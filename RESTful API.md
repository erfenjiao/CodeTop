![image-20230811093512990](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811093512990.png)



# RESTful API 是什么？

谈谈你对RESTful API的理解？

![image-20230811093722399](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811093722399.png)

web客户端与服务端之间的通信，接口规范性就成了问题

所以一套结构清晰、符合标准、易于理解、扩展方便让大部分人都能够理解接受的接口风格就显得越来越重要，而RESTful风格的接口(RESTful API)刚好有以上特点，就逐渐被实践应用而变得流行起来。

REST释义

REpresentational State Transfer 表述性的状态转移

Resource REpresentational State Transfer     资源（数据）以某种表现形式进行状态转移

表现形式：传递过程中使用的JSON XML JPEG

> REpresentational adj
>
> State                         n
>
> Transfer                   v

创建账户/修改账户

![image-20230811094014863](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811094014863.png)

传递的是删除的状态，所以是state

![image-20230811094225408](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811094225408.png)



![image-20230811094809383](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811094809383.png)

# Richardson成熟度模型

![image-20230811094953349](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811094953349.png)

模型用来评价与规范的接近程度

## 举例说明

![image-20230811095131153](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811095131153.png)

### Level 0 - 普通实现

![image-20230811095146619](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811095146619.png)

![image-20230811095337367](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811095337367.png)



### Level 1 - 面向资源

![image-20230811095514186](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811095514186.png)

卡号、订单号本身也是资源

### Level 2 - 使用HTTP 动词

![image-20230811095800377](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811095800377.png)



### Level 3 - 超媒体控制

![image-20230811100130639](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811100130639.png)

超媒体作为应用状态的引擎

通俗的理解

![image-20230811100429292](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811100429292.png)

## 优点

规范 API 设计

增强 API 可读性

方便前后端协作



## 面试题

![image-20230811100625771](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811100625771.png)

### URL 设计规范

![image-20230811100717546](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811100717546.png)

Identifer Design 标识符设计

资源ID应该作为路径的一部分，紧跟在资源名后 : /users?id=1

对于多级资源，父资源应该在子资源之前 : users/1/educations/2





![image-20230811101625886](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811101625886.png)

无状态：服务器不能保存客户端信息，每一次从客户端发送的请求中，要包含所有必须的状态信息，会话信息由客户端保存。服务器根据这些状态信息来处理请求。



从大体样式了解URL路径组成之后，对于RESTful API的URL具体设计的规范如下：

1.  不用大写字母，所有单词使用英文且小写。
2.  连字符用中杠"-"而不用下杠"_"
3.  正确使用 "/"表示层级关系,URL的层级不要过深，并且越靠前的层级应该相对越稳定
4.  结尾不要包含正斜杠分隔符"/"
5.  URL中不出现动词，用请求方式表示动作
6.  资源表示用复数不要用单数
7.  不要使用文件扩展名





### Status Code

![image-20230811103427644](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811103427644.png)

### Header 

Content-Type    请求或者响应消息体中的数据类型

Content-Length  消息体长度，字节为单位

Last-Modified      上次资源修改的日期时间改变

> 缓存时间有没有在期限时间内，如果在，直接使用缓存数据

Cache-Control      基于TTL的缓存值（以秒为单位）

### Content-Type 

![image-20230811104150224](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811104150224.png)

### Response Body

![image-20230811104441728](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811104441728.png)

![image-20230811104623610](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811104623610.png)

# 使用 RESTful API 目的

看 URL 就知道要什么

看 HTTP Method 就知道要干什么

看 HTTP Sttaus Code 就知道结果如何



# 案例

![image-20230811105014133](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811105014133.png)

![image-20230811105122812](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811105122812.png)

  ![image-20230811105336532](/home/erfenjiao/.config/Typora/typora-user-images/image-20230811105336532.png)



