## macOS 安装Mongodb的PHP扩展



本文涉及的操作系统以及软件版本为：

1. macOs High Sierra 10.13.6

  		2. PHP 7.1.16
  		3. Apache/2.4.33

PHP与Apache都是 macOS 系统自带的



#### 关于```PECL```

 一、简介

> ### What is PECL?
>
> PECL is a repository for PHP Extensions, providing a directory of all known extensions and hosting facilities for downloading and development of PHP extensions.
>
> The packaging and distribution system used by PECL is shared with its sister, PEAR.

简单的说就是PHP扩展仓库，为PHP扩展提供托管和下载服务。

通过 ```PEAR``` 可以对 ```PECL``` 扩展进行下载和安装。

二、安装 ```PEAR```

在命令行输入以下命令下载 ```go-pear.phar``` 文件以及安装

```zsh
# 下载go-pear.phar
$ curl -O https://pear.php.net/go-pear.phar
# 安装
$ sudo php -d detect_unicode=0 go-pear.phar
```

在安装过程中需要进行一下简单的配置

首先修改安装根目录(Installation base)

- 输入1，按回车
- 输入 ```/usr/local/pear```
- 按回车

然后修改二进制文件的目录

- 输入4，按回车
- 输入 ```/usr/local/bin```
- 按回车

修改完以上这两项后，按回车继续安装。

以上安装步骤参考的官方文档，其它平台的安装方式亦可参照官方文档 → [传送门](https://pear.php.net/manual/en/installation.getting.php)

三、检测是否安装成功

在命令行输入 ```pear version``` 如果出现以下结果，则安装成功了。

```zsh
PEAR Version: 1.10.9
PHP Version: 7.1.16
Zend Engine Version: 3.1.0
```



#### 使用 ```PCEL``` 安装 mongodb 扩展

在命令行里输入以下命令安装 mongodb 扩展

```zsh
$ sudo pecl install mongodb
```

我在执行此命令的时候出现了两个错误

其中一个如下：

```zsh 
Cannot find autoconf. Please check your autoconf installation and the
$PHP_AUTOCONF environment variable. Then, rerun this script.
```

遇到这个是没有安装 autoconf，可以使用 Homebrew 安装

```zsh
$ brew install autoconf
```

第二个错误类似如下（出错时没有记录下来，跟以下大致相同）：

```zsh
/usr/include/php/ext/pcre/php_pcre.h:29:10: fatal error: 'pcre.h' file not found
#include "pcre.h"
         ^
1 error generated.
```

遇到这个是没有安装 pcre，可以使用 Homebrew 安装

```zsh
$ brew install pcre
```

可以使用 ```brew list``` 查看以上的依赖是否安装成功

上面两个错误解决后，再次执行命令

```zsh
$ sudo pecl install mongodb
```

等一会mongodb扩展就会安装编译完毕，想要查看是否安装成功，可以输入以下命令

```zsh
$ pecl list
Installed packages, channel pecl.php.net:
=========================================
Package Version State
mongodb 1.5.3   stable
```

然后输入一下命令查看 `mongodb.so` 的路径，记住这个路径

```zsh
$ pecl list-files mongodb | grep mongodb.so
src  /usr/lib/php/extensions/no-debug-non-zts-20160303/mongodb.so
```

然后编辑 `php.ini` 文件，在最后另起一行加上

```extension="/usr/lib/php/extensions/no-debug-non-zts-20160303/mongodb.so"```

上面的路径对应你自己的路径(就是前面说的需要记住的那个路径)

然后重启 `apache` 

在命令行输入 `php -i | grep mongodb` 就能看到已经加载mongodb的扩展了



至此，所以的步骤都已完成。



#### 后记

这里说的只是 mongodb 的扩展安装，其它的 PHP扩展也可以使用 ```pecl``` 来安装