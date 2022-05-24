# 命令行触发dubbo

## telnet远程连接到dubbo

```undefined
telnet 127.0.0.1 20880
```

## 查看提供服务的接口

```shell
dubbo>ls
com.test.service.TestInfoQueryService
```

## ls 接口名对外提供的方法

```shell
dubbo>ls com.test.service.TestInfoQueryService
queryByInfoCode
queryInfo
```

## 调用服务

invoke 接口名.方法名(参数) 进行调用

```javascript
invoke XxxService.xxxMethod(1234, "abcd", {"prop" : "value"}): 调用服务的方法
invoke com.xxx.XxxService.XxxService.xxxMethod(1234, "abcd", {"prop" : "value"}): 调用全路径服务的方法
invoke xxxMethod(1234, "abcd", {"prop" : "value"}): 调用服务的方法(自动查找包含此方法的服务)
invoke xxxMethod({"name":"zhangsan","age":12,"class":"org.apache.dubbo.qos.legacy.service.Person"}) :当有参数重载，或者类型转换失败的时候，可以通过增加class属性指定需要转换类(传入class，解决重载问题)
当参数为Map<Integer,T>，key的类型为Integer时，建议指定类型。例如invoke com.xxx.xxxApiService({"3":0.123, "class":"java.util.HashMap"})


invoke com.caocao.risk.offline.strategy.api.service.IRiskCheckService.submit({"eventId":80002,"biz":1,"context":{"orderNo":123132321,"customerNo":123312,"customerMobile":"15180626533","customerDeviceId":"dasdsaddas","driverNo":12312,"driverMobile":"15180626533"},"class":"com.caocao.risk.offline.strategy.api.request.RiskAsyncRequest"
})

invoke com.caocao.risk.offline.strategy.api.service.IRiskCheckService.submit({"eventId":80002,"outerRequestId":"13132","biz":16,"context":{"originalFee":8971,"orderNo":383092180116217,"bizType":16,"driverOverLt":0.0,"driverTimeLg":113.35762,"realKm":0.0,"cityCode":"0762","driverTimeLt":23.160993,"orderStatus":7,"eventType":80002,"customerDeviceId":"Duqqt2YbFcVUF9SzrAhdZ1CtgjfQxTnr-QbNXL_q9nGn9CRA6Fab2bC_jA745R1Ib2eWTT5s76VW5cXOpJXg3gpA","biz":16,"driverDeviceId":"D2udaaydvw8wsrKroU1nJKEI1yddR0dl_0eGL1Z-KQNa0Xf7","getOnTime":1648468197000,"driverOverLg":0.0,"driverMobile":"18102616483","useTime":1648468800000,"thanksFee":0,"customerMobile":"18102737237","customerNo":114850217,"driverNo":844573743,"class":"java.util.HashMap"},"class":"com.caocao.risk.offline.strategy.api.request.RiskAsyncRequest"})


invoke com.caocao.risk.offline.strategy.api.service.IRiskCheckService.submit({"eventId":80002,"biz":16,"context":{"originalFee":8971,"orderNo":383092180116217,"bizType":16,"driverOverLt":0.0,"driverTimeLg":113.35762,"realKm":0.0,"cityCode":"0762","driverTimeLt":23.160993,"orderStatus":7,"eventType":80002,"customerDeviceId":"Duqqt2YbFcVUF9SzrAhdZ1CtgjfQxTnr-QbNXL_q9nGn9CRA6Fab2bC_jA745R1Ib2eWTT5s76VW5cXOpJXg3gpA","biz":16,"driverDeviceId":"D2udaaydvw8wsrKroU1nJKEI1yddR0dl_0eGL1Z-KQNa0Xf7","getOnTime":1648468197000,"driverOverLg":0.0,"driverMobile":"18102616483","useTime":1648468800000,"thanksFee":0,"customerMobile":"18102737237","customerNo":114850217,"driverNo":844573743,"class":"java.util.HashMap"},"class":"com.caocao.risk.offline.strategy.api.request.RiskAsyncRequest"})

curl localhost:8020/risk-offline-strategy/controller/check?originalFee=8971


invoke com.caocao.risk.terminal.identify.api.AppletSubscribeNoticeApi.subscribe({"orderNo":12345679,"uid":7890000,"status":0,"msgType":1,"originEnumCode":9,"customerMobile":"15180626533","requestTime":1648863891258,"class":"com.caocao.risk.terminal.identify.param.SubscribeNoticeParam"})
bizType=~[16]&&!empty(clientType)&&clientType=~[1,3]&&!empty(driverDeviceId)&&size(driverDeviceId)==48
bizType=~[16]&&!empty(clientType)&&clientType=~[2]&&!empty(driverDeviceId)&&size(driverDeviceId)==88
```

