## github上传项目和blog
1.本地新建项目和blog
2.github新建仓库
3.本地init
	
	 git init
4.把所有文件add和commit
	
	git add .
	git commit -m "注释"
5.第一次的时候需要和远端仓库建立连接，以https为例（如果是ssh请看下面如何在github上添加ssh key）
	
	git remote add origin "你的远程仓库地址"
* 注意如果用https是没有问题的，但是如果使用ssh的话，需要在本地生成公钥和私钥，并且在远程配置公钥

6.推送到远程

	git push origin master
	
##### 在 github 上添加 SSH key
1.首先需要检查你电脑是否已经有 SSH key ，运行 git Bash 客户端，检查是否已经存在 id_rsa.pub 或 id_dsa.pub 文件
	
	$ cd ~/.ssh
	$ ls
2.创建一个 SSH key 

	$ ssh-keygen -t rsa -C "your_email@example.com"
* -t 指定密钥类型，默认是 rsa ，可以省略。
* -C 设置注释文字，比如邮箱。
* -f 指定密钥文件存储文件名。

3.把你的公钥添加到github上
