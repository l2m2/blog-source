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

###  jTessBoxEditor

jTessBoxEditor是训练Tesseract的工具。[下载地址](http://vietocr.sourceforge.net/training.html)。我当前下载的版本是2.3.1。

1. 在白纸上书写了所有数字和小写字母。如下图：

   ![](/images/tesseract-ocr-3.jpg)

2. 使用[在线工具](https://cn.office-converter.com/tiff-converter)将jpg转换成tiff格式，重命名为`l2m2hw.normal.exp0.tif`。

   tiff命名规则：`${lang}.${fontname}.exp${num}`

3. 生成box文件

   ```bash
   $ tesseract --psm 6 --oem 3 l2m2hw.normal.exp0.tif l2m2hw.normal.exp0 makebox
   Tesseract Open Source OCR Engine v5.0.0-alpha.20201127 with Leptonica
   Page 1
   ```

4. 打开jTessBoxEditor, `Box Editor`->`Open`,  选择刚才生成的tif文件。

   ![](/images/tesseract-ocr-4.png)

5. 在左侧修正字符后，保存

6. 创建font_properties文件

   ```bash
   $ echo normal 0 0 0 0 0 > font_properties
   ```
   
   语法：`<fontname> <italic> <bold> <fixed> <serif> <fraktur>`
   
7. 生成tr文件

   ```bash
   $ tesseract --psm 6 --oem 3 l2m2hw.normal.exp0.tif l2m2hw.normal.exp0
    nobatch box.train
   Tesseract Open Source OCR Engine v5.0.0-alpha.20201127 with Leptonica
   Page 1
   ```

8. 生成unicharset文件

   ```bash
   $ unicharset_extractor l2m2hw.normal.exp0.box
   Extracting unicharset from box file l2m2_hw.normal.exp0.box
   Other case A of a is not in unicharset
   Other case B of b is not in unicharset
   Other case C of c is not in unicharset
   Other case D of d is not in unicharset
   Other case E of e is not in unicharset
   Other case F of f is not in unicharset
   Other case G of g is not in unicharset
   Other case H of h is not in unicharset
   Other case I of i is not in unicharset
   Other case J of j is not in unicharset
   Other case K of k is not in unicharset
   Other case L of l is not in unicharset
   Other case M of m is not in unicharset
   Other case N of n is not in unicharset
   Other case O of o is not in unicharset
   Other case P of p is not in unicharset
   Other case Q of q is not in unicharset
   Other case R of r is not in unicharset
   Other case S of s is not in unicharset
   Other case T of t is not in unicharset
   Other case V of v is not in unicharset
   Other case W of w is not in unicharset
   Other case U of u is not in unicharset
   Other case X of x is not in unicharset
   Other case Y of y is not in unicharset
   Other case Z of z is not in unicharset
   Wrote unicharset file unicharset
   ```

9. 生成聚集字符特征文件

   ```bash
   $ mftraining -F font_properties -U unicharset -O l2m2hw.normal.exp0.tr
   
   Read shape table shapetable of 0 shapes
   Warning: no protos/configs for Joined in CreateIntTemplates()
   Warning: no protos/configs for |Broken|0|1 in CreateIntTemplates()
   Warning: no protos/configs for 1 in CreateIntTemplates()
   Warning: no protos/configs for 2 in CreateIntTemplates()
   Warning: no protos/configs for 3 in CreateIntTemplates()
   Warning: no protos/configs for 4 in CreateIntTemplates()
   Warning: no protos/configs for 5 in CreateIntTemplates()
   Warning: no protos/configs for 6 in CreateIntTemplates()
   Warning: no protos/configs for 7 in CreateIntTemplates()
   Warning: no protos/configs for 9 in CreateIntTemplates()
   Warning: no protos/configs for 0 in CreateIntTemplates()
   Warning: no protos/configs for a in CreateIntTemplates()
   Warning: no protos/configs for b in CreateIntTemplates()
   Warning: no protos/configs for c in CreateIntTemplates()
   Warning: no protos/configs for d in CreateIntTemplates()
   Warning: no protos/configs for e in CreateIntTemplates()
   Warning: no protos/configs for f in CreateIntTemplates()
   Warning: no protos/configs for g in CreateIntTemplates()
   Warning: no protos/configs for h in CreateIntTemplates()
   Warning: no protos/configs for i in CreateIntTemplates()
   Warning: no protos/configs for j in CreateIntTemplates()
   Warning: no protos/configs for k in CreateIntTemplates()
   Warning: no protos/configs for l in CreateIntTemplates()
   Warning: no protos/configs for m in CreateIntTemplates()
   Warning: no protos/configs for n in CreateIntTemplates()
   Warning: no protos/configs for o in CreateIntTemplates()
   Warning: no protos/configs for p in CreateIntTemplates()
   Warning: no protos/configs for q in CreateIntTemplates()
   Warning: no protos/configs for r in CreateIntTemplates()
   Warning: no protos/configs for s in CreateIntTemplates()
   Warning: no protos/configs for t in CreateIntTemplates()
   Warning: no protos/configs for v in CreateIntTemplates()
   Warning: no protos/configs for w in CreateIntTemplates()
   Warning: no protos/configs for u in CreateIntTemplates()
   Warning: no protos/configs for x in CreateIntTemplates()
   Warning: no protos/configs for y in CreateIntTemplates()
   Warning: no protos/configs for z in CreateIntTemplates()
   Done!
   ```

10. 生成字符正常化特征文件

    ```
    
    ```

    

## Reference

- https://en.wikipedia.org/wiki/Optical_character_recognition
- https://tesseract-ocr.github.io/tessdoc/
- https://www.jianshu.com/p/3326c7216696
- https://towardsdatascience.com/simple-ocr-with-tesseract-a4341e4564b6

