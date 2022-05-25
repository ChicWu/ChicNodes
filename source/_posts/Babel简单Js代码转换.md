---
title: Babel简单js代码转换
date: 2018-08-30 19:17:50
top: 9
tags:
	nodejs
---
　　近期get一个需求：需要对原始的js代码转换功能进行改进，在原始的通过正则匹配转换前增加一层更加准确的ast(抽象解析树)转换。初期的想法是使用jdk9，acorn和babel都进行测试然后选择最优。然后通过实践后发现jdk9只能做到对js的ast解析无法对ast进行更改以及ast转换为js操作，acorn与babel两者都是基于node.js的模块都可以实现js的转换功能但是相比而言acorn的相关文献太少且不完整，另一方面acorn主要是侧重于对js代码的解析在进行js转换的过程中发现有很多的坑步履艰难，而babel则不一样在js的解析方面是基于acorn有着与acorn几乎相同的解析速度，并且在转换方面有着一套完整的开发体系以及有着一套支持多国语言的[开发文档](https://github.com/jamiebuilds/babel-handbook/blob/master/README.md)学习的效率远高于acorn。
<!-- more -->

## 一，简介
### 1.1 ast(抽象语法树):
　　抽象语法树（abstract syntax tree或者缩写为AST），或者语法树（syntax tree），是源代码的抽象语法结构的树状表现形式，这里特指编程语言的源代码。和抽象语法树相对的是具体语法树（concrete syntaxtree），通常称作分析树（parse tree）。一般的，在源代码的翻译和编译过程中，语法分析器创建出分析树。一旦AST被创建出来，在后续的处理过程中，比如语义分析阶段，会添加一些信息。
　　通过AST可以对源代码进行更为准确的定位以及更换，并且有着正则无法实现的上下文结合，可以对源码进行手术刀级别的修改。
　　推荐两个在线生成AST工具：[Astexplorer](https://astexplorer.net/)，[Esprima](http://esprima.org/demo/parse.html#)
### 1.2 nodejs:
　　Nodejs框架是基于V8的引擎，是目前速度最快的Javascript引擎。chrome浏览器就基于V8，同时打开20-30个网页都很流畅。Nodejs标准的web开发框架Express，可以帮助我们迅速建立web站点，比起PHP的开发效率更高，而且学习曲线更低。
　　JS是脚本语言，脚本语言需要一个解析器才能运行。对于写在HTML页面里面的JS，浏览器充当了解析器的角色。而对于需要独立运行的JS，NodeJS就是一个解析器可以通过命令直接运行js。
　　每种解析器就是一个运行环境，不但允许JS定义各种数据结构，进行各种计算，还允许JS使用运行环境提供的内置对象和方法做一些事情。例如运行在浏览器中的JS的用途是操作DOM，浏览器就提供了document之类的内置对象。而运行在NodeJS中的JS的用途是操作磁盘文件或者搭建HTTP服务器，NodeJS就相应提供了fs、http等内置对象。
　　另外Node.js生态圈也是非常的强大有则很多有些的第三法模块比如本次使用的babel,acorn，Bloger的福音Hexo等
### 1.3 babel：
　　Babel 是一个通用的多用途 JavaScript 编译器。通过 Babel 你可以使用（并创建）下一代的 JavaScript，以及下一代的 JavaScript 工具。作为一种语言，JavaScript 在不断发展，新的标准／提案和新的特性层出不穷。 在得到广泛普及之前，Babel 能够让你提前（甚至数年）使用它们。 Babel 把用最新标准编写的 JavaScript 代码向下编译成可以在今天随处可用的版本。 这一过程叫做“源码到源码”编译， 也被称为转换编译（transpiling，是一个自造合成词，即转换＋编译。以下也简称为转译）。 例如，Babel 能够将新的 ES2015 箭头函数语法：
　　`const square = n => n * n;
　　转译为：
```
    const square = function square(n) {
        return n * n;
      };
```
　　不过 Babel 的用途并不止于此，它支持语法扩展，能支持像 React 所用的 JSX 语法，同时还支持用于静态类型检查的流式语法（Flow Syntax）。更重要的是，Babel 的一切都是简单的插件，谁都可以创建自己的插件，利用 Babel 的全部威力去做任何事情。再进一步，Babel 自身被分解成了数个核心模块，任何人都可以利用它们来创建下一代的 JavaScript 工具。已经有很多人都这样做了，围绕着 Babel 涌现出了非常大规模和多样化的生态系统。
　　以上描述来自Babel官方github，也就是说Babel可以进行js代码的转换并且拥有这丰富的插件来按照指定规则来转换，最重要的还可以通过自定义插件。(本次主要使用babel的基本语法，插件的编写在后期博客中讲解)
## 二，安装教程
### 2.1 nodejs:
　　从[官网]()上点击对应的版本下载安装包，然后无脑下一步进行安装。
　　配置环境变量将node.js的安装路径添加到Path当中(为了能够在windos中全局调用node.js命令)
　　现在就可以通过`node xxx.js`执行js文件了
### 2.2 babel:
　　`babel-cli`
　　Babel 的 CLI 是一种在命令行下使用 Babel 编译文件的简单方法。其安装的方法有两种：全局安装和在项目中安装，官方推荐的是在项目中安装，个人也比较认同这一点。
　　首先建立一个文件夹比如`babel`可以在node.js安装目录下也可以在其他目录下，建议不在node.js安装目录下，创建后在打开cmd窗口并进入该目录进行下一步操作。
##### a）全局安装
　　通过命令`$ npm install --global babel-cli`在babel-cli安装完后就可以通过执行`$ babel SimpleTest.js`来执行我们的第一个文件。不过这只是简单的将js结果输出到控制台
　　使用`$ babel example.js --out-file compiled.js`或者`$ babel example.js -o compiled.js`可以将结果写入到指定的文件。
　　如果我们想要把一个目录整个编译成一个新的目录，可以使用 --out-dir 或者 -d`$ babel src --out-dir lib` `$ babel src -d lib`
　　如果想要卸载全局安装的babel的话可以通过执行`$ npm uninstall --global babel-cli`
##### b) 在项目中安装
　　通过命令`$ npm install --save-dev babel-cli`完成安装后你的`package.json`应该如下所示：
```$ package.json
    json{
      "name": "my-project",
      "version": "1.0.0",
      "devDependencies": {
        "babel-cli": "^6.0.0"
      }
    }
```
　　如果我们不想直接从命令行运行 Babel 了，取而代之我们将把运行命令写在 npm scripts 里，这样可以使用 Babel 的本地版本。只需将 `scripts` 字段添加到你的 `package.json` 文件内并且把 babel 命令写成 build 字段。.
```$ package.json
      {
        "name": "my-project",
        "version": "1.0.0",
    +   "scripts": {
    +     "build": "babel src -d lib"
    +   },
        "devDependencies": {
          "babel-cli": "^6.0.0"
        }
      }
```
然后在终端里运行：`npm run build`就可以了
## 三， 进行js代码转换
### 3.1 babel的解析引擎
　　Babel使用的引擎是babylon，babylon并非由babel团队自己开发的，而是fork的acorn项目，不过acorn引擎只提供基本的解析ast的能力。
　　遍历还需要配套的babel-travesal, 替换节点需要使用babel-types，而这些开发，在Babel的插件体系开发下，变得一体化了。
### 3.2 使用babel做js的代码转换
　　进行js代码的转换主要分为三个步骤：js代码解析为AST树，对AST树进行遍历修改，将修改后的AST树转换为新的js代码。
#### a) js代码解析为AST树
　　[`babylon`](https://github.com/babel/babylon)<--点一下，打来惊喜
　　将js源码转换为AST用到的模块叫：babylon，Babylon 是 Babel 的解析器。最初是 从Acorn项目fork出来的。Acorn非常快，易于使用，并且针对非标准特性(以及那些未来的标准特性) 设计了一个基于插件的架构。
　　运行以下命令进行安装：
　　`$ npm install --save babylon`
　　使用的方法如下：
```
    const babylon = require('babylon');
    
    const code = `function square(n) {
      return n * n;
    }`;  
    console.log(babylon.parse(code));
```
　　也可以通过如下的方法传递选项给parse()
```
    babylon.parse(code, {
      sourceType: "module", // default: "script"
      plugins: ["jsx"] // default: []
    });
```
　　sourceType 可以是 "module" 或者 "script"，它表示 Babylon 应该用哪种模式来解析。 "module" 将会在严格模式下解析并且允许模块定义，"script" 则不会。
　　由于 Babylon 使用了基于插件的架构，因此有一个 plugins 选项可以开关内置的插件。 注意 Babylon 尚未对外部插件开放此 API 接口，不排除未来会开放此API。
　　要查看完整的插件列表，请点击这个[链接](https://github.com/babel/babylon/blob/master/README.md#plugins)
#### b) 对AST树进行遍历修改
　　[`babel-traverse`](https://github.com/babel/babel/tree/master/packages/babel-traverse)<--点一下，打开惊喜
　　Babel Traverse模块维护了整棵树的状态，并且负责替换、移除和添加节点。
　　运行以下命令安装：
　　`$ npm install --save babel-traverse`
#### c) 将修改后的AST树转换为新的js代码
　　[`babel-generator`](https://github.com/babel/babel/tree/master/packages/babel-generator)<--点一下,打开惊喜
　　Babel Generator模块是 Babel 的代码生成器，它读取AST并将其转换为代码和源码映射。
　　`$ npm install --save babel-generator`
#### d) 案例代码
```$ SimpleTest.js
    //引入fs模块
    var fs = require('fs');
    //引入babel模快
    const babylon = require('babylon');
    const Traverse = require('babel-traverse').default;
    const generator = require('babel-generator').default;
    //读取js文件
    var code =fs.readFileSync('./JsFile0.js', 'utf8');
    //修改js
    var paths = "";
    var ast = babylon.parse(code);
    Traverse(ast, {
      enter(path) {
        if (
          path.node.type === "Identifier" &&
          path.node.name === "n"
        ) {
          path.node.name = "x";
        }
      }
    });
    //写入生成新的js并写入文件
    fs.writeFile('./JsFile0.js',generator(ast).code,function(err){
        if(err) console.log('写文件操作失败');
        else console.log('写文件操作成功');
    });
```
```$ JsFile0.js
    a === n;
```
* 1 如果运行成功的话会在终端中输出`写文件操作成功`且JsFile0.js中的n会替换成x。
* 2 对于node.js当中的模块引入有两种方式`import`和`require`两种用法自行百度，我在这里说一下我用的是第二种方法因为import对于nodejs7.0以及之前的版本并不能识别。

### 3.3 通过java执行node.js
　　通过java代码直接执行cmd命令-->通过cmd来调用node.js执行js代码，原js打印到终端的信息会打印到控制台
　　注意事项：
　　　　1，java代码执行的cmd命令为项目的根目录 “/”为cmd中的下一级目录
　　　　2，如果执行的js文件用有引用到其他的文件相对路径是相对的cmd执行路径所以最好写成绝对路径如：“C:\\a\\b.js”
　　　　3，请耐心往下接着看，代码并非可以直接运行

```
    import java.io.*;

    public void nodeRunJ() {
            try {
                String line = null;
                String command = "node ./AstForBabel/SimpleTest.js";
                Process p = Runtime.getRuntime().exec(command);
                BufferedReader stdout = new BufferedReader(new InputStreamReader(
                        p.getInputStream()));
                while ((line = stdout.readLine()) != null) {
                    System.out.println(line);
                }
                stdout.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
```
* 由于编写的js代码用有用到babel模块中的一些函数，所以代码并非可以直接运行需要现在你的java工程中将之前安装的babel模块同样的安装在项目当中。
* 其实通过测试也可以不用重新安装可以将之前的`babel`目录下的文件直接粘贴到工程当中就可以。如果想要确定是否好使，稳妥起见最好现在项目的目录下打开终端运行一下node命令看是否可以运行


## 四， FAQ
### * 在安装过程中出现错误，无法找到`package.json`
　　可以先执行`$ npm init`命令然后按照提示回车,回车完成安装生成一个初始化的环境，然后再执行babel的安装
### * bbb
### * ccc