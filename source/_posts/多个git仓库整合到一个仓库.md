---
title: 多个git仓库整合到一个仓库
date: 2020-11-20 14:28:45
tags:
---


# 多个git仓库整合到一个仓库  

> https://www.cnblogs.com/arnoldlu/p/11130600.html  



## 新建summary仓库

新建一个summary仓库，用于整合一系列git仓库。

```bash
git clone <http_url>/summary.git
cd summary
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```



## 将其它git仓库merge到summary中

```bash
# 将memory.git作为别名alias_memory加入到当前仓库中
git remote add alias_memory <http_url>/memory.git
# 从alias_memory拉取数据到summary仓库
git fetch alias_memory
# 将alias_memory/master分支内容对checkout到summary仓库的branch_memory分支
git checkout -b branch_memory alias_memory/master
# 切换到summary仓库的master分支
git checkout master
# 将branch_memory分支合并到master分支;加 --allow-unrelated-histories 解决Git中fatal: refusing to merge unrelated histories
git merge branch_memory --allow-unrelated-histories
```



## 提交代码

至此就将memory仓库的内容merge到了summary仓库中。

但是此时summary中目录结构和memory一样，就需要将新建一个memory目录，并将memory仓库中对应文件移到summary仓库的memory目录中。

```bash
mkdir memory
git mv xxx memory
git commit -s -m "Merge memory.git to memory."
git push -u origin master
```

依次重复上面内容，即可将多个git仓库合并到summary中。

可以用以下命令一次push所有分支到远程仓库

```bash
git push --all origin -u
```

