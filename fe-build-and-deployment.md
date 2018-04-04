# 搭建一个基于 Gulp 的前端自动化部署系统

本文主要介绍从 0 开始，如何搭建一个前端自动化部署系统。

## 构建

前端项目的构建主要包含：js, css 的文件合并、js, css 的预处理、js, css 的压缩与优化、js, css 的语法检查、图片压缩与优化、资源文件加版本号、资源文件发布到 CDN、内联脚本压缩与优化等，并且支持增量构建。

下面说明一下如何实现上面所列的功能。

### 依赖包

例子 package.json 文件：
```json
{
  "name": "demo",
  "engines": {
    "node": ">=8.0.0"
  },
  "devDependencies": {
    "aliyun-sdk": "^1.11.5",
    "autoprefixer": "^8.2.0",
    "babel-core": "^6.26.0",
    "babel-preset-env": "^1.6.1",
    "babel-register": "^6.26.0",
    "cssnano": "^3.10.0",
    "del": "^3.0.0",
    "eslint": "^4.15.0",
    "eslint-config-google": "^0.9.1",
    "eslint-plugin-html": "^4.0.1",
    "eslint-plugin-local": "^1.0.0",
    "gulp": "^3.9.1",
    "gulp-autoprefixer": "^5.0.0",
    "gulp-concat": "^2.6.1",
    "gulp-cssnano": "^2.1.2",
    "gulp-eslint": "^4.0.1",
    "gulp-if": "^2.0.2",
    "gulp-imagemin": "^4.1.0",
    "gulp-load-plugins": "^1.5.0",
    "gulp-match": "^1.0.3",
    "gulp-postcss": "^7.0.1",
    "gulp-resource-hash": "^0.1.7",
    "gulp-size": "^3.0.0",
    "gulp-sizereport": "^1.2.0",
    "gulp-sourcemaps": "^2.6.4",
    "gulp-uglify": "^3.0.0",
    "gulp-util": "^3.0.8",
    "htmlparser2": "^3.9.2",
    "image-size": "^0.6.2",
    "lodash": "^4.17.4",
    "minimatch": "^3.0.4",
    "moment": "^2.20.1",
    "run-sequence": "^2.2.1",
    "shortid": "^2.2.8",
    "strip-comments": "^0.4.4",
    "strip-css-comments": "^3.0.0",
    "through2": "^2.0.3",
    "uglify-js": "^3.3.16"
  },
  "dependencies": {},
  },
  "browserslist": [
    "ie >= 8",
    "android >= 4.2",
    "ios >= 8",
    "> 1%"
  ]
}
```

通过在项目根目录执行 `npm install` 安装这些依赖包。

### 目录结构

``` bash
.
├── src/
│   ├── www/                    # 网站根目录(wwwroot)
│   │   ├── index.html          # config file for eslint with extra settings only for unit tests
│   │   ├── static/             # module assets (processed by webpack)
│   │   │   ├── js/             # config file for eslint with extra settings only for unit tests
│   │   │   │   └── ...         # config file for eslint with extra settings only for unit tests
│   │   │   ├── css/            # config file for eslint with extra settings only for unit tests
│   │   │   │   └── ...         # config file for eslint with extra settings only for unit tests
│   │   │   ├── img/            # config file for eslint with extra settings only for unit tests
│   │   │   │   └── ...         # config file for eslint with extra settings only for unit tests
├── .babelrc                    # babel config
├── .editorconfig               # indentation, spaces/tabs and similar settings for your editor
├── .eslintrc.js                # eslint config
├── .eslintignore               # eslint ignore rules
├── .gitignore                  # sensible defaults for gitignore
├── .postcssrc.js               # postcss config
├── package.json                # build scripts and dependencies
```
