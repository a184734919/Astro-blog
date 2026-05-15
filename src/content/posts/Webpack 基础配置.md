---
title: Webpack 基础配置
published: 2024-06-01
description: 'Webpack 入口和出口的详细介绍和学习笔记'
image: ''
tags: ["Webpack","基础"]
category: '前端工具'
draft: false
lang: 'zh-CN'
---

## Webpack 基础配置概述

Webpack 是一个静态模块打包工具，它会递归地构建依赖关系图，将所有模块打包成一个或多个 bundle。理解 Webpack 的基础配置是掌握前端工程化的关键。

## 核心配置概念

### Entry（入口）

Entry 是 Webpack 构建的起点，Webpack 会从这个文件开始分析依赖关系图。

**单入口配置：**

```javascript
module.exports = {
  entry: './src/index.js',
  
  // 或者使用对象形式
  entry: {
    main: './src/index.js'
  }
};
```

**多入口配置：**

```javascript
module.exports = {
  entry: {
    app: './src/app.js',
    admin: './src/admin.js',
    vendor: './src/vendor.js'
  }
};
```

**数组入口：**

```javascript
module.exports = {
  entry: {
    app: ['./src/app.js', './src/polyfill.js']
  }
};
```

### Output（输出）

Output 配置告诉 Webpack 如何将编译后的文件输出到磁盘。

**基础配置：**

```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  }
};
```

**多入口输出：**

```javascript
module.exports = {
  entry: {
    app: './src/app.js',
    admin: './src/admin.js'
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js',
    clean: true // 每次构建前清理 dist 目录
  }
};
```

**使用占位符：**

```javascript
module.exports = {
  output: {
    path: path.resolve(__dirname, 'dist'),
    // [name] - 入口名称
    // [hash] - 整个构建的 hash
    // [chunkhash] - 特定 chunk 的 hash
    // [contenthash] - 文件内容的 hash
    filename: '[name].[contenthash:8].js',
    // 动态导入的文件名
    chunkFilename: '[name].[contenthash:8].chunk.js',
    // 资源文件名
    assetModuleFilename: 'assets/[name].[hash:8][ext]',
    // 公共路径
    publicPath: '/'
  }
};
```

### Mode（模式）

Mode 配置 Webpack 的运行模式，会影响内置优化和错误处理。

```javascript
module.exports = {
  mode: 'development', // 或 'production' 或 'none'
  
  // development 模式特点：
  // - 使用 eval-source-map 生成 source map
  // - 启用 NamedModulesPlugin
  // - 不压缩代码
  
  // production 模式特点：
  // - 使用 TerserPlugin 压缩代码
  // - 启用 FlagDependencyUsagePlugin
  // - 启用 FlagIncludedChunksPlugin
  // - 启用 ModuleConcatenationPlugin
  // - 启用 NoEmitOnErrorsPlugin
  // - 启用 OccurrenceOrderPlugin
  // - 启用 SideEffectsFlagPlugin
  // - 启用 TerserPlugin
};
```

### Module（模块）

Module 配置决定了 Webpack 如何处理不同的文件类型。

**基础 Loader 配置：**

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: 'babel-loader'
      },
      {
        test: /\.css$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: true,
              importLoaders: 1
            }
          }
        ]
      }
    ]
  }
};
```

**Rule 配置选项：**

```javascript
module.exports = {
  module: {
    rules: [
      {
        // 条件匹配
        test: /\.jsx?$/, // 正则表达式匹配
        include: /src/,   // 必须匹配的路径
        exclude: /node_modules/, // 必须排除的路径
        
        // Loader 配置
        use: [
          {
            loader: 'babel-loader',
            options: {
              cacheDirectory: true
            }
          }
        ],
        
        // 资源类型
        type: 'javascript/auto', // 'javascript/auto' | 'javascript/dynamic' | 'javascript/esm' | 'json' | 'webassembly/sync' | 'webassembly/async' | 'asset' | 'asset/source' | 'asset/resource' | 'asset/inline'
        
        // 生成依赖图时不解析匹配的文件
        noParse: /jquery/,
        
        // 解析器选项
        parser: {
          requireEnsure: false
        },
        
        // 生成器选项
        generator: {
          filename: '[name][ext]'
        },
        
        // 垂直使用顺序
        enforce: 'pre', // 'pre' | 'post' | undefined
      }
    ]
  }
};
```

**资源模块配置（Webpack 5）：**

```javascript
module.exports = {
  module: {
    rules: [
      // 处理图片
      {
        test: /\.(png|jpg|jpeg|gif|webp|svg)$/,
        type: 'asset',
        parser: {
          dataUrlCondition: {
            maxSize: 8 * 1024 // 8KB 以下转为 base64
          }
        },
        generator: {
          filename: 'images/[name].[hash:8][ext]'
        }
      },
      // 处理字体
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        type: 'asset/resource',
        generator: {
          filename: 'fonts/[name].[hash:8][ext]'
        }
      },
      // 处理 JSON
      {
        test: /\.json$/,
        type: 'json' // 或 'json/auto'
      }
    ]
  }
};
```

### Resolve（解析）

Resolve 配置 Webpack 如何查找模块文件。

```javascript
module.exports = {
  resolve: {
    // 模块查找路径
    modules: ['node_modules', 'src'],
    
    // 扩展名自动解析
    extensions: ['.js', '.jsx', '.json', '.css'],
    
    // 路径别名
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@utils': path.resolve(__dirname, 'src/utils'),
      'react-dom': '@hot-loader/react-dom'
    },
    
    // 模块主文件
    mainFields: ['module', 'main'],
    
    // 默认入口文件
    mainFiles: ['index', 'main'],
    
    // 解析符号链接
    symlinks: true,
    
    // 缓存解析结果
    cacheWithContext: true,
    
    // 强制特定模块使用特定解析方式
    aliasFields: ['browser'],
    
    // 解析后的文件绝对路径
    fullySpecified: false
  }
};
```

### Devtool（Source Map）

Devtool 控制 source map 的生成方式，影响构建速度和调试体验。

```javascript
module.exports = {
  // 开发环境：速度快，调试方便
  devtool: 'eval-source-map', // 最好的开发环境选择
  
  // 生产环境：不暴露源码，性能好
  devtool: 'source-map', // 生成独立的 source map 文件
  
  // 其他选项：
  // eval - 速度最快，但不生成 source map
  // cheap-eval-source-map - 生成编译后的代码映射
  // cheap-module-eval-source-map - 生成原始源码映射（推荐开发环境）
  // source-map - 生成完整的 source map（推荐生产环境）
  // hidden-source-map - 不暴露 source map 引用
  // nosources-source-map - 只显示行号，不显示源码
  // inline-source-map - 内联 source map
  // eval-cheap-source-map - eval + cheap + source-map
  // eval-cheap-module-source-map - eval + cheap + module + source-map
};
```

### DevServer（开发服务器）

DevServer 提供了开发时的 HTTP 服务和热替换功能。

```javascript
const path = require('path');

module.exports = {
  devServer: {
    // 服务器基础路径
    static: {
      directory: path.join(__dirname, 'public'),
      publicPath: '/static',
      serveIndex: true,
      watch: true
    },
    
    // 服务器端口
    port: 3000,
    
    // 自动打开浏览器
    open: true,
    
    // 启用热模块替换
    hot: true,
    
    // 启用压缩
    compress: true,
    
    // 启用 gzip
    client: {
      overlay: {
        errors: true,
        warnings: false
      },
      progress: true
    },
    
    // 代理配置
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    },
    
    // 历史模式支持
    historyApiFallback: true,
    
    // HTTPS 配置
    https: false,
    // https: {
    //   key: fs.readFileSync('path/to/private.key'),
    //   cert: fs.readFileSync('path/to/certificate.pem')
    // },
    
    // 响应头配置
    headers: {
      'X-Custom-Header': 'value'
    },
    
    // 服务器日志
    devMiddleware: {
      index: true,
      writeToDisk: false,
      stats: 'minimal'
    }
  }
};
```

### Externals（外部扩展）

Externals 配置可以从 bundle 中排除某些依赖，从外部获取。

```javascript
module.exports = {
  externals: {
    // 字符串形式
    'react': 'React',
    'react-dom': 'ReactDOM',
    
    // 正则表达式
    /^lodash$/i: 'lodash',
    
    // 函数形式
    function ({ request }, callback) {
      if (/^your-library$/.test(request)) {
        return callback(null, 'your-library ' + request);
      }
      callback();
    },
    
    // 对象形式
    'library-name': {
      commonjs: 'library-name',
      commonjs2: 'library-name',
      amd: 'library-name',
      root: 'Library'
    }
  }
};
```

### Cache（缓存）

Cache 配置可以显著提升重复构建的速度。

```javascript
module.exports = {
  cache: {
    type: 'filesystem', // 'memory' | 'filesystem'
    cacheDirectory: path.resolve(__dirname, '.cache/webpack'),
    maxAge: 1000 * 60 * 60 * 24, // 缓存有效期 24 小时
    compression: 'gzip',
    idleTimeout: 60000, // 1 分钟无操作停止缓存
    idleTimeoutAfterLargeChanges: 1000 // 大变化后 1 秒停止缓存
  }
};
```

### Performance（性能）

Performance 配置用于控制构建结果的体积警告。

```javascript
module.exports = {
  performance: {
    hints: 'warning', // false | 'error' | 'warning'
    maxEntrypointSize: 250000, // 入口文件最大体积 250KB
    maxAssetSize: 250000, // 资源文件最大体积 250KB
    assetFilter: function(assetFilename) {
      // 只对某些文件进行性能检查
      return assetFilename.endsWith('.js');
    }
  }
};
```

## 完整配置示例

```javascript
const path = require('path');
const { DefinePlugin } = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const isProduction = process.env.NODE_ENV === 'production';

module.exports = {
  // 入口配置
  entry: {
    main: './src/index.js',
    vendor: './src/vendor.js'
  },
  
  // 输出配置
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: isProduction ? '[name].[contenthash:8].js' : '[name].js',
    chunkFilename: '[name].[contenthash:8].chunk.js',
    assetModuleFilename: 'assets/[name].[hash:8][ext]',
    publicPath: '/',
    clean: true
  },
  
  // 模式配置
  mode: isProduction ? 'production' : 'development',
  
  // Source Map 配置
  devtool: isProduction ? 'source-map' : 'eval-source-map',
  
  // 模块配置
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            cacheDirectory: true,
            presets: [
              ['@babel/preset-env', {
                targets: '> 0.25%, not dead'
              }],
              ['@babel/preset-react', {
                development: !isProduction
              }]
            ]
          }
        }
      },
      {
        test: /\.css$/,
        exclude: /\.module\.css$/,
        use: [
          isProduction ? MiniCssExtractPlugin.loader : 'style-loader',
          {
            loader: 'css-loader',
            options: {
              importLoaders: 1
            }
          },
          'postcss-loader'
        ]
      },
      {
        test: /\.module\.css$/,
        use: [
          isProduction ? MiniCssExtractPlugin.loader : 'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: {
                localIdentName: isProduction 
                  ? '[hash:base64:5]' 
                  : '[name]__[local]--[hash:base64:5]'
              },
              importLoaders: 1
            }
          },
          'postcss-loader'
        ]
      },
      {
        test: /\.(png|jpe?g|gif|webp|svg)$/i,
        type: 'asset',
        parser: {
          dataUrlCondition: {
            maxSize: 8 * 1024
          }
        },
        generator: {
          filename: 'images/[name].[hash:8][ext]'
        }
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/i,
        type: 'asset/resource',
        generator: {
          filename: 'fonts/[name].[hash:8][ext]'
        }
      }
    ]
  },
  
  // 解析配置
  resolve: {
    extensions: ['.js', '.jsx', '.json'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@utils': path.resolve(__dirname, 'src/utils')
    },
    modules: ['node_modules', 'src']
  },
  
  // 插件配置
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html',
      filename: 'index.html',
      minify: isProduction ? {
        removeComments: true,
        collapseWhitespace: true,
        removeRedundantAttributes: true,
        useShortDoctype: true,
        removeEmptyAttributes: true,
        removeStyleLinkTypeAttributes: true,
        keepClosingSlash: true,
        minifyJS: true,
        minifyCSS: true,
        minifyURLs: true
      } : false
    }),
    new DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development'),
      'process.env.API_URL': JSON.stringify(process.env.API_URL || '/api')
    })
  ],
  
  // 开发服务器配置
  devServer: {
    static: {
      directory: path.join(__dirname, 'public')
    },
    port: 3000,
    hot: true,
    open: true,
    compress: true,
    historyApiFallback: true,
    client: {
      overlay: {
        errors: true,
        warnings: false
      },
      progress: true
    },
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    }
  },
  
  // 缓存配置
  cache: {
    type: 'filesystem'
  },
  
  // 性能配置
  performance: {
    hints: isProduction ? 'warning' : false,
    maxEntrypointSize: 500000,
    maxAssetSize: 300000
  },
  
  // 优化配置
  optimization: {
    minimize: isProduction,
    runtimeChunk: 'single',
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all'
        },
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'initial',
          priority: -10,
          reuseExistingChunk: true
        }
      }
    }
  }
};
```

## 配置文件拆分

对于复杂的配置，可以拆分成多个文件：

```javascript
// webpack.common.js - 通用配置
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash:8].js',
    clean: true
  },
  module: {
    rules: [
      // 通用 loader 配置
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html'
    })
  ],
  resolve: {
    extensions: ['.js', '.jsx', '.json'],
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  }
};

// webpack.dev.js - 开发环境配置
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'development',
  devtool: 'eval-source-map',
  devServer: {
    static: './dist',
    hot: true,
    open: true,
    port: 3000
  },
  optimization: {
    runtimeChunk: 'single'
  }
});

// webpack.prod.js - 生产环境配置
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = merge(common, {
  mode: 'production',
  devtool: 'source-map',
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader']
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash:8].css'
    })
  ],
  optimization: {
    minimize: true,
    splitChunks: {
      chunks: 'all'
    }
  }
});
```

## 最佳实践

1. **路径处理**：始终使用 `path.resolve()` 处理路径，确保跨平台兼容性
2. **环境区分**：根据环境变量配置不同的优化策略
3. **缓存利用**：合理配置缓存和持久化构建
4. **代码分割**：合理配置 code splitting 减少初始加载时间
5. **Source Map**：开发环境使用快速选项，生产环境使用完整选项
6. **Loader 顺序**：注意 loader 的执行顺序是从右到左，从下到上
7. **版本控制**：排除构建产物和缓存目录
8. **文档注释**：为复杂配置添加注释说明
9. **模块化配置**：大型项目应该拆分配置文件
10. **类型检查**：使用 `webpack-cli --init` 可以生成带类型检查的配置

## 注意事项

1. Webpack 5 移除了许多插件和 loader 的内置支持，需要显式配置
2. 文件系统缓存需要配合正确的 `cacheDirectory` 配置
3. 资源模块替代了 `file-loader`、`url-loader`、`raw-loader`
4. 模块联邦是 Webpack 5 的新特性，用于微前端架构
5. `target` 配置影响打包结果的运行环境
6. `externals` 需要配合 CDN 使用
7. `browserslist` 配置会影响 `@babel/preset-env` 的转换结果

## 总结

Webpack 基础配置是前端工程化的基石，涵盖了入口、输出、模块处理、解析路径等核心概念。掌握这些配置能够帮助我们构建高效、可维护的前端项目。合理的基础配置为后续的 Loader、Plugin 配置和性能优化奠定了坚实基础。

在实际项目中，建议根据项目需求逐步完善配置，从简单的单入口开始，逐步添加复杂的功能如多入口、代码分割、环境优化等。记住配置的本质是为了解决特定的工程化问题，避免过度配置。