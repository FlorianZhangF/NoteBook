# webpack的简单使用

## 第一步 初始化node

#### npm init 

## 第二步 安装webpack开发环境

#### npm install webpack --save-dev

## 第三步 使用webpack对文件进行打包

#### webpack hello.js hello.bundle.js
#### hello.js是要打包的文件
#### hello.bundle.js是目标文件

### 列名说明
- Asset 打包生成的文件
- Size 文件大小
- Chunks 此次打包的分块
- Chunk Names 此次打包的块名称

-----------------------------------------------------------

# 进阶webpack

## 使用loader

## 第一步 先安装loader

#### npm install css-loader style-loader --save-dev

## 第二步 在require的内容中指定css-loader和style-loader

#### require('style-loader!css-loader!./style.css')

#### css-loader使得webpack可以操作.css的文件

#### style-loader将css-loader处理完的文件,新建一个style标签,插入到html中

## webpack 其他选项

- --watch 文件变动自动打包
- --progress 显示打包进度
- --display-modules 显示打包的模块
- --display-reasons 显示打包的原因

## 补充说明

#### 项目文件夹说明
- src 项目文件
- dist 资源文件

# webpack配置文件详解

#### 直接执行webpack,会默认读取目录下的webpack.config.js文件

## 参数说明

#### context 上下文,默认值为运行脚本的根目录
#### entry 入口文件
#### output 生成文件配置
##### path 生成路径
##### filename 生成的文件

## 插件使用

### html-webpack-plugin

#### 执行 npm install html-webpack-plugin --save-dev 安装插件

#### 参数列表

- filename 生成文件(可用占位符来使用动态名字)
- template 模板文件
- inject 生成的js放在模板文件的哪个标签里('head'、'body'等)
- chunks 指定需要包含的模块
- excludechunks 指定排除的模块

#### 参数都可以在模板页中使用
#### 模板语法 
- 取某个key值 <%= htmlWebpackPlugin.options.title %>
- 一段脚本语言 
```
遍历htmlWebpackPlugin.options
<% for(var key in htmlWebpackPlugin.files){ %>
<%= key %> : <%= JSON.stringify(htmlWebpackPlugin.files[key])  %>
<% } %>
```

### 使得模板页直接将模块内容添加到生成的文件中,以减少http请求
#### 在模板页中加入 
```
<script type="text/javascript">
        <%= 
            compilation.assets[htmlWebpackPlugin.files.chunks.main.entry.substr(htmlWebpackPlugin.files.publicPath.length)].source() 
        %>
</script>
```

