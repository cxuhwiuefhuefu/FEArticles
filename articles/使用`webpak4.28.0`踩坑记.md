> 文章首发于我的[GitHub博客](https://github.com/cxuhwiuefhuefu/FEArticles)，如果觉得不错，欢迎给个start哈




# 使用`webpak4.28.0`踩坑记


实现功能为：将JS文件丶CSS文件和图片文件打包输出，html文件动态引入打包后的bundle.js丶CSS文件和图片

[demo地址](https://github.com/cxuhwiuefhuefu/studyWebpack)



#### 1. babel升级到7以后
- 需要引入@babel/core
- @babel/core只是babel-core的新版本




#### 2. webpack打包html里面img后src为“[object Module]”问题
- 如果使用`"file-loader": "^4.2.0"`或者`"file-loader": "^2.0.0"`却可以正常打包
- file-loader在新版本中`esModule`默认为true，因此手动设置为false



#### 3. webpack4.0以上用3.x extract-webpack-plugin 打包会不兼容，
- extract-webpack-plugin升级就可以了
- `npm install --save-dev extract-text-webpack-plugin@4.0.0-beta.0`

 

#### 4.使用webpack打包后点击index.html页面控制台提示Failed to load resource: net::ERR_FILE_NOT_FOUND的解决方法
- 在webpack.config.js文件中进行如下修改
  ```js
  output: {    
    publicPath: './'
  }
  ```
- 在package.json文件中进行如下修改  
  ```js
  {
    "homepage":"./"
  }
  ``` 




#### 如果你看完觉得这篇文章不错，帮我两件小事
- 关注公众号<text style="color: blue; font-weight: bold;">【前端应届生】</text>，持续为你推送精彩文章。
  
  <img style="width: 200px" src="https://imgkr.cn-bj.ufileos.com/51a2dad3-ae5b-4f66-9717-f0a36e4c68a7.png">
- 加我备注<text style="color: blue" style="color: blue;">【加群】</text>拉你进超级前端交流群，和各大厂的同学一起交流学习，共同进步。
  
  <img style="width: 200px" src="https://imgkr.cn-bj.ufileos.com/36e0b949-54df-4698-9ca2-d7e39ed0e2c0.jpg">  