# Webpack loader的使用
## babel-loader
### 安装babel
#### 运行 npm install babel-loader babel-core --save-dev

### 继续安装 babel的插件 babel-latest
#### 运行 npm install --save-dev babel-preset-latest

### loaders参数,loaders是数组
- test 正则表达式
- loader 加载器 要加上-loader
- exclude 去除指定路径下的文件 babel特有的？
- include 包含指定路径下的文件 babel特有的？
- query babel特有的 例 query:{presets:['latest']},表示转换成最新版本

## css-loader和style-loader
- test 正则表达式
- loader 加载器 要加上-loader

## postcss-loader 给样式加浏览器前缀
### 安装 postcss-loader和autoprefixer
### 使用 
#### 在根目录下新建文件postcss.config.js
#### 输入以下内容即可，不用改webpack.config.js
```
module.exports={
    plugins:[
        require('autoprefixer')({
            browsers:['last 5 versions']
        })
    ]
};
```
