# Git神器| SourceTree安装使用教程

SourceTree 是 Windows 和Mac OS X 下免费的Git客户端管理工具。支持创建、克隆、提交、push、pull 和合并等操作。

一、sourcetree的安装

\1. 下载sourcetree

下载链接：[Sourcetree | Free Git GUI for ](https://link.zhihu.com/?target=https%3A//www.sourcetreeapp.com/)Mac and Windows

\2. 安装sourcetree

点击安装，第一个创建Bitbucket账户可跳过初始设置，加载SSH密钥，选择否（后面使用git生成）。

二、sourcetree的git配置

\1. 下载并安装git

下载链接：[Git](https://link.zhihu.com/?target=https%3A//git-scm.com/)，选择安装路径后进行安装。

\2. 生成秘钥。打开Git，执行命令：ssh-keygen -t rsa，默认安装路径在"C:\Users\Administrator\.ssh”目录下，一直按回车，不用输入密码。

![img](https://pic3.zhimg.com/80/v2-8f51072e8aa43f0c6e14f99608329dd6_720w.webp)

你可以在C:\Users\Administrator\.ssh”目录下查看你生成的秘钥。

![img](https://pic1.zhimg.com/80/v2-231f9100d66a6d8560b26f55166779a0_720w.webp)

\3. 然后在gitlab上绑定自己刚刚生成的公钥

在settings->SSH Keys->Key->Add yey

![img](https://pic3.zhimg.com/80/v2-4d7b3c150b8b2eba59530d3cb92d7c8a_720w.webp)

其中，Key框内把刚刚生成的id_rsa.pub中的内容复制进来，然后点击Add key。

三、sourcetree拉取代码

\1. 打开sourcetree，点击"工具-->选项-->一般"，选择SSH密钥的位置为刚刚生成密钥的路径，SSH客户端选择OpenSSH，点击确定。

![img](https://pic2.zhimg.com/80/v2-464198d81a0c0bf4dc07cd92bf94d3e5_720w.webp)

\2. 点击"文件-->克隆/新建"，打开克隆tab，输入信息后，点击克隆就可以将代码拉到本地。（如果是已有git项目，选择Add，浏览项目指定文件夹，然后添加；如果不是git项目，先选择Create，浏览项目指定文件夹，然后添加）

![img](https://pic1.zhimg.com/80/v2-f54b59dcd7728705d8aa2f171127d504_720w.webp)

源路径：拉取项目的git路径

目标路径：要保存该项目的本地路径

名字：一般会根据目标路径自动获取填充