~~~shell
一、开发分支（dev）上的代码更新后，要合并到 master 分支
git checkout dev     #切换到dev分支
git pull             #将远程更新的代码同步到本地
git checkout master  #切换到master
git merge dev        #将dev分支合并到master上
git push -u origin master   #提交
~~~

~~~shell
二、当master代码改动了，需要更新开发分支（dev）上的代码
git checkout master 
git pull 
git checkout dev
git merge master 
git push -u origin dev
~~~

~~~shell
三、a分支代码更新，将a分支代码提交到新分支b
git branch b
git checkout a
git add xxxxxx
git commit -m "xxxx"
git push -u origin a:b
~~~

~~~shell
四、a分支代码更新，将a分支代码合并到旧分支b上
git checkout b
git pull origin b
git merge a
git push origin b
~~~

~~~shell
五、常用git 命令
git branch aaa    创建aaa分支
git checkout bbb         切换到bbb分支
git pull origin bbb   表示将远程origin主机的bbb分支拉取过来和本地的当前分支进行合并
git status     查看当前状态
git branch -a   查看当前都有那些分支
git clean -xfd  清除掉本地被修改的文件
git reset --hard 回退代码至当前版本
~~~

~~~shell
六、当代码提交错误分支，需要回滚时，git 回退远程代码到指定版本
git reset --hard 3266044634a9ac3891b015b3666d99e70aeaab98（回到版本） 
 
git push origin HEAD --force
~~~

