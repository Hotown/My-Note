# 为什么需要DTO

## 传统的前端请求

`webApplication`->`webService`->`database`

```sequence
webApplication->webService: getRecording(long id)
webService->Database: getRecording(long id)
Database-->webService: getRecording(long id)
webService-->webApplication: getRecording(long id)
```

**缺点：**
多次请求，耗费网络资源，效率差。

## 采用DTO模型

`aClient`->`aServer`->`customerDTO`

**then**

`aClient`->`customerDTO`->`aClient`

```sequence
aClient->aServer: getCustomerDTO
aServer-->customerDTO: 创建
aClient->customerDTO: GetTitle()
customerDTO-->aClient:
aClient->customerDTO: GetName()
customerDTO-->aClient:
```

**优点：**

1. 可以根据现有的类代码，构造出DTO对象
2. 减少请求次数，效率高
3. 按需组织DTO对象，页面需要的字段我才组织，不需要的我不组织，可以避免传输整个表的字段，一定程度上提高了安全性。


