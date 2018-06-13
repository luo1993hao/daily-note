### 错误收集
1. 
```

org.springframework.http.converter.HttpMessageNotReadableException: JSON parse error: Cannot deserialize value of type `java.util.Date` from String "201
8-05-03 11:09:41": not a valid representation (error: Failed to parse Date 
value '2018-05-03 11:09:41': Cannot parse date "2018-05-03 11:09:41": while it

seems to fit format 'yyyy-MM-dd'T'HH:mm:ss.SSSZ', parsing fails (leniency? 
null)); nested exception is 
com.fasterxml.jackson.databind.exc.InvalidFormatException: Cannot deserialize 
value of type `java.util.Date` from String "2018-05-03 11:09:41": not a valid 
representation (error: Failed to parse Date value '2018-05-03 11:09:41': 
Cannot parse date "2018-05-03 11:09:41": while it seems to fit format 
'yyyy-MM-dd'T'HH:mm:ss.SSSZ', parsing fails (leniency? null))

 at [Source: (PushbackInputStream); line: 1, column: 678] (through reference 
 chain: com.c503.lng.api.entity.StationBasic["createTime"])
```
解决：

----
2. spring boot 集成fastJson
```
https://blog.csdn.net/cjq2013/article/details/76421101
```
```
你用的是spring-boot的2.0.x版本吧 ，WebMvcConfigurerAdapter被WebMvcConfigurer代替了
```