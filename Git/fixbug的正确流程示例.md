```sh
场景：
视图云项目已经完成了1.0版本、2.0版本，正在进行3.0版本开发，这是昌乐现场的1.0版本发现了一个需要紧急修复的bug，并且这个bug在2.0版本以及正在进行的3.0版本中均存在，都需要修复。

场景模拟：

第1次提交：
xucl:gittest xualiang$ vim Hello.java
xucl:gittest xualiang$ cat Hello.java 
a
b
xucl:gittest xualiang$ git add .
xucl:gittest xualiang$ git commit -m "add a and b"
[master (root-commit) 2856781] add a and b
 1 file changed, 2 insertions(+)
 create mode 100644 Hello.java

第2次提交：
xucl:gittest xualiang$ vim Hello.java 
xucl:gittest xualiang$ cat Hello.java 
a
b
c
d
xucl:gittest xualiang$ git commit -am "add c and d"
[master 9a0a388] add c and d
 1 file changed, 2 insertions(+)

第3次提交：
xucl:gittest xualiang$ vim Hello.java 
xucl:gittest xualiang$ cat Hello.java 
a
b
c
d
e
f
xucl:gittest xualiang$ git commit -am "add e and f"
[master 8a3f520] add e and f
 1 file changed, 2 insertions(+)

将第1次提交标记为v1.0，将第2次提交标记为v2.0:
xucl:gittest xualiang$ git tag v1.0 285678
xucl:gittest xualiang$ git tag v2.0 9a0a38
xucl:gittest xualiang$ git log --pretty=oneline --graph
* 8a3f52066d2d3a1ee4689a6538e479330db46304 (HEAD -> master) add e and f
* 9a0a388cf560beca818da01ae67046373018bc49 (tag: v2.0) add c and d
* 28567818babfeb01390ecb80abc956474b991c63 (tag: v1.0) add a and b

从master分支的V1.0标签创建分支并在分支上修复bug：
xucl:gittest xualiang$ git checkout -b hotfix-01 v1.0
Switched to a new branch 'hotfix-01'
xucl:gittest xualiang$ vim Hello.java 
xucl:gittest xualiang$ cat Hello.java 
1
b
xucl:gittest xualiang$ git commit -am "fix bug 01, change a to 1"
[hotfix-01 847084f] fix bug 01, change a to 1
 1 file changed, 1 insertion(+), 1 deletion(-)

将hotfix-01分支合并到master分支：
xucl:gittest xualiang$ git checkout master
Switched to branch 'master'
xucl:gittest xualiang$ git merge hotfix-01
Auto-merging Hello.java
Merge made by the 'recursive' strategy.
 Hello.java | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
xucl:gittest xualiang$ git log --pretty=oneline --graph
*   d4b687345439cc0abaa6a8a4e0314068b8238a81 (HEAD -> master) Merge branch 'hotfix-01'
|\  
| * 847084f52ef41d9d91f00bdbbf24570b5980d2b0 (hotfix-01) fix bug 01, change a to 1
* | 8a3f52066d2d3a1ee4689a6538e479330db46304 add e and f
* | 9a0a388cf560beca818da01ae67046373018bc49 (tag: v2.0) add c and d
|/  
* 28567818babfeb01390ecb80abc956474b991c63 (tag: v1.0) add a and b

在master分支上创建版本v1.1:
xucl:gittest xualiang$ git tag v1.1 847084
xucl:gittest xualiang$ git log --pretty=oneline --graph
*   d4b687345439cc0abaa6a8a4e0314068b8238a81 (HEAD -> master) Merge branch 'hotfix-01'
|\  
| * 847084f52ef41d9d91f00bdbbf24570b5980d2b0 (tag: v1.1, hotfix-01) fix bug 01, change a to 1
* | 8a3f52066d2d3a1ee4689a6538e479330db46304 add e and f
* | 9a0a388cf560beca818da01ae67046373018bc49 (tag: v2.0) add c and d
|/  
* 28567818babfeb01390ecb80abc956474b991c63 (tag: v1.0) add a and b

把v1.1checkout下来，检查是否正确：
xucl:gittest xualiang$ git checkout -b testv1.1 v1.1
Switched to a new branch 'testv1.1'
xucl:gittest xualiang$ cat Hello.java 
1
b

在master分支的v2.0标签处拉出一个分支，用于2.0版本的问题修复，有两种代码修改方法，一种是纯手工修改，一种是合并之前的hotfix-01分支，具体采用哪种方法，需要视情况而定。

将hotfix-01分支合并到develop分支（如果3.0版本已经存在release分支了，则需要合并到release分支，因为release分支后续完成时会合并到develop分支，所以不用担心develop分支缺少hotfix的代码，当然如果develop分支的开发工作急需这个bug fix，等不到release分支的完成，页可以合并到develop分支），保证3.0版本以及后续所有版本都不会存在这个bug

```

