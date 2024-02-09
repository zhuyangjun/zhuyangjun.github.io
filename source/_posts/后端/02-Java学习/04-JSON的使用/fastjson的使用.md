## `fastjson`的使用

### 一、`JSONObject`是什么？

#### 1.1、`JSONObject`简介

```
1.JSONObject是一种数据结构
2.主要用于存储JSON数据的数据结构
3.JSON数据结构(key-value结构)
4.JSONObject结构可使用put方法给json对象添加元素
5.JSONObject可很快的转换成字符串,我们也可将其它对象转换为JSONObject对象 
```

#### 1.2、`JSON`简介

```
JSON是一种轻量级数据交换格式;常用于客户端和服务器端通信;它易于读/写,又与语言无关
{
	"name": "lucy",
	"age": 12
}
JSONObject继承自JSON
JSON是Fastjson的一个主要类

JSON中常见的方法:
   toJSONString(Object):将指定的对象序列化成Json表示形式
   parseObject(String, Class):将json反序列化为指定的Class模式
```

#### 1.3、`fastjson`的使用

```
引入依赖：
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.46</version>
</dependency> 
```

#### 1.4、`JSONObject`中的常用方法

```
	方法名										备注说明
containsValue(Object value)				  判断JSONObject是否包含此value值
containsKey(Object key)					  判断JSONObject是否包含此key值
get(Object key)							 通过key获取对应的key-value对象
										底层是先调用Map的get方法获取对象
										当获取的对象为空并且key为数值型则转成字符串型再次调用Map的get方法
```

