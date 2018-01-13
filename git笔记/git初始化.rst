******
Git初始化
******

.. contents:: 目录
   :depth: 3

查看Git版本
===========
使用如下命令：

``git --version``

创建版本库及第一次提交
======================
配置Git
---------
在开始Git之旅之前，我们需要设置一下Git的配置变量，这是一次性的工作。即这些设置会在全局文件(用户主目录下的.gitconfig)或系统文件(如/etc/gitconfig)中做永久的记录。

1. 告诉Git当前用户的姓名和邮件地址，配置的用户名和邮件地址将在版本库提交时用到。

   .. code-block:: shell
   
       git config --global user.name "lzh"
       git config --global user.email "lzh08022040@gmail.com"

   因为Git是分布式版本控制系统，所以，每个机器都必须自报家门：你的名字和Email地址。你也许会担心，如果有人故意冒充别人怎么办？这个不必担心，首先我们相信大家都是善良无知的群众，其次，真的有冒充的也是有办法可查的。

   注意git config命令的--global参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。

2. 设置一些Git别名，以便可以使用更为简洁的子命令。
     
   .. code-block:: shell
     
         # 如果拥有系统管理员权限，希望注册的命令别名能够被所有用户使用，可以执行如下命令
         sudo git config --system alias.st status
         sudo git config --system alias.ci commit
         sudo git config --system alias.co checkout
         sudo git config --system alias.br branch
         # 只在本用户的全局配置中添加Git命令别名
         git config --global alias.st status
         git config --global alias.ci commit
         git config --global alias.co checkout
         git config --global alias.br branch

   输入 ``git  ci`` 即相当于 ``git commit`` ;输入 ``git st`` 即相当于 ``git status`` 。

3. 在Git命令输出中开启颜色显示。

   ``git config --global color.ui true``

创建Git版本库
---------------
可以直接进入到工作目录中，通过执行git init命令完成版本库的初始化。

首先，进入一个新的目录，进入该目录后，执行git init创建版本库。

``git init``

或者不用进入新的目录，直接在命令后面输入新的目录，会自动完成目录的创建。

``git init demo``

使用 ``ls -aF`` 查看该命令创建的隐藏目录。这个隐藏的.git目录就是Git版本库(又叫仓库)。

增加文件到版本库
-------------------

``git add demo.txt``

提交文件到版本库
-------------------
使用 ``git commit`` 命令完成提交。在提交过程中需要输入提交说明，这个要求对于Git来说是强制性的，提交说明需要使用参数 ``-m`` 如：

``git commit -m "进步鸟的第一次提交"``

.. code-block:: shell
    
   D:\gitdome>git commit -m "第一次提交"
   [master (root-commit) a7ed4c4] 第一次提交
   1 file changed, 0 insertions(+), 0 deletions(-)
   create mode 100644 test.txt

命令输出结果分析：

- 从命令输出的第一行可以看出，此次提交是提交在名为master的分支上，且是该分支的第一个提交(root-commit)，提交ID为“a7ed4c4”。
- 从命令输出的第二行可以看出，此次提交修改了一个文件。
- 从命令输出的第三行可以看出，此次提交创建了新文件test.txt。

为什么工作区根目录下有一个.git目录
=================================
对于Git来说，版本库位于工作区根目录下的 ``.git`` 目录中，且仅此一处，在工作区的子目录下则没有任何其他跟踪文件或目录。

当在Git工作区的某个子目录下执行操作的时候，会在工作区目录中依次向上递归查找 ``.git`` 目录，找到的 ``.git`` 目录就是工作区对应的版本库， ``.git`` 所在的目录就是工作区的根目录，文件 ``.git/index`` 记录了工作区文件的状态(实际上是暂存区的状态)。

在非git命令工作区执行git命令时会因找不到 ``.git`` 目录而报错。

当我们处于深层目录中时，又什么办法知道Git版本库的位置呢？如何才能知道工作区的根目录在哪里呢？可以用Git的一个底层命令来实现，具体操作过程如下：

1. 在工作区中建立目录 ``a/b/c`` ，进入到该目录中。

   .. code-block:: shell
  
      cd /path/to/my/workspace/demo/
      mkdir -p a/b/c
      cd /path/to/my/workspace/demo/a/b/c

2. 显示版本库 ``.git`` 目录所在的位置

   .. code-block:: shell
  
      git rev-parse --git-dir
      /path/to/my/workspace/demo/.git

3. 显示工作区根目录

   .. code-block:: shell
   
       git rev-parse --show-toplevel
       /path/to/my/workspace/demo

4. 相对于工作区根目录的相对目录

   .. code-block:: shell
   
       git rev-parse --show-prefix
       a/b/c/

5. 显示从当前目录后退到工作区根的深度

   .. code-block:: shell
   
       git rev-parse --show-cdup
       ../../../

从存储安全的角度上来讲，讲版本库放在工作区目录下有点“把鸡蛋装在一个篮子里”的味道。如果忘记了工作区中还有版本库，当直接从工作区的根执行目录删除操作时就会连版本库一并删除。

Git克隆可以降低因为版本库和工作区混杂在一起而导致的版本库被破坏的风险。可以通过克隆操作在本机另外的磁盘/目录中建立Git克隆，并在工作区有新的提交时，手动或自动地执行向克隆版本库的推送操作。如果使用网络协议，还可以实现在其它机器上建立克隆，这样就更安全了(双机备份)。

git config 命令的各参数有何区别
=============================

如下命令用来操作该文件：

- 执行下面的命令，将打开 ``path/to/my/workspace/demo/.git/config`` 文件进行编辑。

  .. code-block:: shell
  
      cd /path/to/my/workspace/demo
      git config -e

- 执行下面的命令，将打开 ``/home/当前用户名称/.gitconfig`` (用户主目录下的.gitconfig文件)全局配置文件进行编辑。

  ``git config -e --global``

- 执行下面的命令，将打开 ``/etc/gitconfig`` 系统级配置文件进行编辑。如果Git安装在非标准位置，则这个系统级的位置文件也可能是在另外的位置。

  ``git config -e --system``

Git的三个配置文件分别是版本库级别的配置文件、全局配置文件(用户主目录下)和系统级配置文件(/etc目录下)。其中版本库级别的配置文件的优先级最高，全局配置文件次之，系统级配置文件优先级最低。依次被覆盖。

Git配置文件采用的是INI文件格式。 ``git config`` 命令可以用于读取和更改INI配置文件的内容。使用只带一个参数的 ``git config <section>.<key>`` 命令可用来读取INI配置文件中某个配置的键值。

