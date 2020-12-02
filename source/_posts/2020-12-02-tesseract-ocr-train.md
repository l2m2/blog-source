---
title: tesseract-ocr训练
toc: false
date: 2020-12-02 10:02:00
description: Windows下安装tesseract-ocr, 训练
tags:
- OCR
---

## tesseract-ocr

**OCR** : 光学字符识别(Optical character recognition)，是将打字，手写或印刷的文本图像转换成计算机文本。输入可以是扫描的文档，文档的照片，还是场景的照片（身份证照片、驾驶证、广告牌）。

**Tesseract**：开源的OCR引擎，在[Apache 2.0协议](http://www.apache.org/licenses/LICENSE-2.0)下可用。

## 安装

[下载地址](https://github.com/UB-Mannheim/tesseract/wiki)，我当前下载的版本是`tesseract-ocr-w64-setup-v5.0.0-alpha.20201127.exe`，按照步骤安装就行。我的安装路径是`E:\Tesseract-OCR`。

**用命令行工具测试一下**

```bash
$ tesseract tesseract-ocr-1.png tesseract-ocr-1.output
Tesseract Open Source OCR Engine v5.0.0-alpha.20201127 with Leptonica
Warning: Invalid resolution 0 dpi. Using 70 instead.
Estimating resolution as 108
```

我输入的图片是

![](/images/tesseract-ocr-1.png)

输出的结果 tesseract-ocr-1.output.txt中的文本是

```
tesseract-ocr
Hello World!
```

可以添加`-l `选项指定语言，如果不指定，默认是eng.

默认安装后，语言列表仅有eng和osd。

```bash
$ tesseract --list-langs
List of available languages (2):
eng
osd
```

## 训练

自己在笔记本上手写数字和字母，然后再试试。

```bash
$ tesseract tesseract-ocr-2.jpg tesseract-ocr-2.output
```

我输入的图片是

![](/images/tesseract-ocr-2.jpg)

输出的结果 tesseract-ocr-2.output.txt中的文本（用记事本打开）是

![](/images/tesseract-ocr-output-2.png)

没识别，所以我们需要训练我手写的数字和字母库。



## Reference

- https://en.wikipedia.org/wiki/Optical_character_recognition
- https://tesseract-ocr.github.io/tessdoc/