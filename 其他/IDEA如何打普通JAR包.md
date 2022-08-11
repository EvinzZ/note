# IDEA如何打普通JAR包

## 正文

习惯了用maven命令打包，有点忘记了如何打一个普通的jar包了，特此记录一下。
 jar包分两种：一种是有main函数的可以直接执行的jar包，一种是没有main函数，不可以直接执行的jar包（通常是工具包）

## 普通JAR包（不可以直接执行的jar）

1 点击project structure 找到Artifacts 点击加号，选择jar --Empty

![img](../images/%E5%9B%BE%E7%89%87-10646566.png)

[图片.png](https://img.hacpai.com/file/2019/10/图片-10646566.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)

2 修改jar名字,并把右边的compile output拉到左边的jar里面 然后确定保存

![img](../images/%E5%9B%BE%E7%89%87-c4163b8e.png)

[图片.png](https://img.hacpai.com/file/2019/10/图片-c4163b8e.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)

3 点击build 选择build artifacts 进行build就可以了。

![img](../images/%E5%9B%BE%E7%89%87-62176d38.png)

[图片.png](https://img.hacpai.com/file/2019/10/图片-62176d38.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)

4 对应的jar就打包完成了。

![img](../images/%E5%9B%BE%E7%89%87-cfa413d6.png)

[图片.png](https://img.hacpai.com/file/2019/10/图片-cfa413d6.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)

## 可直接执行JAR包（有main函数）

重复上面1 2步操作
 3 然后点击create Mainfest 选择项目目录，直接确定即可

![img](../images/%E5%9B%BE%E7%89%87-8ef1e15a.png)

[图片.png](https://img.hacpai.com/file/2019/10/图片-8ef1e15a.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)

4 点击jar名称，然后设置对应的main函数位置。设置完毕点击确定即可

![img](../images/%E5%9B%BE%E7%89%87-53ab7f7f.png)

[图片.png](https://img.hacpai.com/file/2019/10/图片-53ab7f7f.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)

5 点击build，选择对应的artifacts 进行build就可以了。

6 测试是否成功，不报错，正确执行main里面的代码就成功了。

![img](../images/%E5%9B%BE%E7%89%87-73f2c137.png)

[图片.png](https://img.hacpai.com/file/2019/10/图片-73f2c137.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)