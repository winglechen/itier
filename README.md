 Version 1.0

此程序在 node 0.6.6 ~ 0.7.4 下测试通过

你可以按照以下流程来部署测试程序:

####1 运行 ```sample``` 目录下的 ```service.js```

   ```
      node service.js &
   ```

   这个进程会在3561,3562 端口提供http 服务,来模拟真实的数据源
   用curl 测试如下：

   ```
      curl http://localhost:3561/?fm=m
   ```

   response :

   ```
      { "columns" : ["id","sex"],
        "data" : [ 
          [1,"m"],
          [2,"m"],
          [5,"m"],
          [7,"m"],
          [9,"m"]
        ]
      }
   ```

   ```
      curl http://localhost:3562/?minId=4
   ```

   response:

   ```
      { "columns" : ["id","name","type"], 
        "data" : [ 
          [5,"n5","t5"],
          [6,"n6","t6"],
          [7,"n7","t7"],
          [8,"n8","t8"],
          [9,"n9","t9"],
          [10,"n10","t10"]
        ]
      }
   ````

####2 确认loader 配置正确
   在 ```src/source/``` 下，```service_a.js``` ,```service_b.js``` 为以上两个端口服务相应的loader，
   如果默认的3561,3562 端口被占用，请确认其配置做相应修改

####3 确认sql 模板配置
   目前iTier 仅支持三级目录配置，查看 ```resources/test/sources/test.json```,
   比如 其中有 ```test_service_a``` ,```test_service_b``` 相应的sql模板:

   ```
   "test_service_a" : {
     "config":"select id as num from service_a.table where ##fm## order by num DESC limit 1,2",
     "params":{
       "fm":[{
         "type":"string",
         "pos":"##fm##"
       }]
     }
   }
   ```


####4 启动 iTier 服务 
   运行 ```run.sh``` ,默认会在 3459 端口进行监听（配置见 ```conf.js```）

####5 测试服务 
   现在你可以测试在sql模板配置的所有服务
   测试服务：```test_service_a```:
   测试命令：

   ```
	curl http://127.0.0.1:3459/test/sources/test/test_service_a/where/fm:eq:f
   ```

   后台操作：后台会对 ```test_service_a``` 的模板算子 ``` ##fm## ``` 替换为 ```fm=m```
   返回结果：

   ```
      { "columns" : ["num"],
        "data":[
          [8],
          [6]
        ]
      }
   ```
  
   测试服务：```test_servie_b```

   测试命令：

   ```
	curl http://127.0.0.1:3459/test/sources/test/test_service_b/where/minId:eq:3
   ```

   返回结果：

   ```
      { "columns":["name"],
        "data":[
          ["n9"],
          ["n8"],
          ["n7"],
          ["n6"],
          ["n5"],
          ["n4"],
          ["n10"]
        ]
      }
   ```

   测试服务：```test_innerjoin```

   测试命令：

   ```
	curl http://127.0.0.1:3459/test/sources/test/test_innerjoin/where/fm:eq:f/where/minId:eq:3
   ```

   后台操作：替换后的 sql 语句为

   ```
   "select * from service_a.table as sa inner join service_b.table as sb on sa.id=sb.id where fm=f and minId=3"
   ```

   返回结果：

   ```
      { "columns" : ["id","sex","name","type"],
        "data" : [
          [4,"f","n4","t4"],
          [6,"f","n6","t6"],
          [8,"f","n8","t8"],
          [10,"f","n10","t10"]
        ]
      }
   ```
