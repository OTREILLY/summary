MySQL支持数值、日期/时间和字符串(字符)类型。 </br>
一、数值 </br>

- 整数



- 小数 </br>
```
float(M,D)、double(M、D)的用法规则：
D表示浮点型数据小数点之后的精度，假如超过D位则四舍五入，即1.233四舍五入为1.23，1.237四舍五入为1.24
M表示浮点型数据总共的位数，D=2则表示总共支持五位，即小数点前只支持三位数，所以我们并没有看到1000.23、10000.233、100000.233这三条数据的插入，因为插入都报错了
不指定M、D的时候，会按照实际的精度来处理。

float、double类型存在精度丢失问题，即写入数据库的数据未必是插入数据库的数据，而decimal无论写入数据中的数据是多少，都不会存在精度丢失问题，这就是我们要引入decimal类型的原因，decimal类型常见于银行系统、互联网金融系统等对小数点后的数字比较敏感的系统中。

. float/double在db中存储的是近似值，而decimal则是以字符串形式进行保存的
. decimal(M,D)的规则和float/double相同，但区别在float/double在不指定M、D时默认按照实际精度来处理而decimal在不指定M、D时默认为decimal(10, 0)

```

二、日期/时间

|type         | 大小           | 格式  | desc  |
| :-------------: | :---: | :----------: | :----------: |
| date      | 3 | yyyy-MM-dd | 日期 |
| time      | 3      | HH:mm:ss | 时间 |
| year | 1 | yyyy | 年 | 
| datetime | 8 | yyyy-MM-dd HH:mm:ss | 日期+时间 |
| timestamp | 4 | yyyy-MM-dd HH:mm:ss | 日期+时间 |


· datetime与timestamp能存储的时间范围不同，datetime的存储范围为1000-01-01 00:00:00——9999-12-31 23:59:59，timestamp存储的时间范围为19700101080001——20380119111407
· datetime默认值为空，当插入的值为null时，该列的值就是null；timestamp默认值不为空，当插入的值为null的时候，mysql会取当前时间
· datetime存储的时间与时区无关，timestamp存储的时间及显示的时间都依赖于当前时区
· 在实际工作中，一张表往往我们会有两个默认字段，一个记录创建时间而另一个记录最新一次的更新时间，这种时候可以使用timestamp类型来实现：

三、字符串




参考：

https://www.cnblogs.com/xrq730/p/8446246.html
