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