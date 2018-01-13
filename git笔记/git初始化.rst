******
Git初始化
******

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

