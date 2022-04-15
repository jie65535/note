---
tags: #java.awt #Graphics2D #TextRender
---

# Graphics2D 绘制文字

本文主要记录使用 `Graphics2D` 进行文本渲染相关内容。

## 参考代码

```java
package com.test.testImage;

import java.awt.Color;
import java.awt.Font;
import java.awt.FontMetrics;
import java.awt.Graphics2D;
import java.awt.RenderingHints;
import java.awt.image.BufferedImage;
import java.io.File;

import javax.imageio.ImageIO;

public class Graphics2DTest {

    public static void main(String[] args) {
        try {
            String text = "文字居中";
            int width = 500;
            int height = 400;
            // 创建BufferedImage对象
            BufferedImage image = new BufferedImage(width, height,BufferedImage.TYPE_INT_RGB);
            // 获取Graphics2D
            Graphics2D g2d = image.createGraphics();
            // 设置背景色
            g2d.setBackground(new Color(255,255,255));
            //g2d.setPaint(new Color(0,0,0));
            // 设置前景色
            g2d.setColor(Color.red);
            // 绘制填充矩形 填充背景
            g2d.clearRect(0, 0, width, height);
            // 载入字体
            Font font=new Font("宋体",Font.PLAIN,64);
            // 设置字体
            g2d.setFont(font);
            // 设置抗锯齿
            g2d.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
            // 计算文字长度，计算居中的x点坐标
            // 获取字体度量参数
            FontMetrics fm = g2d.getFontMetrics(font);
            // 测量文本长度
            int textWidth = fm.stringWidth(text);
            // 获得文本居中的起点x位置
            int widthX = (width - textWidth) / 2;
            // 表示这段文字在图片上的位置(x,y) .第一个是你设置的内容。
            g2d.drawString(text,widthX,100);
            // 释放对象
            g2d.dispose();
            // 保存文件
            ImageIO.write(image, "jpg", new File("D:/test.jpg"));
        }
        catch(Exception ex) {
            ex.printStackTrace();
        }
    }
}
```

## 图像缓冲区
```java
BufferedImage image = new BufferedImage(width, height,BufferedImage.TYPE_INT_RGB);
Graphics2D g2d = image.createGraphics();
```

## 抗锯齿
```java
g2d.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
```

## 测量文本
```java
// 获取字体度量参数
FontMetrics fm = g2d.getFontMetrics(font);
// 测量文本长度
int textWidth = fm.stringWidth(text);
```

## 导出文件
```java
ImageIO.write(image, "jpg", new File("D:/test.jpg"));
```