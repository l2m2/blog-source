---
title: MacOS玩Python3的正确姿势
toc: true
date: 2020-03-31 22:26:00
description: 使用pyenv + virtualenvwrapper
tags:
- Python
- MacOS
---

## Pyenv

**核心原则**

> "The basic premise of all Python development is to never use the system Python. You do not *want* the Mac OS X 'default Python' to be 'python3.' You want to never care about default Python."

Python开发的前提是**永远不用系统自带的Python!** 

永远不必关心默认Python是啥！

**使用pyenv**

pyenv 可以帮助你在开发或者生产环境里安装和管理多个 Python 版本。

安装

```bash
$ brew install pyenv
```

使用pyenv安装Python3.7.5

```bash
$ pyenv install 3.7.5
```

现在，通过pyenv安装了Python 3，我们将其设置为pyenv环境的全局默认版本。

```bash
$ pyenv global 3.7.5
$ pyenv version
3.7.5 (set by /Users/l2m2/.pyenv/version)
```

我们需要将以下内容添加到`.bash_profile`

```bash
$ echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bash_profile
```
让`.bash_profile`生效，并测试当前的python版本
```bash
$ source .bash_profile 
$ which python
/Users/l2m2/.pyenv/shims/python
$ python -V
Python 3.7.5
$ pip -V
pip 19.2.3 from /Users/l2m2/.pyenv/versions/3.7.5/lib/python3.7/site-packages/pip (python 3.7)

```

我们通过上面的方式管理多个python版本，并按照自己的需要设置默认的python版本。

## virtualenvwrapper

顾名思义，这货就是大名鼎鼎的`virtualenv`的包装，用于配置Python虚拟环境。使用虚拟环境可以更好的隔离每个项目各自的包依赖，更靠谱。

安装

```bash
$ pyenv global 3.7.5
# Be sure to keep the $() syntax in this command so it can evaluate
$ $(pyenv which python3) -m pip install virtualenvwrapper
```

配置

```bash
$ echo 'export WORKON_HOME=~/.virtualenvs' >> .bash_profile
$ echo 'mkdir -p $WORKON_HOME' >> .bash_profile
$ echo '. ~/.pyenv/versions/3.7.5/bin/virtualenvwrapper.sh' >> .bash_profile
```

打开一个新的终端，或者在当前终端执行`exec /bin/bash -l`, 可以看到`virtualenvwrapper`在初始化环境

```bash
$ exec /bin/bash -l
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/premkproject
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/postmkproject
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/initialize
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/premkvirtualenv
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/postmkvirtualenv
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/prermvirtualenv
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/postrmvirtualenv
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/predeactivate
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/postdeactivate
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/preactivate
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/postactivate
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/get_env_details
```

现在可以愉快的玩耍虚拟环境了

创建一个虚拟环境testenv1

```bash
$ mkvirtualenv testenv1
created virtual environment CPython3.7.5.final.0-64 in 344ms
  creator CPython3Posix(dest=/Users/l2m2/.virtualenvs/testenv1, clear=False, global=False)
  seeder FromAppData(download=False, pip=latest, setuptools=latest, wheel=latest, via=copy, app_data_dir=/Users/l2m2/Library/Application Support/virtualenv/seed-app-data/v1.0.1)
  activators BashActivator,CShellActivator,FishActivator,PowerShellActivator,PythonActivator,XonshActivator
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/testenv1/bin/predeactivate
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/testenv1/bin/postdeactivate
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/testenv1/bin/preactivate
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/testenv1/bin/postactivate
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/testenv1/bin/get_env_details
(testenv1) $ 
```

退出虚拟环境testenv1

```bash
(testenv1) $ deactivate
```

切换虚拟环境

```bash
(testenv2) $ workon testenv1
(testenv1) $ 
```

删除虚拟环境

```bash
$ rmvirtualenv testenv2
```

## 最佳实践

用你的项目名称作为虚拟环境的名称。

为了简化命令，我们在`~/.bashrc`中创建三个函数。

```bash
mkv() {
  mkvirtualenv $(basename $(pwd))
}
ev() {
  deactivate
}
rv() {
  rmvirtualenv $(basename $(pwd))
}
```
然后进入项目所在目录执行mkv，便会创建以此项目为名的虚拟环境
```bash
$ cd django-demo1
$ mkv
created virtual environment CPython3.7.5.final.0-64 in 293ms
  creator CPython3Posix(dest=/Users/l2m2/.virtualenvs/django-demo1, clear=False, global=False)
  seeder FromAppData(download=False, pip=latest, setuptools=latest, wheel=latest, via=copy, app_data_dir=/Users/l2m2/Library/Application Support/virtualenv/seed-app-data/v1.0.1)
  activators BashActivator,CShellActivator,FishActivator,PowerShellActivator,PythonActivator,XonshActivator
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/django-demo1/bin/predeactivate
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/django-demo1/bin/postdeactivate
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/django-demo1/bin/preactivate
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/django-demo1/bin/postactivate
virtualenvwrapper.user_scripts creating /Users/l2m2/.virtualenvs/django-demo1/bin/get_env_details
(django-demo1) $ workon
django-demo1
(django-demo1) $ ev
```

这样下次你进入项目启用虚拟环境。只需要：

```bash
$ cd django-demo1/
$ workon .
(django-demo1) $ 
```

当然，你可以安全的删除当前的虚拟环境。

```bash
$ cd django-demo1/
$ rv
```

## Reference

- https://opensource.com/article/19/5/python-3-default-mac
- https://opensource.com/article/19/6/python-virtual-environments-mac

