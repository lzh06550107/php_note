******
Git暂存区
******

.. contents:: 目录
   :depth: 3

修改不能直接提交吗
==================
当你修改demo.txt文件时，如，在这个文件后面追加一行。可以使用下面的命令实现内容的追加：

``echo "Nice to meet you." >> demo.txt``

这时可以通过执行 ``git diff`` 命令看到修改后的文件与版本库中的文件的差异。（实际上这句话有问题，与本地比较的不是版本库中的文件，而是一个中间状态的文件。）

.. code-block:: shell

    git diff
    D:\gitdome>git diff
    diff --git a/test.txt b/test.txt
    index e69de29..f6b1c5a 100644
    --- a/test.txt
    +++ b/test.txt
    @@ -0,0 +1 @@
    +Nice to meet you.

既然文件修改了，那么是否可以提交呢？

.. code-block:: shell

    D:\gitdome>git commit -m "Append a nice line."
    On branch master
    Changes not staged for commit:
     (use "git add <file>..." to update what will be committed)
    (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   test.txt

    no changes added to commit (use "git add" and/or "git commit -a")

上面的信息显示对于master分支的提交，因为暂存区该文件没有改变而失败。它建议或者使用 ``git add`` 添加修改到暂存区或者 ``git checkout`` 从版本库中获取文件来覆盖当前文件修改。

是否提交成功的判断依据：

1. 查看提交日志( ``git log --pertty=oneline`` )，如果提交成功，应该有新的提交记录出现。
2. 执行 ``git diff`` 可以看到与之前相同的差异输出，这样说明了之前的提交没有成功。
3. 执行 ``git status`` 查看文件状态，可以看到文件处于修改状态。
4. 精简指令 ``git status -s`` 。

这里，我们根据反馈信息执行“添加”操作。

``git add demo.txt``

执行前面的命令之后，再次执行 ``git diff`` 之后没有任何输出？？这时如果与HEAD(当前版本库的头指针)或master分支(当前工作分支)进行比较，就会发现有差异。这个差异才是正常的，因为尚未真正提交。这时如果直接提交( ``git commit`` )，加入提交任务的 demo.txt 文件的更改就会被提交入库了。但是先不忙着执行提交，再执行一些操作，看看是否会被彻底地搞糊涂。

(1) 继续修改一下demo.txt文件(在文件后面再追加一行)。

``echo "Bye-Bye." >> demo.txt``

(2) 然后执行 ``git status`` ,查看一下状态：

.. code-block:: shell

    D:\gitdome>git status
    On branch master
    Changes to be committed:
      (use "git reset HEAD <file>..." to unstage)

            modified:   test.txt

    Changes not staged for commit:
      (use "git add <file>..." to update what will be committed)
      (use "git checkout -- <file>..." to discard changes in working directory)

            modified:   test.txt

状态输出存在前面两种不同状态输入的杂合体。

(3) 如果显示精简的状态输出，也会看到前面两种精简输出的杂合体。

.. code-block:: shell

    D:\gitdome>git status -s
    MM test.txt

上面M字符存在不同的位置表示不同的意思：

- 位于第一列的字符M的含义是：版本库中的文件与处于中间状态(提交暂存区，stage)中的文件相比有改动；
- 位于第二列的字符M的含义是：工作区当前的文件与处于中间状态(提交暂存区，stage)中的文件相比有改动；
- 在执行 ``git add`` 命令之前，这个M位于第二列(第一列是一个空格)，在执行完 ``git add `` 之后，字符M位于第一列(第二列是空白)，当再次修改当前文件时，第一列的空白变为M。

即现在demo.txt有三个不同的版本，一个在工作区，一个在等待提交的暂存区，还有一个是版本库中最新版本的demo.txt 。通过不同的参数调用 ``git diff`` 命令可以看到不同状态下demo.txt文件的差异。

(1) 不带任何选项和参调用 ``git diff`` 显示工作区的最新改动，即工作区与提交任务(提交暂存区，stage)中相比的差异。

.. code-block:: shell

    D:\gitdome>git diff
    diff --git a/test.txt b/test.txt
    index f6b1c5a..6fd9a69 100644
    --- a/test.txt
    +++ b/test.txt
    @@ -1 +1,2 @@
    -Nice to meet you.
    +Nice to meet you.
    +branch master.

(2) 将工作区和HEAD(当前工作分支)相比，会看到更多的差异。

.. code-block:: shell

    D:\gitdome>git diff HEAD
    diff --git a/test.txt b/test.txt
    index e69de29..6fd9a69 100644
    --- a/test.txt
    +++ b/test.txt
    @@ -0,0 +1,2 @@
    +Nice to meet you.
    +branch master.

(3) 通过参数 ``--cached`` 或 ``--staged`` 调用 ``git diff`` 命令，看到的是提交暂存区和版本库中文件的差异。


理解git暂存区(stage)
===================


git diff魔法
===========


不要使用git commit -a
====================

搁置问题，暂存状态
==================


