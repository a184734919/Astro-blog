---
title: Webpack 性能优化
published: 2024-07-13
description: '构建速度优化的详细介绍和学习笔记'
image: ''
tags: ["Webpack","优化"]
category: '前端工具'
draft: false
lang: 'zh-CN'
---

## Webpack 性能优化概述

Webpack 性能优化是提升前端开发体验和应用性能的关键环节。性能优化主要分为两个维度：构建速度优化和产物体积优化。通过合理的配置和策略，可以显著提升构建效率，减少应用加载时间。

## 构建速度优化

### 1. 利用缓存机制

#### Loader 缓存

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
            // 缓存压缩
            cacheCompression: false,
            // 缓存标识符
            cacheIdentifier: 'babel-cache-v1'
          }
        }
      }
    ]
  }
};
```

#### Webpack 文件系统缓存

```javascript
module.exports = {
  cache: {
    type: 'filesystem', // 'memory' | 'filesystem'
    cacheDirectory: path.resolve(__dirname, '.cache/webpack'),
    cacheLocation: path.resolve(__dirname, '.cache/webpack'),
    // 缓存名称
    name: 'cache',
    // 缓存版本
    version: '1.0.0',
    // 缓存过期时间
    maxAge: 1000 * 60 * 60 * 24, // 24小时
    // 缓存压缩
    compression: 'gzip',
    // 空闲超时
    idleTimeout: 60000,
    // 大改动后超时
    idleTimeoutAfterLargeChanges: 1000,
    // 内存使用限制
    maxMemoryGenerations: 1,
    // 强制刷新缓存
    buildDependencies: {
      config: [__filename]
    }
  }
};
```

### 2. 减少构建范围

#### 精确的 include/exclude

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        // 只处理 src 目录
        include: [
          path.resolve(__dirname, 'src')
        ],
        // 明确排除不需要处理的目录
        exclude: [
          path.resolve(__dirname, 'node_modules'),
          path.resolve(__dirname, 'build'),
          path.resolve(__dirname, 'dist'),
          path.resolve(__dirname, '.cache')
        ],
        use: 'babel-loader'
      }
    ]
  }
};
```

#### 优化 resolve 配置

```javascript
module.exports = {
  resolve: {
    // 减少模块查找路径
    modules: [path.resolve(__dirname, 'node_modules')],
    
    // 减少扩展名查找
    extensions: ['.js', '.jsx', '.json'],
    
    // 合理使用路径别名
    alias: {
      '@': path.resolve(__dirname, 'src'),
      'react': path.resolve(__dirname, 'node_modules/react')
    },
    
    // 禁用不必要的解析
    symlinks: false,
    
    // 减少 mainFiles 查找
    mainFiles: ['index']
  }
};
```

#### 使用 noParse

```javascript
module.exports = {
  module: {
    // 不解析已知没有依赖的模块
    noParse: /jquery|lodash/,
    
    // 函数形式
    noParse: function(content) {
      return /jquery/.test(content);
    }
  }
};
```

### 3. 多线程并行处理

#### Thread Loader

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'thread-loader',
            options: {
              // 工作进程数量
              workers: require('os').cpus().length - 1,
              // 工作进程并行任务数
              workerParallelJobs: 50,
              // 池超时时间
              poolTimeout: 2000,
              // 池并行任务数
              poolParallelJobs: 50,
              // 池名称
              name: 'babel-pool'
            }
          },
          {
            loader: 'babel-loader',
            options: {
              cacheDirectory: true
            }
          }
        ]
      }
    ]
  }
};
```

#### HappyPack

```javascript
const HappyPack = require('happypack');

module.exports = {
  plugins: [
    new HappyPack({
      id: 'js',
      loaders: [{
        loader: 'babel-loader',
        options: {
          cacheDirectory: true
        }
      }],
      threads: 4,
      verbose: true
    })
  ],
  
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: 'happypack/loader?id=js'
      }
    ]
  }
};
```

### 4. 优化开发服务器

#### 快速刷新

```javascript
module.exports = {
  devServer: {
    hot: true,
    // 只编译改动的内容
    liveReload: false,
    // 客户端配置
    client: {
      // 覆盖层配置
      overlay: {
        errors: true,
        warnings: false
      },
      // 进度显示
      progress: true
    },
    // 监听配置
    watchOptions: {
      poll: 1000, // 轮询间隔
      aggregateTimeout: 300, // 延迟重新编译
      ignored: /node_modules/ // 忽略目录
    }
  }
};
```

#### 静态文件优化

```javascript
module.exports = {
  devServer: {
    static: {
      directory: path.join(__dirname, 'public'),
      publicPath: '/static',
      serveIndex: true,
      watch: {
        ignored: /node_modules/
      }
    },
    // 压缩
    compress: true,
    // 开启 Gzip
    headers: {
      'X-Content-Type-Options': 'nosniff',
      'Content-Encoding': 'gzip'
    }
  }
};
```

### 5. 分离开发和生产配置

```javascript
// webpack.common.js
const commonConfig = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js'
  },
  module: {
    rules: [
      // 通用规则
    ]
  },
  plugins: [
    // 通用插件
  ]
};

// webpack.dev.js
const { merge } = require('webpack-merge');
const commonConfig = require('./webpack.common.js');

const devConfig = {
  mode: 'development',
  devtool: 'eval-cheap-module-source-map',
  devServer: {
    hot: true,
    port: 3000
  },
  optimization: {
    runtimeChunk: 'single',
    removeAvailableModules: false,
    removeEmptyChunks: false,
    splitChunks: false
  }
};

module.exports = merge(commonConfig, devConfig);

// webpack.prod.js
const { merge } = require('webpack-merge');
const commonConfig = require('./webpack.common.js');

const prodConfig = {
  mode: 'production',
  devtool: 'source-map',
  optimization: {
    minimize: true,
    splitChunks: {
      chunks: 'all'
    }
  }
};

module.exports = merge(commonConfig, prodConfig);
```

## 产物体积优化

### 1. 代码分割

#### 入口分割

```javascript
module.exports = {
  entry: {
    main: './src/main.js',
    vendor: './src/vendor.js',
    polyfill: './src/polyfill.js'
  },
  
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash:8].js',
    chunkFilename: '[name].[contenthash:8].chunk.js'
  }
};
```

#### 动态导入

```javascript
// 路由懒加载
const Home = () => import(/* webpackChunkName: "home" */ './views/Home');
const About = () => import(/* webpackChunkName: "about" */ './views/About');
const Contact = () => import(/* webpackChunkName: "contact" */ './views/Contact');

// 条件导入
if (process.env.NODE_ENV === 'development') {
  import('./devtools').then(module => {
    module.init();
  });
}

// 预取和预加载
import(/* webpackPrefetch: true */ './path/to/LoginModal.js');
import(/* webpackPreload: true */ './path/to/component.js');

// 带注释的导入
const Home = import(/* webpackChunkName: "pages/home" */ './views/Home')
  .then(module => module.default)
  .catch(error => {
    console.error('Failed to load Home:', error);
    // 降级处理
    return FallbackComponent;
  });
```

#### SplitChunks 配置

```javascript
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      minSize: 30000,
      maxSize: 244000,
      minChunks: 1,
      maxAsyncRequests: 30,
      maxInitialRequests: 30,
      automaticNameDelimiter: '~',
      cacheGroups: {
        // 第三方库
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          reuseExistingChunk: true,
          name: 'vendors'
        },
        
        // React 核心库
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
          priority: 20,
          name: 'react',
          chunks: 'all'
        },
        
        // UI 库
        ui: {
          test: /[\\/]node_modules[\\/](ant-design|element-ui|material-ui)[\\/]/,
          priority: 15,
          name: 'ui',
          chunks: 'all'
        },
        
        // 工具库
        utils: {
          test: /[\\/]node_modules[\\/](lodash|axios|dayjs)[\\/]/,
          priority: 10,
          name: 'utils',
          chunks: 'all'
        },
        
        // 公共代码
        common: {
          name: 'common',
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true,
          chunks: 'initial'
        },
        
        // 默认配置
        defaultVendors: {
          reuseExistingChunk: true,
          priority: -10
        },
        
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    },
    
    // 运行时代码分离
    runtimeChunk: {
      name: entrypoint => `runtime-${entrypoint.name}`
    }
  }
};
```

### 2. Tree Shaking

#### ES6 模块支持

```javascript
// 确保 babel 配置正确
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              ['@babel/preset-env', {
                modules: false // 关键：不要转译 ES6 模块
              }]
            ]
          }
        }
      }
    ]
  },
  
  optimization: {
    usedExports: true,
    sideEffects: true
  }
};
```

#### package.json 配置

```json
{
  "sideEffects": [
    "*.css",
    "*.scss",
    "*.less",
    "./src/styles/*.css",
    "./src/components/**/*.css"
  ]
}
```

#### 标记无副作用模块

```javascript
// 在 package.json 中
{
  "sideEffects": false
}

// 或者指定有副作用的文件
{
  "sideEffects": [
    "src/styles/*.css",
    "src/polyfills.js"
  ]
}
```

### 3. 压缩和混淆

#### JavaScript 压缩

```javascript
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        parallel: true,
        sourceMap: true,
        terserOptions: {
          ecma: 2015,
          warnings: false,
          parse: {},
          compress: {
            drop_console: true,
            drop_debugger: true,
            dead_code: true,
            unused: true,
            if_return: true,
            join_vars: true,
            sequences: true,
            typeofs: true,
            pure_funcs: ['console.log', 'console.info', 'console.debug']
          },
          mangle: {
            safari10: true,
            keep_classnames: false,
            keep_fnames: false
          },
          output: {
            comments: false,
            beautify: false,
            ascii_only: true
          }
        },
        extractComments: false
      })
    ]
  }
};
```

#### CSS 压缩

```javascript
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');

module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      new CssMinimizerPlugin({
        parallel: true,
        sourceMap: true,
        minimizerOptions: {
          preset: [
            'default',
            {
              discardComments: { removeAll: true },
              normalizeWhitespace: false,
              minifyFontValues: { removeQuotes: false }
            }
          ]
        }
      })
    ]
  }
};
```

#### HTML 压缩

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html',
      minify: {
        collapseWhitespace: true,
        removeComments: true,
        removeRedundantAttributes: true,
        removeScriptTypeAttributes: true,
        removeStyleLinkTypeAttributes: true,
        useShortDoctype: true,
        minifyCSS: true,
        minifyJS: true,
        minifyURLs: true
      }
    })
  ]
};
```

### 4. 资源优化

#### 图片优化

```javascript
const ImageMinimizerPlugin = require('image-minimizer-webpack-plugin');
const { extendDefaultPlugins } = require('svgo');

module.exports = {
  module: {
    rules: [
      {
        test: /\.(jpe?g|png|gif|svg|webp)$/i,
        type: 'asset',
        parser: {
          dataUrlCondition: {
            maxSize: 8 * 1024
          }
        },
        generator: {
          filename: 'images/[name].[contenthash:8][ext]'
        }
      }
    ]
  },
  
  plugins: [
    new ImageMinimizerPlugin({
      minimizer: {
        implementation: ImageMinimizerPlugin.squooshMinify,
        options: {
          encodeOptions: {
            mozjpeg: {
              quality: 75
            },
            webp: {
              lossless: false,
              quality: 75
            },
            avif: {
              cqLevel: 33,
              denoiseLevel: 25
            },
            pngquant: {
              quality: [0.65, 0.90],
              speed: 4
            }
          }
        }
      },
      generator: [
        {
          preset: 'webp',
          implementation: ImageMinimizerPlugin.squooshGenerate,
          options: {
            encodeOptions: {
              webp: {
                lossless: false,
                quality: 75
              }
            }
          }
        },
        {
          preset: 'avif',
          implementation: ImageMinimizerPlugin.squooshGenerate,
          options: {
            encodeOptions: {
              avif: {
                cqLevel: 33,
                denoiseLevel: 25
              }
            }
          }
        }
      ]
    })
  ]
};
```

#### 字体优化

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/i,
        type: 'asset/resource',
        generator: {
          filename: 'fonts/[name].[contenthash:8][ext]'
        },
        use: [
          {
            loader: 'font-minify-loader',
            options: {
              // 字体优化选项
              removeUnusedGlyphs: true,
              whitelist: [] // 保留的字形
            }
          }
        ]
      }
    ]
  }
};
```

#### 资源压缩

```javascript
const CompressionPlugin = require('compression-webpack-plugin');
const BrotliPlugin = require('brotli-webpack-plugin');

module.exports = {
  plugins: [
    // Gzip 压缩
    new CompressionPlugin({
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/,
      threshold: 10240,
      minRatio: 0.8,
      compressionOptions: {
        level: 9
      }
    }),
    
    // Brotli 压缩
    new BrotliPlugin({
      asset: '[path].br[query]',
      test: /\.(js|css|html|svg)$/,
      threshold: 10240,
      minRatio: 0.8,
      compressionOptions: {
        level: 11
      }
    })
  ]
};
```

### 5. 外部依赖处理

#### CDN 引入

```javascript
module.exports = {
  externals: {
    react: 'React',
    'react-dom': 'ReactDOM',
    vue: 'Vue',
    axios: 'axios',
    lodash: '_',
    moment: 'moment',
    jquery: 'jQuery'
  }
};

// HTML 中引入 CDN
// <script src="https://cdn.jsdelivr.net/npm/react@17/umd/react.production.min.js"></script>
// <script src="https://cdn.jsdelivr.net/npm/react-dom@17/umd/react-dom.production.min.js"></script>
```

#### 预构建依赖

```javascript
const { DllPlugin } = require('webpack');

// webpack.dll.config.js
module.exports = {
  mode: 'production',
  entry: {
    vendor: ['react', 'react-dom', 'axios', 'lodash', 'moment']
  },
  output: {
    path: path.resolve(__dirname, 'dll'),
    filename: '[name].dll.js',
    library: '[name]_[hash]'
  },
  plugins: [
    new DllPlugin({
      name: '[name]_[hash]',
      path: path.resolve(__dirname, 'dll/[name]-manifest.json')
    })
  ]
};

// webpack.config.js
const { DllReferencePlugin } = require('webpack');

module.exports = {
  plugins: [
    new DllReferencePlugin({
      manifest: require('./dll/vendor-manifest.json')
    })
  ]
};
```

## 运行时性能优化

### 1. 缓存策略

#### 长期缓存

```javascript
module.exports = {
  output: {
    filename: '[name].[contenthash:8].js',
    chunkFilename: '[name].[contenthash:8].chunk.js',
    assetModuleFilename: 'assets/[name].[contenthash:8][ext]'
  },
  
  optimization: {
    moduleIds: 'deterministic',
    runtimeChunk: 'single',
    splitChunks: {
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all'
        }
      }
    }
  }
};
```

#### Service Worker 缓存

```javascript
const WorkboxWebpackPlugin = require('workbox-webpack-plugin');

module.exports = {
  plugins: [
    new WorkboxWebpackPlugin.GenerateSW({
      clientsClaim: true,
      skipWaiting: true,
      runtimeCaching: [
        {
          urlPattern: /^https:\/\/api\.example\.com/,
          handler: 'NetworkFirst',
          options: {
            cacheName: 'api-cache',
            expiration: {
              maxEntries: 100,
              maxAgeSeconds: 30 * 24 * 60 * 60 // 30天
            }
          }
        },
        {
          urlPattern: /\.(png|jpe?g|gif|svg|webp)$/i,
          handler: 'CacheFirst',
          options: {
            cacheName: 'images-cache',
            expiration: {
              maxEntries: 200,
              maxAgeSeconds: 60 * 24 * 60 * 60 // 60天
            }
          }
        }
      ]
    })
  ]
};
```

### 2. 代码懒加载

#### 路由级懒加载

```javascript
// React Router 示例
import { lazy, Suspense } from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));
const Dashboard = lazy(() => import('./routes/Dashboard'));

function App() {
  return (
    <Router>
      <Suspense fallback={<div>Loading...</div>}>
        <Switch>
          <Route exact path="/" component={Home} />
          <Route path="/about" component={About} />
          <Route path="/dashboard" component={Dashboard} />
        </Switch>
      </Suspense>
    </Router>
  );
}
```

#### 组件级懒加载

```javascript
import { lazy, Suspense } from 'react';

const Modal = lazy(() => import('./components/Modal'));
const Chart = lazy(() => import('./components/Chart'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading component...</div>}>
        <Modal />
        <Chart />
      </Suspense>
    </div>
  );
}
```

#### 代码分割优化

```javascript
// 预加载重要路由
const Home = lazy(() => import(/* webpackPrefetch: true */ './routes/Home'));
const Dashboard = lazy(() => import(/* webpackPrefetch: true */ './routes/Dashboard'));

// 预加载资源
const LoginModal = lazy(() => import(/* webpackPreload: true */ './components/LoginModal'));

// 按需加载
const HeavyComponent = lazy(() => import(/* webpackChunkName: "heavy" */ './components/HeavyComponent'));
```

## 监控和分析

### 1. 构建分析

#### Bundle Analyzer

```javascript
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'server',
      analyzerHost: '127.0.0.1',
      analyzerPort: 8888,
      openAnalyzer: true,
      generateStatsFile: true,
      statsFilename: 'stats.json',
      statsOptions: { source: false },
      defaultSizes: 'gzip',
      excludeAssets: null,
      excludeModules: null,
      logLevel: 'info'
    })
  ]
};
```

#### 构建统计

```javascript
module.exports = {
  stats: {
    colors: true,
    hash: true,
    version: true,
    timings: true,
    assets: true,
    chunks: true,
    modules: false,
    reasons: false,
    children: false,
    source: false,
    errors: true,
    errorDetails: true,
    warnings: true,
    publicPath: true,
    modulesSort: 'size',
    assetsSort: 'size'
  }
};
```

### 2. 性能监控

#### 构建时间统计

```javascript
const SpeedMeasurePlugin = require('speed-measure-webpack-plugin');

module.exports = new SpeedMeasurePlugin({
  outputFormat: 'human',
  outputTarget: console.log,
  pluginNames: true,
  granularLoaderData: true
}).wrap({
  // webpack 配置
});
```

#### 自定义性能统计

```javascript
class BuildStatsPlugin {
  apply(compiler) {
    compiler.hooks.done.tap('BuildStatsPlugin', (stats) => {
      const buildTime = stats.endTime - stats.startTime;
      const assets = stats.compilation.getAssets();
      
      console.log('\n=== Build Statistics ===');
      console.log(`Build Time: ${(buildTime / 1000).toFixed(2)}s`);
      console.log(`Assets Count: ${assets.length}`);
      console.log(`Total Size: ${this.formatBytes(stats.compilation.outputOptions.totalSize)}`);
      
      // 显示最大的文件
      const largestAssets = assets
        .sort((a, b) => b.size - a.size)
        .slice(0, 5);
      
      console.log('\n=== Largest Assets ===');
      largestAssets.forEach(asset => {
        console.log(`${asset.name}: ${this.formatBytes(asset.size)}`);
      });
    });
  }
  
  formatBytes(bytes) {
    if (bytes === 0) return '0 Bytes';
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return Math.round(bytes / Math.pow(k, i) * 100) / 100 + ' ' + sizes[i];
  }
}
```

## 最佳实践

### 1. 开发环境配置

```javascript
module.exports = {
  mode: 'development',
  devtool: 'eval-cheap-module-source-map',
  cache: {
    type: 'filesystem'
  },
  devServer: {
    hot: true,
    liveReload: false,
    client: {
      overlay: {
        errors: true,
        warnings: false
      },
      progress: true
    },
    watchOptions: {
      poll: 1000,
      aggregateTimeout: 300,
      ignored: /node_modules/
    }
  },
  optimization: {
    runtimeChunk: 'single',
    removeAvailableModules: false,
    removeEmptyChunks: false,
    splitChunks: false,
    usedExports: false,
    sideEffects: false,
    concatenateModules: false
  }
};
```

### 2. 生产环境配置

```javascript
module.exports = {
  mode: 'production',
  devtool: 'source-map',
  cache: {
    type: 'filesystem'
  },
  output: {
    filename: '[name].[contenthash:8].js',
    chunkFilename: '[name].[contenthash:8].chunk.js',
    assetModuleFilename: 'assets/[name].[contenthash:8][ext]'
  },
  optimization: {
    minimize: true,
    nodeEnv: 'production',
    moduleIds: 'deterministic',
    runtimeChunk: 'single',
    usedExports: true,
    sideEffects: true,
    concatenateModules: true,
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          reuseExistingChunk: true,
          name: 'vendors'
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    }
  },
  performance: {
    hints: 'warning',
    maxEntrypointSize: 250000,
    maxAssetSize: 250000
  }
};
```

### 3. 大型项目配置

```javascript
module.exports = {
  // 多入口配置
  entry: {
    main: './src/main.js',
    admin: './src/admin.js',
    vendor: './src/vendor.js'
  },
  
  // 优化构建范围
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        include: [path.resolve(__dirname, 'src')],
        exclude: [path.resolve(__dirname, 'node_modules')],
        use: [
          {
            loader: 'thread-loader',
            options: {
              workers: require('os').cpus().length - 1
            }
          },
          {
            loader: 'babel-loader',
            options: {
              cacheDirectory: true,
              cacheCompression: false
            }
          }
        ]
      }
    ]
  },
  
  // 缓存配置
  cache: {
    type: 'filesystem',
    buildDependencies: {
      config: [__filename]
    }
  },
  
  // 代码分割
  optimization: {
    splitChunks: {
      chunks: 'all',
      maxInitialRequests: Infinity,
      minSize: 20000,
      maxSize: 244000,
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          reuseExistingChunk: true
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    }
  }
};
```

## 性能优化检查清单

### 构建速度优化
- [ ] 启用 Loader 缓存
- [ ] 启用 Webpack 文件系统缓存
- [ ] 精确配置 include/exclude
- [ ] 优化 resolve 配置
- [ ] 使用多线程处理
- [ ] 分离开发和生产配置
- [ ] 使用 noParse 排除无依赖模块
- [ ] 优化开发服务器配置
- [ ] 减少构建范围
- [ ] 使用 Dll 预构建

### 产物体积优化
- [ ] 实现代码分割
- [ ] 启用 Tree Shaking
- [ ] 压缩 JavaScript 代码
- [ ] 压缩 CSS 代码
- [ ] 压缩 HTML 代码
- [ ] 优化图片资源
- [ ] 优化字体资源
- [ ] 使用外部依赖
- [ ] 配置长期缓存
- [ ] 移除未使用的代码

### 运行时性能优化
- [ ] 实现路由懒加载
- [ ] 实现组件懒加载
- [ ] 配置 Service Worker
- [ ] 使用 CDN 加速
- [ ] 实现预加载和预取
- [ ] 优化缓存策略
- [ ] 使用 HTTP/2
- [ ] 配置 Gzip 压缩
- [ ] 配置 Brotli 压缩
- [ ] 优化资源加载顺序

### 监控和分析
- [ ] 配置 Bundle Analyzer
- [ ] 设置性能阈值
- [ ] 统计构建时间
- [ ] 监控资源大小
- [ ] 设置性能预算
- [ ] 配置 CI/CD 检查
- [ ] 使用 Lighthouse
- [ ] 监控真实性能
- [ ] 定期审查优化效果
- [ ] 保持配置更新

## 注意事项

1. **平衡性**：在构建速度和产物体积之间找到平衡点
2. **渐进式优化**：不要一次性应用所有优化策略
3. **测试验证**：每次优化后都要进行功能测试
4. **环境差异**：开发和生产环境的配置要有明显差异
5. **版本兼容**：注意 Webpack 版本升级带来的变化
6. **团队协作**：建立统一的优化标准和最佳实践
7. **持续监控**：定期检查性能指标和优化效果
8. **文档记录**：记录优化过程和决策理由

## 总结

Webpack 性能优化是一个系统性的工程，需要从构建速度、产物体积、运行时性能等多个维度进行综合考虑。通过合理配置缓存、代码分割、压缩策略和监控工具，可以显著提升前端应用的构建效率和运行性能。

在实际项目中，建议根据具体需求和优先级选择合适的优化策略，避免过度优化。持续监控和分析是保持高性能的关键，定期审查和调整配置能够确保项目长期保持良好的性能表现。

记住性能优化是一个持续的过程，随着项目的发展和新技术的出现，需要不断调整和优化构建配置。