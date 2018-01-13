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
在版本库 ``.git`` 目录下有一个index文件。当执行 ``git status`` 命令扫描工作区改动的时候，先依据 ``.git/index`` 文件中记录的(用于跟踪工作区文件的)时间戳、长度等信息判断工作区文件是否改变，如果工作区文件的事件戳改变了，说明文件的内容可能被改变了，需要打开文件，读取文件的内容，与更改前的原始文件相比较，判断文件内容是否被更改。如果文件内容没有改变，则将该文件新的时间戳记录到 ``.git/index`` 文件中。因为如果要判断文件是否更改，使用时间戳、文件长度等信息进行比较要比通过文件内容比较要快的多，所以Git这样的实现方式可以让工作区状态扫描更快速地执行，这也是Git高效的原因之一。

文件 ``.git/index`` 实际上就是一个包含文件索引的目录树，像是一个虚拟的工作区。在这个虚拟工作区的目录树中，记录了文件名和文件的状态信息(时间戳和文件长度等)。文件的内容并没有存储在其中，而是保存在Git对象库 ``.git/objects`` 目录中，文件索引建立了文件和对象库中对象实体之间的对应。

.. image:: ./image/git-stage.png
   :align: center

在这个图中，我们可以看到部分 Git 命令是如何影响工作区和暂存区(stage, index)的。

- 图中左侧为工作区，右侧为版本库。在版本库中标记为 ``index`` 的区域是暂存区(stage, index)，标记为 ``master`` 的是 ``master`` 分支所代表的目录树。
- 图中我们可以看出，此时 ``HEAD`` 实际是指向 ``master`` 分支的一个“游标”。所以图示的命令中出现 ``HEAD`` 的地方可以用 ``master`` 来替换。
- 图中的 ``objects`` 标识的区域为 Git 的对象库，实际位于 ``.git/objects`` 目录下。
- 当对工作区修改(或新增)的文件执行 ``git add`` 命令时，暂存区的目录树被更新，同时工作区修改(或新增)的文件内容被写入到对象库中的一个新的对象中，而该对象的ID 被记录在暂存区的文件索引中。
- 当执行提交操作(git commit)时，暂存区的目录树写到版本库(对象库)中， ``master``  分支会做相应的更新。即 ``master`` 最新指向的目录树就是提交时原暂存区的目录树。
- 当执行 ``git reset HEAD`` 命令时，暂存区的目录树会被重写，被 ``master``  分支指向的目录树所替换，但是工作区不受影响。
- 当执行 ``git rm --cached <file>`` 命令时，会直接从暂存区删除文件，工作区则不做出改变。
- 当执行 ``git checkout .`` 或者 ``git checkout -- <file>`` 命令时，会用暂存区全部或指定的文件替换工作区的文件。这个操作很危险，会清除工作区中未添加到暂存区的改动。
- 当执行 ``git checkout HEAD .`` 或者 ``git checkout HEAD <file>`` 命令时，会用 ``HEAD`` 指向的 ``master`` 分支中的全部或者部分文件替换暂存区和以及工作区中的文件。这个命令也是极具危险性的，因为不但会清除工作区中未提交的改动，也会清除暂存区中未提交的改动。

git diff魔法
===========


不要使用git commit -a
====================

搁置问题，暂存状态
==================


