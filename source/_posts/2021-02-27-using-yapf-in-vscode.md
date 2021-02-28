---
title: VS Code中使用yapf对Python代码进行格式化
toc: false
date: 2021-02-27 15:51:00
description: ...
tags:
- VS Code
- Python
---

## yapf

yapf是一个Python代码的格式化工具。

**安装**

```bash
$ pip install -U yapf
```

或者在使用快捷键（command + K + F）进行格式化时会有如下图所示的提醒.

![](/images/yapf-01.png)

然后选择安装`Use yapf`

## 在VS Code中配置

我们可以在VS Code的全局Settings中配置，也可以在当前工程的.vscode/setttings.json中配置。

下面演示的是在当前工程中配置。

`.vscode/settings.json`

```json
{ 
  "[python]": {
    "editor.formatOnSave": true
  },
  "python.formatting.provider": "yapf"
}
```

其中editor.formatOnSave是指在保存时便会自动格式化。

## 配置yapf

在项目的根目录中添加一个yapf的自定义配置文件`.style.yapf`来进行style定制。

`.style.yapf`

```
[style]
based_on_style = pep8
INDENT_WIDTH = 2
CONTINUATION_INDENT_WIDTH = 2
```

更多参数可以在[这里](https://github.com/google/yapf/blob/main/yapf/yapflib/style.py#L445)查阅。

配置文件用法的详细说明：

```
YAPF will search for the formatting style in the following manner:
Specified on the command line
In the [style] section of a .style.yapf file in either the current directory or one of its parent directories.
In the [yapf] section of a setup.cfg file in either the current directory or one of its parent directories.
In the [style] section of a ~/.config/yapf/style file in your home directory.
```

## Reference

- https://github.com/google/yapf