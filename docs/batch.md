
## 调用方式

### API 接收地址

```
https://pipeline.qiniu.com
```

### API 返回内容

**响应报文** 

* 如果请求成功,返回 HTTP 状态码`200`:

```
HTTP/1.1 200 OK
```

* 如果请求失败,返回包含如下内容的 JSON 字符串（已格式化,便于阅读）:

```
{
    "error":   "<errMsg    string>"
}
```

* 如果请求包含数据获取,则返回相应数据的 JSON 字符串；

## 数据源相关接口

### 创建批量计算数据源

**请求语法**

 ```
 POST /v2/datasources/<DataSourceName>
 Content-Type: application/json
 Authorization: Pandora <auth>
 {
     "region": <Region>,
     "type": <Type>,
     "spec": <Spec>,
     "noVerifySchema": <NoVerifySchema>,
     "workflow": <WorkflowName>,
     "schema": [
       {
         "key": <Key>,
         "valtype": <ValueType>,
         "required": <Required>
       },
       ...
     ]
 }
 ```
 
**参数说明**

|名称|类型|必填|描述|
|:---|:---|:---|:---|
|DataSourceName|string|是|数据源名称</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`</br>1-128个字符，支持小写字母、数字、下划线</br>必须以大小写字母或下划线开头|
|region|string|是|计算与存储所使用的物理资源所在区域</br>目前仅支持“nb”(华东区域)|
|type|string|是|数据源类型，可选值为[`kodo`,`hdfs`,`fusion`]|
|spec|json|是|指定该数据源自身属性相关的信息|
|noVerifySchema|bool|否|是否推断数据源，默认值为`false`。当值为`true`时，会使用用户填写的`schema`，不会主动触发推断`schema`操作|
|workflowName|string|否|指定当前数据源所属的工作流名称，且该工作流必须提前创建|
|schema|array|是|字段信息|
|schema.key|string|是|字段名称</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`</br>1-128个字符,支持小写字母、数字、下划线</br>必须以大小写字母或下划线开头|
|schema.valtype|string|是|字段类型</br>目前仅支持：</br>`boolean`：布尔类型</br>`long`：整型</br>`date`：RFC3339日期格式</br>`float`：64位精度浮点型</br>`string`：字符串|
|schema.required|bool|是|是否必填</br>用户在传输数据时`key`字段是否必填|

 当 type 为`hdfs`的时候 spec 定义如下:
 
|名称|类型|必填|描述|
|:---|:---|:---|:---|
|spec.paths|array|是|包含一个或者多个hdfs文件路径</br>例如：`hdfs://192.168.1.1:9000/usr/local`</br>可以使用系统默认魔法变量，下文中详解|
|spec.fileType|string|是|文件类型，合法取值为`json`、`csv`、`text`和`parquet`|
|spec.delimiter|string|否|csv文件分割符，当文件类型为csv时，delimiter为必填项|
|spec.containsHeader|bool|否|csv文件是否包含header标志，当文件类型为csv时，containsHeader为必填项|

 当type为`kodo`的时候 spec 定义如下:
 
|名称|类型|必填|描述|
|:---|:---|:---|:---|
|spec.bucket|string|是|对象存储数据中心名称，</br>命名规则：4-63个字符，支持字母、数字、中划线|
|spec.keyPrefixes|array|否|包含一个或者多个文件前缀；</br>命名规则：0-128个字符，不支持英文 `\`、`<`、`>`符号|
|spec.fileType|string|是|文件类型，合法取值为`json`、`csv`、`text`和`parquet`|
|spec.delimiter|string|否|csv文件分割符，当文件类型为csv时，delimiter为必填项|
|spec.containsHeader|bool|否|csv文件是否包含header标志，当文件类型为csv时，containsHeader为必填项|

 当type为`fusion`的时候 spec 定义如下:

|名称|类型|必填|描述|
|:---|:---|:---|:---|
|spec.domains|array|是|融合cdn的域名(目前仅支持单域名)|
|spec.fileFilter|string|否|CDN文件过滤规则</br>命名规则：0-64个字符</br>最终的文件地址是'文件前缀+文件过滤规则'</br>请注意过滤规则不要和文件前缀有重复. 注：当数据源为fusion时，支持2种类型的文件过滤表达式（1：固定时间范围，精确到年月日小时。如果2017-06-01 05:00 ~ 2017-07-01 05:00 2.相对时间范围: 设置魔法变量 `fiveDaysAgo = $(now) - 5d` 则fileFilter可以写为 $(fiveDaysAgo) ~ $(now) 范围：五天前到当前调度时间。)|

!> 注意：`region`参数的选择以降低传输数据的成本为原则，请尽量选择离自己数据源较近的区域。

### 更新批量计算数据源

**请求语法**

 ```
 PUT /v2/datasources/<DataSourceName>
 Content-Type: application/json
 Authorization: Pandora <auth>
 {
     "spec": <Spec>,
     "schema": [
       {
         "key": <Key>,
         "valtype": <ValueType>,
         "required": <Required>
       },
       ...
     ]
 }
 ```

!> 注意: 更新批量计算数据源字段信息的时候，不允许减少字段，也不允许更改字段的类型。


### 根据名称查看批量计算数据源

**请求语法**

```
GET /v2/datasources/<DataSourceName>
Authorization: Pandora <auth>
```

**响应报文**

 ```
 {
     "region": <Region>,
     "type": <Type>,
     "spec": <Spec>,
     "schema": [
       {
         "key": <Key>,
         "valtype": <ValueType>,
         "required": <Required>
       },
       ...
     ]
 }
 ```


### 列举批量计算数据源

**请求语法**

```
GET /v2/datasources
Authorization: Pandora <auth>
```

**响应报文**

 ```
 {
   "datasources": [
     {
       "name": <DataSourceName>,
       "region": <Region>,
       "type": <Type>,
       "spec": <Spec>,
       "schema": [
         {
           "key": <Key>,
           "valtype": <ValueType>,
           "required": <Required>
         },
         ...
       ]
     },
     ...
   ]
 }
 ```

### 根据名称删除批量计算数据源

**请求语法**

```
DELETE /v2/datasources/<DataSourceName>
Authorization: Pandora <auth>
```

## 批量计算任务相关接口

### 创建批量计算任务

**请求语法** 

```
POST /v2/jobs/<JobName>
Content-Type: application/json
Authorization: Pandora <auth>
{
	"srcs":[
		{
			"name":<DataSourceName|JobName>,
			"type":<DataSource|Job>,
			"tableName": <TableName>
		},
		...
	],
   "computation": {
       "code": <Code>,
       "type": <SQL>
   }, 
   "container": {
       "type": <ContainerType>,
       "count": <ContainerCount>
   },  
   "scheduler":{
       "type": <crontab|loop|manual|depend>,
       "spec": {
           "crontab": <0 0 0/1 * *>,
           "loop": <1h|3m|....>
       } 
   },
   "params":[
       {
           "name":<ParamName>,
           "default":<ParamValue>
       },
       ...
   ]
}
```

**参数说明**

|名称|类型|必填|描述|
|:---|:---|:---|:---|
|srcs|array|是|数据来源|
|srcs.name|string|是|数据源名称或离线任务名称|
|srcs.type|string|是|数据来源节点类型|
|srcs.tableName|string|是|数据来源的表名称</br>命名规则：1-128个字符，支持字母、数字、下划线，必须以字母开头|
|computation|object|是|计算方式|
|computation.code|string|是|代码片段，可以使用魔法变量|
|computation.type|string|是|代码类型，支持SQL|
|container|map|是|计算资源|
|container.type|string|是|规格，目前支持：1U2G ，1U4G，2U4G，4U8G，4U16G ，8U16G ，8U32G ，16U32G，16U64G|
|container.count|int|是|数量，所选规格 * 数量 <= 100U|
|scheduler|map|是|调度|
|scheduler.type|string|是|调度方式，定时、循环或单次执行三选一，下游任务是依赖模式|
|scheduler.spec.crontab|string|否|定时执行，当调度方式选择为'定时'，此项必填；必须为crontab类型|
|scheduler.spec.loop|string|否|循环执行，当调度方式选择为'循环'，此项必填；</br>其值以`m`(分钟)和`h`(小时)为单位，由数字与单位组成，例如:`5m`|
|params|array|否|魔法变量，系统默认自带6个魔法变量：`$(year)=当前年份`、`$(mon)=当前月份`、`$(day)=当前日期`、`$(hour)=当前小时`、`$(min)=当前分钟`、`$(sec)=当前秒数`；用户也可以自行定义魔法变量|
|params.name|string|是|变量名称,命名规则：1-64个字符，支持大小写字母、数字、下划线，大小写字母或下划线开头|
|params.default|string|是|默认值，命名规则：0-64个字符|

> scheduler.type 为 depend 的时候，container 依赖上游的配置，该配置不填
> 
> scheduler.type 为 manual 和 depend 的时候 spec 可以不填
> 
> scheduler.type 为 loop时，不填该 spec.loop 或者该字段为 0，则默认持续运行该任务
> 
> scheduler.type 为 depend 的时候，params 依赖上游的配置，该配置不填

!> 注1：scheduler.type 如果是 depend 模式，代表这个离线任务依赖某个上游的离线任务。首先 srcs 内有且仅有一个离线任务数据源。同时该任务不能指定调度的模式、魔法变量和容器规格。这些全部使用上游依赖的离线任务。

!> 注2：srcs.tableName 在此任务的 srcs 中以及依赖的所有上游任务的 srcs 中不能重复。

> 文件前缀+文件过滤规则举例说明：
> 
> 文件前缀：`audit-log/xxx-log-`
> 
> 文件过滤规则： `$(year)-$(mon)-$(day)`
> 
> 最终的文件地址为： `audit-log/xxx-log-2017-06-01`

### 更新批量计算任务

**请求语法** 

```
PUT /v2/jobs/<JobName>
Content-Type: application/json
Authorization: Pandora <auth>
{
	"srcs":[
		{
			"name":<DataSourceName|JobName>,
			"fileFilter":<fileFilter>,
			"type":<DataSource|Job>,
			"tableName": <TableName>
		},
		...
	],
   "computation": {
       "code": <Code>,
       "type": <SQL>
   }, 
   "container": {
       "type": <ContainerType>,
       "count": <ContainerCount>
   },
   "scheduler":{
       "type": <crontab|loop|manual|depend>,
       "spec": {
           "crontab": <0 0 0/1 * *>,
           "loop": <1h|3m|....>
       }
   },
   "params":[
       {
           "name":<ParamName>,
           "default":<ParamValue>
       },
       ...
   ]
}
```

!> 注意：更新时候 srcs, code, container, scheduler, params 校验逻辑和创建的时候相同。下游计算任务不指定container, scheduler, params。更新逻辑为全量更新。需要提前获取所有信息。

### 列举批量计算任务信息

**请求语法** 

```
GET /v2/jobs?srcDatasource=[DataSourceName]&srcJob=[JobName]
Authorization: Pandora <auth>

```

**响应报文** 

```
{
	"jobs":[
		{
		    "name": <JobName>,
			"srcs":[
				{
					"name":<DataSourceName|JobName>,
					"fileFilter":<fileFilter>,
					"type":<DataSource|Job>,
					"tableName": <TableName>
				},
				...
			],
		   "scheduler":{
		       "type": <crontab|loop|manual|depend>,
		       "spec": {
		           "crontab": <0 0 0/1 * *>,
		           "loop": <1h|3m|....>
		       }
		   },
		   "computation": {
		       "code": <Code>,
		       "type": <SQL>
		   }, 
		   "container": {
		       "type": <ContainerType>, 
		       "count": <ContainerCount>
		   },
		   "params":[
		       {
		           "name":<ParamName>,
		           "default":<ParamValue>
		       },
		       ...
		   ]，
		   "schema": [
		       {
		           "key": <Key>,
		           "valtype": <ValueType>
		       },
		       ...
		   ]
		},
		...
	]
}
```

**参数说明** 


|名称|类型|必填|描述|
|:---|:---|:---|:---|
|srcDatasource|string|否|依赖离线数据源名字|
|srcJob|string|否|依赖离线计算任务名字|

!> 注意: srcDataSource 和 srcJob 不能同时存在。当两个参数都不指定的时候列举出所有的离线计算任务。


### 获取单个批量计算任务信息

**请求语法** 

```
GET /v2/jobs/<JobName>
Content-Type: application/json
Authorization: Pandora <auth>

```

**响应报文** 

```
{
	"srcs":[
		{
			"name":<DataSourceName|JobName>,
			"fileFilter":<fileFilter>,
			"type":<DataSource|Job>,
			"tableName": <TableName>
		},
		...
	],
   "scheduler":{
       "type": <crontab|loop|manual|depend>,
       "spec": {
           "crontab": <0 0 0/1 * *>,
           "loop": <1h|3m|....> 
		  } 
	},
   "computation": {
       "code": <SqlCode>,
       "type": <SQL>
   }, 
   "container": {
       "type": <ContainerType>, 
       "count": <ContainerCount>
   },
   "params":[
   		{
   			"name":<ParamName>,
   			"default":<ParamValue>
   		},
   		...
   ]，
   "schema": [
      {
        "key": <Key>,
        "valtype": <ValueType>
      },
      ...
    ]
}
```



### 启动批量计算任务

**请求语法**

```
POST /v2/jobs/<JobName>/actions/start
Authorization: Pandora <auth>
{
	"params":[
   		{
   			"name":<ParamName>,
   			"value":<ParamValue>
   		},
   		...
   ]
}
```

**参数说明**

|名称|类型|必填|描述|
|:---|:---|:---|:---|
| params |array|否|定义运行时的魔法参数值|
| params.name |string|是|魔法变量名字|
| params.value |string|是|运行时魔法变量值|



### 停止批量计算任务

**请求语法**

```
POST /v2/jobs/<JobName>/actions/stop
Authorization: Pandora <auth>
```


### 删除批量计算任务信息

**请求语法** 

```
DELETE /v2/jobs/<JobName>
Authorization: Pandora <auth>
```


### 获取数据源schema

**请求语法** 

```
POST /v2/schemas
Content-Type: application/json
Authorization: Pandora <auth>
{
	"type": <Kodo|HDFS|Fusion>, 
	"spec": <Spec>
}
```

**参数说明**

Type 为 Kodo 时，Spec结构：

|名称|类型|必填|描述|
|:---|:---|:---|:---|
| spec.bucket |string|是|对象存储的存储空间|
| spec.keyPrefixes |array|否|一个或者多个文件前缀|
| spec.fileType |string|是|文件类型，合法取值为`json`, `parquet`, `text`|

Type 为 HDFS 时，Spec 结构：

|名称|类型|必填|描述|
|:---|:---|:---|:---|
| spec.paths |array|是|一个或者多个文件路径|
| spec.fileType |string|是|文件类型，合法取值为`json`, `parquet`, `text`|


Type为 Fusion 时，Spec 为空结构体.


**响应内容** 

```
json or parquet return :
{
	"schema": [
		{
			"key": <Key>,
			"valtype": <ValueType>
		},
		...
	]
}

text return :
{
	"schema": [
		{
			"key": <text>,
			"valtype": <String>
		},
		...
	]
}

```
## 批量计算任务导出接口 

### 导出数据至云存储

**请求语法**

```
POST /v2/jobs/<JobName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "type": <kodo>,
    "spec": {
         "bucket": <Bucket>,
         "keyPrefix": <Prefix|Path>,
         "format": <ExportFormat>,
         "delimiter": <Delimiter>,
         "containsHeader":<True|False>
         "compression": <compression>,
         "retention": <Retention>,
         "partitionBy": <PartitionBy>，
         "fileCount": <FileCount>,
         "saveMode": <SaveMode>
	}
}
```

**参数说明**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|type|string|是|导出的类型，目前允许的值为"kodo"|
|bucket|string|是|对象存储bucket名称|
|keyPrefix|string|否|导出的文件名的前缀，当离线任务的`scheduler`是`manual`的时候，就是文件名</br>命名规则：0-128个字符，不支持英文 `:` 、`\`、`<`、`>`符号|
|format|string|否|文件导出格式,支持`json`、`csv`、`text`、`orc`、`parquet`五种类型|
|delimiter|string|否|csv文件分割符，当文件类型为csv时，delimiter为必填项|
|containsHeader|bool|否|csv文件是否包含header标志，当文件类型为csv时，containsHeader为必填项|
|compression|string|否|压缩类型, 具体支持类型与`format`值相关，详见`注1`|
|retention|int|否|数据储存时限,以天为单位,当不大于0或该字段为空时,则永久储存|
|partitionBy|array|否|指定作为分区的字段，为一个字符数组，合法的元素值是字段名称|
|fileCount|int|是|计算结果导出的文件数量，应当大于0，小于等于1000|
|saveMode|string|否|计算结果的保存模式：overwrite(默认) 文件已经存在则覆盖掉， append 在已有的文件上追加，errorIfExists 文件已经存在的时候报错误，ignore 文件已经存在 则认为跑成功了，不报错|

!> 注1: 当用户指定`format`为`json`、`csv`或`text`时, `compression`仅支持`none`(不压缩)、`bzip2`, `gzip`, `lz4`, `snappy`和`deflate`; 当用户指定`format`为`orc`时, `compression`仅支持`none`(不压缩)、`snappy`, `zlib`和`lzo`; 当用户指定`format`为`parquet`时, `compression`仅支持`none`(不压缩)、`snappy`, `gzip`和`lzo`。

!> 注2: `keyPrefix`字段表示导出文件名称的前缀,该字段可选,默认值为""(生成文件名会自动加上时间戳格式为`yyyy-MM-dd-HH-mm-ss`),如果使用了一个或者多个魔法变量时不会自动添加时间戳,支持魔法变量,采用`$(var)`的形式求值,目前可用的魔法变量var如下:

* `year` 上传时的年份
* `mon` 上传时的月份
* `day` 上传时的日期
* `hour` 上传时的小时
* `min` 上传时的分钟
* `sec` 上传时的秒钟

### 导出数据至 HDFS

**请求语法**

```
POST /v2/jobs/<JobName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "type": <hdfs>,
    "spec": {
         "path": <path_to_write>,
         "format": <ExportFormat>,
         "delimiter": <Delimiter>,
         "compression": <compression>,
         "partitionBy": <PartitionBy>，
         "fileCount": <FileCount>,
         "saveMode": <SaveMode>
	}
}
```

**参数说明**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|type|string|是|导出的类型，目前允许的值为"kodo"|
|path|string|是|导出的路径。例如：`hdfs://192.168.1.1:9000/usr/local`；命名规则：0-128个字符，不支持英文 `\`、`<`、`>`符号|
|format|string|否|文件导出格式,支持,`json`、`csv`、`text`、`orc`、`parquet`五种类型|
|delimiter|string|否|csv文件分割符，当文件类型为csv时，delimiter为必填项|
|compression|string|否|压缩类型, 具体支持类型与`format`值相关，详见`注1`|
|partitionBy|array|否|指定作为分区的字段，为一个字符数组，合法的元素值是字段名称|
|fileCount|int|是|计算结果导出的文件数量，应当大于0，小于等于1000|
|saveMode|string|否|计算结果的保存模式：overwrite(默认) 文件已经存在则覆盖掉， append 在已有的文件上追加，errorIfExists 文件已经存在的时候报错误，ignore 文件已经存在 则认为跑成功了，不报错|

!> 注1: 当用户指定`format`为`json`、`csv`或`text`时, `compression`仅支持`none`(不压缩)、`bzip2`, `gzip`, `lz4`, `snappy`和`deflate`; 当用户指定`format`为`orc`时, `compression`仅支持`none`(不压缩)、`snappy`, `zlib`和`lzo`; 当用户指定`format`为`parquet`时, `compression`仅支持`none`(不压缩)、`snappy`, `gzip`和`lzo`。

!> 注2: `path`字段支持魔法变量,采用`$(var)`的形式求值,目前可用的魔法变量var如下:

* `year` 上传时的年份
* `mon` 上传时的月份
* `day` 上传时的日期
* `hour` 上传时的小时
* `min` 上传时的分钟
* `sec` 上传时的秒钟

### 导出数据至 HTTP 地址

**请求语法**

```
POST /v2/jobs/<JobName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
  "type": <http>,
  "spec": {
    "host": <Host>,
    "uri": <RequestURI>,
    "format": <ExportFormat>
  }
}
```

**参数说明**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
| host |string|是|服务器地址（ip或域名）</br>例如:`https://pipeline.qiniu.com` </br>或 `127.0.0.1:7758`|
| uri |string|是|请求资源路径（具体地址,不包含ip或域名）</br>例如:`/test/repos`|
| format |string|否|导出方式</br>支持`text`和`json`</br>如果没有填写此项，默认为`text`|

!> 注1:  当导出格式为`text`时，导出的服务端需要支持`Content-Type(MimeType)`为`text/plain`的请求; 当导出格式为`json`时，导出服务端需要支持`Content-Type(MimeType)`为`application/json`的请求。
!> 注2: `text`的导出数据格式为：`Key1=Value1\tKey2=Value2\tKey3=Value3`; `json`的导出数据格式为：`{"Key1": Value1, "Key2": Value2, "Key3": Value3}`。

### 导出数据至日志检索服务

**请求语法**

```
POST /v2/jobs/<JobName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
  "type": <logdb>,
  "spec": {
        "destRepoName": <DestRepoName>,
        "omitInvalid": <OmitInvalid>,              
        "doc": {
            "LogdbRepoField1": <#JobField1>,
            "LogdbRepoField2": <#JobField2>,
            "LogdbRepoField3": <#JobField3>,         
            ......
        }
}
```

**参数说明**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
| destRepoName |string|是|日志仓库名称|
| omitInvalid |bool|否|是否忽略无效数据，默认值为 false|
| doc |map|是|字段关系说明</br> `JobField`表示离线Job的字段名称</br>`LogdbRepoField`表示目标日志仓库字段名称|

### 导出数据至时序数据库服务

**请求语法**

```
POST /v2/jobs/<JobName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
  "type": <tsdb>,
  "spec": {
        "destRepoName": <DestRepoName>,
        "omitInvalid": <OmitInvalid>, 
        "series": <Series>,             
        "tags": {
        	"tag1": <#JobField1>,
        	"tag2": <#JobField2>,
        	...
        },
        "fields": {
        	"field1": <#JobField3>,
        	"field2": <#JobField4>,
        	...
        },
        "timestamp": <#JobField5> 
}
```

**参数说明**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
| destRepoName |string|是|日志仓库名称|
| omitInvalid |bool|否|是否忽略无效数据，默认值为 false|
| tags |map|是|索引字段|
| fields |map|是|普通字段|
| timestamp |string|否|时间戳字段，`JobField`表示离线 Job 的字段名称|

### 导出数据至报表平台

**请求语法**

```
POST /v2/jobs/<JobName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
  "type": <report>,
  "spec": {
        "dbName": <DBName>,
        "tableName": <TableName>,
        "columns": {
            "column1": <#JobField1>,
            "column2": <#JobField2>,
            "column3": <#JobField3>,
            ...
        },
        "saveMode": <SaveMode>
}
```

**参数说明**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|dbName|string|是|数据库名称|
|tableName|bool|是|数据表名称|
|saveMode|string|否|计算结果的保存模式：overwrite(默认) 默认重写整张表， append 在已有的表数据上追加数据。默认append模式|
|columns |map|是|字段关系说明</br> `JobField`表示离线Job的字段名称</br>`columnN`表示报表服务数据表字段名称|


### 更新导出任务

**请求语法**

```
PUT /v2/jobs/<JobName>/exports/<ExportName>
Content-Type: application/json
Authorization: Pandora <auth>
{
    "spec": <Spec>
}
```


### 按照名称查看导出任务

**请求语法**

```
GET /v2/jobs/<JobName>/exports/<ExportName>
Authorization: Pandora <auth>
```

**响应报文**

```
{
    "type": <Type>,
    "spec": <Spec>
}
```


### 列举导出任务

**请求语法**

```
GET /v2/jobs/<JobName>/exports
Authorization: Pandora <auth>
```

**响应报文**

```
Content-Type: application/json
{
  "exports": [
    {
      "name": <ExportName>,
      "type": <Type>,
      "spec": <Spec>
    },
    ...
  ]
}
```


### 按照名称删除导出任务

**请求语法**

```
DELETE /v2/jobs/<JobName>/exports/<ExportName>
Authorization: Pandora <auth>
```


## 查看历史批量计算任务

**请求语法**

```
GET /v2/jobs/<jobName>/history?page=1&size=20&sortBy=endTime&status=xx&runId=-1
Content-Type: application/json
Authorization: Pandora <auth>
```

**参数说明**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|page|int|否|分页页数|
|size |int|否|分页大小|
|sortBy |string|否|根据哪个字段排序, endTime代表按照字段升序排列，-endTime按照逆序排列|
|status |string|否|<Ready、Successful、Failed、Running、Canceled, Restarting, Cancelling>|
|runId |int|否|查询某个运行批次的状态，-1代表最近一次|


**响应报文**

```
{
"total": <TotalCnt>, # 总共运行批次
"history":[
	{
		"runId" : <RunId>, # 运行批次
		"batchTime": "<batchTime>", #批次时间，每个批次第一次被调度的时间，是不可变的。
		"startTime": <StartTime>,
		"endTime": <EndTime>,
		"duration": <Duration>,
		"status": <Ready、Successful、Failed、Running、Canceled>,
		"message": <Message>
	},
  ]
}
```

**参数说明**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|total|int|是|总运行批次次数|
|history|array|是|运行历史|
|history.runId|int|是|运行批次|
|history.batchTime|string|是|批次时间|
|history.startTime |string|否|启动时间|
|history.endTime |string|否|终止事件，如果为Running，则为当前时间|
|history.duration |int|否|批次运行时间，单位秒|
|history.status |string|否|<Ready、Successful、Failed、Running、Canceled>|
|history.message |string|否|运行、出错信息，比如运行成功、内存溢出、数据损坏|

## 停止批次任务

**请求语法**

```
POST /v2/batch/actions/stop
Content-Type: application/json
Authorization: Pandora <auth>
{
    "jobName": "<jobName>",
    "runId": "<runId>"
}

```
**参数说明**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|jobName|string|是|job名称|
|runId|string|是|待操作的运行批次ID|


**响应报文**

```
{
  "preStatus": "Running",
  "postStatus": "Canceled"
}
```

**参数说明**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|preStatus|string|是|停止前状态|
|postStatus|string|是|停止后状态|

**状态说明**

|状态|说明|备注|
|:---|:---|:---|
|Ready|就绪|不允许停止操作|
|Successful|成功|不允许停止操作|
|Failed|失败|不允许停止操作|
|Running|运行中|允许停止操作|
|Canceled|已停止|不允许停止操作|
|Restarting|重启中|用于重启中间状态，不允许任何操作|
|Cancelling|停止中|用于停止中间状态，不允许任何操作|


## 重跑批次任务

**请求语法**

```
POST /v2/batch/actions/rerun
Content-Type: application/json
Authorization: Pandora <auth>
{
    "jobName": "<jobName>",
    "runId": "<runId>"
}

```

**参数说明**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|jobName|string|是|job名称|
|runId|string|是|待操作的运行批次ID|


**响应报文**

```
{
  "preStatus": "Failed",
  "postStatus": "Running",
  "rerunCount": 2
}
```

**参数说明**

|参数|类型|必填|说明|
|:---|:---|:---:|:---|
|preStatus|string|是|重跑前状态|
|postStatus|string|是|重跑后状态|
|rerunCount|int|是|重跑次数|

**状态说明**

|状态|说明|备注|
|:---|:---|:---|
|Ready|就绪|不允许重跑|
|Successful|成功|允许重跑|
|Failed|失败|允许重跑|
|Running|运行中|不允许重跑（请先停止）|
|Canceled|已停止|允许重跑|
|Restarting|重启中|用于重启中间状态，不允许任何操作|
|Cancelling|停止中|用于停止中间状态，不允许任何操作|

## 魔法变量接口
### 创建魔法变量

**请求语法**

 ```
 POST /v2/variables/<VariableName>
 Content-Type: application/json
 Authorization: Pandora <auth>
 {
     "type": <Type>,
     "value": <Value>,
     "format": <Format>,
 }
 ```
 
**参数说明**

|名称|类型|必填|描述|
|:---|:---|:---|:---|
|VariableName|string|是|魔法变量名称</br>命名规则: `^[a-zA-Z_][a-zA-Z0-9_]{0,127}$`</br>1-128个字符，支持小写字母、数字、下划线</br>必须以大小写字母或下划线开头|
|type|string|是|魔法变量类型，可选值为`time`和`string`，其中`time`类型表示该变量的取值与当前时间有关，如`$(now)-5h`，其中`h`表示小时，也支持`d`和`m`，分别表示天和分钟|
|value|string|是|魔法变量值，如`time`类型变量可为`$(now)-5h`；`string`类型常量可为`2017-01-01`|
|format|string|可选|仅当type为`time`时需要填写，表示指定的时间格式，如`yyyy-MM-dd HH:mm:ss`|


!> 备注：系统默认可用的魔法变量如下:

* `year` 表示`type`为`time`；`value`为当前年份；`format`为 `yyyy`, 如`2017`
* `mon` 表示`type`为`time`；`value`为当前月份；`format`为 `MM`, 如`01`
* `day` 表示`type`为`time`；`value`为当前日期；`format`为 `dd`, 如`01`
* `hour` 表示`type`为`time`；`value`为当前小时；`format`为 `HH`, 如`00`
* `min` 表示`type`为`time`；`value`为当前分钟；`format`为 `mm`, 如`00`
* `sec` 表示`type`为`time`；`value`为当前秒数；`format`为 `ss`, 如`00`
* `date` 表示`type`为`time`；`value`为当前秒数；`format`为 `yyyy-MM-dd`, 如`2017-01-01`
* `now` 表示`type`为`time`；`value`为当前时间；`format`为 `yyyy-MM-dd HH:mm:ss`, 如`2017-01-01 00:00:00`


### 更新魔法变量

**请求语法**

 ```
 PUT /v2/variables/<VariableName>
 Content-Type: application/json
 Authorization: Pandora <auth>
 {
     "type": <Type>,
     "value": <Value>,
     "format": <Format>
 }
 ```

### 根据名称查看魔法变量

**请求语法**

```
GET /v2/variables/<VariableName>
Authorization: Pandora <auth>
```

**响应报文**

 ```
 {
     "name": <Name>,
     "type": <Type>,
     "value": <Value>,
     "format": <Format>
 }
 ```


### 列举魔法变量

**请求语法**

```
GET /v2/variables?type=<Type>
Authorization: Pandora <auth>
```

**响应报文**

 ```
 {
   "variables": [
     {
        "name": <VariableName>,
        "type": <Type>,
        "value": <Value>,
        "format": <Format>
     },
     ...
   ]
 }
 ```
!> 注意：请求url中`type`参数仅支持`system`和`user`, 分别表示列举出系统内置变量和用户本身以创建变量。


### 根据名称删除魔法变量

**请求语法**

```
DELETE /v2/variables/<VariableName>
Authorization: Pandora <auth>
```

## 错误代码及相关说明

| 错误码 | 错误描述 |
| :---  | :----- |
|500	|服务器内部错误 |
|411	|E18003: 缺少内容长度|
|400	|E18004: 无效的内容长度|
|413	|E18005: 请求实体过大|
|400	|E18006: 请求实体为空|
|400	|E18007: 请求实体格式非法|
|400	|E18008: 字段长度超过限制|
|409	|E18101: 仓库已经存在|
|404	|E18102: 仓库不存在|
|400	|E18103: 无效的仓库名称|
|400	|E18104: 无效的日期格式|
|400	|E18105: 仓库Schema为空|
|400	|E18106: 无效的字段名称|
|400	|E18107: 不支持的字段类型|
|400	|E18108: 源仓库数量超过限制|
|400	|E18109: 无效的仓库模式|
|400	|E18110: 无效的字段格式|
|404	|E18111: 字段不存在|
|400	|E18112: 仓库上存在着级联的转换任务或者导出任务|
|409	|E18113: 仓库处于删除状态中|
|400	|E18117: Plugin名称不合法|
|404	|E18120: 共享资源池不存在|
|404	|E18122: 导出的仓库在logd中不存在|
|202	|E18124: 仓库处于创建中|
|400	|E18125: 读取gzip的打点请求体出错|
|409	|E18201: 计算任务已经存在|
|404	|E18202: 计算任务不存在|
|415	|E18203: 计算任务类型不支持|
|409	|E18204: 计算任务的源仓库与目的仓库相同|
|409	|E18205: 目的仓库已经被其他转换任务占用|
|400	|E18206: 目的仓库必须通过删除计算任务的方式删除|
|400	|E18207: 计算任务描述格式非法|
|400	|E18208: 计算任务interval非法|
|400	|E18209: 计算任务中的SQL语句非法|
|400	|E18211: 计算任务中plugin输出字段类型非法|
|400	|E18212: 仓库的区域信息和数据中心不相符|
|400	|E18213: 计算任务中容器类型非法|
|400	|E18214: 计算任务中容器数量非法|
|409	|E18215: 共享资源池处于使用中|
|404	|E18216: Plugin不存在|
|409	|E18217: Plugin已存在|
|409	|E18218: 共享资源池已存在|
|400	|E18219: Plugin上传内容长度不一致|
|400	|E18220: Plugin上传内容的MD5不一致|
|400	|E18221: 共享资源池名称非法|
|400	|E18222: 共享资源池的区域信息不一致|
|400	|E18223: 不能向计算任务的目标仓库中打点|
|409	|E18301: 导出任务已经存在|
|404	|E18302: 导出任务不存在|
|400	|E18303: 提交导出任务失败|
|400	|E18304: 删除导出任务失败|
|400	|E18305: 导出任务出现错误|
|401  |bad token：鉴权不通过 、token已过期、机器时间未同步|
|400  |E18639: 工作流名称不合法|
|409  |E18640: 工作流已存在|
|404  |E18641: 工作流不存在|
|400  |E18642: 工作流的格式非法|
|400  |E18644: 当前工作流禁止启动|
|400  |E18645: 当前工作流禁止停止|






