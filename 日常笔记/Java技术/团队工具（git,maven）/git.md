## 上传本地项目到git
1. `git init`   建本地仓库
2. `git remote add origin http://luo1993haoyu@192.168.1.116/lizuzhao1/lng.git`
建本地分支，关联远程仓库


##  常见错误
1. 
```
E:\lng>git remote add origin 
http://luo1993haoyu@192.168.1.116/lizuzhao1/lng.git
fatal: remote origin already exists.
```
解决办法
 1、先输入$ git remote rm origin
    2、再输入$ git remote add origin git@github.com:djqiang/gitdemo.git 就不会报错了！
2.
```
E:\lng>git pull origin master
fatal: Couldn't find remote ref master

```
3.
```
git push -u origin master
remote: Access denied
fatal: unable to access 'http://luo1993haoyu@192.168.1.116/lizuzhao1/lng.git/': The requested URL returned error: 403
```
问题原因
没有连接到git，在idea中可以是因为一开始输入了账号密码，连接到了git

解决办法

1. The repository must be accessible over http://, https:// or git://. 
2. If your HTTP repository is not publicly accessible, add authentication information to the URL: https://username:password@gitlab.company.com/group/project.git. 
3. The import will time out after 15 minutes. For repositories that take longer, use a clone/push combination. 
4. To migrate an SVN repository, check out this document. 
