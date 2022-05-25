---
title: InnerClass
date: 2018-09-12 14:08:55
top: 8
tags:
---
　　`震惊！外部类竟可以访问内部类private变量！！！`
<!-- more -->
　　标题致敬著名的某头条，在最近的开发过程中写了一段类似与这样的代码，惊奇的发现外部类竟然能对内部类做这样的事情！！
```
    public class OuterClass {
        private String outerStr;
        public void test(){
            new InnerClass().innerStr = "外部类也可以修改内部类私有变量" ;
        }
        public class InnerClass{
            private String innerStr;
            public void test1(){
                outerStr = "内部类可以直接修改外部类私有变量";
            }
        }
    }
```
　　以我的暴脾气，既然遇见了一个问题就要刨根问底然后我有写了如下的测试，分别测试了成员内部类和方法内部类的私有变量以及私有方法的，并且还特地的将成员内部类与方法内部类写成了一样的。发现并不会出现类冲突，但是在方法内方法内部类会覆盖掉成员内部类。
```
    public class OuterClass {
        private String outerStr = "外部类的私有变量";
        //成员内部类
        private class InnerClass{
            private InnerClass(){
                System.out.println("外部类可以调用成员内部类的私有构造方法");
            }
            private String innerStr = "成员内部类的私有变量";
            private void test(){
                System.out.println("成员内部类可以直接调用"+outerStr);
            }
        }
        @Test
        public void test(){
            System.out.println("外部类可以通过实例对象调用"+new InnerClass().innerStr);
            new InnerClass().test();
            //方法内部类
            class InnerClass{
                private InnerClass(){
                    System.out.println("外部类可以调用方法内部类的私有构造方法");
                }
                private String innerStr = "方法内部类的私有变量";
                private void test(){
                    System.out.println("方法内部类可以直接调用"+outerStr);
                }
            }
            //注意要有先后顺序
            System.out.println("外部类可以通过实例对象调用"+new InnerClass().innerStr);
            new InnerClass().test();
        }
    }

```
　　`运行结果：`
```
    外部类可以调用成员内部类的私有构造方法
    外部类可以通过实例对象调用成员内部类的私有变量
    外部类可以调用成员内部类的私有构造方法
    成员内部类可以直接调用外部类的私有变量
    外部类可以调用方法内部类的私有构造方法
    外部类可以通过实例对象调用方法内部类的私有变量
    外部类可以调用方法内部类的私有构造方法
    方法内部类可以直接调用外部类的私有变量
```
　　言归正传，下面正式开始全面解析内部类。
# 为什么使用内部类

# 内部类与外部类的联系

## 内部类访问外部类

## 外部类访问内部类

# 内部类的分类

## 成员内部类

## 方法内部类

## 匿名内部类

## 静态内部类



