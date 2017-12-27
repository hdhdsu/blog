## Git 简介
1.建立本地仓库
*  新建本地仓库

        git init

  此时当前文件夹就是一个新的仓库。该目录下会多出一个被隐藏的.git文件夹，里面的文件是用来跟踪管理版  本库的，一般情况下不建议手动修改。

*  从远端仓库检出

        git clone [远端仓库地址]

该命令执行成功后，远端仓库会被检出到本地。

2.本地仓库
本地仓库我们可以理解它有三个区域：工作区，暂存区，版本库。后两者不可见，存在于被隐藏的文件夹.git中。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-8e7388e36a60fe98.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*  工作区
可见文件的区域，我们对文件的操作都是针对工作区的。
当对工作区的文件进行变更，这时候，这些变更没有被提交（暂存区或版本库），用git status
命令可以看到以下状态，工作区是脏的。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-765d9752abe012e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


*  暂存区
顾名思义，将变更提交的暂存区，提交到版本库的过渡。

        git add [file]

add命令可以将工作区的变更添加到暂存区。（git add .可以将所有变更添加到暂存区） 此时再用git status
命令可以看到以下状态。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-7a8b5fc28f4a79f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


*  版本库
对已经提交的变更进行版本管理。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-c9841e41b7796f75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

每个版本节点中存有当前提交的一些文件信息，如果文件后续变更后，还需要之前的文件信息，则可以从版本库中检出之前的文件信息。commit命令可以将暂存区的变更保存至版本库，形成一个新的版本节点。

        git commit -m '[变更的描述]'

同样的，用git status查看状态，工作区是干净的，没有需要提交的内容。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-de996ff035d687a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*  HEAD
到这里我们可以看到，版本库里面有许多的版本节点，那么我们的工作区到底显示的是哪个版本的文件，是在对哪个版本下的文件进行变更呢？HEAD就起到这样一个作用，类似于当前活跃版本节点的指针：HEAD指向的位置，就是我们当前工作区所在的版本节点，包括后面会提到的分支切换，其实也是HEAD指针指向的版本节点的切换。比如，我们用reset指令将工作区的内容切换到之前的版本（描述为'UrlUtil modified'）：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-90399e7a5a2f7e08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到提示**HEAD is now at c0f2e25 UrlUtil modified**。

3. 远端仓库
远端仓库是这样的一个角色，所有人都可以从远端仓库托管的文件克隆一份到自己的电脑上，并且各自把各自的提交推送到远端仓库里，也从远端仓库中拉取别人的提交。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-88eacb8cc7048646.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对远端仓库我们有两个基本的操作，这两个操作都是在本地版本库和远端版本库间的操作。
*  推送本地版本至远端

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-ca59b541586b7a27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可见上图中，本地版本库有一个新的提交版本没有推送到远端，这时候远端没有我们本地最新版本的信息，需要执行以下命令将本地的最新版本信息推送到远端。

        git push

执行之后，就把本地的最新版本推送到远端了。

*  注意：如果是自己创建的代码仓库并且是第一次提交，那么需要添加远程的代码库到配置：

        git remote add origin https://github.com/zhong635725959/droplook.git

将远端版本拉到本地仓库类似的，如果别人对把他们的变更提交到远端，这时候我们本地版本库就比远端落后几个版本，我们可以通过pull命令将远端的版本拉到本地。

        git pull

##### 文件冲突
如果我们想要把本地的变更push到远端的时候，远端版本库被别人提交过了，这时候就会出现提交失败的情况，就像下图的情况。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-c48339d5adc85dc3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时候我们需要3步：
1.将远端最新的版本拉到本地

        git pull

这时候，如果远端的变更跟本地变更没有冲突，则会快速合并，即可以跳过下一步，直接进行第3步。若有冲突，则会提示以下信息：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-70f9f8b77fc837d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.合并远端仓库和本地的变更首先需要手动解决冲突的文件。git status查看状态，打开both modified
路径下的文件，会出现如下提示信息：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-02e0b2674b00a436.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

手动将冲突的文件修改后，再执行以下步骤提交。

        git add .
        git commit -m '[提交的描述（合并远端冲突）]'

3.提交本地变更

        git push

到执行了这条push之后，我们才算把之前想要提交到远端的代码push到了远端仓库。

#### 分支&分支管理
分支可以帮助我们更好地**互不干扰地**进行协同开发。

当我们新建一个git仓库时，默认是在master分支上。

*  根据当前版本号上的内容 新建一个分支

        # 新建一个分支，并且切换到该分支上(HEAD指向新的分支)
        git checkout -b [分支名]
        # 新建一个分支，不切换git branch [分支名]

*  分支切换
        git checkout [分支名]

*  分支合并
        # 将其他分支合并到当前分支
        git merge [其他分支名]

merge过程中也可能会遇到有冲突的情况，需要我们手动解决冲突，类似于上面提到的**文件冲突**。

#### git回退
1.使用[Git](http://lib.csdn.net/base/git) log命令查看所有的历史版本，获取某个历史版本的id，假设查到历史版本的id是

    139dcfaa558e3276b30b6b2e5cbbb9c00bbdca96。

2.回退

    git reset --hard 139dcfaa558e3276b30b6b2e5cbbb9c00bbdca96 

3.推送到远程仓库

    git push -f -u origin master  
#### git 撤销add
1.先 git status 查看状态
2.git reset HEAD 文件

#### Git Flow分支管理模型
*  **master**随时可发布的代码，这里的代码都是测试完成的，已上线的；在master打tag，所有上线了的版本的tag都打在这里；
*  **develop**永远是功能最新最全的分支，所有开发完成的功能都合并到该分支上；这个分支上的代码不一定都是测试完成的，大部分都是待测试的；
*  hotfix修复线上代码的 bug，它派生于master分支，修改完成并且测试完成后，合并到master和develop上；
*  feature某个功能点正在开发阶段的代码，它派生于develop分支，开发完成后，合并到develop上等待测试；
*  release要上线的功能，它派生于develop分支，测试完成可以上线后，分别合并到master和develop上；派生出某个release后，develop分支可以接着开发别的功能而不影响到上个release；

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-d86f0e380b854419.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)