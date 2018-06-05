### git 分布式版本控制系统

一、 git版本库 (.git/)
```
        .
      ├── COMMIT_EDITMSG
      ├── FETCH_HEAD
      ├── HEAD
      ├── ORIG_HEAD
      ├── config
      ├── description
      ├── hooks/
      ├── index             //暂存区 （stage）
      ├── info/
      ├── logs/
      ├── objects/
      └── refs/
```
```
  git init                              //初始化本地本地
  git add <file> / git add .            //把要提交的所有修改放到暂存区（Stage）
  git commit -m "desc"                  //一次性把暂存区的所有修改提交到分支
  git pull (git fetch + git merge) 
  git push    <br>
  
```  
  
【git remote】  <br>
```
  git remote add [remote_name] [url]  <br>
  git remote rm [remote_name]  <br>
  git remote -v   <br>

        origin	https://github.com/OTREILLY/summary.git (fetch)
        origin	https://github.com/OTREILLY/summary.git (push)
```
git remote show origin
```
* remote origin
  Fetch URL: https://github.com/OTREILLY/summary.git
  Push  URL: https://github.com/OTREILLY/summary.git
  HEAD branch: master
  Remote branch:
    master tracked
  Local ref configured for 'git push':
    master pushes to master (up to date)
```


【git log】
git log [--pretty=oneline]




### 问题：
1. git pull出错：fatal: refusing to merge unrelated histories
原因： 本地与远程分支有修改同一内容
解决：git pull origin master --allow-unrelated-histories, 允许带冲突的pull到本地，然后本地解决冲突

2. git push 出错：error: failed to push some refs to
原因：远程分支有最新修改提交
解决：git pull 

3. 





### 参考

https://git-scm.com/book/zh/v2
