###### 安装前准备，首先安装node和git

1、 安装hexo

>$ npm install hexo-cli -g

>$ npm install hexo-deployer-git --save

2、新建文件夹，进入，并进行初始化

>$ hexo init

3、创建静态页面

>hexo generate （hexo g  也可以） 

4、本地启动：

>$ hexo server  

5、部署步骤：

>$ hexo clean

>$ hexo generate

>$ hexo deploy

5、部署步骤：

>$ hexo clean

>$ hexo generate

>$ hexo deploy

6、常用命令：

>$ hexo new "postName" #新建文章

>$ hexo new page "pageName" #新建页面

>$ hexo generate #生成静态页面至public目录

>$ hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）

>$ hexo deploy #将.deploy目录部署到GitHub

>$ hexo help  #查看帮助

>$ hexo version  #查看Hexo的版本

7、其他：

hexo分为站点配置文件和主题配置文件，
   主题的安装(以next为例)： 

   >$ cd your-hexo-site

   >$ git clone https://github.com/iissnan/hexo-theme-next themes/next

   在站点配置文件 _config.yml中，

   >theme: next

   使用调试命令验证主题：
   >hexo s --debug

8、部署到GitHub上

   看看本机是否存在SSH keys,打开Git Bash,并运行:
   >$ cd ~/. ssh

   如果没有，创建;否则忽略该步骤：
   >$ ssh-keygen -t rsa -C "your_email@example.com"

   拷贝你的SSh秘钥(id_rsa.pub)至GitHub的配置的SSh key中：
   >$ clip < ~/.ssh/id_rsa.pub

   配置用户信息：
   >$ git config --global user.name "username"//用户名

   >$ git config --global user.email "usernameg@xxx.com"//填写自己的邮箱

   测试是否连通：
   >$ ssh -T git@github.com

   下一步就是去hexo的站点配置文件，修改
   ![](https://user-gold-cdn.xitu.io/2018/1/11/160e4f2f37cb4b1f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)