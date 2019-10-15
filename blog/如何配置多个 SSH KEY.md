### 如何配置多个 SSH KEY

本文涉及的操作系统为：```macOs High Sierra 10.13.6```



#### 1、为什么需要配置多个 SSH KEY

公司的项目是托管在腾讯的 GIT 服务上的，而我还需要用到 github ，此时就需要配置两个 SSH KEY，如果还有其他的代码托管服务的，也就还需要配置更多的 SSH KEY，因此写下这篇文章说明如何配置多个 SSH KEY。



#### 2、如何配置多个 SSH KEY

- 使用命令生成多套 SSH KEY

  ```zsh
  $ ssh-keygen -t rsa -C "your_email@example.com" -f ~/.ssh/id_rsa_example
  ```

  例如我现在需要生成两个 SSH KEY，假设邮箱分别是 ```tencent@company.com``` 和 ```liam@private.com``` ，在 ```~/.ssh``` 目录下生成两队公私钥，分别命名为 ```id_rsa_company``` 和 ```id_rsa_private``` ，则敲的命令如下

  ```zsh
  # 生成第一对公私钥
  $ ssh-keygen -t rsa -C "tencent@company.com" -f ~/.ssh/id_rsa_company
  # 生成第二对公私钥
  $ ssh-keygen -t rsa -C "liam@private.com" -f ~/.ssh/id_rsa_private
  ```

  执行完上面的程序后，就会看到 ```~/.ssh``` 目录下会对了四个文件

  ```zsh
  id_rsa_company.pub       id_rsa_private.pub       id_rsa_company      id_rsa_private
  ```

- 添加 ```config``` 配置文件，指明不同的 SSH KEY 和不同的应用之间的映射关系

  在 ```~/.ssh``` 目录下，新建 `config` 文件，并添加一下内容（我这里以我上面生成的两个 SSH KEY 作为例子，对应的内容要替换成你自己生成 SSH KEY ）

  ```zsh
  ###############
  #  tgit的公钥  #
  ###############
  Host git.cloud.tencent.com
  HostName git.cloud.tencent.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa_company
  
  ###############
  # github的公钥 #
  ###############
  Host github.com
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa_private
  ```

- 然后将以上配置的映射关系的公钥添加到网站应用里面就大功告成了