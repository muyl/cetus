# 分库(sharding)版配置文件说明

分库(sharding)版配置文件包括用户配置文件（users.json）、变量处理配置文件（variables.json）、分库版本的分片规则配置文件（sharding.json）和分库版本的启动配置文件（shard.conf），具体说明如下：

##  1.users.json

```
{
        "users":        [{
                        "user": "XXXX",
                        "client_pwd":   "XXXXXX",
                        "server_pwd":   "XXXXXX"
                }, {
                        "user": "XXXX",
                        "client_pwd":   "XXXXXX",
                        "server_pwd":   "XXXXXX"
                }]
}
```

users.json用来配置用户登陆信息，采用键值对的结构，其中键是固定的，值是用户在MySQL创建的登陆用户名和密码。

其中user的值是用户名；client_pwd的值是前端登录Cetus的密码；server_pwd的值是Cetus登录后端的密码。

例如：

```
{
       "users":        [{
                       "user": "root",
                       "client_pwd":   "123",
                       "server_pwd":   "123456"
               }, {
                       "user": "test",
                       "client_pwd":   "456",
                       "server_pwd":   "123456"
               }]
}
```

我们配置了2个用户名root和test。其中root用户前端登录Cetus的密码是123，Cetus登录后端的密码是123456；test用户前端登录Cetus的密码是456，Cetus登录后端的密码是123456。

##  2.variables.json

Cetus支持部分会话级系统变量的设置，可以通过在variables.json配置允许发送的值和静默处理的值，如下：

```
{
  "variables": [
    {
      "name": "XXXXX",
      "type": "XXXX",
      "allowed_values": ["XXX"]
    },
    {
      "name": "XXXXX",
      "type": "XXXX",
      "allowed_values": ["XXX"],
      "silent_values": ["XX"]
    }
  ]
}
```

variables.json同样采用键值对的结构，其中键是固定的，值是用用户自定义的。

其中name的值是需要设置的会话级系统变量的名称；type的值是变量的类型，可以为string或string-csv逗号分隔的字符串值，目前尚未支持int类型；allowed_values的值是指定允许设定的变量值，可以使用通配符\*表示此变量设任意值都允许；silent_values的值是指定静默处理的值，可以使用通配符\*，表示此变量设任意值都静默处理。

**注意：配置过allowed_values才能走到静默处理流程**

例如：

```
{
 "variables": [
   {
     "name": "sql_mode",
     "type": "string-csv",
     "allowed_values":
     ["STRICT_TRANS_TABLES",
       "NO_AUTO_CREATE_USER",
       "NO_ENGINE_SUBSTITUTION"
     ]
   },
   {
     "name": "connect_timeout",
     "type": "string",
     "allowed_values": ["*"],
     "silent_values": ["10", "100"]
   }
 ]
}
```

我们配置了sql_mode变量和connect_timeout变量。其中sql_mode变量的类型是string-csv（逗号分隔的字符串值），指定了允许设定的变量有STRICT_TRANS_TABLES、NO_AUTO_CREATE_USER和NO_ENGINE_SUBSTITUTION；connect_timeout变量的类型是string（字符串），此变量设任意值都允许，指定静默处理的值为10和100。

## 3.sharding.json

```
{
  "vdb": [
    {
      "id": X,
      "type": "XXX",
      "method": "XXXX",
      "num": X,
      "partitions": {"XXXX1": [X,X], "XXXX2": [X,X], "XXXX3": [X,X], "XXXX4": [X,X]}
    },
    {
      "id": X,
      "type": "XXX",
      "method": "XXXXX",
      "num": X,
      "partitions": {"XXXX1": XXXXXX, "XXXX2": XXXXXX, "XXXX3": XXXXXX,"XXXX4": XXXXXX}
    }
  ],
  "table": [
    {"vdb": X, "db": "XXXX", "table": "XXX", "pkey": "XX"},
    {"vdb": X, "db": "XXXX", "table": "XXX", "pkey": "XX"},
    {"vdb": X, "db": "XXXX", "table": "XXX", "pkey": "XX"},
    {"vdb": X, "db": "XXXX", "table": "XXX", "pkey": "XX"}
  ]
}
```

sharding.json是分库版本的分库规则配置文件，同样采用键值对的结构，其中键是固定的，值是由用户自定义。

其中vdb逻辑db，包含属性有id、type、method、num和partitions，id的值是逻辑db的id，type的值是分片键的类型，method的值是分片方式，num的值是hash分片的底数（range分片的num为0），partitions是分组名和分片范围的键值对,其中键和值都是用户自定义的；table是分片表，包含属性有vdb、db、table和pkey，vdb的值是逻辑db的id，db的值是物理db名，table的是分片表名，pkey的值是分片键。

例如：

```
{
 "vdb": [
   {
     "id": 1,
     "type": "int",
     "method": "hash",
     "num": 8,
     "partitions": {"data1": [0,1], "data2": [2,3], "data3": [4,5], "data4": [6,7]}
   },
   {
     "id": 2,
     "type": "int",
     "method": "range",
     "num": 0,
     "partitions": {"data1": 124999, "data2": 249999, "data3": 374999,"data4": 499999}
   }
  ],
 "table": [
   {"vdb": 1, "db": "employees_hash", "table": "dept_emp", "pkey": "emp_no"},
   {"vdb": 1, "db": "employees_hash", "table": "employees", "pkey": "emp_no"},
   {"vdb": 2, "db": "employees_range", "table": "dept_emp", "pkey": "emp_no"},
   {"vdb": 2, "db": "employees_range", "table": "employees", "pkey": "emp_no"},
 ]
}
```

我们配置了两种vbd分片规则，第一种规则的id为1，分片键类型是int，分片方法是hash，hash分片的底数为8，一共分了4组，分组名为data1的分片范围为0和1，分组名为data2的分片范围为2和3，分组名为data3的分片范围为4和5，分组名为data4的分片范围为6和7；第二种规则的id为2，分片键类型是int，分片方法是range，range无底数num设为0，一共分了4组，分组名为data1的分片范围为0-124999，分组名为data2的分片范围为125000-249999，分组名为data3的分片范围为250000-374999，分组名为data4的分片范围为37500-499999；

分片表table涉及两个物理db，为employees_hash和employees_range，其中employees_hash采用第一种分片规则，表dept_emp的分片键为emp_no，表employees的分片键为emp_no，employees_range采用第二种分片规则，表dept_emp的分片键为emp_no，表employees的分片键为emp_no。

##  4.shard.conf

```
[cetus]
# Loaded Plugins
plugins=XXXX,XXXX

# Proxy Configuration
proxy-address=XXX.XXX.XXX.XXX:XXXX
proxy-backend-addresses=XXX.XXX.XXX.XXX:XXXX@XXXX1,XXX.XXX.XXX.XXX:XXXX@XXXX2,XXX.XXX.XXX.XXX:XXXX@XXXX3,XXX.XXX.XXX.XXX:XXXX@XXXX4

# Admin Configuration
admin-address=XXX.XXX.XXX.XXX:XXXX
admin-username=XXXX
admin-password=XXXX

# Backend Configuration
default-db=XXX
default-username=XXXX

# Log Configuration
log-file=XXXX
log-level=XXXX
```

shard.conf是分库版本的启动配置文件，在启动Cetus时需要加载，配置文件同样采用key＝value的形式，其中key是固定的，可参考[Cetus 启动配置选项说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-configuration.md)，value是用户自定义的。

例如：

```
[cetus]
# Loaded Plugins
plugins=shard,admin

# Proxy Configuration
proxy-address=127.0.0.1:1234
proxy-backend-addresses=127.0.0.1:3361@data1,127.0.0.1:3362@data2,127.0.0.1:3363@data3,127.0.0.1:3364@data4

# Admin Configuration
admin-address=127.0.0.1:5678
admin-username=admin
admin-password=admin

# Backend Configuration
default-db=test
default-username=dbtest

# Log Configuration
log-file=cetus.log
log-level=debug
```

我们配置了分库版本的启动选项，其中plugins的值是加载插件的名称，分库（sharding）版本需加载的插件为shard和admin；

proxy-address的值是Proxy监听的IP和端口，我们设置为127.0.0.1:1234；proxy-backend-addresses的值是后端的IP和端口，需要同时指定group（@group），本例分为4个group，分别data1的127.0.0.1:3361、data2的127.0.0.1:3362、data3的127.0.0.1:3363、data4的127.0.0.1:3364；

admin-address的值是管理模块的IP和端口，我们设置为127.0.0.1:5678；admin-username的值是管理模块的用户名，我们设置为admin；admin-password的值是管理模块的密码明文，我们设置为admin；

default-db的值是默认数据库，当连接未指定db时，使用的默认数据库名称，我们设置为test；default-username的值是默认登陆用户名，在Proxy启动时自动创建连接使用的用户名，我们设置为dbtest；

log-file的值是日志文件路径，我们设置为当前安装路径下的cetus.log；log-level的值是日志记录级别，可选 info | message | warning | error | critical(default)，我们设置为debug；这些是必备启动选项，其他可选性能配置详见[Cetus 启动配置选项说明](https://github.com/Lede-Inc/cetus/blob/master/doc/cetus-configuration.md)。

**注：**

**以上配置文件中.json文件名称不可变，.conf文件可自定义名称，并利用命令行加载**

**启动配置文件shard.conf 常用参数：**

**1）default-pool-size=\<num\>，设置刚启动的连接数量**

**2）max-pool-size=\<num\>，设置最大连接数量**

**3）max-resp-size=\<num\>，设置最大响应大小，一旦超过此大小，则会报错给客户端**

**4）enable-client-compress=\[true\|false\]，支持客户端压缩**

**5）enable-tcp-stream=\[true\|false\]，启动tcp stream，无需等响应收完就发送给客户端**

**6）master-preferred=\[true\|false\]，除非注释强制访问从库，否则一律访问主库**

**7）reduce-connections=\[true\|false\]，自动减少过多的后端连接数量**
