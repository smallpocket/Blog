---
title: GitHub：Start
type: tags
tags:
  - start
  - github
date: 2018-09-28 13:13:46
categories: 版本控制
description:
---
# Git原理

## 直接记录快照，而非差异比较

Git 和其它版本控制系统(包括 Subversion 和近似工具)的主要差别在于 Git 对待数据的方法。 概念上来区分，其它大部分系统以文件变更列表的方式存储信息。 这类系统(CVS、Subversion、Perforce、Bazaar 等等)将它们保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异。存储每个文件与初始版本的差异，如下图所示 - 

![img](assets/919150744_33189.png)

Git 不按照以上方式对待或保存数据。 反之，Git 更像是把数据看作是对小型文件系统的一组快照。 每次你提交更新，或在 Git 中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。 为了高效，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个 *快照流*。如下图所示 - 

![img](assets/480150745_86943.png)

这是 Git 与几乎所有其它版本控制系统的重要区别。 因此 Git 重新考虑了以前每一代版本控制系统延续下来的诸多方面。 Git 更像是一个小型的文件系统，提供了许多以此为基础构建的超强工具，而不只是一个简单的 VCS。 稍后我们在 Git 分支讨论 Git 分支管理时，将探究这种方式对待数据所能获得的益处。

## 近乎所有操作都是本地执行

在 Git 中的绝大多数操作都只需要访问本地文件和资源，一般不需要来自网络上其它计算机的信息。 如果你习惯于所有操作都有网络延时开销的集中式版本控制系统，Git 在这方面会让你感到速度之神赐给了 Git 超凡的能量。 因为你在本地磁盘上就有项目的完整历史，所以大部分操作看起来瞬间完成。

举个例子，要浏览项目的历史，Git 不需外连到服务器去获取历史，然后再显示出来——它只需直接从本地数据库中读取。 你能立即看到项目历史。 如果想查看当前版本与一个月前的版本之间引入的修改，Git 会查找到一个月前的文件做一次本地的差异计算，而不是由远程服务器处理或从远程服务器拉回旧版本文件再来本地处理。

这也意味着你离线或者没有 VPN 时，几乎可以进行任何操作。 如你在飞机或火车上想做些工作，你能愉快地提交，直到有网络连接时再上传。 如你回家后 VPN 客户端不正常，你仍能工作。 使用其它系统，做到如此是不可能或很费力的。 比如，用 Perforce，你没有连接服务器时几乎不能做什么事；用 Subversion 和 CVS，你能修改文件，但不能向数据库提交修改(因为你的本地数据库离线了)。 这看起来不是大问题，但是你可能会惊喜地发现它带来的巨大的不同。

## Git 保证完整性

Git 中所有数据在存储前都计算校验和，然后以校验和来引用。 这意味着不可能在 Git 不知情时更改任何文件内容或目录内容。 这个功能建构在 Git 底层，是构成 Git 哲学不可或缺的部分。 若你在传送过程中丢失信息或损坏文件，Git 就能发现。

Git 用以计算校验和的机制叫做 SHA-1 散列(hash，哈希)。 这是一个由 40 个十六进制字符(`0-9` 和 `a-f`)组成字符串，基于 Git 中文件的内容或目录结构计算出来。 SHA-1 哈希看起来是这样：

```shell
24b9da6552252987aa493b52f8696cd6d3b0037
```

Git 中使用这种哈希值的情况很多，你将经常看到这种哈希值。 实际上，Git 数据库中保存的信息都是以文件内容的哈希值来索引，而不是文件名。

## Git三种状态

请注意！如果你希望后面的学习更顺利，记住下面这些关于 Git 的概念。 Git 有三种状态，你的文件可能处于其中之一：**已提交(committed)、已修改(modified)和已暂存(staged)。** 

- 已提交表示数据已经安全的保存在本地数据库中。 
- 已修改表示修改了文件，但还没保存到数据库中。 
- 已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

由此引入 Git 项目的三个工作区域的概念：Git 仓库、工作目录以及暂存区域。工作目录、暂存区域以及 Git 仓库如下图所示 -

![img](assets/744160702_48164.png)

Git 仓库目录是 Git 用来保存项目的元数据和对象数据库的地方。 这是 Git 中最重要的部分，从其它计算机克隆仓库时，拷贝的就是这里的数据。

工作目录是对项目的某个版本独立提取出来的内容。 这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。

暂存区域是一个文件，保存了下次将提交的文件列表信息，一般在 Git 仓库目录中。 有时候也被称作‘索引’，不过一般说法还是叫暂存区域。

基本的 Git 工作流程如下：

1. 在工作目录中修改文件。
2. 暂存文件，将文件的快照放入暂存区域。
3. 提交更新，找到暂存区域的文件，将快照永久性存储到 Git 仓库目录。

如果 Git 目录中保存着的特定版本文件，就属于已提交状态。 如果作了修改并已放入暂存区域，就属于已暂存状态。 如果自上次取出后，作了修改但还没有放到暂存区域，就是已修改状态。 

## 术语

![img](assets/683090701_88630.jpg)

- workspace：工作区
  - 通过`git init`创建的代码库的所有文件但是不包括`.git`文件(版本库)
- index/stage：暂存区，也叫索引
  - 通过`git add ./*/*Xxx/Xxxx*` 添加的修改,都是进入到暂存区了,肉眼不可见 通过 `git status` 可以看到修改的状态。
- repository：仓库区（本地仓库），也叫存储库
- remote：远程仓库

### code review

对代码进行评阅

### bugfix

修复bug

### pull request

提交合并请求

### merge

合并其他人提出的pull请求，对pull进行审核，若满足则进行冲突解决并合并到项目当中

# Git快速入门

能够配置并初始化一个仓库(`repository`)、开始或停止跟踪(`track`)文件、暂存(`stage`)或提交(`commit`)更改。 本章也将演示如何配置 Git 来忽略指定的文件和文件模式、如何迅速而简单地撤销错误操作、如何浏览项目的历史版本以及不同提交(`commits`)间的差异、如何向远程仓库推送(`push`)以及如何从远程仓库拉取(`pull`)文件。

***远程仓库是什么？***

Repository(仓库)包含的内容 - Git的目标是管理一个工程，或者说是一些文件的集合，以跟踪它们的变化。Git使用Repository来存储这些信息。一个仓库主要包含以下内容(也包括其他内容)：

- 许多commit objects
- 到commit objects的指针，叫做heads
- Git的仓库和工程存储在同一个目录下，在一个叫做.git的子目录中。

## 创建Repository(略过)

## 获取Git仓库

有两种取得 Git 项目仓库的方法。第一种是从一个服务器克隆一个现有的 Git 仓库。第二种是在现有项目或目录下导入所有文件到 Git 中；

## 更新提交到仓库中

### 记录每次更新到仓库

在一个真实的Git仓库当中，对一些文件作出修改，完成一个阶段性目标后，提交本次更新到仓库中。

工作目录下的每一个文件都不外乎这两种状态：已跟踪或未跟踪。 已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后，它们的状态可能处于未修改，已修改或已放入暂存区。 工作目录中除已跟踪文件以外的所有其它文件都属于未跟踪文件，它们既不存在于上次快照的记录中，也没有放入暂存区。 初次克隆某个仓库的时候，工作目录中的所有文件都属于已跟踪文件，并处于未修改状态。

编辑过某些文件之后，由于自上次提交后你对它们做了修改，Git 将它们标记为已修改文件。 我们逐步将这些修改过的文件放入暂存区，然后提交所有暂存了的修改，如此反复。所以使用 Git 时文件的生命周期如下：

![img](assets/212220731_17394.png)

### 检查当前文件状态

要查看哪些文件处于什么状态，可以用 `git status` 命令。 如果在克隆仓库后立即使用此命令，会看到类似这样的输出：

```shell
$ git status
```

### 跟踪新文件

使用命令 `git add` 开始跟踪一个文件。 所以，要跟踪 `mytext.txt` 文件，运行：

```shell
$ git add mytext.txt
```

此时再运行 `git status` 命令，会看到 `mytext.txt` 文件已被跟踪，并处于暂存状态：

```shell
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   mytext.txt
```

只要在 *Changes to be committed* 这行下面的，就说明是已暂存状态。 如果此时提交，那么该文件此时此刻的版本将被留存在历史记录中。`git add` 命令使用文件或目录的路径作为参数；如果参数是目录的路径，该命令将递归地跟踪该目录下的所有文件。

### 暂存已修改文件

现在我们来修改一个已被跟踪的文件。 如果修改了一个名为 `README.md` 的已被跟踪的文件，打开文件 `README.md`并编辑其中的内容，在文件的未尾加入一行内容：”这是暂存已修改文件示例”，然后运行 `git status` 命令，会看到下面内容：

```shell
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   mytext.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   README.md 
```

文件 `README.md` 出现在 *Changes not staged for commit* 这行下面，说明已跟踪文件的内容发生了变化，但还没有放到暂存区。要暂存这次更新，需要运行 `git add` 命令。 这是个多功能命令：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等。 将这个命令理解为“添加内容到下一次提交中”而不是“将一个文件添加到项目中”要更加合适。 现在让我们运行 git add 将”`README.md`“放到暂存区，然后再看看 `git status` 的输出：

```shell
$ git add README.md

Administrator@MY-PC /F/worksp/git-start (master)
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   README.md
        new file:   mytext.txt


Administrator@MY-PC /F/worksp/git-start (master)
$


Shell
```

现在两个文件都已暂存，下次提交时就会一并记录到仓库。 假设此时，想要在 `README.md` 里再加条注释， 重新编辑存盘后，准备好提交。不过且慢，先向 “README.md” 文件加入一点内容，再运行 `git status` ，如下所示 - 

```shell
$ echo "Add new Line content 1002 " >> README.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   mytext.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   README.md
```

在上面可以看到，README.md出现在了两个地方

实际上 Git 只不过暂存了运行 `git add` 命令时的版本， 如果现在提交，`README.md` 的版本是最后一次运行 `git add` 命令时的那个版本，而不是运行 `git commit` 时，在工作目录中的当前版本。 所以，运行了 `git add` 之后又作了修订的文件，需要重新运行 `git add` 把最新版本重新暂存起来：

```shell
$ git add README.md

Administrator@MY-PC /F/worksp/git-start (master)
$ git status
warning: LF will be replaced by CRLF in README.md.
The file will have its original line endings in your working directory.
On branch master
Your branch is up-to-date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   README.md
        new file:   mytext.txt
```

### 状态简览

`git status` 命令的输出十分详细，但其用语有些繁琐。 如果你使用 `git status -s` 命令或 `git status --short` 命令，将得到一种更为紧凑的格式输出。 运行 `git status -s`，状态报告输出如下：

```shell
$ git status -s
 M README.md
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
```

- 新添加的未跟踪文件前面有 ?? 标记
- 新添加到暂存区中的文件前面有 A 标记
- 修改过的文件前面有 M 标记。 你可能注意到了 M 有两个可以出现的位置
- 出现在右边的 M 表示该文件被修改了但是还没放入暂存区
- 出现在靠左边的 M 表示该文件被修改了并放入了暂存区。 

例如，上面的状态报告显示： README 文件在工作区被修改了但是还没有将修改后的文件放入暂存区,`lib/simplegit.rb` 文件被修改了并将修改后的文件放入了暂存区。 而 Rakefile 在工作区被修改并提交到暂存区后又在工作区中被修改了，所以在暂存区和工作区都有该文件被修改了的记录。

### 忽略文件

一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。 在这种情况下，我们可以创建一个名为 `.gitignore` 的文件，列出要忽略的文件模式。 来看一个实际的例子：

```shell
$ cat .gitignore
*.[oa]
*~


Shell
```

第一行告诉 Git 忽略所有以 `.o` 或 `.a` 结尾的文件。一般这类对象文件和存档文件都是编译过程中出现的。 第二行告诉 Git 忽略所有以波浪符(`~`)结尾的文件，许多文本编辑软件(比如 Emacs)都用这样的文件名保存副本。 此外，你可能还需要忽略 `log`，`tmp` 或者 `pid` 目录，以及自动生成的文档等等。 要养成一开始就设置好 `.gitignore` 文件的习惯，以免将来误提交这类无用的文件。

文件 `.gitignore` 的格式规范如下：

- 所有空行或者以 `＃` 开头的行都会被 Git 忽略。
- 可以使用标准的 `glob` 模式匹配。
- 匹配模式可以以(`/`)开头防止递归。
- 匹配模式可以以(`/`)结尾指定目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号(`!`)取反。

所谓的 `glob` 模式是指 shell 所使用的简化了的正则表达式。 星号(`*`)匹配零个或多个任意字符；`[abc]`匹配任何一个列在方括号中的字符(这个例子要么匹配一个字符 `a`，要么匹配一个字符 `b`，要么匹配一个字符 `c`)；问号(`?`)只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配(比如 `[0-9]` 表示匹配所有 `0` 到 `9` 的数字)。 使用两个星号(`*`) 表示匹配任意中间目录，比如`a/**/z` 可以匹配 `a/z`, `a/b/z` 或 `a/b/c/z`等。

下面再看一个 `.gitignore` 文件的例子：

```shell
# no .a files
*.a

# but do track lib.a, even though you're ignoring .a files above
!lib.a

# only ignore the TODO file in the current directory, not subdir/TODO
/TODO

# ignore all files in the build/ directory
build/

# ignore doc/notes.txt, but not doc/server/arch.txt
doc/*.txt

# ignore all .pdf files in the doc/ directory
doc/**/*.pdf
```

> 提示：GitHub 有一个十分详细的针对数十种项目及编程语言的 `.gitignore` 文件列表，你可以在 <http://github.com/github/gitignore> 找到它。

## 查看已暂存和未暂存的修改

如果 `git status` 命令的输出对于你来说过于模糊，你想知道具体修改了什么地方，可以用 `git diff` 命令。 稍后我们会详细介绍 `git diff`，可能通常会用它来回答这两个问题：

- 当前做的哪些更新还没有暂存？ 
- 有哪些更新已经暂存起来准备好了下次提交？ 

尽管 `git status` 已经通过在相应栏下列出文件名的方式回答了这个问题，`git diff` 将通过文件补丁的格式显示具体哪些行发生了改变。

假如再次修改 `README.md` 文件后暂存，然后编辑 `READ.md` 文件并在文件的最后追加一行内容：”`this is another line 1003`“ 之后先不暂存， 运行 `git status` 命令将会看到：

```shell
$ echo "this is another line 1003 " >> README.md
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   README.md
        new file:   mytext.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   README.md


Administrator@MY-PC /F/worksp/git-start (master)
$


Shell
```

要查看尚未暂存的文件更新了哪些部分，不加参数直接输入 `git diff`：

```shell
$ git diff
diff --git a/README.md b/README.md
index ea161e2..6679481 100644
--- a/README.md
+++ b/README.md
@@ -1,2 +1,3 @@
 Add new Line content 1001
 Add new Line content 1002
+this is another line 1003
warning: LF will be replaced by CRLF in README.md.
The file will have its original line endings in your working directo(END)
```

上面输出显示有加一行“+`this is another line 1003`”，前面带有一个加号：“`+`”。

请注意，`git diff` 本身只显示尚未暂存的改动，而不是自上次提交以来所做的所有改动。 所以有时候你一下子暂存了所有更新过的文件后，运行 `git diff` 后却什么也没有，就是这个原因。

然后用 `git diff --cached` 查看已经暂存起来的变化：(`--staged` 和 `--cached` 是同义词)

```shell
$ git diff --cached
diff --git a/README.md b/README.md
index 2f88ca7..ea161e2 100644
--- a/README.md
+++ b/README.md
@@ -1,2 +1,2 @@
-#git-start
-这是一个 Git 学习使用的Git仓库。
\ No newline at end of file
+Add new Line content 1001
+Add new Line content 1002
diff --git a/mytext.txt b/mytext.txt
new file mode 100644
index 0000000..1820ae1
--- /dev/null
+++ b/mytext.txt
@@ -0,0 +1 @@
+This is my first Git control file
warning: LF will be replaced by CRLF in mytext.txt.
The file will have its original line endings in your working directory.

Administrator@MY-PC /F/worksp/git-start (master)
$
```

如上图中所示，分别对比了两个文件：*README.md* 和 *mytext.txt*，其中绿色的内容表示添加，红色的内容表示删除。

> 注意：`git diff` 的插件版本,在本教程中，我们使用 `git diff` 来分析文件差异。 但是，如果你喜欢通过图形化的方式或其它格式输出方式的话，可以使用 `git difftool` 命令来用 `Araxis` ，`emerge` 或 `vimdiff` 等软件输出 `diff` 分析结果。 使用 `git difftool --tool-help` 命令来看你的系统支持哪些 `Git Diff` 插件。

### 提交更新

现在的暂存区域已经准备妥当可以提交了。 在此之前，请一定要确认还有什么修改过的或新建的文件还没有 `git add` 过，否则提交的时候不会记录这些还没暂存起来的变化。 这些修改过的文件只保留在本地磁盘。 所以，每次准备提交前，先用 `git status` 看下，是不是都已暂存起来了，如果没有暂存起来则要先使用命令：`git add .`将所有文件暂存起来， 然后再运行提交命令 `git commit`：

```shell
$ git status
$ git add .
$ git commit


Shell
```

这种方式会启动文本编辑器以便输入本次提交的说明。 (默认会启用 shell 的环境变量 `$EDITOR` 所指定的软件，一般都是 `vim` 或 `emacs`。使用 `git config --global core.editor` 命令设定你喜欢的编辑软件。)

编辑器会显示类似下面的文本信息(本例选用 Vim 的屏显方式展示)：

```shell
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Changes to be committed:
#    new file:   README
#    modified:   CONTRIBUTING.md
#
this is my commit info note.
~
~
".git/COMMIT_EDITMSG" 9L, 283C


Shell
```

可以看到，默认的提交消息包含最后一次运行 `git status` 的输出，放在注释行里，另外开头还有一空行，供你输入提交说明。完全可以去掉这些注释行，不过留着也没关系，多少能帮你回想起这次更新的内容有哪些。 (如果想要更详细的对修改了哪些内容的提示，可以用 -v 选项，这会将你所做的改变的 `diff` 输出放到编辑器中从而使你知道本次提交具体做了哪些修改。) 退出编辑器时，Git 会丢掉注释行，用输入提交附带信息生成一次提交。如上面示例中，提交的备注信息是：“this is my commit info note.”。

另外，也可以在 `commit` 命令后添加 `-m` 选项，将提交信息与命令放在同一行，如下所示：

```shell
$ git commit -m "this is my commit info note."
[master 463dc4f] Story 182: Fix benchmarks for speed
 2 files changed, 2 insertions(+)
 create mode 100644 README.md


Shell
```

现在已经创建了第一个提交！ 可以看到，提交后它会告诉你，当前是在哪个分支(`master`)提交的，本次提交的完整 SHA-1 校验和是什么(`463dc4f`)，以及在本次提交中，有多少文件修订过，多少行添加和删改过。

请记住，提交时记录的是放在暂存区域的快照。任何还未暂存的仍然保持已修改状态，可以在下次提交时纳入版本管理。 每一次运行提交操作，都是对你项目作一次快照，以后可以回到这个状态，或者进行比较。  

### 跳过使用暂存区域

尽管使用暂存区域的方式可以精心准备要提交的细节，但有时候这么做略显繁琐。 Git 提供了一个跳过使用暂存区域的方式， 只要在提交的时候，给 `git commit` 加上 `-a` 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git add` 步骤：

```shell
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
$ git commit -a -m 'added new benchmarks'
[master 83e38c7] added new benchmarks
 1 file changed, 5 insertions(+), 0 deletions(-)
```

看到了吗？提交之前不再需要 `git add` 文件“`README.md`”了。

### 移除文件

要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除(确切地说，是从暂存区域移除)，然后提交。 可以用 `git rm` 命令完成此项工作，并连带从工作目录中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了。

如果只是简单地从工作目录中手工删除文件，运行 `git status` 时就会在 “*Changes not staged for commit*” 部分(也就是 未暂存清单)看到：

```shell
$ rm mytext.txt
$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    mytext.txt

no changes added to commit (use "git add" and/or "git commit -a")

```

下一次提交时，该文件就不再纳入版本管理了。 如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 `-f`(注：即 `force` 的首字母)。 这是一种安全特性，用于防止误删还没有添加到快照的数据，这样的数据不能被 Git 恢复。

另外一种情况是，我们想把文件从 Git 仓库中删除(亦即从暂存区域移除)，但仍然希望保留在当前工作目录中。 换句话说，你想让文件保留在磁盘，但是并不想让 Git 继续跟踪。 当你忘记添加 `.gitignore` 文件，不小心把一个很大的日志文件或一堆 `.a` 这样的编译生成文件添加到暂存区时，这一做法尤其有用。 为达到这一目的，使用 `--cached` 选项：

```shell
$ git rm --cached mytext.txt
$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        deleted:    mytext.txt
```

`git rm` 命令后面可以列出文件或者目录的名字，也可以使用 `glob` 模式。 比方说：

```shell
$ git rm log/\*.log
```

注意到星号 `*` 之前的反斜杠 `\`， 因为 Git 有它自己的文件模式扩展匹配方式，所以我们不用 shell 来帮忙展开。 此命令删除 `log/` 目录下扩展名为 `.log` 的所有文件。 类似的比如：

```shell
$ git rm \*~
```

该命令为删除以 `~` 结尾的所有文件。

### 移动文件

不像其它的 VCS 系统，Git 并不显式跟踪文件移动操作。 如果在 Git 中重命名了某个文件，仓库中存储的元数据并不会体现出这是一次改名操作。 不过 Git 非常聪明，它会推断出究竟发生了什么，至于具体是如何做到的，我们稍后再谈。

既然如此，当你看到 Git 的 `mv` 命令时一定会困惑不已。 要在 Git 中对文件改名，可以这么做：

```shell
$ git mv file_from file_to
```

它会恰如预期般正常工作。 实际上，即便此时查看状态信息，也会明白无误地看到关于重命名操作的说明：

```shell
$ git mv README.md README
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
```

其实，运行 `git mv` 就相当于运行了下面三条命令：

```shell
$ mv README.md README
$ git rm README.md
$ git add README
```

如此分开操作，Git 也会意识到这是一次改名，所以不管何种方式结果都一样。 两者唯一的区别是，mv 是一条命令而另一种方式需要三条命令，直接用 `git mv` 轻便得多。 不过有时候用其他工具批处理改名的话，要记得在提交前删除老的文件名，再添加新的文件名。

## 查看提交历史

git log

## 撤销操作

## 远程仓库的使用

### 推送到远程仓库

当想分享你的项目时，必须将其推送到上游。 这个命令很简单：`git push [remote-name] [branch-name]`。 当你想要将 `master` 分支推送到 `origin` 服务器时(再次说明，克隆时通常会自动帮你设置好那两个名字)，那么运行这个命令就可以将所做的备份到服务器：

```shell
$ git push origin master
```

只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效。 当你和其他人在同一时间克隆，他们先推送到上游然后你再推送到上游，你的推送就会毫无疑问地被拒绝。 你必须先将他们的工作拉取下来并将其合并进你的工作后才能推送。

# GitHub的使用

- 命令行的难度蛮大的,初入门经常出现各种的问题
- 推荐使用工具source tree轻松解决问题
- source tree需要翻墙注册,自带VPN哦

# 命令

- `git pull`取回远程主机某个分支的更新，再与本地的指定分支合并
- `git fetch`命令用于从另一个存储库下载对象和引用。
- `git checkout`在取回远程主机的更新以后，可以在它的基础上，使用`git checkout`命令创建一个新的分支。

`git add`命令将文件内容添加到索引(将修改添加到暂存区)。也就是将要提交的文件的信息添加到索引库中

`git push`命令用于将本地分支的更新，推送到远程主机。

`git merge`命令用于将两个或两个以上的开发历史加入(合并)一起。

# 私人仓库

本来最近挺想弄个私人仓库的,苦于贫穷,一直没有好的解决办法,终于,被我找到了这个学生优惠,造作呀.

>今天在 HN 上到一则消息，“Free private Github repos for students and edu people |  学生和教育人士可免费申请 Github 私有仓库”。


- 如果你是学生，并且已有 Github 账号，那都可以去申请微型方案（Micro Plan）。私有仓库可用来托管你个人的课程项目、论文或和学位相关的研究。当然了，如果学生需要用于团队的私有仓库，同样可以申请。
- 如果你是教师，或者是学生组织（诸如校园报社和机器人俱乐部）的赞助商，可以申请组织账号（Organization  account ）。


# 参考 #

1. [申请GitHub学生免费私有仓库](https://yq.aliyun.com/ziliao/304951)
2. [武汉理工大学申请校园邮箱的网址](http://zhlgd.whut.edu.cn/tp_up/view?m=up#act=portal/viewhome)
3. [加快git clone 几十倍速度的小方法 （30KB vs 2M）](<https://blog.51cto.com/11887934/2051323>)
4. [易百教程](https://www.yiibai.com/git/getting-started-git-basics.html  )
