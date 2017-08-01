---
layout: post
title: Python把PDF的图片提取出来做新的PDF 
---
&emsp;&emsp;海信A2墨水屏，看PDF太小。放大后刷新太慢。所以想用Python把PDF分割每次看一段
先找了大神的帖子，用PyPDF提取图像。因为我的是扫描件，所以都是灰度图。灰度图提取后用OpenCV
处理。再把处理的文件导入到WORD中，生成PDF。也尝试过用PyPDF直接生成PDF，但是发现PyPDF中没有
相关函数，后来找了好多PDF的库，基本要都是读取。所以就只能用先生成Word再生成PDF的方案了。


先提取PDF的图像
```python
import PyPDF2
import cv2
import numpy as np
import pdb
import win32com


if __name__ == '__main__':
    input1 = PyPDF2.PdfFileReader(open("d:/BaiduYunDownload/test/1.pdf", "rb"))
    pages=input1.getNumPages()
    pic_list = []
    for i in range(pages):
        page_i = input1.getPage(i)
        xObject = page_i['/Resources']['/XObject'].getObject()
        for obj in xObject:
            if xObject[obj]['/Subtype'] == '/Image':
                size = (xObject[obj]['/Width'], xObject[obj]['/Height'])
                data = xObject[obj].getData()
                if xObject[obj]['/ColorSpace'] == '/DeviceRGB':
                    mode = "RGB"
                else:
                    mode = "P"
    
                if xObject[obj]['/Filter'] == '/FlateDecode':
#                    img = Image.frombytes(mode, size, data)
#                    img.save(obj[1:] + ".png")
                     img = np.fromstring(data,np.uint8)
                     img = img.reshape(size[1],size[0])
#                     pdb.set_trace()
                     img = 255-img
                     img = img+1
                     #cut img
                     img = img[:,220:-225]
                     cv2.imwrite("page"+"%05ui"%i+"obj"+obj[1:]+".png",img)
                     pic_list.append("page"+"%05ui"%i+"obj"+obj[1:]+".png")
                     
                elif xObject[obj]['/Filter'] == '/DCTDecode':
#                    img = open(obj[1:] + ".jpg", "wb")
#                    img.write(data)
#                    img.close()
                     img = np.fromstring(data,np.uint8)
                     cv2.imwrite(obj[1:]+".jpg",img)
                elif xObject[obj]['/Filter'] == '/JPXDecode':
#                    img = open(obj[1:] + ".jp2", "wb")
#                    img.write(data)
#                    img.close()
                     img = np.fromstring(data,np.uint8)
                     cv2.imwrite(obj[1:]+".jp2",img)
		    ```


再把jng文件添加到word里
```python

import win32com
from win32com.client import Dispatch, constants
import os
import re

w = win32com.client.Dispatch('Word.Application')
w.Visible=1
doc = w.Documents.Open("d:/BaiduYunDownload/test/1.docx")
notpage=re.compile("page.*png")
j=0
a=os.listdir("d:/BaiduYunDownload/test/")
a.reverse()
for i in a:
        if notpage.match(i)!=None:
          
            doc.InlineShapes.AddPicture("d:/BaiduYunDownload/test/"+i)
          
       
doc.Save()   
```
