---
title: .bash_profile, .bashrc, .bash_login, .profile, .bash_logout的执行顺序
toc: false
date: 2020-01-17 11:07:56
description: 又是Linux菜逼愉快的一天。~_~
tags:
- Linux
---

它们是一个个[shell脚本](https://en.wikipedia.org/wiki/Shell_script)，这些文件包括：

- /etc/profile
- /etc/bashrc
- ~/.bash_profile
- ~/.bashrc
- ~/.bash_login
- ~/.profile
- ~/.bash_logout

## interactive login shell的执行顺序

下面是伪代码：

```shell
execute /etc/profile
IF ~/.bash_profile exists THEN
    execute ~/.bash_profile
ELSE
    IF ~/.bash_login exist THEN
        execute ~/.bash_login
    ELSE
        IF ~/.profile exist THEN
            execute ~/.profile
        END IF
    END IF
END IF
```

当登出交互式shell时，执行顺序是

```shell
IF ~/.bash_logout exists THEN
    execute ~/.bash_logout
END IF
```

**Note**: /etc/bashrc是在~/.bashrc中被执行的

```shell
# cat ~/.bashrc
if [ -f /etc/bashrc ]; then
. /etc/bashrc
fi
```

## interactive no-login shell的执行顺序 

 启动非登录交互式shell时，以下是执行顺序 

```shell
IF ~/.bashrc exists THEN
    execute ~/.bashrc
END IF
```

## 测试

我们用[PS1](https://wiki.archlinux.org/index.php/Bash/Prompt_customization)来测试执行顺序。

**/etc/profile**被执行

添加下面的代码到/etc/profile， 然后重新登录。

```bash
[leon@192 ~]$ grep PS1 /etc/profile
PS1="/etc/profile> "
[leon@192 ~]$ su - leon
Password: 
Last login: Sat Jan  4 04:14:53 CST 2020 on pts/0
/etc/profile> ls
```

**~/.bash_profile**和**~/.bashrc**被执行

同样的，我们把这行代码添加到~/.bash_profile, ~/bash_login, ~/.profile和~/.bashrc中，然后再重新登录。

```bash
export PS1="~/.bash_profile> "
[leon@192 ~]$ grep PS1 ~/.bashrc 
export PS1="~/.bashrc> "
[leon@192 ~]$ grep PS1 ~/.bash_login
export PS1="~/.bash_login> "
[leon@192 ~]$ grep PS1 ~/.profile
export PS1="~/.profile> "
[leon@192 ~]$ su leon
Password: 
~/.bashrc> su - leon
Password: 
Last login: Sat Jan  4 04:25:35 CST 2020 on pts/0
~/.bash_profile> 
```

可以看到，如果shell是非登录交互式shell时，显示的是~/.bashrc。而如果shell是登录交互式shell时，显示的是~/.bash_profile。

**~/.bash_login**被执行

把~/.bash_profile重命名为其他名字，再重新登录。

```bash
~/.bash_profile> mv .bash_profile .bash_profile.bak
~/.bash_profile> su - leon
Password: 
Last login: Sat Jan  4 04:25:45 CST 2020 on pts/0
~/.bash_login> 
```

**~/.profile**被执行

把~/.bash_login重命名为其他名字，再重新登录。

```bash
~/.bash_login> mv .bash_login .bash_login.bak
~/.bash_login> su - leon
Password: 
Last login: Sat Jan  4 04:35:47 CST 2020 on pts/0
~/.profile> 
```

## Reference

-  https://www.thegeekstuff.com/2008/10/execution-sequence-for-bash_profile-bashrc-bash_login-profile-and-bash_logout/ 

-  https://unix.stackexchange.com/questions/129143/what-is-the-purpose-of-bashrc-and-how-does-it-work
-  https://unix.stackexchange.com/questions/38175/difference-between-login-shell-and-non-login-shell 