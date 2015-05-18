###jsonDB
	jsonDB是js的一个类库，实现使用SQL语句对json数据增删改查。
	jsonDB的构建源自于HTML5本地存储的一个应用需求，可以通过sql对json数据进行增删改查，
	同时该类库提供强大的where检索条件，数据排序，limit查询条件限制等数据库基本功能。
	通过jsonDB可以轻松维护一个库/表或多个库/表，而无需额外实现json的数据的维护等，
	在该类库完善以后为简化sql操作，基于jsonDB核心模块扩展了连贯操作模型，简化对jsonDB的操作以及sql语句出错的概率。
	
当前版本的不足： 
-------------------
	1.无法支持查询字段运算 
	2.不支持对没有选取的查询字段排序 
	3.只支持单个字段排序，无法进行组合排序操作 
	4.update、delete语句不支持order by子句，导致limit子句功能弱化 
	5.编写where条件是必须使用()包含where条件 
	6.无法选取深层次的字段作为返回字段 
	7.没有错误或异常解决方案 
	8.不支持外部扩展 
 
jsonDB基础应用示例：
-------------------
```javascript
//创建一张user数据表，并定义一个DB的jsonDB别名  
var data = [{username:'张三',sex:'男',birthday:{year:2000,month:6,day:18}},{username:'李红',sex:'女',birthday:{year:1986,month:9,day:22}}];  
//以下方法可以通过两种方式获取别名，一个是通过init方法获取(推荐)，一个是获取jsonDB()方法的返回值  
//以后示例中都将使用init()方法创建的别名操作数据  
var jDB = jsonDB(data,'user').init('DB');  
  
//插入一条新的数据  
data = {username:'李想',sex:'男',birthday:{year:1990,month:2,day:15}};  
DB.insert(data,'user');  
  
//查询姓名为李红的性别,where条件必须加()否者会出现错误  
var result =DB.query('select sex from user where (username="李红")');  
//结果：[{"sex":"女"}]  
  
//查询2000年之前出生的且按出生年先后排序  
var result =DB.query('select * from user where (birthday.year<2000) order by birthday.year asc');  
//结果：[{"username":"李红","sex":"女","birthday":{"year":1986,"month":9,"day":22}},{"username":"李想","sex":"男","birthday":{"year":1990,"month":2,"day":15}}] 

//查询年龄最小的两个人  
var result =DB.query('select * from user order by birthday.year desc limit 2');  
//查询结果：[{"username":"张三","sex":"男","birthday":{"year":2000,"month":6,"day":18}},{"username":"李想","sex":"男","birthday":{"year":1990,"month":2,"day":15}}]  
  
//修改李红的出生日期  
var result =DB.query('update user set birthday.year=1991 where (username="李红") limit 1');  
//影响条数为一条，可以通过DB.findAll('user')获取全部数据查看是否被修改成功  
  
//删除姓名为李想的数据  
var result =DB.query('delete from user where (username="李想") limit 1');  
//影响条数为一条，可以通过DB.findAll('user')获取全部数据查看是否被删除成功  
  
//数据库中所有数据  
//[{"username":"张三","sex":"男","birthday":{"year":2000,"month":6,"day":18}},{"username":"李红","sex":"女","birthday":{"year":1991,"month":9,"day":22}}]  
```

jsonDB的高级应用示例：
-------------------
```javascript
//向数据表插入一条数据
DB.table('user').add({username:'王帅',sex:'男',birthday:{year:1995,month:10,day:23}});
			
//查询所有人出生日期，并按出生年排序，由于之前使用table('user')定义为user表，所以可以省略，table方法定义是一直有效的，除非是重新设定了
var result = DB.field('username,birthday').order('birthday.year').select();
//查询结果	[{"username":"李红","birthday":{"year":1991,"month":9,"day":22}},{"username":"王帅","birthday":{"year":1995,"month":10,"day":23}},{"username":"张三","birthday":{"year":2000,"month":6,"day":18}}]

//先插入一些数据，方便后面的检索操作
DB.table('user').add({username:'李亨',sex:'男',birthday:{year:2008,month:8,day:8}})
.add({username:'张琦',sex:'男',birthday:{year:1990,month:10,day:23}})
.add({username:'李媛芳',sex:'女',birthday:{year:1985,month:2,day:28}})
.add({username:'李果果',sex:'女',birthday:{year:2002,month:3,day:15}})
.add({username:'张源',sex:'男',birthday:{year:2005,month:12,day:5}})
.add({username:'王丽娜',sex:'女',birthday:{year:1992,month:5,day:12}});

//高级查询之模糊搜索
//查询所有李姓成员(正则查找法)
var result = DB.field(["username"]).where('username.match(/^李/)').select();
//查询结果[{"username":"李红"},{"username":"李亨"},{"username":"李媛芳"},{"username":"李果果"}]

//查询所有李姓成员(字符串搜索法)
var result = DB.field(["username"]).where('username.indexOf("李")=0').select();
//查询结果[{"username":"李红"},{"username":"李亨"},{"username":"李媛芳"},{"username":"李果果"}]

//查询所有在1990年到1995年出生的人
var result = DB.field(["username","birthday"]).where('birthday.year>=1990 and birthday.year<=1995').order('birthday.year').select();
//查询结果：[{"username":"张琦","birthday":{"year":1990,"month":10,"day":23}},{"username":"李红","birthday":{"year":1991,"month":9,"day":22}},{"username":"王丽娜","birthday":{"year":1992,"month":5,"day":12}},{"username":"王帅","birthday":{"year":1995,"month":10,"day":23}}]
```
