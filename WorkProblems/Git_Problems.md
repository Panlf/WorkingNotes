## 问题列表_Git

### 1、Git将已有的项目上传到Github上

```
1、在github上新建的Repository -- 跟需上传项目的项目名称一致
2、在需上传项目的根目录在打开git，并进行以下命令
3、git init
4、git add .
5、git remote add origin [Repository地址]
6、git commit -m "提交说明"
7、git pull --rebase origin main
8、git push origin main
```

### 2、Git放弃本地修改

本地增加、修改、删除了一些文件，现在想远程代码库覆盖本地，使用了`git check -- .`操作，但是只覆盖了修改的部分，新增和删除的部分没有覆盖。
所以需要使用如下操作：
```
git checkout . && git clean -df
```

 `git checkout . `是放弃本地修改，没有提交的可以回到未修改前版本；`git clean -df`删除当前目录下没有被track过的文件和文件夹。

### 3、Git统计代码量

```
git log --author="username" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }'
```

`--author="username" `根据自己的用户名修改

### 4、Git暂存部分文件

```
git stash push <file1> <file2> <file3> [file4 ...]
```

### 5、Git拉取指定远程分支

```
git clone -b [dev] [respository]
```

dev 代表分支，respository代表远程仓库。

### 6、Git删除远程仓库的文件夹，但是本地不影响

因为之前没有把例如.idea、.gradle的文件夹放入到.gitignore，导致这些无用文件上传了，现在想把这些文件忽略掉，同时删除仓库的文件夹，同时不影响本地的文件
```
git rm -r --cached .idea  #--cached不会把本地的.idea删除
git commit -m 'delete .idea dir'
git push origin master
```

### 7、git add不同命令

git添加文件的时候有git add .和git add -u和git add -A命令

- git add . : 这个命令会监控工作区的状态树，会把工作区的所有变化提交到暂存区，包括修改内容的文件（modified）和新文件（new），但是不包括被删除的文件。
- git add -u : （-update缩写）这个命令不会提交新文件（untracked file），只提交被修改（modified）和被删除（delete）文件，不提交新文件（new）。
- git add -A : （-all缩写）是上面两个命令的集合，会提交所有的文件包括新建、修改和删除的。
