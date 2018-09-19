##markdown和github下载使用##
###markdown使用###
- [下载markdownd地址](http://www.markdownpad.com/)
-  #一级标题，##二级标题，###三级标题，如此类推 
-  [] ()   显示创建链接，[]是显示名字，()链接地址
-  >      显示引用  
-  -或*   显示列表
-  ** **   显示加粗
-  ![] ()   显示图片，[]是显示名字，()链接地址
-  < pre >   < /pre>  添加代码
-  ***    显示一条横线
-  * *  我变斜了




##github使用
- 注册,用户名vip20180818 密码：wang831002@
- [git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
- **安装教程**
  * **linux安装**  
  <pre>
   yum install git
  </pre>

  * **windows安装**
    > 在Windows上使用Git，可以从Git官网直接[下载安装程序](https://git-scm.com/downloads)，（网速慢的同学请移步国内镜像），然后按默认选项安装即可。
  * ###**配置Git**
   * 首先在本地创建ssh key；
   <pre>
    $ ssh-keygen -t rsa -C "your_email@youremail.com"   
   </pre>
  后面的your_email@youremail.com改为你在github上注册的邮箱，之后会要求确认路径和输入密码，我们这使用默认的一路回车就行。成功的话会在~/下生成.ssh文件夹，进去，打开id_rsa.pub，复制里面的key。回到github上，进入 Account Settings（账户配置），左边选择SSH Keys，Add SSH Key,title随便填，粘贴在你电脑上生成的key。

  ![](http://www.runoob.com/wp-content/uploads/2014/05/github-account.jpg)

  为了验证是否成功，在git bash下输入：
  <pre>
   $ ssh -T git@github.com
  </pre>

   如果是第一次的会提示是否continue，输入yes就会看到：You've successfully authenticated, but GitHub does not provide shell access 。这就表示已成功连上github。
   
   * 接下来我们要做的就是把本地仓库传到github上去，在此之前还需要设置username和email，因为github每次commit都会记录他们。
   <pre>
    $ git config --global user.name "your name"
    $ git config --global user.email "your_email@youremail.com"
   </pre>  
   * 进入要上传的仓库，右键git bash，添加远程地址：
   <pre>
    $ git remote add origin git@github.com:yourName/yourRepo.git
   </pre>
   后面的yourName和yourRepo表示你再github的用户名和刚才新建的仓库，加完之后进入.git，打开config，这里会多出一个remote "origin"内容，这就是刚才添加的远程地址，也可以直接修改config来配置远程地址。创建新文件夹，打开，然后执行 git init 以创建新的 git 仓库。

###检查仓库
  * 执行如下命令以创建一个本地仓库的克隆版本：
  <pre>
   $ git clone /path/to/repository 
  </pre>
  * 如果是远端服务器上的仓库，你的命令会是这个样子：
  <pre>
   $ git clone username@host:/path/to/repository
  </pre>
###工作流
你的本地仓库由 git 维护的三棵"树"组成。第一个是你的 工作目录，它持有实际文件；第二个是 暂存区（Index），它像个缓存区域，临时保存你的改动；最后是 HEAD，它指向你最后一次提交的结果。

 * 你可以提出更改（把它们添加到暂存区），使用如下命令：
 <pre>
  $ git add <filename>
  $ git add *
 </pre>
 * 这是 git 基本工作流程的第一步；使用如下命令以实际提交改动：<>  <pre>
  $ git commit -m "代码提交信息"
 </pre>	
 现在，你的改动已经提交到了 HEAD，但是还没到你的远端仓库。
 * 你的改动现在已经在本地仓库的 HEAD 中了。执行如下命令以将这些改动提交到远端仓库：
 <pre>
  $ git push origin master
 </pre>
 可以把 master 换成你想要推送的任何分支。
 如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：
 <pre>
  $ git remote add origin (server)
 </pre>
 如此你就能够将你的改动推送到所添加的服务器上去了

###分支###

* 查看分支：
<pre>
$ git branch
</pre>
* 创建分支：
<pre>
$ git branch (name)
</pre>
* 切换分支：
<pre>
$ git checkout (name)
</pre>
* 创建+切换分支：
<pre>
$ git checkout -b (name)
</pre>
* 合并某分支到当前分支：
<pre>
$ git merge (name)
</pre>
* 删除分支：
<pre>
$ git branch -d (name)
</pre>

####**分支使用具体流程：**
* 首先，我们创建dev分支，然后切换到dev分支：
<pre>
git checkout -b dev
Switched to a new branch 'dev'
</pre>
* git checkout命令加上-b参数表示创建并切换，相当于以下两条命令：
<pre>
$ git branch dev
$ git checkout dev
Switched to branch 'dev'
</pre>
* 然后，用git branch命令查看当前分支：
<pre>
$ git branch
* dev
  master
</pre>
* git branch命令会列出所有分支，当前分支前面会标一个*号。然后，我们就可以在dev分支上正常提交，比如对readme.txt做个修改，加上一行,然后提交：
<pre>
$ git add readme.txt 
$ git commit -m "branch test"
[dev b17d20e] branch test
 1 file changed, 1 insertion(+)
</pre>
* 现在，dev分支的工作完成，我们就可以切换回master分支：
<pre>
$ git checkout master
Switched to branch 'master'
</pre>
* 切换回master分支后，再查看一个readme.txt文件，刚才添加的内容不见了！因为那个提交是在dev分支上，而master分支此刻的提交点并没有变：
* 现在，我们把dev分支的工作成果合并到master分支上：
<pre>
$ git merge dev
 Updating d46f35e..b17d20e
 Fast-forward
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
</pre>
* 合并完成后，就可以放心地删除dev分支了：
<pre>
$ git branch -d dev
 Deleted branch dev (was b17d20e).
</pre>
* 删除后，查看branch，就只剩下master分支了：
<pre>
$ git branch
* master
</pre>

###版本回退###
1 git log命令显示从最近到最远的提交日志，如果嫌输出信息太多，看得眼花缭乱的，可以试试加上--pretty=oneline参数
 <pre>
 $ git reflog
bb3f800 (HEAD -> master) HEAD@{0}: reset: moving to bb3f800
6576055 HEAD@{1}: reset: moving to HEAD^
bb3f800 (HEAD -> master) HEAD@{2}: reset: moving to HEAD^
bbbe235 (origin/master) HEAD@{3}: commit: 1
bb3f800 (HEAD -> master) HEAD@{4}: commit: dd
6576055 HEAD@{5}: commit (initial): cc
 </pre>
2 首先，Git必须知道当前版本是哪个版本，在Git中，用HEAD表示当前版本，也就是最新的提交1094adb...（注意我的提交ID和你的肯定不一样），上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100。
<pre>
$ git reset --hard HEAD^
HEAD is now at e475afc add distributed
</pre>
3 现在，你回退到了某个版本，关掉了电脑，第二天早上就后悔了，想恢复到新版本怎么办？找不到新版本的commit id怎么办？在Git中，总是有后悔药可以吃的。当你用$ git reset --hard HEAD^回退到add distributed版本时，再想恢复到append GPL，就必须找到append GPL的commit id。Git提供了一个命令git reflog用来记录你的每一次命令：
<pre>
git reflog
e475afc HEAD@{1}: reset: moving to HEAD^
1094adb (HEAD -> master) HEAD@{2}: commit: append GPL
e475afc HEAD@{3}: commit: add distributed
eaadf4e HEAD@{4}: commit (initial): wrote a readme file

git reset --hard 1094adb
</pre>