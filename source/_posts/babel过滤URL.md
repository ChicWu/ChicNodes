---
title: Babel过滤URL
date: 2018-09-11 16:30:18
top: 7
tags:
	nodejs
---
　　通过nodejs+babel查找所有js文件中的URL
<!-- more -->
```
    var fs = require('fs');
    var path = require('path');
    const babylon = require('babylon');
    const Traverse = require('babel-traverse').default;
    const generator = require('babel-generator').default;
    const Types = require('babel-types');
    const babel = require('babel-core');

    //解析需要遍历的文件夹，我这以E盘根目录为例
    var filePath = path.resolve('C:\\Users\\Administrator.FO13YM4ZWHL5OAN\\Desktop\\babel\\AstForBabel\\conference');
    var reg = /(.js)$/i;

    //调用文件遍历方法
    fileDisplay(filePath);

    /**
     * 文件遍历方法
     * @param filePath 需要遍历的文件路径
     */
    console.log(">>>>>>>>>>>>>>>程序开始>>>>>>>>>>>>>>>");

    function findurl(filedir){
      // urlast转换 
      try{
        var code =fs.readFileSync(filedir, 'utf8');
        var ast = babylon.parse(code);
        Traverse(ast,{
          Identifier(path){
            if (path.node.name === "url") {
              var parentcode = generator(path.parent).code;
              var reg1 = /^(url)[:=\s]/i;
              if (reg1.test(parentcode)) {
                console.log(parentcode);
              };

            }
          }
        })
        // console.log(filedir);
        fs.writeFile(filedir,generator(ast).code,function(err){
            if(err) console.log('写文件操作失败');
        })
      } catch(error){
        // console.log("!!!!!!!!!!" + filedir);
      }  
    }

    function fileDisplay(filePath){
      //根据文件路径读取文件，返回文件列表
      fs.readdir(filePath,function(err,files){
        if(err){
          console.warn(err)
        }else{
          //遍历读取到的文件列表
          files.forEach(function(filename){
            //获取当前文件的绝对路径
            var filedir = path.join(filePath,filename);
            //根据文件路径获取文件信息，返回一个fs.Stats对象
            fs.stat(filedir,function(eror,stats){
              if(eror){
                console.warn('获取文件stats失败');
              }else{
                var isFile = stats.isFile();//是文件
                var isDir = stats.isDirectory();//是文件夹
                if(isFile){
                  if (reg.test(filedir)) {
                    findurl(filedir);
                  }
                }
                if(isDir){
                  fileDisplay(filedir);//递归，如果是文件夹，就继续遍历该文件夹下面的文件
                }
              }
            })
          });
        }
      });
    }


```