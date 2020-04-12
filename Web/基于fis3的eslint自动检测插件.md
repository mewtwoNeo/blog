# 基于fis3的eslint自动检测插件

## 起因
团队发展到一定规模就需要有一个统一的标准去约束成员，统一编码风格。要完成这件事就需要先确定几个事：

- 用什么样的编码规范
- 是自己定义一套还是基于开源标准
- 编码规范怎么在实际开发中执行

## 选择规范
在团队中开了几次头脑风暴，也实际调研了下现有的编码规范。目前比较流向的两个规范：

- feross 的 standard 标准,目前在 github 上有1w多个star [github地址](https://github.com/feross/standard) [官网地址](https://standardjs.com/)
- airbnb 公司的 javascript 标准，目前在 github 上 star 已经突破5w [github地址](https://github.com/airbnb/javascript)

虽然 airbnb 公司的方案已有 5w 个 star，不过 standard 标准大有赶超的意思，所以最后选择了基于standard来做 js 规范。

## 确定方案
好了，现在确定了编码规范，剩下的就是怎么去落地标准。如果只是让每个人去学习标准并不能保证所有人都能严格的执行标准去写代码，所以最好的办法还是在开发阶段就发现问题，并且必须要解决了错误才能继续开发，这样就能保证大家都是一样的标准了。

由于团队使用的是百度的 [fis3](http://fis.baidu.com/) 作为构建工具，所以第一想法就是用 fis3+eslint 来检查代码。
于是就在 fis3 的插件列表中搜了下，找到几款基于 eslint 的 fis3 插件：
![fis3 eslint](https://user-images.githubusercontent.com/7699570/30841341-83aeedf0-a2ae-11e7-83ee-53136643f4c0.png 'fis3 eslint')


但是美中不足的是这些插件大都是在终端中显示错误信息，而且错误信息的格式也不优雅：
![fis3 eslint zsh](https://user-images.githubusercontent.com/7699570/30841386-b64d3bb8-a2ae-11e7-9f56-5363f6c276c3.png 'fis3 eslint zsh')


基于以上原因最终确定自己开发一款基于fis3，eslint和standard标准的插件。

## fis钩子
fis提供了编译阶段插件，打包阶段插件，Deploy 插件和命令行插件 [可参见这篇文档](http://fis.baidu.com/fis3/docs/api/dev-plugin.html)
因为是要在开发阶段实时对代码检测所以选择了编译阶段插件
> 在编译阶段，文件是单文件进行编译的，这个阶段主要是对文件内容的编译分析；这个阶段分为 lint、parser、preprocessor、postprocessor、optimizer 等插件扩展点
> - lint：代码校验检查，比较特殊，所以需要 release 命令命令行添加 -l 参数
> - parser：预处理阶段，比如 less、sass、es6、react 前端模板等都在此处预编译处理
> - preprocessor：标准化前处理插件
> - standard：标准化插件，处理内置语法
> - postprocessor：标准化后处理插件

从官网的这段话可以看出fis已经有了代码检查的扩展点，直接选择在lint上做插件就行。

## fis插件
选定了扩展点，接下来看看fis的插件是怎么开发的
``` javascript
/**
 * Compile 阶段插件接口
 * @param  {string} content     文件内容
 * @param  {File}   file        fis 的 File 对象 [fis3/lib/file.js]
 * @param  {object} settings    插件配置属性
 * @return {string}             处理后的文件内容
 */
module.exports = function (content, file, settings) {
    return content;
};

```
从api上来看还是挺简单的，基本参数都有传给回调函数。下面举一个官网的例子来看看具体是怎么调用的
### 插件
```javascript
// vi my-proj/node_modules/fis3-<type>-<name>/index.js

  module.exports = function (content, file, settings) {
      // 只对 js 类文件进行处理
      if (!file.isJsLike) return content;
      return content += '\n// build ' + Date.now()
  }
```
### 插件
```javascript
// fis-conf.js
  fis.match('*.js', {
      <type>: fis.plugin('<name>', {
          //conf
      })
  })
```
在fis-conf.js文件上配置好扩展点（type）和对应的插件名（fis3-<type>-<name>）就可以。

## 开发
基本开发流程已经清楚了，接下来就是具体的开发了。
我们先来看看fis3-lint-noob-eslint在fis-conf.js中的配置
```javascript
var config = {
    ignoreFiles:[],
    envs: [],
    globals: [],
    rules: {
        // 不必要非得用骆驼拼接法
      "camelcase": 0,
        // 设置为0，代表 == 也可以不必要非得 ===
      "eqeqeq": 0
    }
}


fis.match('*.js', {
    lint: fis.plugin('noob-eslint', config)
})
```
config是对eslint的自定义配置文件。具体配置可以参考[Configuring ESLint](https://eslint.org/docs/user-guide/configuring)
在不传config的情况下，插件默认使用feross的standard标准

### 插件开发

其中会用到一些必要的node包
- eslint 代码检查
- eslint-config-standard standard 标准的配置
- eslint-friendly-formatter 美化输出

```javascript
const formatter = require('eslint-friendly-formatter')
const standard = require('eslint-config-standard')
const CLIEngine = require("eslint").CLIEngine
```

从前面fis官网的例子来看，回调函数里content应该就是需要检查的文件的内容了，我们只需要对取到的文件内容利用eslint按照standard标准检查，然后输出美化后的结果就行。

```javascript
module.exports = function (content) {

    // 实例化并传入配置信息
    const cli = new CLIEngine(config)

    // 返回eslint检测结果
    const report = cli.executeOnText(content)

    // 判断是否有错误信息
    if (report.errorCount || report.warningCount) {

        // 格式化错误信息
        const output = formatter(report.results)

        // 在fis里输出错误信息
        fis.log.info(output)
    }
}
```
fis的命令窗口里就会输出相应文件的错误信息
![fiserror](https://user-images.githubusercontent.com/7699570/40605442-fffbadde-6294-11e8-9619-22f27bac55df.jpg)

最后有个问题就是错误信息只是在命令行里输出了，如果开发人员不去修改错误，程序还是能编译通过，所以可以采取一些强制性的措施，比如我们所做的，如果不修改就不让你预览界面
![fiserrorweb](https://user-images.githubusercontent.com/7699570/40605857-2f26048c-6296-11e8-8f63-507011604ae9.jpg)
