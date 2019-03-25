```shell
xucl:testgit xualiang$ git push origin master
root@10.10.51.54's password: 
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 251 bytes | 251.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: error: refusing to update checked out branch: refs/heads/master
remote: error: By default, updating the current branch in a non-bare repository
remote: error: is denied, because it will make the index and work tree inconsistent
remote: error: with what you pushed, and will require 'git reset --hard' to match
remote: error: the work tree to HEAD.
remote: error: 
remote: error: You can set 'receive.denyCurrentBranch' configuration variable t
remote: error: 'ignore' or 'warn' in the remote repository to allow pushing int
remote: error: its current branch; however, this is not recommended unless you
remote: error: arranged to update its work tree to match what you pushed in som
remote: error: other way.
remote: error: 
remote: error: To squelch this message and still keep the default behaviour, se
remote: error: 'receive.denyCurrentBranch' configuration variable to 'refuse'.
To ssh://10.10.51.54/opt/testgit
 ! [remote rejected] master -> master (branch is currently checked out)
error: failed to push some refs to 'ssh://root@10.10.51.54/opt/testgit'


解决方法：
远程服务器输入以下：
git config 'receive.denyCurrentBranch' warn

这种错误是出于git的一种保护措施 master->master 怕误操作
```

