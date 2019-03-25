- git  clone默认会把远程仓库整个clone下来，但只会在本地默认创建一个master分支。想要clone其他分支，可以使用checkout命令来把远程分支取到本地，并自动建立tracking：

- ```sh
  git checkout -b develop  origin/develop
  ```

- 尝试输入上面命令，把develop克隆到本地，但是失败了：

- ```sh
  $ git checkout -t  origin/develop
     fatal: Cannot update paths and  switch to branch 'develop' at the same time.
     Did you intend to checkout 'origin/develop' which can not be resolved as commit?
  ```

- 用*git remote show origin*命令查看一下远程仓库的状态：

- ```sh
  $ git remote show origin
     \* remote origin
       Fetch URL:  git@github.com:chengshengyang/Login-MVP-Architecture.git
       Push   URL: git@github.com:chengshengyang/Login-MVP-Architecture.git
       HEAD branch:  master
       Remote branches:
         develop new (next fetch will store in  remotes/origin)
         master  tracked
       Local branch configured for 'git pull':
         master merges with remote  master
       Local ref configured for 'git push':
         master pushes to master (local out  of date)
  ```

- 可以看到develop分支的状态是new，而不是跟master分支一样：tracked。这就是导致问题的原因。也就是说develop分支还没有被追踪。执行*git remote update*命令：

- ```sh
  $ git remote update
     Fetching origin
     remote: Counting objects: 81, done.
     remote: Compressing objects: 100% (28/28), done.
     remote: Total 81 (delta 21), reused 12 (delta 12), pack-reused 37
     Unpacking objects: 100% (81/81), done.
     From github.com:chengshengyang/Login-MVP-Architecture
        db7e7b3..82618ec  master      -> origin/master
      \* [new branch]      develop    -> origin/develop
  ```

- 再调用*git remote show origin*命令查看

- ```sh
  $ git remote show origin
     \* remote origin
       Fetch URL:  git@github.com:chengshengyang/Login-MVP-Architecture.git
       Push   URL: git@github.com:chengshengyang/Login-MVP-Architecture.git
       HEAD branch:  master
       Remote branches:
         develop tracked
         master  tracked
       Local branch configured for 'git pull':
         master merges with remote  master
       Local ref configured for 'git push':
         master pushes to master (local out  of date)
  ```

- 这时候develop分支也变成了tracked了，接着执行*git fetch*命令，无任何输出（成功）。问题解决。再次执行克隆远程分支命令

- ```sh
  $ git checkout -b develop  origin/develop
     error: Your local changes to the following files would be overwritten by  checkout:
             .idea/misc.xml
     Please commit your changes or stash them before you can switch branches.
     Aborting
  ```

- 提示本地分支有未提交的内容，先提交，保持工作空间的clean。

- ```sh
  chengshengyang@csy-pc MINGW64 ~/AndroidStudioProjects/Login-MVP-Architecture (master)
     $ git checkout -b develop origin/develop
     Branch develop set up to track remote branch develop from origin.
     Switched to a new branch 'develop'
  
  chengshengyang@csy-pc  MINGW64 ~/AndroidStudioProjects/Login-MVP-Architecture (develop)
  ```

- ok，终于成功啦，现在我在本地克隆了远程仓库的develop分支，并且切换到develop分支了。

- ```sh
  $ git branch
     \* develop
       master
  ```

- 上述问题的解决方案参考：

- - 1.[http://stackoverflow.com/questions/22984262/cannot-update-paths-and-switch-to-branch-at-the-same-time](https://link.jianshu.com/?t=http://stackoverflow.com/questions/22984262/cannot-update-paths-and-switch-to-branch-at-the-same-time)
  - 2.[http://blog.csdn.net/qianggezhishen/article/details/49337169](https://link.jianshu.com/?t=http://blog.csdn.net/qianggezhishen/article/details/49337169)