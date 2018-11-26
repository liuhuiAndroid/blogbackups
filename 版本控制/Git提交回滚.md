想fork一个github上的项目，然后回滚到某个简单的版本然后提交到我的远程分支。

git reset --hard c146cb0
git commit -m "revert to version 2.3.0"
git push -u origin master -f    强制提交
