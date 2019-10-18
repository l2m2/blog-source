---
title: 使用requirement.txt简化Python项目包的安装
toc: false
date: 2019-10-18 13:08:22
description: 运行新的项目时，为了避免逐个包的安装，使用requirement.txt
tags:
- Python
---

1. 在Python项目中生成requirement.txt

   在Python的工作目录下运行 `pip freeze`

   ```bash
   $ pip freeze > requirement.txt
   ```

2. 根据requirement.txt安装Python包

   ```bash
   $ pip install -r requirement.txt
   ```

   