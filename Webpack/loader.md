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
#### 输入以下内容
```
module.exports={
    plugins:[
        require('autoprefixer')({
            browsers:['last 5 versions']
        })
    ]
};
```
#### 然后在css加载器那边最后加上postcss-loader即可
```
{
    test:/\.css$/,
    loaders:[
        'style-loader',
        'css-loader',
        'postcss-loader'
    ]
}
```

### 'css-loader?importLoaders=1' 加上importLoaders参数会将import进来的样式也进行postcss处理

## file-loader 文件加载器，可以加载图片
### 安装file-loader
### npm install file-loader --save-dev
### webpack 配置
```
{
    test:/\.(png|jpg|gif|svg)$/i,
    loader:'file-loader'
}
```
#### /i 表示忽略大小写

### 在模板中要引入相对路径的图片
### ${ require('../../assets/bg.png') }

### 可以自定义生成资源的生成路径以及生成名字
```
{
    test:/\.(png|jpg|gif|svg)$/i,
    loader:'file-loader',
    query:{
        name:'assets/[name]-[hash:5].[ext]'
    }
}
```

## url-loader 
### 和file-loader类似
### 大于设定的大小,则交给file-loader处理;如果小于指定大小,则转换为base64编码
```
{
    test:/\.(png|jpg|gif|svg)$/i,
    loader:'url-loader',
    query:{
        limit:20000,
        name:'assets/[name]-[hash:5].[ext]'
    }
}
```

## image-webpack-loader 对图片进行压缩，通常配合file-loader或者url-loader一起使用
### 安装 npm install image-webpack-loader --save-dev
```
{
    test:/\.(png|jpg|gif|svg)$/i,
    loaders:[
        'url-loader?limit=20000&name=assets/[name]-[hash:5].[ext]',
        'image-webpack-loader'
    ]
}
```

