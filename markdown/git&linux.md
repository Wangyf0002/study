# 1.Git

## 1.1 git配置

配置姓名和邮箱：

```
git config --global user.name "xxx"
git config --global user.email "xxx"
```

## 1.2 git基础指令

>工作区（新建或修改的文件）
>
>->add->
>
>暂存区（授权给git管理但还未上传）
>
>->commit->
>
>本地仓库

查看状态
```
git status
```
添加至暂存区
- `git add .`  ：添加目录下全部文件
- `git add 1.txt`  : 添加某文件
  
提交至仓库
```
git commit -m "备注"
```

查看本地仓库中的文件
```
git ls-files
```

查看日志
- `git log` : 全部日志
- `git log +语句+语句···`：按下列语句条件查看：
   - `--all` ： 显示所有分支
   - `--pretty=online` ： 显示为1行
   - `--abbrev-commit` ： 输出更短的commitId
   - `--graph` ： 以图的形式显示
 - `git reflog` ：全部逻辑日志

版本回退
```
git reset --hard 某commitId
```
>选中即复制，按下滚轮即粘贴

创建忽略列表

 1. 创建gitignore.txt文件
 2. 
