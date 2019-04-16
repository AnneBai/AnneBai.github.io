---
layout: post
title: Github--如何创建pull request
---

主要参考： [How To Create a Pull Request on GitHub | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-create-a-pull-request-on-github)

github上有很多开源项目，每个人都可以阅读和贡献代码，帮助修改错误、增强代码稳健性或丰富功能等。但这样做需要遵循一定的流程和规则，才能使所有人的合作和代码更新有序和可靠地进行。以下是一般步骤：

### 0. 前提
1. 电脑中有安装git;
2. 在github中有自己的账号；
3. 在github中找到某个开源项目；

### 1. 复制代码到本地
#### 1.1 fork仓库

登录github,，在github某个项目仓库右上角有fork操作按钮，点击后可以在自己的账号中建立一个该仓库的副本；fork成功后会自动切换到自己账号中的这个副本仓库页面；

#### 1.2 clone代码到本地
点击 `Clone or download` 按钮，复制代码后，在电脑某个本地文件夹中打开git bash，执行
```
git clone <https://.....>
```
复制成功后进入项目文件夹内，可以看到已经在当前项目的master分支中（默认）；

### 2. 建立新分支
新分支的名字最好与自己建立该分支的目的相符，具有说明性，例如"fix-documentation-typos"或"front-end-migration";

在项目的目录下， 执行
```
git branch <new-branch-name>   // 新建分支
git checkout <new-branch-name> // 切换至新分支
```
或
```
git checkout -b <new-branch-name> // 新建并切换至新分支
```

### 3. 在本地进行修改

#### 3.1 修改代码后，查看当前变更状态
```
git status
```
#### 3.2 提交修改至暂存区
```
git add <file-uri> // 提交某文件修改
// 或
git add -A // 提交所有修改
git add . // 提交所有修改
```
#### 3.3 记录修改，提交commit log需要详细写明 为何修改/代码如何工作等方面信息；
```
git commit -m "detail messages" // 记录该次修改至本地仓库
```
#### 3.4 将提交的修改记录上传至远程--自己fork的仓库副本：
```
git push --set-upstream origin <new-branch-name>
```

### 4. 更新本地仓库
当我们在本地修改代码的同时，可能其他协作者也在他们自己的分支上修改并提交，所以我们提交自己的修改时可能远程的仓库已经相比我们克隆时有了新的更新。在提交自己的修改前， 需要确认自己本地的代码处于最新状态；

#### 4.1 设置远程仓库

当前本地指向的远程仓库默认指向我们自己fork的副本仓库；查看远程仓库列表：
```
git remote // 远程仓库列表
git remote -v // 远程仓库列表， 包含url
```
将其设置为指向原始仓库：
```
git remote add upstream <https://....> // upstream为该远程仓库简称，可以直接引用
```
#### 4.2 拉取最新代码
```
git fetch upstream // 引用远程仓库的简称代替完整url
```

#### 4.3 合并最新代码

切换至master分支, 并执行合并
```
git checkout master
git merge upstream/master // 将远程更新的upstream/master分支合并至本地master分支
```

### 5. 创建Pull Request

在自己fork的仓库副本上，点击页面左边的New pull request按钮，此时github会提醒"可以合并"并显示出填写此pull request的标题和详细comments的编辑框。编辑完成后点击Create pull request按钮。

### 6. 最后
等待原始仓库管理者的回应，可能我们还需要对自己的修改进行重新修改和提交。

