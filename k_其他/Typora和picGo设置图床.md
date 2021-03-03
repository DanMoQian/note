# Typora和picGo设置图床

## 0. typora主题

1. 下载主题

> https://github.com/blinkfox/typora-vue-theme

2. 下载本主题中的`vue.css`、`vue-dark.css`文件和包含字体的`vue`文件夹；
3. 打开 Typora，点击“**偏好设置**” => “**打开主题文件夹**”按钮，将弹出 Typora 的主题文件夹；
4. 将下载好的`vue.css`和`vue-dark.css`文件和包含字体的`vue`文件夹放到 Typora 的主题文件夹中；
5. 关闭并重新打开 Typora，从菜单栏中选择 “**主题**” => “**Vue**” 或者 “**Vue Dark**” 即可。

## 主题修改

Vue主题默认的代码字体太丑，而且伤眼睛，修改字体

参考链接：https://www.jianshu.com/p/5634b207d516

1. 打开 Typora，点击“**偏好设置**” => “**打开主题文件夹**”按钮，将弹出 Typora 的主题文件夹；
2. 修改`vue.css`，如图添加红色框中的`'Microsoft YaHei' `微软雅黑

![image-20210301163233314](https://gitee.com/danmoqi/pictureBed/raw/master/img/image-20210301163233314.png)

## 1.  环境

typora+picGo+nodejs

参考链接：https://zhuanlan.zhihu.com/p/102594554

## 2. 下载picGo

1. 打开 Typora，点击“**偏好设置**” => “**图像**”=> "**下载picgo**"按钮或者去github下载

## 3. 安装之后打开主界面

![img](https://gitee.com/danmoqi/pictureBed/raw/master/img/v2-83da997e41df3d5a366492075e84e8c6_720w.jpg)



### 选择最底下的插件设置，搜索**gitee**



![img](https://gitee.com/danmoqi/pictureBed/raw/master/img/v2-e571ed50235f2f82661b5b1315419bd5_720w.jpg)



### 点击右边的gitee-uploader 1.1.2开始安装

> 这里注意一下，必须要先安装[node.js](https://link.zhihu.com/?target=https%3A//nodejs.org/en/)才能安装插件，没装的自己装一下，然后重启就行。

这个地方有两个插件，我试了一遍，两个都能用，大家看心情选择，先说一下右边这个**gitee-uploader 1.1.2**，用不了的同学就选左边那个，我都会讲一遍配置

------

## 2. 建立gitee（码云）图床库

注册码云的方法很简单，网站引导都是中文，不多说了，我们直接建立自己的图床库。

### 点击右上角的+号，新建仓库



![img](https://gitee.com/danmoqi/pictureBed/raw/master/img/v2-44a4581b8e0ac9a0bc6747ee9b507a0e_720w.jpg)



新建仓库的要点如下：

1. 输入一个仓库名称
2. 其次将仓库设为公开
3. 勾选使用Readme文件初始化这个仓库

**这个选项勾上，这样码云会自动给你的仓库建立master分支，这点很重要!!!** 我因为这点折腾了很久，因为使用github做图床picgo好像会自动帮你生成master分支，而picgo里的gitee插件不会帮你自动生成分支。



![img](https://gitee.com/danmoqi/pictureBed/raw/master/img/v2-11790828fe9ce436ea6d92fbe1c0662f_720w.jpg)



点击创建进入下一步

------

## 3. 配置PicGo

安装了**gitee-uploader 1.1.2**插件之后，我们开始配置插件

### 配置插件的要点如下：

![img](https://gitee.com/danmoqi/pictureBed/raw/master/img/v2-7fd17e45105b65c13c9c7e260a5b6d87_720w.jpg)

- repo：用户名/仓库名称，比如我自己的仓库leonG7/blogImage，找不到的可以直接复制仓库的url

>  ![img](https://pic3.zhimg.com/80/v2-c0bc93a55fc118fb9731371af0c8a702_720w.png)

- branch：分支，这里写上master
- token：填入码云的私人令牌
- path：路径，一般写上img
- customPath：提交消息，这一项和下一项customURL都不用填。在提交到码云后，会显示提交消息，插件默认提交的是 `Upload 图片名 by picGo - 时间`

### 这个token怎么获取，下面登录进自己的码云

1. 点击头像，进入设置



![img](https://gitee.com/danmoqi/pictureBed/raw/master/img/v2-09207edcefff7852c91abcc3df3c5ba0_720w.png)



1. 找到右边安全设置里面的私人令牌



![img](https://gitee.com/danmoqi/pictureBed/raw/master/img/v2-bc612193330f4235b8c887dc95a77f68_720w.jpg)



1. 点击`生成新令牌`，把**projects**这一项勾上，其他的不用勾，然后提交

![img](https://gitee.com/danmoqi/pictureBed/raw/master/img/v2-9d8ba0adb98d509d3de8c1b615c68353_720w.jpg)



这里需要验证一下密码，验证密码之后会出来一串数字，这一串数字就是你的token，将这串数字复制到刚才的配置里面去。

![img](https://gitee.com/danmoqi/pictureBed/raw/master/img/v2-7d9e998d82ae965fe7f1ddd93d59d48a_720w.jpg)



> 注意：这个令牌只会明文显示一次，建议在配置插件的时候再来生成令牌，直接复制进去，搞丢了又要重新生成一个。

### 现在保存你刚才的配置，然后将它设置为默认图床，大功告成。

还有一个插件**gitee 1.2.2-beta**，功能差不多，`刚才那个能用的话就不需要用这个`，配置的内容有点差别，简单说一下：



![img](https://gitee.com/danmoqi/pictureBed/raw/master/img/v2-499cfcbda9cbd6c4f5c8e23f30e382cc_720w.jpg)



- url：图床网站，这里写码云的主页 [https://gitee.com](https://link.zhihu.com/?target=https%3A//gitee.com)
- owner：所有者，写上你的码云账号名，如果你不知道你的账号名，进入你刚才的仓库，浏览器url里面有



![img](https://gitee.com/danmoqi/pictureBed/raw/master/img/v2-f782ba2cb965931f00798322d42d3e54_720w.png)



- repo：仓库名称，只要写上仓库名称就行，比如我自己的仓库blogImage
- path：写上路径，一般是img，**这几个项都不用加“ / “符号**
- token：刚才你获取的个人令牌，两个插件是通用的，如果你用了另一个再来用这个，它会自动读取另一个插件的部分配置，不用重新申请
- message：不用填

------

## 4. 测试

随便选一张图片上传（picgo也支持剪贴板上传，截图工具推荐win10的*Snipaste*神器！），试试看

### 超级快有木有！比github快很多，0.1秒上传，而且导入到你的markdown编辑器里面也是秒识别你的图片内容，而如果是github图床上传太慢不说可能还会出现下面这样识别不出来的问题！



![img](https://gitee.com/danmoqi/pictureBed/raw/master/img/v2-7d18d14d530df892f48202f683947fcc_720w.jpg)



### 上传之后默认复制链接，直接粘贴到你的markdown编辑器里，就可以愉快的进行写作了！

最后推荐一下我的博客写作套件**Typora + PicGo + Snipaste**，Typora写文档，Snipaste一键截图，PicGo一键上传图片返回链接。