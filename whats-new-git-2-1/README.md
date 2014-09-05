原文链接： [What’s new in Git 2.1](http://blogs.atlassian.com/2014/08/whats-new-git-2-1/)

`Git` `2.1`新特性
======================

`git` `2.0.0`发布2个半月后，`2.1.0`作为小版本更新，带来了一大波令人兴奋的新特性。

完整的发布说明文档可以在这里[查看](https://raw.githubusercontent.com/git/git/master/Documentation/RelNotes/2.1.0.txt)，
如果你对`git`社区接触不多，会觉得发布说明文档说明有些太简明了。
这篇文章是我对这次发布中觉得令人兴奋的方面所做的评注。

更好的分页程序缺省设置
------------------

本文引文都是直接提取自发布说明文档，会在其中加上自己的评注。

> 自从很早期的`Git`开始，当调用`less`分页程序时，`LESS`环境变量缺省值成`FRSX`。
`S`选项（截断长文本行而不是折行）从缺省值中删除了，因为对不同的人有不同的说法，这个选项或多或少是个人口味问题。
（比如，`R`选项就合理得多，因为很多不同的输出都是彩色的，而`FX`也是合理的，因为输出常常短于一页。）

如果你没有覆盖过`git`分页程序的缺省值，这个变化意味着`git`命令的分页输出会在终端宽度的地方折行而不是截断行。
下面是`git` `2.1.0`（折行）和`git` `2.0.3`（截断）在右侧的显示的例子：

![](git210leftvsgit200right-600x293.png)

这个只会影响你日志的输出，如果你用的是一个窄的终端，或者在提交消息中有长行。
一般`git`推荐提交日志信息[不要超过72字符宽度](http://stackoverflow.com/questions/2290016/git-commit-messages-50-72-formatting)，
但如果觉得折行很烦，可以通过恢复原来的行为来关闭：

```bash
$ git config core.pager "less -S"
```

当然，分页程序也会用于其它的输出，比如`git blame`，这种情况下由于作者名长度和代风格，可以能会有很长的行。
2.1.0的发布说明文档也指出了可以只在`blame`的分页程序中启用`-S`选项：

```bash
$ git config pager.blame "less -S"
```

如果你对`git`还在使用的缺省`less`选项很好奇，说明如下：

- `-F`：让`less`进程退出，如果输出少于一页。
- `-R`：保证只有`ANSI`颜色转义序列按原始形式输出，这样`git`控制台颜色才能生效。
- `-X`：避免屏幕在`less`启动时被清空。这个也是在`less`输出少于一页时才是有用的。

更好的`Bash`补全
------------------

> 更新了`Bash`的补全脚本（在`contrib/`），对于定义了复杂命令序列的别名能更好的处理。

这个**超酷**！我是一个自定义`git`别名的大粉丝。能够在复杂的别名上用上`git`的`Bash`自动补全，
让这些别名在命令行上使用起来更强大和方便。举个例子，我定义一个可以从日志中`grep`出`JIRA`风格的`issue`主键（如`STASH-123`）的别名：

```bash
issues = !sh -c 'git log --oneline $@ | egrep -o [A-Z]+-[0-9]+ | sort | uniq' -
```

所有的命令行参数传给`git log`命令，这样可以限制提交的范围用于返回`issue`主键。
比如，`git issues -n 1`只会显示我的分支最近一次提交所关联的`issue`主键。
在`2.1.0`中，`git`的`Bash`补全让我就像`git log`命令一样去补全`git`的`issue`别名。

在`git` `2.0.3`下，键入`git issues m<tab>`会退化成缺省的`Bash`补全行为，列出当前目录下`m`开头的文件。
在`git` `2.1.0`下，正确地补全成`master`，就和`git log`命令下补全动作一样。
通过在别名加上空命令前缀`:`前缀，就可以触发`Bash`的补全。
如果要补全的不是别名中的第一个命令，这个很有用。举个例子：

```bash
issues = "!f() { echo 'Printing issue keys'; git log --oneline $@ | egrep -o [A-Z]+-[0-9]+ | sort | uniq; }; f"
```

这个别名不能正常补全，因为`git`不能把`echo`命令识别为补全目标。
但如果加上前缀成`: git log;`，补全就正确了：

```bash
issues = "!f() { : git log ; echo 'Printing issue keys'; git log --oneline $@ | egrep -o [A-Z]+-[0-9]+ | sort | uniq; }; f"
```

这是个可用性巨大改进，如你喜欢编写复杂的别名脚本！
请记住，补全功能的脚本在`contrib/`目录下，不是`git`核心的一部分，
所以如果你要使用这个功能，不要忘了更新`Bash profile`指向新版本的`contrib/completion/git-completion.bash`。

`git commit`命令使用`approxidate`
------------------

> `git commit ‐‐date=<date>`选项有了更多的时间戳格式选项，包括`--date=now`。

当严格的`parse_date()`函数不能解析给的日期字符串时，
`git`提交的`--date`选项现在会回退到`git`酷炫的（也有些怪异的）`approxidate`（大概日期）解析器。
`approxidate`可以处理显而易见的值，像`--date=now`，也允许一些略复杂格式，像`--date="midnight the 12th of october, anno domini 1979"`或是`--date=teatime`。
如果你想了解更多，Alex Peattie有一篇优秀的[关于`git`酷炫日期处理的博文](http://alexpeattie.com/blog/working-with-dates-in-git/)。

更好的路径显示方式`grep.fullname`
------------------

> `git grep`会读取`grep.fullname`配置变量，强制`‐‐full-name`成为缺省。
这可能会让使用脚本的用户出错，这些用户不期望这样的新行为。

省得你去翻`git-grep`的`man`，下面是`--full-name`选项的文档说明：

> --full-name
>
> 当从子目录运行时，命令输出的路径通常相对于当前目录。这个选项强制输出的路径是相对项目的顶级目录。

非常贴心！这个缺省行为非常符合我的工作方式，我常常会运行`git grep`找出一个文件的路径，
拷贝粘贴到一个`XML`文件中（我这么做出卖了其实我是个`Java`开发）。
如果你也觉得有用，只要简单运行：

```bash
$ git config --global grep.fullname true
```

在你的配置文件开启这个选项。

`--global`选项把配置应用到`$HOME/.gitconfig`文件中，这样配置值就会我系统上所有`git`仓库的缺省行为。
如果有必要，你可以也只在仓库级别覆盖配置值。

更聪明的`git replace`
------------------

先停一下！先看看`git replace`做了什么？

简单地说，`git replace`重写`git`仓库中的某个对象并且不保持对应树或是提交的`SHA`不变。
如果你是第一次听到`git replace`并且知道`git`的数据模型，会觉得这样的做法听起来很逆天！
我就是这么觉得。我有另一篇正在写的博文讨论什么时候和为什么要使用这样的功能。
如果现在你想了解更多，看[这篇文章](http://git-scm.com/blog/2010/03/17/replace.html)比看`man`手册好得多，
手册中只有很少且不充分的用例说明。

> `git replace`会读取`--edit`选项，可以编辑一个已有的对象再做替换。

`--edit`选项会`dump`一个对象的内容到一个临时文件，启动你喜欢的编辑器，这样就方便地拷贝和替换这个对象。
要替换`master`分支的最近那次提交，可以简单运行命令：

```bash
$ git replace --edit master
```

或者编辑最近那次提交的`blob`，假设是文件`jira-components/pom.xml`，可以运行命令：

```bash
$ git replace --edit master:jira-components/pom.xml
```

应该这么做？基本上不会 :smile: 大部分情况应该用`git rebase`重写对象，这样会正确的重写提交的`SHA`，保证历史是健全的（`sane history`）。

> `git replace`会读取`--graft`选项，可以编辑父提交。

`--graft`选项是替换一个提交有相同的内容但用不同的父提交的快捷操作。
这可以方便地完成一个稍微正常一点的`git replace`的用例，[缩短`git`历史](http://git-scm.com/blog/2010/03/17/replace.html)。
要替换`master`分支的最近那次提交的父，可以简单运行命令：

```bash
$ git replace master --graft [new parent]..
```

或者要砍掉某个点之后的历史，可以忽略所有父提交让这个提交成为孤儿提交：

```bash
$ git replace master --graft
```

再次说明，没有好的理由基本上不应该这么做。通常重写历史的首选方法是用明智的`git rebase`。

等等，还有还有！
------------------

在`git` `2.1.0`中还有其它很好的内容我没有在一篇文章中涉及到，所以有兴趣可以看看[完整的发布说明文档](https://raw.githubusercontent.com/git/git/master/Documentation/RelNotes/2.1.0.txt)。
由衷地感谢`git`团队又提供了一个高质量和丰富新功能的版本。
如果你有兴趣了解更多有关于`git`的实用小建议和花边新闻，
欢迎在Twitter上关注我（[@kannonboy](https://twitter.com/kannonboy)）和Atlassian开发工具（[@atldevtools](https://twitter.com/atldevtools)）。