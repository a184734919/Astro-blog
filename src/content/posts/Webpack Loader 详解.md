---
title: Webpack Loader 详解
published: 2024-06-15
description: '常用 Loader 配置的详细介绍和学习笔记'
image: ''
tags: ["Webpack","Loader"]
category: '前端工具'
draft: false
lang: 'zh-CN'
---

## Webpack Loader 概述

Loader 是 Webpack 的核心概念之一，它允许 Webpack 处理非 JavaScript 文件。Loader 将文件转换为模块，使其能够被 Webpack 添加到依赖图中。理解 Loader 的工作原理和常用配置对于前端工程化至关重要。

## Loader 基本原理

Loader 本质上是一个 Node.js 模块，它接收源文件内容，处理后返回转换后的内容。

### Loader 的执行顺序

Loader 的执行遵循以下规则：
1. **从右到左**：use 数组中的 loader 按照从右到左的顺序执行
2. **从下到上**：多个 loader 规则按照从下到上的顺序执行
3. **Pitch 阶段**：先执行所有 loader 的 pitch 函数，再执行正常的 loader 函数

```javascript
// 示例：CSS 处理流程
{
  test: /\.css$/,
  use: [
    'style-loader',      // 3. 将 CSS 注入到 style 标签中
    'css-loader',        // 2. 处理 @import 和 url()
    'postcss-loader'     // 1. 处理 CSS 兼容性和转换
  ]
}
```

### Loader 的基本结构

```javascript
// my-loader.js
module.exports = function(source, sourceMap, data) {
  // source: 源文件内容
  // sourceMap: source map 对象
  // data: 在不同 loader 间共享的数据
  
  const callback = this.async(); // 异步 loader
  
  // 处理逻辑
  const processedSource = transform(source);
  
  // 返回结果：错误、源码、source map、元数据
  callback(null, processedSource, sourceMap, {
    version: 1,
    someMeta: 'value'
  });
};

// 同步 loader
module.exports = function(source) {
  return transform(source);
};
```

### Loader 的 Pitch 阶段

```javascript
module.exports.pitch = function(remainingRequest, precedingRequest, data) {
  // remainingRequest: 剩余的请求字符串
  // precedingRequest: 前面的请求字符串
  // data: 存储在 data 对象中，供后续 loader 使用
  
  // 返回值会跳过后续 loader 的正常执行
  // return "module.exports = 'some code';";
};
```

## 常用 Loader 详解

### JavaScript/TypeScript Loaders

#### Babel Loader

Babel Loader 用于转译 ES6+ 代码和 JSX 语法。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            // 缓存转译结果
            cacheDirectory: true,
            cacheCompression: false,
            
            // Babel 配置
            presets: [
              ['@babel/preset-env', {
                targets: {
                  browsers: ['> 0.25%', 'not dead']
                },
                useBuiltIns: 'usage',
                corejs: 3
              }],
              ['@babel/preset-react', {
                runtime: 'automatic', // 或 'classic'
                development: !isProduction
              }],
              ['@babel/preset-typescript', {
                isTSX: true,
                allExtensions: true
              }]
            ],
            plugins: [
              '@babel/plugin-proposal-class-properties',
              '@babel/plugin-proposal-object-rest-spread',
              '@babel/plugin-transform-runtime',
              isProduction ? null : 'react-refresh/babel'
            ].filter(Boolean)
          }
        }
      }
    ]
  }
};
```

**Babel 配置文件：**

```javascript
// babel.config.js
module.exports = {
  presets: [
    ['@babel/preset-env', {
      targets: {
        node: '14'
      }
    }]
  ],
  plugins: [
    '@babel/plugin-transform-runtime'
  ]
};
```

#### TypeScript Loader

TypeScript Loader 用于处理 TypeScript 文件。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'ts-loader',
            options: {
              // 使用 tsconfig.json 配置
              transpileOnly: true, // 只进行语法转换，不做类型检查
              happyPackMode: false, // 启用多进程
              compilerOptions: {
                module: 'esnext',
                target: 'es5'
              }
            }
          }
        ]
      }
    ]
  }
};
```

**使用 TypeScript Loader 的最佳实践：**

```javascript
// 生产环境：分离类型检查和编译
{
  test: /\.tsx?$/,
  exclude: /node_modules/,
  use: [
    {
      loader: 'babel-loader',
      options: {
        presets: [
          ['@babel/preset-env', { modules: false }],
          '@babel/preset-typescript',
          '@babel/preset-react'
        ]
      }
    }
  ]
}

// 开发环境：使用 ts-loader 进行快速开发
{
  test: /\.tsx?$/,
  exclude: /node_modules/,
  use: [
    {
      loader: 'ts-loader',
      options: {
        transpileOnly: true,
        experimentalWatchApi: true
      }
    }
  ]
}
```

### CSS Loaders

#### CSS Loader

CSS Loader 处理 CSS 中的 `@import` 和 `url()` 语句。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        exclude: /\.module\.css$/,
        use: [
          'style-loader', // 3. 注入到 DOM
          {
            loader: 'css-loader',
            options: {
              // 启用 CSS Modules
              modules: false,
              
              // url() 处理
              url: {
                filter: (url, resourcePath) => {
                  // 过滤不需要处理的 url
                  return true;
                }
              },
              
              // @import 处理
              import: true,
              
              // 是否使用 source map
              sourceMap: isDevelopment,
              
              // 自动导入全局 CSS
              importLoaders: 1, // 在当前 loader 前执行的 loader 数量
              
              // 模块导出格式
              esModule: false,
              
              // 自定义类名生成
              getLocalIdent: (context, localIdentName, localName, options) => {
                return `${localName}__${context.resourcePath.replace(/.*[\\\/]/, '')}`;
              }
            }
          },
          'postcss-loader' // 1. 处理 CSS
        ]
      }
    ]
  }
};
```

**CSS Modules 配置：**

```javascript
{
  test: /\.module\.css$/,
  use: [
    'style-loader',
    {
      loader: 'css-loader',
      options: {
        modules: {
          // 类名生成模式
          localIdentName: isDevelopment
            ? '[path][name]__[local]--[hash:base64:5]'
            : '[hash:base64:8]',
          
          // 模块上下文
          context: path.resolve(__dirname, 'src'),
          
          // hash 前缀
          hashPrefix: 'hash',
          
          // 是否允许局部作用域
          exportLocalsConvention: 'asIs', // 'asIs' | 'camelCase' | 'camelCaseOnly' | 'dashes' | 'dashesOnly'
          
          // 命名导入
          namedExport: false,
          
          // 自动导入全局 CSS
          importLoaders: 1
        },
        sourceMap: isDevelopment
      }
    },
    'postcss-loader'
  ]
}
```

#### Style Loader

Style Loader 将 CSS 注入到页面的 style 标签中。

```javascript
{
  loader: 'style-loader',
  options: {
    // 插入位置
    insert: function insertAtTop(element) {
      var parent = document.querySelector('head');
      var lastInsertedElement = window._lastElementInsertedByStyleLoader;

      if (!lastInsertedElement) {
        parent.insertBefore(element, parent.firstChild);
      } else if (lastInsertedElement.nextSibling) {
        parent.insertBefore(element, lastInsertedElement.nextSibling);
      } else {
        parent.appendChild(element);
      }

      window._lastElementInsertedByStyleLoader = element;
    },
    
    // 插入到指定元素前
    // insert: '#style-element',
    
    // 样式属性
    attributes: {
      id: 'my-style',
      nonce: 'abc123'
    },
    
    // 是否启用 source map
    sourceMap: isDevelopment,
    
    // 转换 base64
    esModule: false,
    
    // 自定义转换
    transform: function(cssCode) {
      return cssCode.replace(/console\.log\(.*?\);?/g, '');
    }
  }
}
```

#### MiniCssExtractPlugin Loader

MiniCssExtractPlugin Loader 将 CSS 提取到单独的文件中，通常用于生产环境。

```javascript
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              // publicPath 配置
              publicPath: (resourcePath, context) => {
                return path.relative(path.dirname(resourcePath), context) + '/';
              },
              
              // 启用 HMR
              hmr: isDevelopment,
              
              // 重新加载
              reloadAll: true,
              
              // 模块导出格式
              esModule: false
            }
          },
          'css-loader',
          'postcss-loader'
        ]
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: isDevelopment ? '[name].css' : '[name].[contenthash:8].css',
      chunkFilename: isDevelopment ? '[id].css' : '[id].[contenthash:8].css'
    })
  ]
};
```

#### PostCSS Loader

PostCSS Loader 使用 PostCSS 处理 CSS。

```javascript
// postcss.config.js
module.exports = {
  plugins: [
    require('autoprefixer')({
      browsers: ['> 0.25%', 'not dead']
    }),
    require('cssnano')({
      preset: 'default'
    })
  ]
};

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              // PostCSS 配置文件
              config: {
                path: path.resolve(__dirname, './postcss.config.js')
              },
              
              // source map
              sourceMap: isDevelopment,
              
              // 是否执行插件
              execute: false
            }
          }
        ]
      }
    ]
  }
};
```

### 样式预处理器 Loaders

#### Sass Loader

Sass Loader 处理 Sass/SCSS 文件。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(scss|sass)$/,
        exclude: /\.module\.(scss|sass)$/,
        use: [
          isDevelopment ? 'style-loader' : MiniCssExtractPlugin.loader,
          {
            loader: 'css-loader',
            options: {
              sourceMap: true
            }
          },
          {
            loader: 'sass-loader',
            options: {
              sourceMap: true,
              // Sass 编译器
              implementation: require('sass'),
              // 额外的数据
              additionalData: `
                $primary-color: #3498db;
                $secondary-color: #2ecc71;
              `,
              // 输出样式
              sassOptions: {
                outputStyle: isDevelopment ? 'expanded' : 'compressed',
                indentType: 2,
                indentWidth: 2
              },
              // 警告处理
              warnRuleAsWarning: true
            }
          }
        ]
      }
    ]
  }
};
```

#### Less Loader

Less Loader 处理 Less 文件。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/,
        use: [
          isDevelopment ? 'style-loader' : MiniCssExtractPlugin.loader,
          {
            loader: 'css-loader',
            options: {
              sourceMap: true
            }
          },
          {
            loader: 'less-loader',
            options: {
              sourceMap: true,
              lessOptions: {
                // 全局变量
                modifyVars: {
                  'primary-color': '#3498db',
                  'border-radius': '4px'
                },
                // 严格模式
                math: 'always',
                // 相对路径
                relativeUrls: true
              }
            }
          }
        ]
      }
    ]
  }
};
```

#### Stylus Loader

Stylus Loader 处理 Stylus 文件。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.styl$/,
        use: [
          isDevelopment ? 'style-loader' : MiniCssExtractPlugin.loader,
          'css-loader',
          {
            loader: 'stylus-loader',
            options: {
              sourceMap: true,
              stylusOptions: {
                // 导入路径
                import: [
                  '~nib/lib/nib/index.styl',
                  path.join(__dirname, 'src/styles/variables.styl')
                ],
                // 全局变量
                define: {
                  'primary-color': '#3498db',
                  'font-size-base': '16px'
                },
                // 包含路径
                paths: [
                  path.join(__dirname, 'src/styles')
                ],
                // 是否压缩
                compress: isProduction
              }
            }
          }
        ]
      }
    ]
  }
};
```

### 模板 Loaders

#### HTML Loader

HTML Loader 处理 HTML 文件，导出为字符串。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.html$/,
        use: [
          {
            loader: 'html-loader',
            options: {
              // 属性处理
              attributes: {
                list: [
                  {
                    tag: 'img',
                    attribute: 'src',
                    type: 'src',
                    filter: (tag, attribute, attributes, resourcePath) => {
                      // 过滤某些属性
                      return true;
                    }
                  },
                  {
                    tag: 'img',
                    attribute: 'data-src',
                    type: 'src'
                  },
                  {
                    tag: 'link',
                    attribute: 'href',
                    type: 'src',
                    filter: (tag, attribute, attributes, resourcePath) => {
                      return attributes.rel === 'stylesheet';
                    }
                  }
                ],
                urlFilter: (attribute, value, resourcePath) => {
                  // 过滤 URL
                  return true;
                },
                root: '.'
              },
              // 是否最小化
              minimize: isProduction,
              // 是否支持 ESM
              esModule: false,
              // 来源限制
              sources: {
                list: [
                  '...',
                  {
                    tag: 'img',
                    attribute: 'data-src',
                    type: 'src'
                  }
                ]
              }
            }
          }
        ]
      }
    ]
  }
};
```

#### Pug Loader

Pug Loader 处理 Pug 模板文件。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.pug$/,
        use: [
          {
            loader: 'pug-loader',
            options: {
              pretty: isDevelopment,
              self: false,
              doctype: 'html',
              // 全局变量
              data: {
                siteName: 'My Site',
                currentYear: new Date().getFullYear()
              },
              // 插件
              filters: {
                'my-filter': function(text, options) {
                  return text.toUpperCase();
                }
              }
            }
          }
        ]
      }
    ]
  }
};
```

### 资源 Loaders

#### File Loader

File Loader 处理文件并将其输出到输出目录。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif|svg)$/i,
        type: 'asset',
        // 等同于 file-loader:
        // use: [
        //   {
        //     loader: 'file-loader',
        //     options: {
        //       name: '[name].[contenthash:8].[ext]',
        //       outputPath: 'images/',
        //       publicPath: '/images/',
        //       esModule: false,
        //       context: '',
        //       emitFile: true,
        //       regExp: /\/([a-z0-9]+)([\/\-_][a-z0-9]+)*\.[a-z0-9]+$/i
        //     }
        //   }
        // ]
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/i,
        type: 'asset/resource',
        generator: {
          filename: 'fonts/[name].[contenthash:8][ext]'
        }
      }
    ]
  }
};
```

#### URL Loader

URL Loader 将文件转换为 base64 字符串，用于小文件。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif|svg)$/i,
        type: 'asset',
        parser: {
          dataUrlCondition: {
            maxSize: 8 * 1024 // 8KB 以下转为 base64
          }
        },
        generator: {
          filename: 'images/[name].[contenthash:8][ext]'
        }
      }
    ]
  }
};
```

#### Raw Loader

Raw Loader 将文件内容作为字符串导出。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.txt$/,
        type: 'asset/source'
        // 等同于 raw-loader:
        // use: [
        //   {
        //     loader: 'raw-loader',
        //     options: {
        //       esModule: false
        //     }
        //   }
        // ]
      }
    ]
  }
};
```

### 框架 Loaders

#### Vue Loader

Vue Loader 处理 Vue 单文件组件。

```javascript
const { VueLoaderPlugin } = require('vue-loader');

module.exports = {
  module: {
    rules: [
      {
        test: /\.vue$/,
        use: [
          {
            loader: 'vue-loader',
            options: {
              // 编译模板
              compilerOptions: {
                isCustomElement: tag => tag.startsWith('my-')
              },
              // 热重载
              hotReload: isDevelopment,
              // 自定义块处理
              preLoaders: {},
              postLoaders: {},
              loaders: {
                // 为自定义块配置 loader
                docs: 'docs-loader'
              },
              // CSS 提取
              css: 'css-loader',
              // SCSS 处理
              scss: 'vue-style-loader!css-loader!sass-loader',
              // Less 处理
              less: 'vue-style-loader!css-loader!less-loader'
            }
          }
        }
      }
    ]
  },
  plugins: [
    new VueLoaderPlugin()
  ]
};
```

#### Angular Component Loader

Angular 项目中用于处理组件模板和样式。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(html|css)$/,
        type: 'asset/resource',
        use: {
          loader: 'file-loader',
          options: {
            name: '[name].[hash].[ext]'
          }
        }
      }
    ]
  }
};
```

### 工具 Loaders

#### ESLint Loader

ESLint Loader 在编译时检查代码质量。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        enforce: 'pre', // 在其他 loader 之前执行
        use: [
          {
            loader: 'eslint-loader',
            options: {
              // 配置文件
              configFile: '.eslintrc.js',
              // 缓存
              cache: true,
              cacheIdentifier: 'eslint-cache',
              // 自动修复
              fix: true,
              // 报告格式
              formatter: require('eslint-friendly-formatter'),
              // 输出警告和错误
              emitWarning: true,
              emitError: false,
              // 是否失败
              failOnError: false,
              failOnWarning: false,
              // 忽略模式
              ignorePattern: 'node_modules/',
              // 自定义规则
              rules: {
                'no-console': 'off'
              }
            }
          }
        ]
      }
    ]
  }
};
```

#### Stylelint Loader

Stylelint Loader 检查 CSS 代码质量。

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(css|scss|sass|less|styl)$/,
        exclude: /node_modules/,
        enforce: 'pre',
        use: [
          {
            loader: 'stylelint-custom-processor-loader',
            options: {
              configPath: path.resolve(__dirname, '.stylelintrc.json'),
              emitWarning: true,
              failOnError: false,
              quiet: false,
              formatter: require('stylelint-formatter-pretty')
            }
          }
        ]
      }
    ]
  }
};
```

## 自定义 Loader

### 创建简单的自定义 Loader

```javascript
// uppercase-loader.js - 将内容转换为大写
module.exports = function(source) {
  if (this.cacheable) {
    this.cacheable();
  }
  return source.toUpperCase();
};
```

### 创建带配置的自定义 Loader

```javascript
// prefix-loader.js - 为内容添加前缀
const schema = require('./schema.json');

module.exports = function(source) {
  // 获取 loader 选项
  const options = this.getOptions(schema);
  
  if (this.cacheable) {
    this.cacheable();
  }
  
  const prefix = options.prefix || '/* */';
  return prefix + '\n' + source;
};
```

### 创建异步 Loader

```javascript
// async-loader.js - 异步处理文件
module.exports = function(source) {
  const callback = this.async();
  
  setTimeout(() => {
    callback(null, source + ' // processed asynchronously');
  }, 1000);
};
```

### 创建带 Pitch 的 Loader

```javascript
// cache-bypass-loader.js - 绕过缓存
module.exports.pitch = function(remainingRequest, precedingRequest, data) {
  // pitch 阶段可以提前终止
  if (this.cacheable) {
    this.cacheable(false);
  }
  
  // 将其他 loader 的结果包装
  return `
    module.exports = require(${remainingRequest});
  `;
};

module.exports = function(source) {
  return source;
};
```

### 创建 Pipeline Loader

```javascript
// pipeline-loader.js - 串行执行多个转换
module.exports = function(source) {
  const transformers = this.query.transformers || [];
  
  let result = source;
  for (const transformer of transformers) {
    result = transformer(result);
  }
  
  return result;
};
```

## Loader 配置高级技巧

### Loader 的条件匹配

```javascript
module.exports = {
  module: {
    rules: [
      {
        // 多条件匹配
        test: /\.(js|jsx)$/,
        include: [
          path.resolve(__dirname, 'src'),
          path.resolve(__dirname, 'test')
        ],
        exclude: [
          path.resolve(__dirname, 'src/excluded'),
          /node_modules/
        ],
        issuer: {
          test: /\.(js|jsx)$/,
          issuer: /main\.js$/
        },
        resource: {
          and: [/\.js$/, /exclude/],
          or: [/\.js$/, /\.jsx$/],
          not: [/vendor/]
        },
        use: 'babel-loader'
      }
    ]
  }
};
```

### Loader 的内联使用

```javascript
// 在 import 语句中直接指定 loader
import styles from 'style-loader!css-loader!./styles.css';
import script from 'babel-loader?presets[]=env!./script.js';

// 禁用其他 loader
import styles from '!style-loader!css-loader!./styles.css';

// 只使用特定 loader
import styles from '-!style-loader!css-loader!./styles.css';
```

### Loader 的执行顺序控制

```javascript
module.exports = {
  module: {
    rules: [
      // pre 阶段 - 最先执行
      {
        test: /\.js$/,
        enforce: 'pre',
        use: 'eslint-loader'
      },
      
      // normal 阶段 - 按配置顺序执行
      {
        test: /\.js$/,
        use: 'babel-loader'
      },
      
      // post 阶段 - 最后执行
      {
        test: /\.js$/,
        enforce: 'post',
        use: 'license-checker-loader'
      }
    ]
  }
};
```

### Loader 的性能优化

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            // 启用缓存
            cacheDirectory: true,
            cacheCompression: false,
            // 指定缓存目录
            cacheIdentifier: 'babel-cache-v1',
            // 并行处理
            cache: true
          }
        }
      }
    ]
  },
  // 使用 thread-loader 进行多线程处理
  {
    test: /\.js$/,
    exclude: /node_modules/,
    use: [
      {
        loader: 'thread-loader',
        options: {
          workers: require('os').cpus().length - 1,
          workerParallelJobs: 50,
          poolTimeout: 2000,
          poolParallelJobs: 50,
          name: 'my-pool'
        }
      },
      'babel-loader'
    ]
  }
};
```

## 实战案例

### 多样式语言项目配置

```javascript
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  module: {
    rules: [
      // 普通文件
      {
        test: /\.css$/,
        use: [
          isDevelopment ? 'style-loader' : MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader'
        ]
      },
      
      // Sass/SCSS 文件
      {
        test: /\.scss$/,
        use: [
          isDevelopment ? 'style-loader' : MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader',
          {
            loader: 'sass-loader',
            options: {
              implementation: require('sass'),
              sassOptions: {
                includePaths: [path.resolve(__dirname, 'src/styles')]
              }
            }
          }
        ]
      },
      
      // Less 文件
      {
        test: /\.less$/,
        use: [
          isDevelopment ? 'style-loader' : MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader',
          'less-loader'
        ]
      },
      
      // Stylus 文件
      {
        test: /\.styl$/,
        use: [
          isDevelopment ? 'style-loader' : MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader',
          'stylus-loader'
        ]
      }
    ]
  }
};
```

### 图标处理配置

```javascript
module.exports = {
  module: {
    rules: [
      // SVG Sprites
      {
        test: /\.svg$/,
        oneOf: [
          {
            // 作为 React 组件使用
            resourceQuery: /react/,
            use: '@svgr/webpack'
          },
          {
            // 作为 sprite 处理
            resourceQuery: /sprite/,
            type: 'asset/resource',
            generator: {
              filename: 'sprite/[name].[hash:8][ext]'
            }
          },
          {
            // 普通图片处理
            type: 'asset',
            parser: {
              dataUrlCondition: {
                maxSize: 8 * 1024
              }
            }
          }
        ]
      }
    ]
  }
};
```

### TypeScript + React 配置

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(ts|tsx)$/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'babel-loader',
            options: {
              cacheDirectory: true,
              presets: [
                ['@babel/preset-env', {
                  targets: { browsers: ['> 0.25%', 'not dead'] },
                  modules: false
                }],
                ['@babel/preset-react', {
                  runtime: 'automatic',
                  development: !isProduction
                }],
                ['@babel/preset-typescript', {
                  isTSX: true,
                  allExtensions: true
                }]
              ],
              plugins: [
                ['@babel/plugin-transform-runtime', {
                  corejs: 3,
                  regenerator: true
                }]
              ]
            }
          }
        ]
      }
    ]
  }
};
```

## 注意事项

1. **Loader 顺序**：注意 loader 的执行顺序是从右到左，从下到上
2. **性能影响**：复杂的 loader 会影响构建速度，考虑使用缓存和并行处理
3. **Source Map**：开发环境确保 source map 配置正确，便于调试
4. **缓存策略**：合理使用 cacheDirectory 提升构建速度
5. **错误处理**：配置合适的错误处理和警告级别
6. **模块联邦**：使用 Module Federation 时注意 loader 的配置一致性
7. **版本兼容**：注意 webpack 版本与 loader 版本的兼容性
8. **资源处理**：合理配置资源大小阈值，平衡打包体积和加载性能

## 总结

Webpack Loader 是处理非 JavaScript 模块的核心机制，通过合理的 Loader 配置可以让 Webpack 处理各种类型的文件。理解 Loader 的工作原理、执行顺序和配置技巧对于构建高效的前端工程化方案至关重要。

在实际项目中，根据项目需求选择合适的 Loader 组合，注重构建性能和开发体验的平衡。掌握常用 Loader 的配置方法和自定义 Loader 的开发技巧，能够让我们更好地应对各种复杂的构建需求。