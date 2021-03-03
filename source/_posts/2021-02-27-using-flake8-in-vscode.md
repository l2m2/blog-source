---
title: VS Code中使用flake8对Python进行静态代码检查
toc: false
date: 2021-02-27 16:51:00
description: ...
tags:
- VS Code
- Python
---

## flake8

flake8是一个Python代码的静态检查工具。

**安装**

```bash
$ pip install -U flake8
```

## 在VS Code中配置

使用弹出命令窗口的快捷键（Command + Shift + P）,

![](/images/flake8-01.png)

然后选择flake8

![](/images/flake8-02.png)

`.vscode/settings.json`中将会自动添加如下配置

```json
{
  "python.linting.flake8Enabled": true,
  "python.linting.enabled": true
}
```

## flake8配置

在项目的根目录添加一个flake8的配置文件`.flake8`

```
[flake8]
exclude = .git,__pycache__,__init__.py,.mypy_cache,.pytest_cache
ignore =
  # E111: indentation is not a multiple of four
  E111,
  # E114: indentation is not a multiple of four (comment)
  E114,
  # E121: continuation line under-indented for hanging indent
  E121,
per-file-ignores = 
  # imported but unused
  base.py: F401
  __init__.py: F401
```

ignore是指需要忽略的PEP8规则。在我的项目中，缩进是两个空格，因此忽略了E111和E121。

所有的错误代码可以在[这里](https://pep8.readthedocs.io/en/release-1.7.x/intro.html#error-codes)找到。

配置文件的详细说明：

```
Flake8 supports storing its configuration in the following places:

Your top-level user directory
In your project in one of setup.cfg, tox.ini, or .flake8.
```

我们可以在per-file-ignores中配置需要忽略的错误，也可以直接在源码中通过注释忽略。

```python
from app.db import base  # noqa: F401
```

## Reference

- https://flake8.pycqa.org/

