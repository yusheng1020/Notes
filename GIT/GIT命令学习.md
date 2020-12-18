### GIT命令学习

- #### GIT初始化配置

`git config --global user.name "xx"`    配置自己的git名称

`git config --global user.email xxx`	//配置自己的邮箱

`git config --global`

`git config --system`

`git config `

global指的是登录这台计算机的用户

system指的是当前的系统，不管是什么用户

不填指的是当前文件

优先级大小为：不填>global>system

- #### GIT区域

  工作区---本地代码（沙箱环境）

  暂存区---改完一个确定的文件就放到暂存区

  - `git ls-files -s` 查看缓存区当前的样子

​	版本库---最终一下从暂存区中提交到版本库(.git)

- #### .git文件目录

  hooks--钩子，是在进行add和commit之前或之后执行的脚本，包含客户端或服务端的钩子脚本

  info--包含一个全局性排除文件，那些文件git无需进行管理

  logs---日志文件

  ***objects***--目录存储的是所有的数据内容（数据库）

  - 各文件目录的命名加上内部文件的命名组成的xxx（是一个hash-code）在命令`git cat-file -p xxx`下会还原成原内容（commit信息和保存的数据内容）-t是查看数据类型
  - git的工作无非就是使用`echo 'xxx' | git hash-object -w --stdin`将内容xxx转换成hash-code以`key:value`的形式存入文件objects文件夹中。
  - `git hash-object 文件路径` 返回对应文件内容的键值（hash-code）
  - `git hash-object -w 文件路径`   将指定文件路径中的内容转换成hash-code，并使用git对象存入文件Objects文件夹中，其存储的内容是经过压缩的，但其存储的不是增量（版本之间存储的不是增量，而是当前版本的***最新所有git对象***）

  ***refs***--目录存储指向数据（分支）的提交对象的指针

  config---项目级别的配置文件

​	   description--用来

​	***HEAD***--文件指示目前被检出的分支

​	***index***---文件保存暂存区的信息

- #### LINUX基本操作

  `echo 'xxx' > xx` 	创建文件xx并写入内容xxx

  `clear`  清屏

  `ll` 	文件属性

  - 例：drwxr-xr-x 1 lenovo 197121          0 Jan  7  2019  AppData/

   - 第一个字母表示文件类型
     	- '-'，表示普通文件
        	- 'd'，表示目录文件
   - 紧接着是3*3个字符分组，对于owner,group,others而言，以-rwx（owner）r-x（Group）r-x(Others)为例
     - 普通文件，使用者自己可读，写，执行
     - 同一组的用户可读，不可写，可执行
     - 其他用户可读，不可写，可执行

  - 第二个栏位-文件个数，若为目录文件，则为目录下文件的个数
  - 第三个栏位-该文件/目录的拥有者
  - 第四个栏位-所属的组，每一个使用者都可以拥有一个以上的组，不过大部分的使用者应该都只属于一个组，只有当系统管理员希望给予某使用者特殊权限时，才可能会给他另一个组
  - 第五栏位，表示文件大小。文件大小用byte来表示，而空目录一般都是1024byte，当然可以用其它参数使文件显示的单位不同，如使用ls –k就是用kb莱显示一个文件的大小单位，不过一般我们还是以byte为主。
  - 第六个栏位，表示最后一次修改时间。以“月，日，时间”的格式表示，如Aug 15 5:46表示8月15日早上5:46分。
  - 第七个栏位，表示文件名。我们可以用ls –a显示隐藏的文件名。

  `find xxx` 寻找文件xxx，并打印出文件下的所有文件目录以及文件

  `find xxx -type f`  寻找文件xxx，并打印出文件下的所有文件，不打印目录

  `rm xxx`   删除文件名xxx

  `mv xxx xxxx`  将文件xxx的名称改为xxxx

  `cat`  查看文件内容

  `vim` 使用vim编辑器编辑文件  `:set nu`显示行号

  

- #### GIT对象

  - Git是一个内容寻址文件系统，这意味着Git核心部分是一个简单的键值对数据库，可以向Git仓库中插入任意类型的内容，它会返回一个唯一的键，通过该键可以再任意时刻再次取回内容。
  - `key:value`组成的键值对，key是hash-code，value是内容，这个键值对在Git内部是***数据对象（blob object）***
  - 所以只存单个文件数据内容，而不去记录其文件名
  - git对象（也就是数据对象）表示的是某单个文件的一次次版本
  - 以a.txt（v1_1）文件为例，将内容v1_1转换成hash-code（40个字符，是一个SHA-1哈希值），hash_code_2（hash-code的前两位作为文件目录名称），hash_code_2~（hash-code的非前两位（38位）作为hash-code-2文件夹下的唯一文件的文件名称，文件中存储的是v1_1的压缩版本）

- #### 树对象

  - 记住文件的每一个版本所对应的hash-code并不现实

  - 在Git中，文件名并没有被保存-----我们只保存了文件的内容

  - 一个git对象只能存储一个文件，项目有多个文件

  - 一个树对象包含多个git对象或子树对象的hash-code指针，以及相应的模式，类型，文件信息

  - 树对象表示的是项目的一次次版本

  - `git update-index --add --cacheinfo x xx xxx`  利用update-index命令为xxx文件（xx是文件内容的hash-code,xxx为文件的名称）的首个版本创建一个暂存区。并通过`git write-tree`命令生成树对象（暂存区的快照）将其写入Objects文件夹中，x表示文件模式（100644-普通文件，100755-可执行文件，120000-一个符号链接），--add 此前文件xxx不在暂存区，需要add（***若只是简单的修改已有的文件，则不需要执行--add，所有涉及到add的命令同理***），--cacheinfo，需要添加的文件xxx在Git数据库objects中，而不是位于当前目录下，需要查询出来

  - 拓展：

    - <img src="https://raw.githubusercontent.com/yusheng1020/PicGo_MyPicBed/main/gitLearning/pic_1.png" style="zoom:70%;" />

    - 树对象也可以包含其他的树对象（作为其子树对象）

- #### commit对象（提交对象）

  - ***commit描述信息+commit作者信息+当前tree对象的hash-code+上一个tree对象的commit(parent)的hash-code(为初始版本则无parent)***

  - 底层命令为`echo 'x' | git commit-tree xx -p xxx` 其中x是我们的描述内容，xx是某一个树对象的hash-code，xxx是某一个commit对象的hash-code,最终返回一个commit对象的hash-code。（一般来说，某一个指的是当前最新的对应的对象）
  - 所以commit对象将所有的版本都串了起来，是一种链式结构

- #### 三种对象的总结

  <img src="https://raw.githubusercontent.com/yusheng1020/PicGo_MyPicBed/main/gitLearning/pic_2.png" style="zoom:70%;" />

  - 一个commit对象是对树对象的一个封装，就是一个项目的版本，一个树对象是一个项目版本的快照（包含项目所有的git对象），一个git对象对应的是文件的快照（而不是相对增量）
  - 一次完整的流程必定包含三种对象，并且，一次提交必定只会有一个commit对象和一个tree对象

  

- #### git命令

  - `git init`       //初始化仓库

    - 工作目录中的所有文件不外乎两种状态-----**已跟踪**和**未跟踪**
    - **已跟踪**：指本来就被纳入版本控制管理的文件，在上次快照中有它们的记录，工作一段时间后，他们的状态可能是***已提交***，***已修改***，***已暂存***。
    - **未跟踪(untracked files---new files(新的文件))**：其他非以跟踪文件，没有上次更新的快照，也不在当前的暂存区域。`git add xxx`***将文件xxx(或文件目录xxx下的所有文件)由未跟踪变成已跟踪状态中的已暂存状态***
    - 初次克隆某个仓库的时候，工作目录中的所有文件都是**已跟踪**的文件，且状态未***已提交***，在编辑过某些文件之后，Git将这些文件标为***已修改(changes not staged for commit---modified(修改)---deleted(删除))***，再逐步把这些修改过的文件放到暂存区域，标记为***已暂存(changes to be commited---new files(新加的文件)--modified(修改)--deleted(删除))***，直到最后一次性提交所有这些暂存起来的文件，变成***已提交***

  - `git status`     //查看文件状态

  - `git diff`         //当前做了那些更新（***不包含未跟踪的文件修改***）还没暂存？（**所有已修改相对于最新的版本库**）

  - `git diff --cached/staged`    //有哪些更新是暂存起来准备好了下次提交的?（**当前暂存区相对于最新的版本库**）

  - `git add xxx`   //xxx为上传的文件的名称，从工作区上传到暂存区

    - 当对源项目已有的文件进行修改/添加新的文件的时候，才会进行具体的操作，因此在执行add操作前，会对源文件进行检查，看是否进行了修改/添加。

    - 在这个操作中实际上是进行了两个动作
    - `git hash-object -w xxx`  将xxx文件内容压缩并转换成hash-code再生成一个***git对象***存在objects中
    - 再通过`git update-index --add --chcheinfo x xx xxx` 从数据库拿出hash-code对应于xx的git对象存储在当前的暂存区中（改变原文件xxx中的内容/新添加一个文件xxx时才会执行，因为暂存区在commit后是不会清空的）
    - 这两个命令也可以使用一个合并命令`git update-index --add xxx` 来执行，其中xxx是文件名
    - `git add ./`将当前工作区中的所有修改了的文件进行上传

  - git commit -m "x"     //x是对于当前上传的说明，并从暂存区上传到本地仓库（git commit启动文本编辑器以便输入本次提交的说明，第一行）
    - 对当前的暂存区拍一张快照，使用`git write-tree`命令生成一个***树对象***（内容包含最新的***所有***git对象），将其存入objects数据库中
    - 同时执行`echo 'x' | git commit-tree xx -p xxx`将生成一个***commit对象***存入数据库中
  - `git commit -a -m "xxx"`加一个`-a`操作直接跳过add操作，跳过使用暂存区域（将所有已跟踪的文件暂存起来一并提交）
  - `rm xxx`  在工作区删除文件xxx
  - `git rm x` 删除暂存区中的文件x（相当于执行了`rm x`，`git add x`）
  - `git mv x xx` 将暂存区中的文件x改名为xx（相当于执行了`mv x xx`，`git rm x`，`git add xx`）
  - git remote add origin https://github.com/2293755237/xxx.git    //xxx表示的是仓库的名称
  - git push -u origin master                //将文件上传到github上
  - `git log`       查看提交日志   `q`退出日志
  - `git log --pretty=oneline`  一行显示日志（commit对象）
  - `git log --oneline` 简洁版一行显示日志
  - `git reflog`       查看以往的所有操作（包括版本回退以及提交日志），前面的序号是当前版本的唯一标识号
  - git reset --hard HEAD~2  //版本回退到前两个版本、
  - git reset --hard xxx  //版本回到标识号为xxx版本
  - git reset --hard -- xxx   //找回之前删除的xxx文件
  - git remote rm origin  //删除远程Git仓库
  - git remote -v      //查看远程仓库地址
  - git checkout xxx    //本地删错了，将文件从暂存区中还原回来

- #### Git分支操作

  - 在很多版本控制系统中，常常需要完全创建一个源代码目录的副本，对于大项目来说，这是非常浪费时间的

  - Git处理分支是非常轻量的，创建分支几乎在瞬间完成，并且在不同分支之间的切换操作十分便捷

  - ![](https://raw.githubusercontent.com/yusheng1020/PicGo_MyPicBed/main/gitLearning/pic_3.png)

  - 分支的本质就是一个提交对象

  - HEAD是一个指针，它默认指向master分支，切换分支时就是让HEAD指向不同的分支

  - 再次提交一个commit对象的时候。向前走的是HEAD指针，携带着当前分支向前走

  - `git branch` 显示分支列表

  - `git branch xxx`  创建一个名字为xxx的分支

  - `git checkout xxx`  切换到xxx的分支上去

    - 该操作会改变三个地方***HEAD,工作区，暂存区***
    - 执行该命令前，要保证当前分支是干净的（已提交状态）
    - 如果当前分支上有未暂存的修改（未跟踪）或未提交的暂存（已暂存），在checkout时，会带过去（污染其他分支）或报错（已修改）

  - `git branch`     查看分支列表

  - `git branch -D xxx`   强制删除分支xxx(必须要切到其他分支才能删除)

  - `git branch -d xxx`   删除空的分支 删除已经被合并的分支

  - `git log --oneline --decorate --graph --all` 查看项目分支历史

  - `git config --global alias.xx xxx`  将命令xxx取个别名xx

  - `git branch -v`  查看每个分支的最后一次提交

  - `git branch xxx xx`  新建一个分支xxx，并将分支xxx指向hash-code为xx的commit对象

  - `git checkout -b xxx`  新建一个分支xxx,并切换过去

  - `git merge xxx`  当前分支与分支xxx进行合并

    - fast forward快进合并，是在同一条主线上的合并

    <img src="https://raw.githubusercontent.com/yusheng1020/PicGo_MyPicBed/main/gitLearning/pic_4.png" style="zoom:70%;" />

    - 现在两条线上的合并（典型合并）

    <img src="https://raw.githubusercontent.com/yusheng1020/PicGo_MyPicBed/main/gitLearning/pic_4.png" style="zoom:70%;" />

    		-  ​	回到master，`git merge xxx` 合并xxx分支，若发生冲突，则打开具体的文件进行修改，再进行`git add ./` 并更新到版本库

    

  - **git分支工作流**

    - 开发某个网站
    - 为实现某个新的需求，创建一个分支
    - 在这个分支上展开工作
    - 正在此时，你突然接到通知有个严重的问题需要紧急修复，则按照以下方式处理：
    - 使得当前工作分支都处于***已提交***状态，再切换到你的线上分支
    - 为这个紧急任务新建一个分支，并在其中修复他
    - 在测试通过之后，切换回线上分支，然后合并这个修补分支，最终将改动推送到线上分支，再删除这个修补分支
    - 切换回到你最初工作的分支上，继续工作	

    

  - **Git分支模式**

    - 长期分支（master和develop）与特性分支（topic）,develop是自己拉的分支，topic是对于自己的分支的子功能写完一个合并一个

    <img src="https://raw.githubusercontent.com/yusheng1020/PicGo_MyPicBed/main/gitLearning/pic_6.png" style="zoom:100%;" />

- #### Git存储

  - 有时，当你在项目的一部分上已经工作一段时间后，所有的东西都进入了混乱的状态，而这时你想要切换到另一个分支做一点别的事情。问题是，你不想仅仅因为过会儿回到这一点而为做了一半的工作创建一次提交。
  - `git stash list`  查看存储区
  - `git stash`  会将未完成的修改保存到一个栈上，并保证**工作区**十分干净（实际上是提交了，但在日志中查不到），而你可以在任何时候重新应用这些改动(`git stash apply stash@{x}`这个操作能够还原栈中的第x个元素，但不会删除这个元素，删除可以用命令`git stash drop stash@{x}`)
  - `git stash pop stash@{x}`  应用存储并从栈中扔掉它
  - 不指定`stash@{x}`则默认是最新的存储进度
  - `git stash clear`清空存储区

- #### 数据恢复

  - 

- #### Git撤销操作

  - 操作的前提是文件是已追踪的
  - 工作区
    - `git restore xxx`  工作区中的文件xxx修改回到最初的样子（`git checkout -- xxx`也可）
  - 暂存区
    - 文件`add`入了暂存区，撤回操作`git restore --staged xxx`
  - 版本库
    - 撤回自己的提交（注释写错了，工作区已修改的文件忘记提交到暂存区了）
    - `git add xxx` 再提交一次
    - 再`git commit --amend`重新写注释

- #### 集中式与分布式的区别

  - 集中式（SVN）：SVN每次存储的都是差异，需要的硬盘空间相对小一点，可是回滚速度很慢
    - 优点：代码存在单一服务器上，便于项目管理
    - 缺点：服务器宕机：员工写的代码得不到保障；服务器没了：整个项目的历史记录得不到保障
  - 分布式（Git）：git每次存储的都是整个项目的快照，需要欸但硬盘空间会相对大一些，但git对代码进行了极致的压缩，最终需要的实际空间比SVN大不了多少，而且代码回滚速度快
    - 优点：完全分布式
    - 缺点：命令多



版本一：test.txt(v1)，test2.txt(v1_2)

版本二：test3.txt(v2)

版本三：test.txt(v1,v2)

版本四：test.txt(v1,v2,v3)

###### 数据库：

```python
.git/objects/02/e36c19f1cc0901c2dd7d94f596986ea904cd44    		#版本三test.txt的内容---git对象      
.git/objects/31/4fe298016ec142a282536791a98537edf619f8			#版本三提交信息
.git/objects/36/5e9e187f20d28f03072b20cf00acb4d6803639			#版本一的提交信息
.git/objects/3d/ba9ed9ac32e689750a242fe1751fb09166e8c2			#版本四的提交信息
.git/objects/4b/1d4d4b660c400875dd122de217e942f481a378			#版本四的test.txt的内容------git对象
.git/objects/59/3401d06024c1182241a6a0a9733fc36ad3301c			#版本一的两个git对象------树对象
.git/objects/60/ce2fe88a87d16d8c515f96ea905dae9eaeda94			#版本一test2.txt的内容---git对象
.git/objects/62/6799f0f85326a8c1fc522db584e86cdfccd51f			#版本一test.txt的内容----git对象、
.git/objects/8c/1384d825dbbe41309b7dc18ee7991a9085c46e			#版本二test3.txt的内容---git对象
.git/objects/9a/94b281d542c55b7c85639af9027fd9abae75ae			#版本四的三个git对象------树对象
.git/objects/ad/b49d1d5d72173b825904dfe61b75dc8f7351ed			#版本二的三个git对象------树对象
.git/objects/d6/595174c43ca856243fb568d7740e2c3b3391bc			#版本二的提交信息
.git/objects/e2/3d1dc07353aabc5ce9b39a4fdaaf255ca446e6			#版本三的三个git对象------树对象
```
数据库的存放没有任何线性规律

每次在对项目版本进行提交的时候，暂存区中的文件***不会被清空***，所以暂存区中始终保存着多个***最新的git对象***，当我们对已有的文件在工作区进行修改后执行`add`命令会提交到暂存区进行覆盖，对于新添加的文件提交到暂存区中会在暂存区中新添加一个新的该文件对应的***git对象***

