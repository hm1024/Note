[TOC]
# Git 命令
## Git 命令行操作
### **本地库初始化**
`git init`  // 在想让git管理的文件夹执行，该文件夹就称为git本地仓库

### **设置签名**
（如果email 与github不一样，push后，github上不会有提交记录，也就是有时候，当我们提交后，不会再显示绿方块）

```java
 git config --global user.email "1945883067@q.com"
 git config --global user.email "minghai"
```

### **项目级别/仓库级别：仅在当前本地库范围内有效**

`git config user.name tom_pro`
`git config user.email goodMorning_pro@atguigu.com`

信息保存位置：./.git/config 文件

### **系统用户级别：登录当前操作系统的用户范围**

`git config --global user.name tom_glb`
`git config --global goodMorning_pro@atguigu.com`

信息保存位置：~/.gitconfig 文件

>级别优先级
>**就近原则**：项目级别优先于系统用户级别，二者都有时采用项目级别的签名
>如果只有系统用户级别的签名，就以系统用户级别的签名为准
>二者都没有不允许

### **状态查看**
`git status` // 查看工作区、暂存区状态

### 添加
   `git add [file name]` //将工作区的“新建/修改”添加到暂存区

### **提交**
`git commit -m "commit message" [file name]`
// 将暂存区的内容提交到本地库

### **查看历史记录**
`git log`

> 空格往下翻页，b 向上翻页，q 退出

加参数`--pretty=oneline`漂亮的输出一行

`git log --pretty=oneline`

`git log --oneline`

显示历史命令
`git reflog`

查看分支合并图

`git log --graph`


### **前进后退**

**基于索引值操作[推荐]**
`git reset --hard [局部索引值]`
`git reset --hard a6ace91`

**使用^符号：只能后退**

返回到上一个版本

`git reset --hard HEAD^`

> 注：一个^表示后退一步，可以用`git reset --hard HEAD^^`回退到上一个版本， n 个表示后退 n 步
> HEAD@{移动到当前版本需要多少步}

**使用~符号：只能后退**

回退到前n个版本

`git reset --hard HEAD~n`

> 注：表示后退 n 步

reset 命令的三个参数对比


--soft 参数
- 仅仅在本地库移动 HEAD 指针

--mixed 参数
- 在本地库移动 HEAD 指针， n 重置暂存区


--hard 参数
- 在本地库移动 HEAD 指针
  重置暂存区
  重置工作区

### **删除文件并找回**

- 前提：删除前，文件存在时的状态提交到了本地库。
- 操作：git reset --hard [指针位置]
   - 删除操作已经提交到本地库：指针位置指向历史记录
   - 删除操作尚未提交到本地库：指针位置使用 HEAD

### **比较文件差异**

- git diff HEAD -- readme.txt
   + 将工作区中的文件和暂存区进行比较
- git diff HEAD -- readme.txt 
  命令可以查看工作区和版本库里面最新版本的区别
- git diff [本地库中历史版本][文件名]
   + n 将工作区中的文件和本地库历史记录比较

## 分支管理

### **创建分支**

`git branch [分支名]`

### **查看分支**
`git branch -v`

### **切换分支**
方法一：

`git checkout [分支名]`

>  另外：`git checkout -- file`可以丢弃工作区的修改
>
>  一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
>
>  一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
>
>  总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。
>
>  **注意**：`git checkout -- file`命令中的`--`很重要，没有`--`，就变成了“切换到另一个分支”的命令，
>
>  这里如果文件名与另一个分支相同就切换到分支，如果不相同则与加`--`效果一样



方法二：切换分支

` git switch master`

### 创建+切换分支

`git checkout -b <name>`或者`git switch -c <name>`

### **合并分支**

第一步：切换到接受修改的分支（被合并，增加新内容）上 
`git checkout [被合并分支名]`
第二步：执行 merge 命令，合并某分支到当前分支
`git merge [有新内容分支名]`

>  另外：`git checkout -- file`可以丢弃工作区的修改
>
>  一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
>
>  一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
>
>  总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。

### 删除分支

`git branch -d <name>`

### **解决冲突**

第一步：编辑文件，删除特殊符号

第二步：把文件修改到满意的程度，保存退出

第三步：`git add [文件名]`

第四步：`git commit -m "日志信息"`
> **注意：此时 commit 一定不能带具体文件名**

## GitHub
### **查看当前所有远程地址别名**
`git remote -v`  查看当前所有远程地址别名

### 添加远程地址

`git remote add [别名] [远程地址]`

### **推送**
`git push [别名] [分支名]`

### **克隆**
`git clone [远程地址]`

### **拉取**

`pull=fetch+merge`

`git fetch [远程库地址别名] [远程分支名]`

`git merge [远程库地址别名/远程分支名]`

`git pull [远程库地址别名] [远程分支名]`