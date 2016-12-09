---
layout: post
title:  "图片上传时可能遇到的一些情况..."
date:   2016-12-09 12:01
categories: js image file
---

## component

    <input type="file" accept="image/png, image/jpeg, image/jpg"
        title=""
        onChange={this.handleReUpload} />

## 类型、大小（体积）

    event.target.files[0]: {
    	type, size
    }

此时也可以同时在express里对上传文件大小进行限制


## 图片转换成base64<!--more-->

    inputTypeFileChangeHandler = event => {
        let me = this;
        let reader = new FileReader();
        // onload() 当读取操作成功完成时调用
        reader.onload = function () {
            // reader.result 读取到的文件内容.这个属性只在读取操作完成之后才有效,并且数据的格式取决于读取操作是由哪个方法发起的
            me.setState({
                imageUrl: reader.result
            });
        }
        if (event.target.files[0]) {
            // readAsDataURL()开始读取指定的Blob对象或File对象中的内容
            reader.readAsDataURL(event.target.files[0]);
        }
    }

## 通过base64编码获取图片原始宽高

    let image = new Image();
    image.src = reader.result;
    let {width, height} = image;
    // 校验逻辑
    ...

## 关于上传input的onChange事件

连续上传同一张图片时，不会触发onChange事件，解决思路：每次上传完，先销毁组件，再重新render一个组件...

[阅读更多](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader)
