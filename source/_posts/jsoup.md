---
title: Jsoup
dete: 2018-08-19 14:09:25
top: 9
tags:
---
　　对HTML页面的解析，之前我一般使用HTMLParser，详细见HTMLParser的学习系列 - 学习总结，但是这个项目已经停止更新。现在比较好的解析HTML的控件是Jsoup。本文对Jsoup的用法做个总结.
<!-- more -->
#### 1. 概述

　　Jsoup的主要功能有三部分组成：
　　`a) 从字符串，网页，本地文件等方式生成Documentn`
　　`b) 在生成Doucment后，根据Dom和css或类似jquery的selector语法获取Element，然后再从Elements中获取节点属性、文本、html`
　　`c) 对Element的进行操作，包括HTML的值、节点内容的值和设置节点属性的值
下方每节对以上三点进行逐一演示。`

###### POM.xml 配置

```
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.10.2</version>
</dependency>
```

#### 2.生成Document
　　JSOUP通过不同方式生成Document，主要有以下三种：
　　`a) 字符串`
　　`b) 网页`
　　`c) 本地文件`
###### 2.1 从字符串生成Document
　　关键方法： Jsoup.parse(String html)
```
	public void fromString() {
        String html = "<html><head><title>First parse</title></head>"
                + "<body><p>Parsed HTML into a doc.</p></body></html>";
        Document document = Jsoup.parse(html);
    }
```

###### 2.2 从网页生成Document
　　关键方法： Jsoup.connect()
```
    public void fromURL() {
        Document document;
        try {
        	//通过URL+访问的方法获取Document
            document = Jsoup.connect("https://www.baidu.com/").get();
            // 从document中获取title值
            String title = document.title();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```
###### 2.3 从本地文件生成Document
　　关键方法： Jsoup.connect(File file,String charset)
```
    public void fromFile(){
            File file = new File("filePath");
            try {
                //通过文件+编码集来获取Document
                Document document = Jsoup.parse(file,"utf-8");
                System.out.println(document.title());
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
```
#### 3. 获取Element及节点属性、文本、html
　　在上一节中已经生成Document，下面就可以对这个document进行操作，操作的主要单位是Element，下面介绍如何选取elment及获取elment的内容。
###### 3.1 获取Element，获取的方式分为二种:
　　`a) 通过DOM解析来获取;`
```
    public void extractDataByDOM() throws IOException{
        Document doc = Jsoup.connect("https://www.baidu.com/").get();
        Element lg = doc.getElementById("lg");
        logger.info("getElementById lg = {}", lg);
        Elements links = doc.getElementsByTag("a");
        for (Element link : links) {
          String linkHref = link.attr("href");
          String linkText = link.text();
          logger.info("linkHref={}, linkText={}",linkHref, linkText);
        }
    }
```
　　`b) 通过css或类似jquery的selector语法来获取；`
```
    public void select4J() throws IOException{
        File input = new File("filePath");
        Document document = Jsoup.parse(input, "UTF-8", "http://example.com/");

        // 获取所有的a节点
        Elements links = document.select("a[href]");
        System.out.println(links);
        // 获取img的src以.png结果结尾
        Elements pngs = document.select("img[src$=.png]");
        System.out.println(pngs);
        // 获取class=masthead的div节点
        Element masthead = document.select("div.masthead").first();
        System.out.println(masthead);

        // 获取class=r的h3节点下面的a节点
        Elements resultLinks = document.select("h3.r > a");
        System.out.println(resultLinks);

    }
```
###### 3.2 获取节点属性、文本、html
###### 3.3 Select选择器
#### 4. 设置节点值
　　设置节点值，主要有在以下方式：
　　`a) 设置节点HTML的值`
　　`b) 设置节点内容的值`
　　`c) 设置节点属性的值`
###### 4.1 设置节点HTML的值
　　1. Element.html：使用新的HTML替换旧的值
　　2. Element.prepend：将新html添加到指定节点内部的最前面
　　3. Element.append：将新html添加到指定节点内部的最后面
　　4. Element.wrap:将指定节点封装到html最里面

###### 4.2 设置节点内容的值
　　1. Element.text: 完全替换内容
　　2. Element.prepend:在节点的内容最前面加内容
　　3. Element.append：在节点的内容最后面加内容

###### 4.3 设置节点属性的值
　　1. element.attr: 设置属性
　　2. element.addClass: 设置class
