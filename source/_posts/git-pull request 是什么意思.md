---
title: pull request 是什么意思
date: 2017-07-18 13:00:00
tags: GitHub
categories: git
---

1、git 上有常见的pull request 功能

 ![](http://images2015.cnblogs.com/blog/605655/201601/605655-20160115184616913-1303099209.png)

**2、pull request 的含义**

 解释一：

 有一个仓库，叫Repo A。你如果要往里贡献代码，首先要Fork这个Repo，于是在你的Github账号下有了一个Repo A2。

 然后你在这个A2下工作，Commit，push等。然后你希望原始仓库Repo A合并你的工作，你可以在Github上发起一个Pull Request，意思是请求Repo A的所有者从你的A2合并分支。

 如果被审核通过并正式合并，这样你就为项目A做贡献了。

 解释二：


我尝试用类比的方法来解释一下 pull reqeust。想想我们中学考试，老师改卷的场景吧。你做的试卷就像仓库，你的试卷肯定会有很多错误，就相当于程序里的 bug。

老师把你的试卷拿过来，相当于先 fork。在你的卷子上做一些修改批注，相当于 git commit。

最后把改好的试卷给你，相当于发 pull request，你拿到试卷重新改正错误，相当于 merge。

当你想更正别人仓库里的错误时，要走一个流程：

1.  先 fork 别人的仓库，相当于拷贝一份，相信我，不会有人直接让你改修原仓库的
2.  clone 到本地分支，做一些 bug fix
3.  发起 pull request 给原仓库，让他看到你修改的 bug
4.  原仓库 review 这个 bug，如果是正确的话，就会 merge 到他自己的项目中

至此，整个 pull request 的过程就结束了。





相关地址:[http://www.zhihu.com/question/21682976](http://www.zhihu.com/question/21682976 "http://www.zhihu.com/question/21682976")

