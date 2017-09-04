title: 凤城卫士API文档
date: 2015-11-19 00:40:58
author: Mcl
category: [web]
tags: [web,api]
---

> 本篇为凤城卫士的API文档，包括其url,接受的参数，示例url以及返回值示例。对于REST API的 设计感兴趣的可以移步[这篇文章](http://mclspace.com/2015/11/03/restful-note/)

注: 本篇中的url均具有前缀 */api/v1* ,这里为简便起见，进行了省略.

<!-- more -->

#用户信息

## 授权验证 <!--这边应该POST,因为重新生成了token -->
`/auth/local` GET 验证用户名密码，并重新生成token，返回基本的用户信息
接受参数:

- id 	String 用户id
- password 		String 用户密码

示例 : `GET localhost:3000/api/v1/auth/local?id=44220&password=44220`

```
{
  "id": "44220",
  "role": "high-level",
  "description": "干事1",
  "name": "",
  "accessToken": "d827b0ef-4e21-4cb7-b89f-c0b601fbe4b1"
}
```

## 重置密码 <!--这边应该用PUT ,因为用户更新了密码 -->
 `/users/changePassword` POST 重置用户密码，重新生成盐值和hash存储到数据库中，返回修改后的用户基本信息
接受参数:

- id 	String 用户id
- password 		String 用户密码
- new_password 		String 新的用户密码

示例 : `POST localhost:3000/api/v1/users/changePassword`

```
{
  "id": "44220",
  "role": "high-level",
  "description": "干事1",
  "name": "",
  "accessToken": "d827b0ef-4e21-4cb7-b89f-c0b601fbe4b1"
}
```

## 获得用户信息 //这边应该用/users/:id
`/users/details` GET 获取用户详细信息,如最近的主题以及最近的回复

接受参数:

- access_token 		String 用户的token

示例 :  `GET localhost:3000/api/v1/users/details?access_token=e25be7a2-0553-4104-a210-bde3873bfdb6`

```
{
  "id": "13852862738",
  "role": "low-level",
  "description": "海陵区城西街道情报员",
  "name": "高月",
  "recent_posts": [
    {
      "_id": "564aef7bd443b5243c7e826b",
      "author_id": "13852862738",
      "content": "测试情报",
      "important": false,
      "create_at": "2015-11-17T09:12:27.217Z"
    }
  ],
  "recent_replies": [
    {
      "_id": "564aef8fd443b5243c7e826d",
      "author_id": "13852862738",
      "content": "呵呵",
      "create_at": "2015-11-17T09:12:47.664Z"
    },
    {
      "_id": "564aef85d443b5243c7e826c",
      "author_id": "13852862738",
      "content": "评论一个",
      "create_at": "2015-11-17T09:12:37.741Z"
    }
  ]
}
```

# 情报信息
## 获取情报列表
`/posts` GET 获取帖子列表

接受参数:

- limit 	Number 每页的情报数量
- page 		Number 页数
- access_token 		String 用户的token

示例: `GET localhost:3000/api/v1/posts?access_token=e25be7a2-0553-4104-a210-bde3873bfdb6&limit=5&page=1`

```
[
  {
    "_id": "564aef7bd443b5243c7e826b",
    "author_id": "13852862738",
    "content": "测试情报",
    "important": false,
    "create_at": "2015-11-17T09:12:27.217Z",
    "author": {
      "name": "高月"
    }
  },
  {
    "_id": "564ae2c08bb772bf39386e2b",
    "author_id": "44121",
    "content": "hello",
    "important": true,
    "create_at": "2015-11-17T08:18:08.800Z",
    "author": {
      "name": ""
    }
  }
]
```

## 获取某个帖子的具体内容
`/posts/:id` GET 获取某个帖子的具体内容

接受参数:

- id	String 帖子的id
- access_token 		String 用户的token

示例: `GET localhost:3000/api/v1/posts/564aef7bd443b5243c7e826b?access_token=e25be7a2-0553-4104-a210-bde3873bfdb6`

```
{
  "_id": "564aef7bd443b5243c7e826b",
  "author_id": "13852862738",
  "content": "测试情报",
  "important": false,
  "photos": [],
  "create_at": "2015-11-17T09:12:27.217Z",
  "author": {
    "name": "高月"
  },
  "replies": [
    {
      "_id": "564aef85d443b5243c7e826c",
      "author_id": "13852862738",
      "post_id": "564aef7bd443b5243c7e826b",
      "content": "评论一个",
      "create_at": "2015-11-17T09:12:37.741Z",
      "author": {
        "name": "高月"
      }
    },
    {
      "_id": "564aef8fd443b5243c7e826d",
      "author_id": "13852862738",
      "post_id": "564aef7bd443b5243c7e826b",
      "content": "呵呵",
      "create_at": "2015-11-17T09:12:47.664Z",
      "author": {
        "name": "高月"
      }
    }
  ]
}
```

## 创建情报
`/posts` POST 创建情报，返回情报基本内容

参数: 

- content	String 帖子的内容,不能为空
- important 	Boolean 是否为紧急情报，默认为false
- photos 	[String] 情报的配图，目前默认只能上传一张
- access_token		String 用户的token

示例: `POST localhost:3000/api/v1/posts?access_token=e25be7a2-0553-4104-a210-bde3873bfdb6`

```
{
  "_id": "564c34da38cad35f288846c9",
  "author_id": "13852862738",
  "content": "测试消息 from postman",
  "important": true,
  "create_at": "2015-11-18T08:20:42.379Z"
}
```

## 删除情报 
`/posts/:id` DELETE 删除情报，返回空文档。

接受参数:

- id 	String 待删除的情报的id
- access_token 		String 用户的token

示例: `DELETE localhost:3000/api/v1/posts/564c34da38cad35f288846c9?access_token=e25be7a2-0553-4104-a210-bde3873bfdb6`

```
{}
```

# 回复

## 创建回复
`replys` POST 创建一个回复，返回该回复

接受参数:

- post_id 		String  回复的情报的id
- content 		String  回复内容
- access_token 	String  用户的token

示例: `POST localhost:3000/api/v1/replys?access_token=e25be7a2-0553-4104-a210-bde3873bfdb6`

```
{
  "_id": "564c7234d1739e4f0b056796",
  "post_id": " 564c65c34086d9a105d78243",
  "content": "添加一条回复",
  "author_id": "13852862738",
  "create_at": "2015-11-18T12:42:28.692Z"
}
```

## 删除回复
`replys/:id` DELETE 删除某个回复,返回空文档

接受参数:
- id String 该回复的id
- access_token String 用户的token

示例: `DELETE localhost:3000/api/v1/replys/564c7234d1739e4f0b056796?access_token=e25be7a2-0553-4104-a210-bde3873bfdb6`

```
{}
```