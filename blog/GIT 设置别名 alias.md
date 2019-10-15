#### GIT 设置别名 alias

本文涉及的操作系统为：```macOs High Sierra 10.13.6```



使用 git 命令的时候，有些命令可以使用别名替代，提高工作效率

下面我就说一下如何设置别名已经我常用的别名

#### 设置别名的方法

#####  - 编辑配置文件设置别名

我的系统的 git 的配置文件为 `~/.gitconfig` ， 编辑该配置文件

```zsh
# 在 [alias] 下面的就是设置的别名，=号左边的命令等价于右边的命令
[alias]
				# 比如下面一行表示：输入 st 命令时，就等于输入 status 命令
        st = status
	
```



#####  - 利用全局命令修改配置

可直接输入以下命令，设置全局配置

```zsh
# 比如设置：输入 st 命令时，就等于输入 status 命令
$ git config --global alias.st 'status'
```



#### 我常用的别名

我常用到的别名主要是一下这些

```zsh
[alias]
        st = status
        br = branch
        co = checkout
        ci = commit -m
```

