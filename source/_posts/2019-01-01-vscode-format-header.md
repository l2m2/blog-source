---
title: VS Code自动添加文件头注释
toc: false
date: 2019-01-01 16:11:38
description: 更方便的添加文件头注释
tags:
- VS Code
---
- 添加VS Code扩展 `psioniq File Header`

  直接在扩展商店中搜索安装即可。
- 按照自己的要求修改扩展的配置
```yaml
    "psi-header.config": {
        "author": "leon.li",
        "authorEmail": "l2m2lq@gmail.com"
    },
    "psi-header.templates": [
        {
            "language": "*",
            "template": [
                "@File: <<filename>>",
                "@Author: <<author>>(<<authoremail>>)",
                "@Date: <<filecreated('YYYY-MM-DD hh:mm:ss')>>",
                "@Last Modified By: <<author>>(<<authoremail>>>)",
                "@Last Modified Time: <<dateformat('YYYY-MM-DD hh:mm:ss')>>"
            ]
        }
    ]
```
- 使用

  F1 -> Header Insert
  
  或者
  
  快捷键（默认的是按两次Ctrl + Alt + H ）