# fed-e-task-02-02
### 一、简答题
#### 1、Webpack 的构建流程主要有哪些环节？如果可以请尽可能详尽的描述 Webpack 打包的整个过程。
- 构建流程
    - 根据entry配置找到入口文件
    - 查找入口文件中import、require语句解析推断出这个文件所依赖的资源模块
    - 依次解析每个资源模块对应的依赖
    - 得到整个项目资源模块依赖关系树
    - 递归这个树找到每个节点对应的资源文件
    - 根据配置中的rules属性找到这个模块对应的加载器，然后交给它去加载这个模块
    - 最后将加载的结果输出到打包结果文件中

#### 2、Loader 和 Plugin 有哪些不同？请描述一下开发 Loader 和 Plugin 的思路。
- Loader 和 Plugin 不同点
    - loader是一个转换器，将文件A编译成列外一个文件B
    - plugin是一个扩展器，主要针对loader结束后的操作。在webpack打包过程中，它不直接操作文件，而是基于事件工作机制监听webpack打包过程中的某些节点执行相应任务
- 开发一个loader
    - 在文件中定义一个导出函数，这个函数接受加载的文件内容作为输入参数
    - 通过这个函数内部对文件内容进行加工
    - 通过加工后，函数最终需要返回一段标准的JavaScript代码（一般将结果作为当前模块导出。如：module.exports = 'xxx'）
- 开发一个plugin
    - 定义一个类，类中实现了apply方法
    - webpack在启动时会自动调用apply方法，apply方法接受一个compile对象参数
    - 通过compile中的hooks对象访问到对应的钩子，并使用tap进行注册钩子函数，如：compile.hooks.emit.tap('MyPlugin', compilation => {})
    - 钩子函数中第一个参数传递当前插件名称，第二个参数挂载这个钩子上的回调函数
    - 通过回调函数中传递的compilation（此次打包的上下文）对象获取当中的资源信息
    - 通过遍历compilation.assets[name].source()方法拿到打包每个资源文件的内容
    - 通过操作后，使用compilation.assets[name] = {}覆盖当前资源文件内容
    - 需要指定source属性方法返回内容，指定size属性方法返回文件大小
    - 最后在plugin中使用new插件类方式使用插件
### 二、编程题
#### 1、使用 Webpack 实现 Vue 项目打包任务
- 项目地址：[vuw-app-webpack](https://github.com/Xwatson/vue-app-webpack)

- 创建公共配置文件[webpack.common.js](https://github.com/Xwatson/vue-app-webpack/blob/master/webpack.common.js)，配置入口、出口、loader
    - 使用了vue-loader、vue-loader/lib/plugin处理vue文件
    - 使用babel-loader处理js文件
    - 在vue和js文件处理之前进行一次eslint校验
    - 使用url-loader处理图片文件，并设置10kb以下文件转换base64格式输出
    - 使用HtmlWebpackPlugin输出html模板
- 创建开发环境配置文件[webpack.dev.js](https://github.com/Xwatson/vue-app-webpack/blob/master/webpack.dev.js)
    - 设置mode类型为development
    - 因为使用eslint控制每行最多80个字符，则使用cheap-module-eval-source-map输出行级map文件提高频繁修改文件编译效率
    - 使用devServer开启hotOnly热更新，使用contentBase指定静态资源路径
    - 开发环境下没有使用单独提取css文件，单独配置了css和less的loader
    - 使用DefinePlugin定义开发环境下BASE_URL
    - 最后使用webpack-merge将common文件与当前dev文件合并导出
- 创建生产环境配置文件[webpack.prod.js](https://github.com/Xwatson/vue-app-webpack/blob/master/webpack.prod.js)
    - 设置mode类型为production
    - 覆盖公共output.filename，这里使用contenthash:8输出文件名称
    - 生产环境不设置source-map
    - 使用usedExports忽略导出成员没使用会被压缩掉，开启了splitChunks提取公共模块，minimizer配置压缩css文件
    - 生产环境重新配置了css和less的loader器，这里使用了MiniCssExtractPlugin单独提取css文件
    - 其他插件方面使用DefinePlugin定义生产环境BASE_URL，CleanWebpackPlugin打包前清除输出目录，CopyWebpackPlugin复制public文件到dist，MiniCssExtractPlugin种配置文件名称为contenthash:8
    - 最后使用webpack-merge将common文件与当前dev文件合并导出
- 配置[package.json](https://github.com/Xwatson/vue-app-webpack/blob/master/package.json)中script运行脚本