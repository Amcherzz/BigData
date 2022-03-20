## git版本控制

发展历程：

文件 ：多个文件

本地：本地单个文件，多个版本信息

集中式 ：SVN，文件在集中一个地方，本地只能少量文件

分布式：文件中心和本地都可以存放多个版本

操作步骤：  git  管理文件夹

```linux
# 1 进入文件夹
# 2 初始化
git init
# 3 管理文件
git add .  # 当前文件夹下所有文件
git status  # 检测当前目录下文件状态
```

三种状态变化：

- 红色：新增或修改的文件  -->git add 文件名  转绿色
- 绿色：已经被git管理起来  --> git commit -m '描述信息'
- 生成版本

查看版本记录

```
git log
```

三大区域：

工作区、暂存区、版本库

工作区：修改自动变红

工作--》暂存：git add

暂存--》版本库：git commit



## 常用命令

- 回滚

  ```
  git reset --hard 版本号
  ```

- 若在回滚到回滚前的状态，需用git reflog查看版本，再用git reset

  ```
  git reflog
  git reset --hard 版本号
  ```

- 分支 线上出现bug，创建分支，修复后合并到master

  ```
  git branch
  git branch dev  # 创建分支
  git checkout dev   # 切换到dev分支
  
  # master合并其他分支
  # 首先切换回master
  git checkout master
  git merge dev  # 将dev合并到master
  
  # 注意合并的时候多个分支可能会产生冲突，需要手动解决后再提交合并
  ```

- 删除分支

  ```
  git branch -d 分支名称
  ```

## 工作流

至少两个分支

master 正式

dev  开发，稳定后合入正式



云端：

1. 注册账号
2. 创建仓库
3. 本地代码推送

 ```
 1. 远程仓库起别名
 	git remote add origin 远程仓库地址
 2. 远程推送代码
 	git push -u origin 分支
 ```



```
1. 切换dev分支
	git checkout dev
2. 合并master中的代码
    git merge master
3. 提交代码
    git add .
    git commit -m'xxx'
    git push origin dev

回家中继续开发
1. 切换dev分支
git checkout dev
2. 拉最新代码
git pull origin dev
3. 修改代码
4. 提交代码
	git add .
    git commit -m'xxx'
    git push origin dev
```

开发完毕上线

```
1. dev合并到master
	git checkout master
	git merge dev
	git push origin master
2. 把dev分支也推送到远程
	git checkout dev
	git merge master
	git push origin dev
```

注意在相同行更改后，合并中可能出现冲突，在pull中也可能出现冲突

```
git pull origin dev
上面等同于下面两次
git fetch origin dev  # 拉到本地版本库
git merge origin/dev  # 本地版本库到工作区
```

```
# 注意不要合并已提交到远程仓库的代码，不然会比较乱
# 多个记录合并到一个记录
git rebase


3. 使用场景，用git pull时候有冲突产生分叉
git pull origin dev
更改为
git fetch origin dev
git rebase origin/dev

过程中发生冲突如何解决
git add .
git rebase --continue
```

用工具快速解决冲突

beyond compare

1. 安装beyond compare

2. 在git 中配置

   ```
   git config --local merge.tool bc3
   git config --local mergetool.path '/user/local/bin/bcomp'
   git config --local mergetool.keepBackup false
   ```

3. 应用beyond compare解决冲突

## 命令总结



- 添加远程连接 别名

  ```
  git remote add origin 地址
  ```

- 推送代码

  ```
  git push origin dev
  ```

- 下载代码

  ```
  git clone 地址
  ```

- 拉取代码

  ```
  git pull origin dev
  等价于
  git fetch origin dev
  git merge origin/dev
  ```

- 保持代码提交整洁(变基)

  ```
  git rebase 分支
  ```

- 记录图形展示

  ```
  git log --graph --pretty=format:"%h %s"
  ```

### git flow 工作流

dev（某个功能模块） --> review --> release --> master





## 补充内容

- 检查安装是否成功

  ```
  git --version
  ```

- git 配置

  ```
     1.local，针对当前仓库的配置，git配置默认为local级别
        git config [--local] user.name 本仓库的用户名
        git config [--local] user.email 本仓库的用户邮箱
     2.global，针对当前系统用户的仓库
        git config --global user.name
        git config --global user.email
     3.system，针对当前操作系统所有用户的仓库。（该级别通常不用于配置用户信息）
        git config --system user.name
        git config --system user.email
  ```

  

- 查看git配置

  ```
  查看配置:
  
  git config --list --local       ##只能在仓库里面起作用, 普通路径git不管理  
  git config --list --global    
  git config --list --system   
  ```

  

- 提交  工作区  --> 暂存区 --> 仓库

  ```
  git add -u  添加追踪的文件到暂存区
  git add . 添加所有文件
  ```

- 常用的命令

  ```
  # 重命名文件
   git mv readme readme.md 
  ```

  相当于

  mv readme readme.md 删除文件readme  创建新文件readme.md 

  git add readme.md 添加到暂存区 

  git rm readme 将原来的文件删除掉

- 日志常用

  ```
  # 查看所有分支的历史
  git log --all 
  # 查看图形化的 log 地址
  git log --all --graph 
  # 查看单行的简洁历史。
  git log --oneline 
  # 查看最近的四条简洁历史。
  git log --oneline -n4 
  # 查看所有分支最近 4 条单行的图形化历史。
  git log --oneline --all -n4 --graph 
  # 跳转到git log 的帮助文档网页
  git help --web log 
  # 帮助日志
  git help --web log
  ```

- gitk 图形化界面

- 切换分支

  ```
  # 切换分支命令
  git checkout 分支名称
  ```

  

### git文件目录

- COMMIT_EDITMSG
- config               当前 git 的配置文件
- description      （仓库的描述信息文件）
- HEAD             （指向当前所在的分支），例如当前在 develop 分支，实际指向地址是 refs/heads/develop
- hooks [文件夹]
- index
- info [文件夹]
- logs [文件夹]
- objects [文件夹]    （存放所有的 git 对象，对象哈希值前 2 位作为文件夹名称，后 38 位作为对象文件名, 可通过 git cat-file -p 命令，拼接文件夹名称+文件名查看）
- ORIG_HEAD
- refs [文件夹] 
  - heads  （存放当前项目的所有分支）
  - ags      (存放的当前项目的所有标签，又叫做里程碑)
- 
- cat 命令， 功能：用来显示文件。 例如 cat text.md 显示 text.md 文件的内容
- ls -al 命令， 表示列出当前目录下的所有文件（包括隐藏文件）
- git cat-file -t 命令 ， 查看 git 对象的类型
- git cat-file -p 命令， 查看 git 对象的内容
- git cat-file -s 命令， 查看 git 对象的大小



分支使用

git checkout [-b]

-b 创建并切换至分支

git checkout



```
git rebase
# 让代码提交变简洁  多个提交记录整合成一个
# 注意不要rebase 远程创库已有版本
```

- git rebase
  - 让代码提交变简洁  多个提交记录整合成一个
  - 注意不要rebase 远程创库已有版本
  - 让分支简洁，多个提交记录合并
- git merge和git rebase区别，后者简洁

如果拉远程代码有冲突会产生分叉

git pull 改为 git fetch origin/master  -> git rebase origin/master



git rebase 产生冲突怎么办

-> 解决冲突  -> git add .  -> git rebase --continue



**rebase 会合并commit 信息，注意所操作范围没有被push到远程代码仓库**



```
git commit --amend 
# 更改最后提交记录 会重新生成 SHA-1值
# ** 不要在push之后再进行操作**

# 合并commit
git rebase -i HEAD~3

git rm --cached filename  # untrace a file 
git rm filename  #会删除文件

# 更改文件名
git mv file_from file_to 

git branch --merged
git branch -d branchname  # 删除分支
```

