# KOKO-MONI
## 介绍
本项目是一个网络空间搜索引擎监控平台，本项目聚合了 Fofa、Hunter、Quake、Zoomeye 和 Threatbook 的数据源，并对获取到的数据进行去重与清洗。


该项目可以用于蓝方监控自身资产公网暴露以及 SRC 项目新增资产进行监控。
## 功能
- 聚合 Fofa、Hunter、Quake、Zoomeye 和 Threatbook 的数据源，快速查询相关资产信息。
- 自动化监控，可定时进行资产信息爬取，及时发现新增资产。
- 支持钉钉，推送加消息提醒，让您能够及时发现异常情况。
- 提供 Web 检索界面，使得查询操作更加方便。
## 安装
解压安装包

按照config.demo.yaml 内的注释填写配置项 保存为config.yaml

启动 ./koko-moni

## UI
![图片.png](./img/1.png)

![图片.png](./img/2.png)

![图片.png](./img/2.JPG)

## 搜索语法

本项目使用了 [ZED](https://github.com/brimdata/zed) 作为结构化数据搜索引擎
可参考[zed官方文档](https://zed.brimdata.io/docs/language/overview)

### 简单运算
```
int
tlen >= 0

时间
timestamp >= 2023-01-08T05:55:22.200Z

字符串
title=="404"


```
如果带有特殊符号(中文)的键 根下可以使用 this[""] 引用
this["status-code"] >10
this["content-length"]==158
非根
abc["测试"]=="123456"

多层复杂json结构
{"a":{"b":{"c":"123"}}}
this["a"].b.c=="123"
{"a":{"b":["123","456"]}}
this["a"].b[0]=="123"
### 强制类型转换

```

cast(数据,<类型>)

字符串日期转time

cast("2022-09-19T18:11:05.545961703+08:00",<time>)

字符串ip转ip

cast("1.1.1.1",<ip>)

也可以简写成

time("2022-09-19T18:11:05.545961703+08:00")
ip("1.1.1.1")

用例

cidr_match(1.1.0.0/16,ip(host))  //匹配ip是否在cidr内


time(timestamp) >= 2022-01-08T05:55:22.200Z
```
### 常用操作
#### 排序
升序排序
sort dns_names   
降序排序
sort -r dns_names
#### in
判断某个值是否在数组内
```
{"test":[301,200]}
200 in test
```


#### 聚合查询
##### 统计计数
count() by key
输出  key,count()
与sort组合使用
count() by title|sort -r count
![图片.png](https://cdn.nlark.com/yuque/0/2023/png/25577536/1674188968503-469e5ab7-c470-4a47-adb5-2e41cbaa8e08.png#averageHue=%23efa695&clientId=uc4ecdc4a-1236-4&from=paste&height=539&id=u16a78f8a&name=%E5%9B%BE%E7%89%87.png&originHeight=674&originWidth=582&originalType=binary&ratio=1&rotation=0&showTitle=false&size=51309&status=done&style=none&taskId=ue774514d-576f-4122-9cb3-c4b693f8419&title=&width=465.6)




## API

### 全局参数

鉴权url参数 `key` 

在配置文件中设置(secret_key)


### 测试推送

请求方式: GET

请求URL: `/api/testpush`

请求参数: 无


### 聚合查询

请求方式: GET

请求URL: `/api/aggregate`

请求参数: 

| 参数名 | 必选 | 类型 | 说明 |
| --- | --- | --- | --- |
| current | 是 | int | 当前页数 |
| pageSize | 是 | int | 每页数据量 |
| query | 是 | string | ZQ查询语句 |

响应格式: JSON

响应示例: 

```
{
  "data": {
    "count": 92,
    "elapsed": 226,
    "finger": [
      {
        "finger": "Fofa",
        "count": 75
      },
      {
        "finger": "Hunter",
        "count": 17
      },
      {
        "finger": "Nginx",
        "count": 7
      },
      {
        "finger": "Lua",
        "count": 7
      }
    ],
    "ipcount": 77,
    "port": [
      {
        "port": "443",
        "count": 57
      },
      {
        "port": "80",
        "count": 26
      }
    ],
    "title": [
      {
        "count": 29
      },
      {
        "title": "301 Moved Permanently",
        "count": 16
      },
      {
        "title": "302 Found",
        "count": 11
      }
    ]
  },
  "message": "ok",
  "success": true
}
```

### 搜索

请求方式: GET

请求URL: `/api/search`

请求参数: 

| 参数名 | 必选 | 类型 | 说明 |
| --- | --- | --- | --- |
| current | 是 | int | 当前页数 |
| pageSize | 是 | int | 每页数据量 |
| query | 是 | string | ZQ查询语句 |

响应格式: JSON

响应示例: 

```
{
	"data": [{
		"banner": "HTTP/1.1 403 Forbidden\r\nConnection: close\r\nContent-Length: 9\r\nContent-Type: application/octet-stream\r\nDate: Thu, 16 Feb 2023 17:11:21 GMT\r\nServer: Tengine\r\n\r\n\r\n",
		"body_length": 9,
		"commonname": "",
		"date": "2023-03-13 21:59:41",
		"dnsnames": "",
		"fingerprint": ["Fofa"],
		"host": "1.1.com",
		"ip": "1.1.1.1",
		"loc": "[中国 上海 上海]",
		"organization": "Huawei Cloud Service data center",
		"port": "443",
		"status_code": 403,
		"title": "",
		"tls": "Version:  v3\nSerial Number: 123456546\nSignature Algorithm: SHA256-RSA",
		"url": "https://1.1.com:443/"
	}],
	"message": "ok",
	"success": true,
	"total": 92
}
```