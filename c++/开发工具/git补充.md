## 远程仓库

远程仓库的别名为origin

## 本地仓库与远程仓库交互协议

一般是选择ssh

更快

主要使用公钥和私钥

将客户端的ssh公钥拷贝服务端中，这样可以实现免密码登录

## git基础原理

### git的git的四个区域

![image-20240401000042806](../../picture/image-20240401000042806.png)

**Workspace**： 工作区，就是你平时存放项目代码的地方

**Index / Stage**： 暂存区，用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息

**Repository**： 仓库区（或版本库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本

**Remote**： 远程仓库，托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换



**这里有一个地方值得注意就是从Remote中拉取代码有两种方式**

**一种是fecth一种是pull**

**git pull相当于直接把代码拉取到工作区然后修改工作区的文件**

**git fetch则是把远程仓库的修改先拉取到本地仓库 的origin(远程仓库在本地的分支，由git维护)**

将远程仓库包含分支的最新commit-id到本地文件

然后再git merge把修改的内容拷贝到工作区

### git的工作流程

git的工作流程一般是这样的：

1、在工作目录中添加、修改文件；

2、将需要进行版本管理的文件add到暂存区域；

3、将暂存区域的文件commit到git仓库；

4、本地的修改push到远程仓库，如果失败则执行第5步

5、git pull将远程仓库的修改拉取到本地，如果有冲突需要修改冲突。回到第三步

因此，git管理的文件有三种状态：已修改（modified）,已暂存（staged）,已提交(committed)



### git 文件的四种状态

![image-20240401000328445](../../picture/image-20240401000328445.png)

Untracked:   未跟踪, 此文件在文件夹中，但并没有加入到git库，不参与版本控制， 通过git add 状态变为Staged。
Unmodify:   文件已经入库且未修改, 即版本库中的文件快照内容与文件夹中完全一致，这种类型的文件有两种去处，如果它被修改， 而变为Modified，如果使用git rm移出版本库, 则成为Untracked文件。
Modified：文件已修改，仅仅是修改，并没有进行其他的操作，这个文件也有两个去处，通过git add可进入暂存staged状态，使用git checkout 则丢弃修改，返回到unmodify状态, 这个git checkout即从库中取出文件，覆盖当前修改
Staged：暂存状态，执行git commit则将修改同步到库中，这时库中的文件和本地文件又变为一致，文件为Unmodify状态。





## 版本回退

### 尚未commit

git restore --staged  文件名

直接取消暂存区的修改，但是如果此时工作区的文件已经被修改，那么不会从暂存区中的文件来覆盖工作区的文件



git restore -s  文件名

工作区的文件未修改，发现了

### 已经commit，尚未push，如何回退

有时候，我们在本地的修改，可能提交到了本地

仓库，但是尚未push到远程仓库，针对这一种场景的

回滚是比较简单的，如下图所示：

![image-20240409222518441](../../picture/image-20240409222518441.png)

origin/master指向了5ff5433b这个commit，本地

有三个commit，现在想针对这三个commit作回滚，可

以使用git reset命令来做，reset参数如下意思：

  --soft – 缓存区和工作目录都不会被改变

​    --mixed – 默认选项。你指定的提交同步，但工作目录不受影响

​    --hard – 缓存区和工作目录缓存区和都同步到你指定的提交

  git reset --hard HEAD~{n}就是把HEAD指针回退n个版本（commit），并且使用该commit的内容覆盖掉

工作区的内容，即丢弃了前面n个commit的修改和当前工作区的修改。







git reset --soft //回退到暂存区

giut reset --mixed //回退到工作区

git rest --hard //直接删除这个提交，本地文件全部回退，仿佛没有编辑过或者新添加过这个文件一样

上面三个最后一个参数是head^代表回退一个版本

（这个不会删除文件，只会管理修改，要删除文件需要自己删除）

### 清空工作区的修改

git checkout 将工作区情况



### 如果已经提交到远程仓库

你只能先把远程仓库的代码拉取下来，回退并提交到本地仓库，再提交到远程仓库



## 本地仓库的整理操作

注意本地仓库的整理操作一定是分支的commit是自己提交的，别人不会依赖你，不要在master分支和dev分支进行这种操作

commit 加上. 代表把暂存区的内容全部提交

### 补充提交到上一次commit

因为忘记有代码修改了没提交，但是又不能再来一个commit

新文件还是先gid add

然后git commit --amend （复用你的上一次提交）

输入这个命令会使用vs code打开一个文件其中就有上一次提交的commit信息，你可以再其中添加新的信息然后保存关闭这个命令就执行成功





Git会新增加一个commit-id覆盖了上一次的commit-id, 这样漏掉的文件会合并到上一次的提交，然后我们也修改了

提交message的规范，大家可以通过git log –p去查看这次内容。当然我们除了添加“漏掉”的文件，也可以删除

“误修改”的文件。最后使用git push –fore强制推送修改后的commit。

### 修改前n次提交的commit信息

如果我们想要修改比较久远的commit message的格式不太符合规范，怎么去修改呢？比如我想修改从bed58d54e

之后所有的commit message的内容？

![image-20240409011137994](../../picture/image-20240409011137994.png)

```shell
$ git rebase –i bed58d54e # 打算从bed58d54e（不包含bed58d54e）之后所有的commit 的message

```

输入上面一条命令后，会弹出下文这个窗口，上半部分是一些pick命令，下半部分是一些提示内容，不过有意思的是此时的commit内容的排列和git log里的排列是反的，也就是倒序的。如果我们仅仅修改commit message，需要把打算修改的commit的**对应pick命令修改为rewor**d，然后保存。

![image-20240409011230311](../../picture/image-20240409011230311.png)

之后会弹出编辑commit message的内容，如果我们有多处commit需要被修改的话会多次弹出vim编辑窗口。

如果在修改前所有的commit都已经push到远程仓库的话，我们需要使用git push --force强制推送到远程仓库。

### 修改历史提交记录（补充提交的文件）

如果在某一个commit少提交了内容，这个内容又不能在当前commit进行提交应该如何做？

 ![image-20240402220848140](../../picture/image-20240402220848140.png)

从这里我们可以看到我们的当前提交超出了origin master两个，我们是不能修改origin master的，但是我们可以修改origin master之后的提交

git rebase -i（这样就相当于要最近的两次提交）

-i后面的参数表示不要合并的commit的hash值，代表这个commit以及在这之前的commit都不要合并

输入上面这个命令打开了一个文件

![image-20240402221423944](../../picture/image-20240402221423944.png)

将第一个pick修改为e

第二个pick修改为p

然后就进入了一个变基的状态

![image-20240402221521984](../../picture/image-20240402221521984.png)

在改状态只能使用上面的两个命令，一个是amend追加，一个是还原为正常的状态

现在对文件a进行修改，其中添加一行hello world

然后添加到暂存区git add a.txt

然后将这个修改，添加到刚刚文件设置为e对应的commit中去

git commit --amend

然后再弹出的文件中再额外添加一个message

![image-20240402221910734](../../picture/image-20240402221910734.png)

保存退出，然后输出

![image-20240402221948954](../../picture/image-20240402221948954.png)

然后我们现在还处于rebase的状态，我已经不想再修改了，就可以使用

git rebase --continue会回到master的状态

![image-20240402222810372](../../picture/image-20240402222810372.png)

然后你就发现你的倒数第二个提交被修改了，然后你也可以通过cat a.txt确切的看到有那些内容被修改了

### 合并提交（倒数第二和倒数第一合并）

**但是我们现在又想吧倒数第二个和倒数第一个合并为1个提交**

![image-20240402223147398](../../picture/image-20240402223147398.png)

这里有个参数s，就是将这个提交合并到上一个提交的参数

，如果我们想将倒数第一个合并到倒数第二个提交就是将b,txt前面的pick改为s

然后保存退出

它和前面有一点不同的是，如果使用s，保存退出后马上会进入下面这个文件

![image-20240402224516882](../../picture/image-20240402224516882.png)

修改两个提交记录为

![image-20240402224757529](../../picture/image-20240402224757529.png)

然后保存退出

再通过git log查看

![image-20240402224830350](../../picture/image-20240402224830350.png)

可以发现两个提交变成只有一个提交了，

**变基的时候如果再倒数第三个提交进行了额外的修改，那么倒数第二个提交，倒数第一个提交都会变化**

如果修改了commit那么commit的id也会发生变化

所以一般rebase只会出现在

我们有一个任务需要开发一个功能，然后从dev分支新建一个work分支，在work分支可以进行我们的变基操作

## 分支操作

创建一个新分支develop，提交了一个c文件

在master分支中又提交了一个d文件

如果直接在master分支合并develop分支

最后git log 不仅会有提交c的记录，而且git默认会把这次合并也作为一个commit进行一个提交

![image-20240402230511310](../../picture/image-20240402230511310.png)

**如果master对应一个远程分支**

**实际开发过程应该怎么做**？

那么我们需要在master分支上面合并develop分支之前，先git fetch拉取远程分支的最新代码

然后合并到本地master分支，合并成功后就在master分支合并本地的develop分支，有冲突就修改冲突并提交，然后将合并成功后的master分支推送到远程

**（上面这个有问题）**

实际应该是

![image-20240402231124013](../../picture/image-20240402231124013.png)

1.git fetch, git merge origin master

2.切换到develop分支，然后git merge master 

如果出现冲突先合并冲突并提交

> 冲突出现的原因：
>
> 同一个文件，相同行或者相近一两行
>
> 解决冲突原则：
>
> 因为我是拉取已经提交的远程代码，所以我们不能影响远程的代码，就移动我们自己的代码然后进行合并

冲突出现的样子

![image-20240402231843774](../../picture/image-20240402231843774.png)

=号线到HEAD这是我们本地develop的修改

=号线到master是本地master合并了远程master的修改是最新且已经提交的修改

最简单的处理就是将HEAD那行 ，==那行，master那行删除，然后保存退出

此时状态是一个(develop|MERAGING)的状态，我们需要将刚刚修改的文件进行一个add commit就解决了这个冲突

git commit的时候需要加上一个-i的操作，代表这是一个解决冲突的commit

如：

![image-20240402232059513](../../picture/image-20240402232059513.png)

git commit api.h -i -m "fix merge"

使用完上面的命令状态重新回到develop的状态



> 如果处于develop分支，想要合并master分支
>
> 也可以使用git rebase master
>
> 这是因为这种合并就不会新建支线
>
> 因为develop分支和master分支都有共同的祖先

3.测试原来的代码是否会出现问题



4.如果没有问题就直接git commit提交到develop分支



5.切换到master分支，然后git merge develop，git branch -d develop

再git push origin master



## 推送到远程分支

### commit命令

git commit –a –m “commit messeages”

添加的-a参数会把当前暂存区里所有的修改（包括删除操作）都提交，但是那些尚未添加到暂存区的内容是不会提交的，网上有很多的博客内容说-a参数会把尚未add的文件也提交了，这个说法是错误的。

git commit --amend

 这也是我们经常用的命令，他会把此次提交追加到上一次的commit内容里。

### 推送到远程分支

git push命令用于将本地分支的更新，推送到远程主机。

它的格式与git pull命令相仿。

git push <远程主机名> <本地分支名>:<远程分支名>

注意，分支推送顺序的写法是<来源地>:<目的地>，所以git pull是<远程分支>:<本地分支>，而git push是<本地分支>:<远程分支>。例如：

 git push origin master：refs/for/master

如果省略远程分支名，则表示将本地分支推送与之存在”追踪关系”的远程分支(通常两者同名)，如果该远程分支不存在，则会被新建。

git push origin master

上面命令表示，将本地的master分支推送到origin主机的master分支。如果后者不存在，则会被新建。



**如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支。**

> git push origin :master  # 等同于 git push origin --delete master
>
> 上面命令表示删除origin主机的master分支。



如果当前分支与远程分支之间存在追踪关系，则本地分支和远程分支都可以省略。

git push origin

上面命令表示，将当前分支推送到origin主机的对应分支。

如果当前分支只有一个追踪分支，那么主机名都可以省略。

git push

### 冲突

Git push的时候如果有冲突，会显示如下，此时我们必须去修复这个冲突。

![image-20240409010422866](../../picture/image-20240409010422866.png)



## 暂存修改

大家可能也遇到这么一种情况，你正在聚精会神地开发某个feature或者是在重构（FT-12345分支），但是突然线上冒出了一个

BUG，你是哪个牛逼的人，于是去解BUG，但是目前的修改怎么办才好呢？如果提交则会产生一个没有意义的commit。此时我们可以

使用git stash这个神器了。我保证你用了它之后会爱上它。

  比如我现在FT-12345的分支上做了修改，内容如下：

![image-20240409223627804](../../picture/image-20240409223627804.png)

  现在我要切到master分支上去修改BUG了，那在切换到master分支前我们可以暂存修改，即：

​      git stash  # 暂存修改

​      git stash pop # 从缓存里取出修改

调用git stash后就可以使用git checkout master分支上去修复bug了，修复完了之后再git checkout FT-12345后git stash pop



> ​	$ git stash  # 将工作区的修改保存到缓存区，默然取名为：
>
> ​      WIP on <branch_name> ： <latest_commit_id> <latest_commit_message>
>
> ​     $ git stash save <name>  # 将工作区的修改保存到缓存区，且取名为name
>
> ​     $ git stash pop  # 取出缓存区栈顶（即最近一次）的内容，并且会删除此次pop的内容
>
> ​     $ git stash list  # 查看缓存里所有存储的修改
>
> ​     $ git stash apply stash@{X} # 取出stash里的内容，X为序号，但是不会删除stash@{X}
>
> ​     $ git stash drop stash@{X} # 删除stash@{X}
>
> ​     $ git stash clear  # 删除缓存区里所有的记录