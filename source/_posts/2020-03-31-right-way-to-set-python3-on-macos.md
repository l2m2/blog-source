---
title: MacOS玩Python3的正确姿势
toc: false
date: 2020-03-31 22:26:00
description: 推荐使用pyenv
tags:
- Python
- MacOS
---

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

## Reference

- https://opensource.com/article/19/5/python-3-default-mac
- https://opensource.com/article/19/6/python-virtual-environments-mac