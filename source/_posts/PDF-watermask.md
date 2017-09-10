---
title: 利用iTextSharp向pdf文件中添加图片水印
categories:
  - Program
tags:
  - C# pdf
date: 2017-09-10 11:18:30
---

由于某些需要，我需要向pdf文件中插入图片水印，经过往上搜索发现iTextSharp这个文件操作库就可以实现我的需求。其实向pdf文件中插入水印就是重新读取pdf文件，再朝文件中写入新增的图片。

<!--more-->

怎么加载iTextSharp库就不啰嗦了，这里贴一下添加图片水印的代码，以防忘记。

```c#
			String srcPdf = "I:\\文档文件\\xxx.pdf";
            String dstPdf = "C:\\Users\\xiaoxing\\Desktop\\xxxxx.pdf";
            String imagepath = "C:\\Users\\xiaoxing\\Desktop\\xxxx\\qrcode.jpg";
            iTextSharp.text.Image img = Image.GetInstance(imagepath);
            PdfReader reader = new PdfReader(srcPdf);
            PdfStamper stamp = new PdfStamper(reader, new FileStream(dstPdf, FileMode.Create));

            PdfContentByte page;
            float width = reader.GetPageSize(1).Width;
            float height = reader.GetPageSize(1).Height;
            int num = reader.NumberOfPages;
            int margin = 100;
            img.ScalePercent(20);
            
            for (int i = 1; i <= num; ++i)
            {
                page = stamp.GetOverContent(i);

                img.SetAbsolutePosition(width-img.ScaledWidth-10,10);//设置图片插入的位置，左下角为坐标系（0，0）位置
                page.AddImage(img);
            }

            stamp.Close();
            reader.Close();
```

