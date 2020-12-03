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

[下载地址](https://github.com/UB-Mannheim/tesseract/wiki)，我当前下载的版本是`tesseract-ocr-w64-setup-v4.1.0.20190314.exe`，按照步骤安装就行。我的安装路径是`E:\Tesseract-OCR`。

**用命令行工具测试一下**

```bash
$ tesseract tesseract-ocr-1.png tesseract-ocr-1.output
Tesseract Open Source OCR Engine v4.0.0.20190314 with Leptonica
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

可以直接输出到stdout

```bash
$ tesseract tesseract-ocr-1.png stdout
tesseract-ocr
Hello World!

Warning: Invalid resolution 0 dpi. Using 70 instead.
Estimating resolution as 108
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

我们有如下一张图片，进行识别

![](/images/tesseract-ocr-2.png)

```bash
$ tesseract tesseract-ocr-2.png stdout
40293847 S565647386291L0

Warning: Invalid resolution 0 dpi. Using 70 instead.
Estimating resolution as 147

```

识别的结果显然不是我们想要的，接下来我们对它进行训练。

2. 使用[在线工具](https://cn.office-converter.com/tiff-converter)将jpg转换成tiff格式，重命名为`l2m2hw.normal.exp0.tif`。

   tiff命名规则：`${lang}.${fontname}.exp${num}`

2. 生成box文件

   ```bash
   $ tesseract --psm 7 --oem 3 l2m2hw.normal.exp0.tif l2m2hw.normal.exp0 makebox
   Tesseract Open Source OCR Engine v4.0.0.20190314 with Leptonica
   Page 1
   Warning: Invalid resolution 0 dpi. Using 70 instead.
   ```

   通过帮助命令可以查看psm和oem参数的可选项及含义

   ```bash
   $ tesseract --help-psm
   Page segmentation modes:
     0    Orientation and script detection (OSD) only.
     1    Automatic page segmentation with OSD.
     2    Automatic page segmentation, but no OSD, or OCR. (not implemented)
     3    Fully automatic page segmentation, but no OSD. (Default)
     4    Assume a single column of text of variable sizes.
     5    Assume a single uniform block of vertically aligned text.
     6    Assume a single uniform block of text.
     7    Treat the image as a single text line.
     8    Treat the image as a single word.
     9    Treat the image as a single word in a circle.
    10    Treat the image as a single character.
    11    Sparse text. Find as much text as possible in no particular order.
    12    Sparse text with OSD.
    13    Raw line. Treat the image as a single text line,
          bypassing hacks that are Tesseract-specific.
   $ tesseract --help-oem
   OCR Engine modes:
     0    Legacy engine only.
     1    Neural nets LSTM engine only.
     2    Legacy + LSTM engines.
     3    Default, based on what is available.
   ```

3. 打开jTessBoxEditor (运行E:\jTessBoxEditor\training.bat）, `Box Editor`->`Open`,  选择刚才生成的tif文件。

   jTessBoxEditor是训练Tesseract的辅助工具。[下载地址](http://vietocr.sourceforge.net/training.html)。我当前下载的版本是2.3.1。

   ![](/images/tesseract-ocr-4.png)

4. 在左侧修正字符后，保存

5. 创建font_properties文件

   ```bash
   $ echo normal 0 0 0 0 0 > font_properties
   ```

   语法：`<fontname> <italic> <bold> <fixed> <serif> <fraktur>`

   到这一步，你应该有这些文件：

   ```bash
   $ ls
   font_properties  l2m2hw.normal.exp0.box  l2m2hw.normal.exp0.tif
   ```

6. 生成训练文件 .tr

   ```bash
   $ tesseract l2m2hw.normal.exp0.tif l2m2hw.normal.exp0 nobatch box.train
   Tesseract Open Source OCR Engine v4.0.0.20190314 with Leptonica
   Page 1
   Warning: Invalid resolution 0 dpi. Using 70 instead.
   Estimating resolution as 147
   APPLY_BOXES:
      Boxes read from boxfile:      20
      Found 20 good blobs.
   Generated training data for 2 words
   ```

7. 生成unicharset字符集文件

   ```bash
   $ unicharset_extractor l2m2hw.normal.exp0.box
   Extracting unicharset from box file l2m2hw.normal.exp0.box
   Wrote unicharset file unicharset
   ```

8. 生成字符特征文件 shapetable

   ```bash
   $ shapeclustering -F font_properties -U unicharset -O l2m2hw.unichaset l2m2hw.normal.exp0.tr
   Reading l2m2hw.normal.exp0.tr ...
   Building master shape table
   Computing shape distances...
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances...
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances...
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances... 0
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances... 0
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances... 0
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances... 0
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances... 0
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances... 0
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances... 0
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances... 0
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances... 0
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances... 0
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances...
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances...
   Stopped with 0 merged, min dist 999.000000
   Computing shape distances... 0 1 2 3 4 5 6 7 8 9
   Stopped with 0 merged, min dist 0.319728
   Master shape_table:Number of shapes = 10 max unichars = 1 number with multiple unichars = 0
   
   ```

9. 生成特征字符文件 pffmtable，intemp

   ```bash
   $ mftraining -F font_properties -U unicharset -O l2m2hw.unichaset l2m2hw.normal.exp0.tr
   Done!
   Read shape table shapetable of 10 shapes
   Reading l2m2hw.normal.exp0.tr ...
   Warning: no protos/configs for Joined in CreateIntTemplates()
   Warning: no protos/configs for |Broken|0|1 in CreateIntTemplates()
   ```

10. 生成字符正常化特征文件 normproto

    ```bash
    $ cntraining l2m2hw.normal.exp0.tr
    Reading l2m2hw.normal.exp0.tr ...
    Clustering ...
    
    Writing normproto ...
    ```

    到这一步，你有这些文件了。

    ```bash
    $ ls
    font_properties         l2m2hw.normal.exp0.tif  normproto   unicharset
    inttemp                 l2m2hw.normal.exp0.tr   pffmtable
    l2m2hw.normal.exp0.box  l2m2hw.unichaset        shapetable
    ```

11. 将4个字符特征文件重命名

    ```bash
    $ mv inttemp l2m2hw.inttemp
    $ mv normproto l2m2hw.normproto
    $ mv pffmtable l2m2hw.pffmtable
    $ mv shapetable l2m2hw.shapetable
    ```

12. 合并训练文件

    ```bash
    $ combine_tessdata l2m2hw.
    ```

## Reference

- https://en.wikipedia.org/wiki/Optical_character_recognition
- https://tesseract-ocr.github.io/tessdoc/
- https://towardsdatascience.com/simple-ocr-with-tesseract-a4341e4564b6

