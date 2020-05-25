---
title: 将Excel文件转为PDF的几种方案
toc: false
date: 2020-05-25 13:37:00
description: Windows下可直接调用Office接口。Linux下可使用GOTENBERG或者Wine。
tags:
- Excel2PDF
---

最近项目中遇到了生成PDF的问题，我们选用了先生成Excel, 再将Excel文件转换成PDF的方式。使用这种方式的好处是可以让客户更方便的制作模板，这在我们的业务场景中很重要。

为了跨平台，我们选用的Excel操作库是[LibXL](https://www.libxl.com/) , 需要付费使用。

那么如何将Excel文件转换成PDF就成了我们的子问题。

## Windows 

如果客户服务器 是Windows, 我们可以直接调用Office 接口来实现。

这样做的前提是需要在使用的电脑安装好Office软件。

下面是一段C#代码示例：

```c#
Microsoft.Office.Interop.Excel.Application excelApplication;
Microsoft.Office.Interop.Excel.Workbook excelWorkbook;
excelApplication = new Microsoft.Office.Interop.Excel.Application();
excelApplication.ScreenUpdating = false;
excelApplication.DisplayAlerts = false;
excelWorkbook = excelApplication.Workbooks.Open(workbookPath);
if (excelWorkbook == null) {
    excelApplication.Quit();
    excelApplication = null;
    excelWorkbook = null;
    return false;
}
var exportSuccessful = true;
try {
    excelWorkbook.ExportAsFixedFormat(Microsoft.Office.Interop.Excel.XlFixedFormatType.xlTypePDF, outputPath);
} catch (System.Exception ex) {
    exportSuccessful = false;   
}  finally {
    excelWorkbook.Close();
    excelApplication.Quit();
    excelApplication = null;
    excelWorkbook = null;
}
```

更详细的代码在[这里](https://github.com/l2m2/excel2pdf)。

## Linux

### 使用 GOTENBERG

`GOTENBERG` 是 一个 开源项目，当前开源协议是 MIT. 它支持将 HTML, Markdown, Office文件转换成 PDF。

下面的测试以 CentOS 7 环境为例。

先拉取docker镜像。

```bash
$ docker pull thecodingmachine/gotenberg
```

再run一个容器。

```bash
$ docker run --name excel2pdf --rm -p 9182:3000 -d thecodingmachine/gotenberg
```

一个示例的Python代码：

```python
def excel2pdf(xlsx_path, pdf_path):
    url = "http://192.168.0.118:9182/convert/office"
    files = { "files" : (os.path.basename(xlsx_path), open(xlsx_path, "rb"))}
    response = requests.post(url, files=files)
    open(pdf_path, "wb").write(response.content)
```

### 使用 Wine

**Wine**是在[x86](https://zh.wikipedia.org/wiki/X86)、[x86-64](https://zh.wikipedia.org/wiki/X86-64)容许[类Unix操作系统](https://zh.wikipedia.org/wiki/类Unix系统)在[X Window System](https://zh.wikipedia.org/wiki/X_Window_System)运行[Microsoft Windows](https://zh.wikipedia.org/wiki/Microsoft_Windows)程序的软件。

此方案未测试。此处不详述。

## References

- https://www.libxl.com/
- https://hub.docker.com/r/thecodingmachine/gotenberg
- https://github.com/thecodingmachine/gotenberg
- https://phoenixnap.com/kb/how-to-install-wine-on-centos