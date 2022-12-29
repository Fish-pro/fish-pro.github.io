---
layout: post
title: python PIL分享图片合成
author: Fish-pro
tags:
- python
- PIL
date: 2019-08-11 13:56 +0800
---
笔者在开发过程中，遇到了这样的需求，需要将合成如下图所示的分享图，用户可以点击图片保存分享，其中红色框内的数据都是变化的

![其中红色框内的数据都是变化的，二维码携带用户和商品信息](https://img-blog.csdnimg.cn/20191216172454902.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzU2ODA2MA==,size_16,color_FFFFFF,t_70)
笔者在查阅资料后发现：阿里云oss图片可以添加水印对图片进行处理，最多可支持三个图片水印，所有的图片都要求在同一个buket下，而笔者开发系统有拉取微信头像地址，不是企业oss的buket，如果要使用还得上传到buket，并且阿里云oss图片水印明确指出，不支持水印图片内切圆，所以不满足项目需求。
最后决定自己本地合成处理图片，目前该版本未经高并发等专业测试，所以有很多不足之处，如果路过的你有更好的解决方案，希望能多多指教帮助成长。
我的做法是使用python 的PIL库实现图片的处理，在本地合成图片后将图片上传至oss，然后用oss处理图片圆角，以下为相关处理方法
```
import requests
from PIL import Image, ImageFont, ImageDraw, ImageFilter
import qrcode

def download_image(url, name):
    """下载图片到本地"""
    r = requests.get(url)
    with open(name, 'wb') as f:
        f.write(r.content)
        
def circle_new(path):
	"""头像内切圆"""
    ima = Image.open(path).convert("RGBA")
    ima = ima.resize((72, 72))
    r2 = 72
    circle = Image.new('1', (r2, r2), 0)
    draw = ImageDraw.Draw(circle)
    draw.ellipse((0, 0, r2, r2), fill=(255))
    alpha = Image.new('RGBA', (r2, r2), (255, 255, 255, 255))
    alpha = alpha.convert("1")
    alpha.paste(circle, (0, 0))
    ima.putalpha(alpha)  # 蒙层图片
    # ima.show()
    # ima.save(r".\head.png")
    return ima

def create_code(url, QRcenter=r".\QRcenter.jpg"):
	“”合成二维码“”“”
    qr = qrcode.QRCode(
        version=5, error_correction=qrcode.constants.ERROR_CORRECT_H, box_size=8, border=4)
    qr.add_data(url)
    qr.make(fit=True)

    img = qr.make_image()
    img = img.convert("RGBA")
    icon = Image.open(QRcenter)  # 这里是二维码中心的图片

    img_w, img_h = img.size
    factor = 4
    size_w = int(img_w / factor)
    size_h = int(img_h / factor)

    icon_w, icon_h = icon.size
    if icon_w > size_w:
        icon_w = size_w
    if icon_h > size_h:
        icon_h = size_h
    icon = icon.resize((icon_w, icon_h), Image.ANTIALIAS)

    w = int((img_w - icon_w) / 2)
    h = int((img_h - icon_h) / 2)
    icon = icon.convert("RGBA")
    img.paste(icon, (w, h), icon)
    # img.show()  # 显示图片,可以通过save保存
    return img

def image_tool(urlName=None, url=None, centerUrl=None, price=None, nickName=None, fontPath=None, logoPath=None,
              coursePath=None, avatarPath=None):
    target = Image.new('RGBA', (600, 696), "white")

    # 顶部宣传图
    box = (0, 0, 600, 439)  # 区域
    region = Image.open(coursePath)
    region = region.convert("RGBA")
    region = region.resize((600, 439))

    # 企业logo
    box1 = (0, 626, 600, 696)
    region1 = Image.open(logoPath)
    region1 = region1.convert("RGBA")
    region1 = region1.resize((600, 70))

    # 用户头像
    box2 = (20, 530, 92, 602)
    region2 = circle_new(avatarPath)
    region2 = region2.resize((72, 72))

    # 二维码
    box3 = (472, 461, 580, 569)
    region3 = create_code(url, centerUrl)
    region3 = region3.convert("RGBA")
    region3 = region3.resize((108, 108))

    # 粘贴图片
    target.paste(region, box)
    target.paste(region1, box1)
    target.paste(region2, box2)
    target.paste(region3, box3)

    # 写入介绍
    draw = ImageDraw.Draw(target)
    font = ImageFont.truetype(fontPath, 30)
    # (fl, il) = math.modf(price)
    # priceList = [str(il), str(int(fl * 10))]
    draw.text((20, 456), "成功邀请后，将获得", "#333333", font)
    draw.text((293, 456), str(price) + "元收益", "#FFCC33", font)

    # 写入昵称
    font1 = ImageFont.truetype(fontPath, 28)
    draw.text((101, 546), nickName + "推荐", "#333333", font1)
    # 写入固定文字
    font2 = ImageFont.truetype(fontPath, 24)
    draw.text((478, 573), "扫码购买", "#333333", font2)
    x, y = target.size
    p = Image.new('RGBA', target.size, (255, 255, 255))
    p.paste(target, (0, 0, x, y), target)
    # p.show()
    p.save(urlName)  # 保存图片


def get_res_url(backgroundUrl, avatarBase64=None, courseBase64=None, price=None, nickName=None):
	"""处理图片圆角，注释部分为最先考虑的阿里云水印写法，有兴趣的可以看一看"""
    # text_Info1 = get_base64_txt("成功邀请后，将获得")
    # text_Info2 = get_base64_txt(str(price / 100) + "元收益")
    # text_Info3 = get_base64_txt(nickName + "推荐")
    # text_Info4 = get_base64_txt("扫码购买")

    resStr = backgroundUrl
    resStr += "?x-oss-process=image/resize,w_600,h_696"
    # resStr += "/watermark,image_" + logoBase64 + ",x_0,y_637"
    resStr += "/watermark,image_" + avatarBase64 + ",x_508,y_94/rounded-corners,r_30"
    # resStr += "/watermark,image_" + courseBase64 + ",x_0,y_0"
    # resStr += "/watermark,type_d3F5LXplbmhlaQ,size_30,text_" + text_Info1 + ",color_333333,x_20,y_456"
    # resStr += "/watermark,type_d3F5LXplbmhlaQ,size_30,text_" + text_Info2 + ",color_FFCC33,x_293,y_456"
    # resStr += "/watermark,type_d3F5LXplbmhlaQ,size_28,text_" + text_Info3 + ",color_333333,x_101,y_546"
    # resStr += "/watermark,type_d3F5LXplbmhlaQ,size_24,text_" + text_Info4 + ",color_333333,x_478,y_573"
    return resStr


if __name__ == "__main__":
    code_tool()
```
由于涉及大量的io操作，第一次分享图片合成很慢，但是可以用数据库记录用户的分享，如果一旦合成就数据库记录，第二次直接返回信息。但是这样会出现的问题是，如果用户修改信息或者分享的商品信息修改，那么分享图信息依然不会修改，这一点笔者目前还没有想到很好的方法。
本地操作需要将图片拉取到本地存储，然后PIL才能操作图片，所以会有图片命名的处理，要求所有图片不重名，并且图片使用完后要将图片删除，不然图片会存储在服务器，导致服务器压力。

> 综上考虑：其实分享图合成如果只包含少部分信息，那么阿里云是最好的选择，只需要转码拼接就行了。但是如果出现内切圆这样的需求就处理不了了，可以尝试get图片到本地再上传到自身buket下，或者注册时直接存储圆形头像，但是处理都比较麻烦。如果你有更好的解决方法，希望留言互相学习，谢谢~~