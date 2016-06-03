http://www.cnblogs.com/egger/archive/2013/06/14/3135847.html

#查询操作
1. 集合查询方法 find() 
2. 查询内嵌文档
3. 查询操作符(内含 数组查询)  
"$gt" 、"$gte"、 "$lt"、 "$lte"、"null查询"、"$all"、"$size"、"$in"、"$nin"、
"$and"、"$nor"、"$not"、"$or"、"$exists"、"$mod"、"$regex"、"$where"、"$slice"、"$elemMatch"

####1.1集合查询方法find()
一个条件  
>   db.user.find({age:16})
  
多个条件，逗号分隔
>   db.user.find({age:28,sex:"male"})

指定返回的键  
通过find 的第二个参数来指定返回的键  
若find不指定第二个参数，查询操作默认返回查询文档中所有键值  
集合user包含 _id,name,age,sex，email等键,如果查询结果想只显示集合中的"name"和"age"键，可以使用如下查询返回这些键
>   db.users.find({}, {"name" ： 1, "age" ： 1})

_id"这个键总是被返回，即便是没有指定也一样。但是我们可以显示的将其从查询结果中移除掉
>   db.users.find({}, {"name" ： 1, "age" ： 1, "_id"：0})

在第二个参数中，指定键名且值为1或者true则是查询结果中显示的键；若值为0或者false，则为不显示键


####1.3查询操作符
下面我们将配合查询操作符来执行复杂的查询操作，比如元素查询、 逻辑查询 、比较查询操作  
使用下面的比较操作符"$gt" 、"$gte"、 "$lt"、 "$lte"(分别对应">"、 ">=" 、"<" 、"<=")，组合起来进行范围的查找  
查询年龄为16-18岁(包含16但不含18)的用户：  
>   db.user.find( { age: { $gte: 16 ,$lt:18} } 

我们可以使用"$ne"来进行"不相等"操作。例如查询年龄不为18岁的用户：  
>   db.user.find( { age: {$ne:18} } 

精确匹配日期要精确到毫秒,然而我们通常只是想得到关于一天、一周或者是一个月的数据，我们可以使用"gt"、"gt"、"lt"进行范围査询  
例如，要査找在1990年1月1日出生的用户：
>    start = new Date("1990/01/01")  
>    db.users.find({"birthday" ： {"$lt" ： start}})

####键值为null查询操作
$in"判断键值是否为null,"$exists"判定集合中文档是否包含该键  
>   //集合中有一条sex键值为null的文档  
>   {"name":"xiaoming","age":20,"sex":"male"}  
>   {"name":"xiaohong","age":22,"sex":"female"}  
>   {"name":"lilei","age":24,"sex":null}  

>   //返回文档中存在sex键，且值为null的文档  
>   db.users.find({sex:{$in:[null],$exists:true }})  

>   //返回文档中存在birthday键，且值为null的文档   
>   //文档没有birthday键，所以结果为空  
>   db.users.find({birthday:{$in:[null],$exists:true }})  

我们也可以运行如下语句：  
>   db.users.find({sex:null})  

我们最好使用db.users.find({sex:{in:[null],in:[null],exists:true }})这种格式。  

####"$all"
匹配那些指定键的键值中包含数组，而且该数组包含条件指定数组的所有元素的文档,数组中元素顺序不影响查询结果  

>   语法: { field: { $all: [ <value> , <value1> ... ] }  

查询出在集合inventory中 tags键值包含数组，且该数组中包含appliances、school、 book元素的所有文档:  
>   db.inventory.find( { tags: { $all: [ "appliances", "school", "book" ] } }   

文档中键值类型不是数组，也可以使用$all操作符进行查询操作，如下例所示"$all"对应的数组只有一个值，那么和直接匹配这个值效果是一样的  
>   //查询结果是相同的，匹配amount键值等于50的文档  
>   db.inventory.find( { amount: {$all:[50]}} )  
>   db.inventory.find( { amount: 50}} )    

要是想查询数组指定位置的元素，则需使用key.index语法指定下标，例如下面查询出tags键值数组中第2个元素为"school"的文档：  
>   db.inventory.find({"tags.1":"school"})  
数组下标都是从0开始的，所以查询结果返回数组中第2个元素为"school"的文档  

####"$size"
用其查询指定长度的数组  
语法：{field: {$size: value} }  
查询集合中tags键值包含有3个元素的数组的所有文档：
> db.inventory.find({tags:{$size:3}})

size必须制定一个定值，不能接受一个范围值，不能与其他查询子句组合(比如"gt")  
但有时查询需求就是需要一个长度范围，这种情况创建一个计数器字段，当你增加元素的同时增加计数器字段值。 
>  //每一次向指定数组添加元素的时候,"count"键值增加1(充当计数功能)  
>   db.collection.update({ $push : {field: value}, $inc ：{count ： 1}})  
>   //比较count键值实现范围查询  
>   db.collection.find({count : {$gt:2}})  

####"$in"  
匹配键值等于指定数组中任意值的文档。类似sql中in.  
语法: { field: { $in: [<value1>, <value2>, ... <valueN> ] } }
####"$nin"  
匹配键不存在或者键值不等于指定数组的任意值的文档。类似sql中not in(SQL中字段不存在使用会有语法错误).  
语法: { field: { $nin: [ <value1>, <value2> ... <valueN> ]} }　　

查询出amount键值为16或者50的文档：  

>   db.inventory.find( { amount: { $in: [ 16, 50 ] } } )  

>   //查询出amount键值不为16或者50的文档  
>   db.inventory.find( { amount: { $nin: [ 16, 50 ] } } )  
>   //查询出qty键值不为16或50的文档，由于文档中都不存在键qty,所以返回所有文档   
>   db.inventory.find( { qty: { $nin: [ 16, 50 ] } } )  


文档中键值类型不是数组，也可以使用$all操作符进行查询操作，如下例所示"$in"对应的数组只有一个值，那么和直接匹配这个值效果是一样的。  
>   //查询结果是相同的，匹配amount键值等于50的文档  
>   db.inventory.find( { amount: {$in:[50]}} )  
>   db.inventory.find( { amount: 50}} )  

####"$and" 
指定一个至少包含两个表达式的数组，选择出满足该数组中所有表达式的文档。$and操作符使用短路操作，若第一个表达式的值为“false”,余下的表达式将不会执行。  
语法: { $and: [ { <expression1> }, { <expression2> } , ... , { <expressionN> } ] }  
查询name键值为“t1”,amount键值小于50的文档：  
>   db.inventory.find({ $and: [ { name: "t1" }, { amount: { $lt：50 } } ] } )

对于下面使用逗号分隔符的表达式列表，MongoDB会提供一个隐式的$and操作：  
>   //等同于{ $and: [ { name: "t1" }, { amount: { $lt：50 } } ] }  
>   db.inventory.find({ name: "t1" , amount: { $lt：50 }} )  

####"$nor"
执行逻辑NOR运算,指定一个至少包含两个表达式的数组，选择出都不满足该数组中所有表达式的文档。  
语法: { $nor: [ { <expression1> }, { <expression2> }, ... { <expressionN> } ] }  
查询name键值不为“t1”,amount键值不小于50的文档：  
>   db.inventory.find( { $nor: [ { name: "t1" }, { qty: { $lt: 50 } } ] } )  

若是文档中不存在表达式中指定的键，表达式值为false; false nor false 等于 true,所以查询结果返回集合中所有文档：  
>   db.inventory.find( { $nor: [ { sale: true }, { qty: { $lt: 50 } } ] } )  

####"$not"
执行逻辑NOT运算，选择出不能匹配表达式的文档 ，包括没有指定键的文档。$not操作符不能独立使用，必须跟其他操作一起使用（除$regex）  
语法: { field: { $not: { <operator-expression> } } }  
查询amount键值不大于50（即小于等于50）的文档数据  
>   db.inventory.find( { amount: { $not: { $gt: 50 } } } ) //等同于db.inventory.find( { amount:  { $lte: 50 } } )  

查询条件中的键gty，文档中都不存在无法匹配表示，所以返回集合所有文档数据。  
>   db.inventory.find( { gty: { $not: { $gt: 50 } } } ）  

####"$or" 
执行逻辑OR运算,指定一个至少包含两个表达式的数组，选择出至少满足数组中一条表达式的文档。  
语法: { $or: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] }  
查询集合中amount的键值大于50或者name的键值为“t1”的文档：  
>   db.inventory.find( { $or: [ { amount: { $gt: 50 } }, { name: "t1" } ] } )  

####"$exists"   
如果$exists的值为true,选择存在该字段的文档；若值为false则选择不包含该字段的文档(我们上面在查询键值为null的文档时使用"$exists"判定集合中文档是否包含该键)。  
语法: { field: { $exists: <boolean> } }  
>   //查询不存在qty字段的文档（所有文档）  
>   db.inventory.find( { qty: { $exists: false } })  
>   //查询amount字段存在，且值不等于16和58的文档  
>   db.inventory.find( { amount: { $exists: true, $nin: [ 16, 58 ] } } )   

如果该字段的值为null，$exists的值为true会返回该条文档，false则不返回。  
>   //向集合中插入一条amount键值为null的文档  
>   {"name":"t4","amount":null,"tags":[ "bag", "school", "book" ]}  
>   //0条数据  
>   db.inventory.find( { amount: { $exists: false } } )  
>   //所有的数据  
>   db.inventory.find( { amount: { $exists: true } } )    

####"$mod"  
匹配字段值对（divisor）取模，值等于（remainder）的文档。  
语法: { field: { $mod: [ divisor, remainder ]} }  
查询集合中 amount 键值为 4 的 0 次模数的所有文档，例如 amount 值等于 16 的文档  
>   db.inventory.find( { amount: { $mod: [ 4, 0 ] } } )  

有些情况下(特殊情况键值为null时)，我们可以使用mod操作符替代使用求模表达式的mod操作符替代使用求模表达式的where操作符，因为后者代价昂贵。   
>   db.inventory.find( { $where: "this.amount % 4 == 0" } )  
 注意：返回结果怎么不一样。因为有一条文档的amount键值为null,javascript中null进行数值转换，会返回"0"。所以该条文档匹配where操作符求模式了表达式。当文档中字段值不存在null，就可以使用where操作符求模式了表达式。当文档中字段值不存在null，就可以使用mod替代$where的表达式.  
 
####"$regex"
操作符查询中可以对字符串的执行正则匹配。 MongoDB使用Perl兼容的正则表达式（PCRE)库来匹配正则表达式  
我们可以使用正则表达式对象或者$regex操作符来执行正则匹配：  
>    //查询name键值以“4”结尾的文档  
>   db.inventory.find( { name: /.4/i } );  
>   db.inventory.find( { name: { $regex: '.4', $options: 'i' } } );  

###options（使用options（使用regex ）

*   i  　如果设置了这个修饰符，模式中的字母会进行大小写不敏感匹配。
*   m   默认情况下，PCRE 认为目标字符串是由单行字符组成的(然而实际上它可能会包含多行).如果目标字符串 中没有 "\n"字符，或者模式中没有出现“行首”/“行末”字符，设置这个修饰符不产生任何影响。
*   s    如果设置了这个修饰符，模式中的点号元字符匹配所有字符，包含换行符。如果没有这个修饰符，点号不匹配换行符。
*   x    如果设置了这个修饰符，模式中的没有经过转义的或不在字符类中的空白数据字符总会被忽略，并且位于一个未转义的字符类外部的#字符和下一个换行符之间的字符也被忽略。 这个修饰符使被编译模式中可以包含注释。  
       注意：这仅用于数据字符。 空白字符 还是不能在模式的特殊字符序列中出现，比如序列 。 
　　注：JavaScript只提供了i和m选项，x和s选项必须使用$regex操作符。  

####"$where"
操作符功能强大而且灵活，他可以使用任意的JavaScript作为查询的一部分,包含JavaScript表达式的字符串或者JavaScript函数  
新建fruit集合并插入如下文档：  
>   //插入两条数据  
>   db.fruit.insert({"apple":1, "banana": 4, "peach" : 4})  
>   db.fruit.insert({"apple":3, "banana": 3, "peach" : 4})   

比较文档中的两个键的值是否相等.例如查找出banana等于peach键值的文档（4种方法）：  
>   //JavaScrip字符串形式  
>   db.fruit.find( { $where: "this.banana == this.peach" } )  
>   db.fruit.find( { $where: "obj.banana == obj.peach" } )  

>   //JavaScript函数形式  
>   db.fruit.find( { $where: function() { return (this.banana == this.peach) } } )  
>   db.fruit.find( { $where: function() { return obj.banana == obj.peach; } } )  

####"$slice (projection)" 
$slice操作符控制查询返回的数组中元素的个数  
语法：db.collection.find( { field: value }, { array: {$slice: count } } );  
此操作符根据参数"{ field: value }" 指定键名和键值选择出文档集合，并且该文档集合中指定"array"键将返回从指定数量的元素。如果count的值大于数组中元素的数量，该查询返回数组中的所有元素的  
$slice接受多种格式的参数 包含负数和数组:  
>   //选择comments的数组键值中前五个元素。  
>   db.posts.find( {}, { comments: { $slice: 5 } } );

>   //选择comments的数组键值中后五个元素。  
>   db.posts.find( {}, { comments: { $slice: -5 } } );

####"$elemMatch(projection)"  
elemMatch投影操作符将限制查询返回的数组字段的内容只包含匹配elemMatch条件的数组元素。  

注意：

* 数组中元素是内嵌文档。
* 如果多个元素匹配$elemMatch条件，操作符返回数组中第一个匹配条件的元素。  
