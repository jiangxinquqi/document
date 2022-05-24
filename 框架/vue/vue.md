# 如何搭建一个vue项目

## 一、环境准备

### 1、安装node.js

    下载地址：https://nodejs.org/zh-cn/

### 2、检查node.js版本

    node -v

### 3、使用淘宝的镜像源

    npm install -g cnpm --registry=https://registry.npm.taobao.org 
    
    以后再用到npm的地方直接用cnpm来代替就好了，因为没有镜像源的话，安装速度比较慢
    
    检查是否安装成功：cnpm -v

## 二、搭建vue环境

### 1、全局安装vue-cli

    这里注意：安装vue-cli对node.js的版本是有要求的
    
    装的两种方式：输入cmd命令
    npm install -g @vue/cli //这个是从国外下载的比较慢
    cnpm install -g @vue/cli //这个是从镜像源下载
    
    查看安装的版本（显示版本号说明安装成功）
    vue --version
    
    如果你原来有版本或者版本比较低，可以升级
    npm update -g @vue/cli
    yarn global upgrade --latest @vue/cli