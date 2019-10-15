## 简述 Gitflow 工作流及实际例子的演示

#### 什么是工作流

> WorkFlow 的字面意思，工作流，即工作流程。在分支篇里，有说过这样的话：因为有分支的存在，才构成了多工作流的特色。事实的确如此，因为项目开发中，多人协作，分支很多，虽然各自在分支上互不干扰，但是我们总归需要把分支合并到一起，而且真实项目中涉及到很多问题，例如版本迭代，版本发布，bug 修复等，为了更好的管理代码，需要制定一个工作流程，这就是我们说的工作流，也有人叫它分支管理策略。
>
> 工作流不涉及任何命令，因为它就是一个规则，完全由开发者自定义，并且自遵守，正所谓无规矩不成方圆，就是这个道理。

上面是引用网上的一段描述，简单来说，工作流就是一套让大家协助完成开发工作的一套代码管理规则。



#### Gitflow 介绍

目前比较流行的工作流主要有三种，分别为 `Git Flow` 、`GitHub Flow` 、`GitLab Flow` 

由于本人公司使用的是 `Git Flow` ，我对此还是比较熟悉，所以这里只介绍 `Git Flow`

 要明白 `Git Flow` 是如何工作的，最主要还是要理解下面这张图：

![](https://raw.githubusercontent.com/JLiamLin/cloudImg/master/img/f2a8e2be7b4515665137cc295d6347a5.png)



主要有五种分支，分别为：

##### master 分支

master 分支的代码是生产环境使用的代码，是最稳定的代码版本。

##### develop 分支

develop 分支是从master分支新建的一个分支，每次迭代新特性都是基于此分支的。

##### feature 分支

feature 分支是从develop 分支新建的，每次迭代新特性都要拉一条feature 分支，确保不同的feature 互不干扰。

##### release 分支

release 分支是用来打包测试使用的分支。当feature 分支开发完成后，需要合并到develop 分支上，然后再从develop 分支新建一条release分支，发布到测试环境用于测试feature 分支新开发的功能。在测试的过程中，有bug也直接在release 分支上修改。没问题后，合并回develop 分支以及master 分支，打上标签，正式上线新功能。

##### hotfixes 分支

hotfixes 分支是用来修复线上版本的bug的，直接从master 分支新建，修复完bug后合并回develop 分支和master 分支，并打上相应的标签。



#### 实际例子演示Git Flow 工作流程

#####  - 项目初始化

1、现在有一个新的项目，小明先创建一个该项目的远程仓库，初始化完成该项目后，推到远程仓库的master 分支。

2、小明（运维角色）创建develop 分支，用于开发使用

```zsh
# 小明本地基于master 分支创建develop 分支
$ git checkout -b develop
# 将develop 分支推到远程仓库，建立跟踪
$ git push -u origin develop
```

至此，新项目的master 分支和develop 分支都已创建完毕，可以开始进行新项目的开发工作了。

##### - 新迭代开发

1、小红（开发角色）接到一个新迭代的需求，该需求的版本号为100110，小红需要先从远程仓库把该项目克隆下来，然后切换到develop 分支，从develop 分支创建一个 feature分支，该分支命名为100110(具体命名规则可按公司规定)，提交到远程仓库，这样别的开发人员要参与到这次需求中来，也要使用该分支，然后就可以在该分支下进行开发工作。

```zsh
# 克隆项目
$ git clone git@github.com:test/newProject.git
# 切换到develop 分支
$ git checkout -b develop origin/develop
# 从develop 分支创建一个feature 分支，该分支以需求版本号(100110)命名
$ git checkout -b 100110
# 将100110 分支推到远程仓库，这样子别的开发人员也就可以基于此版本开发这一次的迭代需求
$ git push origin 100110:100110
```

2、小红完成了100110的需求后，将代码推到远程仓库

```zsh
$ git add .
$ git commit -m "100110版本需求开发"
$ git push origin 100110:100110
```

3、小明（运维角色）接到把100110迭代上测试的请求后，需要将100110 分支(feature 分支)的代码合并到 develop 分支，然后从develop 分支创建出一个release 分支，将release分支的代码部署到测试环境，用于测试，该release 分支命名为release-100110(具体命名规则可按公司规定)。

```zsh
# 切换到develop 分支
$ git checkout develop
# 将 100110 分支合并到 develop 分支上
$ git merge --no-ff 100110
# 基于develop 分支新建release-100110 分支
$ git checkout -b release-100110
# 删除 100110 分支
$ git branch -d 100110
# 然后将release-100110 分支的代码部署到测试环境，通知测试人员测试100110的迭代需求
```

4、测试人员测出了bug 后，通知到小红，小红拉取远程仓库最新的release-100110 分支的代码，进行bug 修复，修复完后再将修改后的代码推到远程仓库，让运维再次将最新的release-100110 分支的代码部署到测试环境供测试人员测试

```zsh
# 拉取远程仓库最新的release-100110 分支的代码
$ git checkout -b release-100110 origin/release-100110
# bug修复完成后，将最新的代码推到远程仓库
$ git add .
$ git commit -m "100110版本xxx bug修复"
$ git push
# 让运维再次将最新的release-100110 分支的代码部署到测试环境
```

5、测试通过没有问题以后，通知运维上线。小明（运维角色）接到把100110迭代上生产环境的请求后，将release-100110 分支 合并到develop 分支和 master 分支，并打好tag 以方便后续跟踪。

```zsh
# 切换到develop 分支
$ git checkout develop
# 将 release-100110 分支合并到 develop 分支上
$ git merge --no-ff release-100110
# 切换到master 分支
$ git checkout master
# 将 release-100110 分支合并到 master 分支上
$ git merge --no-ff release-100110
# 删除 release-100110 分支
$ git branch -d release-100110
# 给新颁布打上tag 标签
$ git tag -a v1.0 -m "新颁布v1.0, 增加 xxx 功能"
$ git push --tags
# 然后将最新的master 分支的代码部署到生产环境，让测试人员测试看有没问题
```

6、最新的master 分支的代码部署到生产环境后， 让测试人员测试有没问题，如果没有问题，100110 迭代从开发到完成上线的流程就完成了。

##### - 线上bug修复

有时遇到线上出现bug ，需要紧急修复

1、发现线上环境出现了个bug，让小红（开发角色）紧急修复。此时小红需要基于 master 分支创建一个热修复分支，也就是图上说的hotfixes 分支，该分支命名为 issue-#100128(具体命名规则可按公司规定)。

```zsh
# 拉取最新的master 分支代码
$ git checkout master
$ git pull
# 基于master 分支创建hotfixes 分支
$ git checkout -b issue-#100128
# 修复完bug后，将代码推到远程仓库
$ git add .
$ git commit -m "100128bug修复， 修复了 xxxx bug修复"
$ git push
# 将issue-#100128分支的代码部署到测试环境， 让测试人员测试看是否有问题
```

2、小明（运维角色）接到把100128bug的修复上线的请求后，将 hotfixes 分支合并到develop 分支 和 master 分支

```zsh
# 切换到develop 分支
$ git checkout develop
# 将 issue-#100128 分支合并到 develop 分支上
$ git merge --no-ff issue-#100128
# 切换到master 分支
$ git checkout master
# 将 issue-#100128 分支合并到 master 分支上
$ git merge --no-ff issue-#100128
# 删除 issue-#100128 分支
$ git branch -d issue-#100128
# 给新颁布打上tag 标签
$ git tag -a v1.1 -m "新颁布v1.1, 修复 xxx 问题"
$ git push --tags
# 然后将最新的master 分支的代码部署到生产环境，让测试人员测试看有没问题
```



#### 后记

Git Flow 的流程大致如此，后期能想到啥再补充