## 第四章 检索数据
### 4.1 Select语句
使用select语句必须至少给出两条信息：**想要检索什么**及**从什么地方检索**。
### 4.2 检索单个列

* **SELECT prod_name FROM products;** 

**为排序数据**：如果没有明确排序查询结果，返回的数据的顺序没有特护意义。
**sql语句大小写**：sql语句不区分大小写，一般sql关键字使用大写，而对所有列和表名使用小写，已于阅读和调试。
### 4.3检索多个列
**SELECT prod_id,prod_name,prod_price FROM products;**
**数据显示**：SQL语句一般返回原始的、无格式的数据。（数据格式化是一个表示问题，而不是一个检索问题）
### 4.4检索所有列

* **SELECT * FROM products;**

如果给定一个**通配符***，则表示返回表中所有列。列的顺序一般是列在表中定义的顺序。但有时表的模式的变化（如添加或者删除列）可能会导致顺序的变化。
**检索未知列**：使用通配符能检索出名字位置的列。
### 4.5检索不同行

* **SELECT DISTINCT  vend_id FROM products;**

**注**：distinct 不同的, 完全分开的
**DISTINCT**指示MySQL只返回不同（唯一）的值。**它必须放在列名的前面**
**不能部分使用DISTINCT**：DISTINCT关键字应用于所有列而不仅是前置列。如果给出**SELECT DISTINCT  vend_id,prod_price FROM products;** 除非指定的两个列都不同，否则所有行都将被检索出来。
### 4.6限制结果

* **SELECT prod_name FROM products LIMIT 5;**

此语句使用select语句检索单个列。**LIMIT 5**指示MySQL返回不多于5行(前五行，从行0开始的五行，即行0到行4)。

* **SELECT prod_name FROM products LIMIT 5,5;**

**LIMIT 5,5**表示MySQL返回从5行开始的5行（即6到10行，即行5开始到行9），第一个数为开始位置，第二个数为要检索的行数。所以带一个值的**LIMIT**总是从第一行开始，给出的数为返回的行数。带两个值的**LIMIT**表示从指定行号为第一个值的位置开始。
**行0**：检索出的第一行为行0而不是行1。所以**LIMIT 1,1**检索出的是第二行，而不是第一行。
**MySQL 5的LIMIT语法**:MySQL 5支持**LIMIT**的另一种代替语法。**LIMIT 4 OFFSET 3**意为从第三行开始取四行，就像**LIMIT 3,4**一样。（注：offset:抵消,弥补）
### 4.7使用完全限定表名

* **SELECT products.prod_name FROM products ;**

* **SELECT products.prod_name FROM crashcourse.products ;**
  其中crashcourse为数据库名，products为表名。
## 第五章 排序检索
  本章介绍select语句的order by 子句，根据需要排序检索出的数据。
### 5.1排序数据
 其实检索出的数据并不是以纯粹的随机顺序显示的。如果不排序，数据一般以他在底层中出现的顺序显示。可以是数据最初添加到表中的数据。如果后来进行过更新或删除，则此顺序将会受到MySQL重用回收存储空间的影响。因此不能（也不应该）依赖该排序顺序。关系数据库设计理论认为，**如果不明确规定排序顺序，则不应该假定检索出的数据的数据的顺序有意义**。
**子句（clause）**:sql语句有子句构成，一个子句通常由一个关键字和所提供的数据组成，子句的例子有SELECT语句的FROM子句。
* **ORDER BY**子句取一个或者多个列的名字，据此对输出进行排序。
* **SELECT prod_name FROM products ORDER BY prod_name;**

这表语句表示MySQL对prod_name列以字母顺序排序数据。
**通过非选择列进行排序**：即要指定顺序的列可以是索要显示的列，也可以是非选择的列（不显示的列）。
### 5.2按多个列排序

*  **SELECT prod_id,prod_price,prod_name 
    FROM products
    ORDER BY prod_price,prod_name;**
*  **多个列排序时，排序顺序完全按所规定的顺序进行**。此sql中按两个列的的结果进行排序---首先按照**prod_price**排序，然后再按**prod_name**排序（即当**prod_price**相同时再按**prod_name**进行排序）。
### 5.3指定排序的方向
数据排序不仅可以按升序排序（从A到Z）**这是默认的排序顺序**,还可以使用**ORDER BY** 子句以降序（从Z到A）排序。为了进行降序排序，必须指定**DESC**关键字。**注**：desc:降序排列（descend 的缩写）
* **SELECT prod_id,prod_price,prod_name 
  FROM products
  ORDER BY prod_price DESC;**

如果打算用多个列排列时怎么办呢？解决如下
* **SELECT prod_id,prod_price,prod_name 
  FROM products
  ORDER BY prod_price DESC，prod_name**

**DESC**关键字只作用于其前面的列名。

**在多个列上降序排序**：在多个列上进行降序排序，必须在每个列上指定**DESC**关键字。

于**DESC**相反的关键字是**ASC**（ASCENDING）,在升序时可以指定它，但因为默认为升序**ASC**，因此一般无需指定。

**区分大小写和排序顺序**：在字典排序中，A被视为与a相同，这是MySQL（和大多数数据库管理系统）的默认行为。

使用**ORDER BY**与**LIMIT**的组合，能够找出一个列中最高或最低的值。
* **SELECT prod_price
  FROM products
  ORDER BY prod_price DESC
  LIMIT 1;**

**ORDER BY子句的位置**：必须在**FROM**子句之后，如果只用**LIMIT**，它必须在**LIMIT**子句之前。
## 第六章 数据过滤
### 6.1使用WHERE子句
* **SELECT prod_name,prod_price
  FROM products 
  WHERE prod_price=2.50;**

**WHERE子句的位置**：同时使用**ORDER ＢY**和**WHERE**子句时，应该让**ORDER BY**位于**WHERE**之后，否则将会有错误。
### 6.2WHERE子句操作符
**WHERE子句操作符**
| 操作符      | 说明               |
| ----------- | ------------------ |
| =           | 等于               |
| <>          | 不等于             |
| !=          | 不等于             |
| <           | 小于               |
| <=          | 小于等于           |
| >           | 大于               |
| >=          | 大于等于           |
| **BETWEEN** | 指定两个值之间的值 |

**MYSQL在执行匹配时默认不区分大小写，所以fuses与FUSes一样**

* **不匹配检索
  SELECT vend_id,prod_name
  FROM products 
  WHERE vend_id<>1003;**

 **如何使用单引号**：单引号用来限定字符串，如果将值与串类型的列进行比较，则需要限定引号，否则不需要。

 * **范围值检查
    SELECT prod_name,prod_price
    FROM products 
    WHERE prod_price BETWEEN 5 AND 10;**
    在使用**BETWEEN**时必须指定两个值——所需范围的低端值和高端值，这两个值必须用**AND**连接，**BETWEEN匹配范围内的 所有值，包括指定的 开始值 和 结束值**。

**NULL** 无值（no value),他与字段包含0、空字符串或仅仅包含空格不同。
* **空值检查
  SELECT prod_name
  FROM products
  WHERE prod_price IS NULL;**
  这条语句返回没有价格（空prod_price字段，不是价格为0）的所有产品的名字
## 第七章 数据过滤
### 7.1组合WHERE子句
***操作符***：用来联结或改变WHERE子句中的子句的关键字。也成为逻辑操作符。
#### 7.1.1 AND操作符
***AND***：用来指示检索满足所有给定的条件。
* **SELECT prod_id,prod_price,prod_name
  FROM products
  WHERE vend_id = 1003 AND prod_price < 10;**
  此sql语句检索由供应商1003制造且价格小于等于10美元的所有产品的名称和价格。
#### 7.1.2 OR操作符
***OR***：用来表示检索匹配任一给定条件的行。
* **SELECT prod_price,prod_name
  FROM products
  WHERE vend_id = 1002 OR vend_id = 1003;**
  此语句检索由任一个指定供应商的所有产品的产品名和价格。
### 7.1.3计算次序
**SQL语言在处理OR操作符前，优先处理AND操作符，（AND在计算次序中优先级更高）**
* 列出价格为10美元（含）以上且由1002或1003制造的所有产品
1.  错误的sql
     * **SELECT vend_id,prod_name,prod_price
        FROM products
        WHERE vend_id = 1002 OR vend_id = 1003 AND prod_price >= 10;**

2.  正确的sql
    * **SELECT vend_id,prod_name,prod_price
      FROM products
      WHERE (vend_id = 1002 OR vend_id = 1003) AND prod_price >= 10;**

**在WHERE语句中使用圆括号**：任何时候使用具有AND和OR操作符的WHERE子句，都应该使用圆括号明确的分明。使用圆括号没有坏处，它能消除歧义。
### 7.2 IN操作符
***IN*** 用来指定匹配值的清单的关键字，功能与OR相当。（IN操作符用来指定条件范围，范围中的每个条件都可以进行匹配）
* **SELECT prod_name,prod_price
  FROM products 
  WHERE vend_id IN (1002,1003)
  ORDER BY prod_name;**
  此select语句检索供应商1002和1003制造的所有产品，并根据prod_name进行升序排序。

* **使用IN操作符的优点**
    1. 在使用长的合法清单时，IN操作符的语法更清楚且更直观。
    2. 在使用IN时，计算的次序易于管理。
    3. IN操作符一般比OR操作符清单执行更快。
    4. IN最大的优点是可以包含其他select语句，使的能够更动态地建立WHERE子句。
### 7.3 NOT操作符
***NOT***：用来否定后跟条件的关键字。
* **SELECT prod_name,prod_price
  FROM products 
  WHERE vend_id NOT IN (1002,1003)
  ORDER BY prod_name;**
  此语句表示匹配1002和1003之外供应商的产品并显示产品名和价格。
## 第八章 用通配符进行过滤
### 8.1 LIKE操作符
***通配符（wildcard）*** :用来匹配值的一部分的特殊字符。
***搜索模式（searchpattern）*** :由字面值、通配符或两者组合构成的搜索条件。
**谓词**：操作符何时不是操作符？答案是它在作为谓词时（predicate）时。从技术上说，**LIKE**是谓词而不是操作符。
### 8.2百分号（%）通配符
在搜索串中，**%表示任何字符出现任意次数**。
* 找出所有以词jet起头的产品的 
  **SELECT prod_id,prod_name
  FROM products
  WHERE prod_name LIKE 'jet%';**
  此例子使用了搜索模式 'jet% ',执行时将检索jet起头的词。%告诉MySQL接受jet之后的任意字符，不管它有多少字符。

**%代表搜索模式中给定的0个、1个或多个字符**。

**注意NULL**：虽然似乎%通配符可以匹配任何东西，但有一个例外NULL;及时是**WHERE prod_name LIKE '%'**也不能匹配用值null作为产品名的行。
### 8.1.2下划线（_）通配符
**_通配符**：只能匹配单个字符。
sql1
* **SELECT prod_id,prod_name
  FROM products
  WHERE prod_name LIKE '_ ton anvil';**
  sql2
* **SELECT prod_id,prod_name
  FROM products
  WHERE prod_name LIKE '% ton anvil';**

sel1与sql2是不对等的，_只能匹配一个字符，不能多也不能少。
### 8.2使用通配符的技巧
MySQL的通配符虽然很有用，但是也是有代价的：通配符搜索的处理一般要比前面讨论的其他搜索所花的时间更长。这里给出通配符使用的的一些技巧：

1. 不要过度使用通配符。如果其他操作符能达到相同的目的，应该使用其他操作符。
2. 在确实需要使用通配符时，除非绝对有必要，否则不要把它们用在搜索模式的开始处。把通配符置于搜索模式的开始处，搜索起来是最慢的。
3. 仔细注意通配符的位置。如果放错地方，可能不会返回想要的数据。
## 第九章 用正则表达式进行搜索
### 9.2使用MySQL正则表达式
#### 9.2.1 基本字符匹配
* **SELECT prod_name 
  FROM products 
  WHERE prod_name REGEXP '.000'
  ORDER BY prod_name;**
  **REGEXP**后跟的为正则表达式，**.** 是正则表达式语言中一个特殊的字符。表示匹配任意一个字符。

**LIKE**是匹配整个行，**REGEXP**匹配行内值，即匹配的值在行内存在即可，即部分匹配整体。
#### 9.2.2进行OR匹配
* **SELECT prod_name 
  FROM products 
  WHERE prod_name REGEXP '1000|2000'
  ORDER BY prod_name;**
  **|** 操作符为正则表达式的OR操作符，可以改写有多个OR的select语句。
#### 9.2.3匹配几个字符之一
* **SELECT prod_name 
  FROM products 
  WHERE prod_name REGEXP '[123] Ton'
  ORDER BY prod_name;**
  **[123]** 定义一组字符，他的意思是匹配1或2或3.


## 第十九章 插入数据
### 19.1数据插入
**INSERT**是用来插入（或添加）行到数据库表中的。插入可以分为以下集中

1. 插入完整行。
2. 插入行的一部分。
3. 插入多行。
4. 插入某些查询结果。
### 19.2插入完整的行
插入完整的行要求指定**表名**和**插入到新行中的值**。
* sql
  **INSERT INTO customers
  VALUES(NULL,
  'Pep E. LaPew',
  '100 Main Street',
  'Los Angles',
  'CA',
  '90046',
  'USA',
  'null',
  'null');**

* **仔细的给出值**：不管使用哪种INSERT语法，都必须各处VALUES的正确数目。如果不提供列名，则必须给每个表列提供一个值。如果提供列名，则必须对每个列出的列给一个值。如果不知这样，将产生一条错误的消息，相应的行插入不如不成功。

* **省略列**：如果表的定义允许，则可以在insert操作中省略某些列。省略的列必须满足以下某个条件：
  1. 该列定义为允许NULL值（无值或空值）。
  2. 在表定义中给出默认值。这表示如果不给出值，将使用默认值。
      如果对表中不允许NULL值且没有默认值的列不给出值，则MySQL将产生一条错误，并且相应的行插入不成功。

 * **提高整体性能**：数据库经常被多个用户访问，对处理什么请求以及用什么次序进行管理是MySQL的任务。INSERT操作可能耗时（特别是有很多索引需要更行时时），而且它可能降低等待处理的SELECT语句的性能。
    如果数据检索是重要的（通常是这样），则你可以通过在INSER和INTO之间添加关键字**LOW_PRIORITY**,指示MySQL降低INSERT语句的优先级，如下所示：
     **INSERT LOW_PRIORITY INTO**
     ps:这也适用于**UPDATE**和**DELETE**语句。
 ### 19.3插入多个行
  1. 插入多行时可以使用多条**INSERT**语句，每条语句用分号结束，一次提交。
  2. 只要将每条**INSERT**语句中的列名（和次序）相同，可以如下组合各条语句：
     **INSERT INTO customers
     VALUES(
     'Pep E. LaPew',
     '100 Main Street',
     'Los Angles',
     'CA',
     '90046',
     'USA',
     'null',
     'null'),
     (
     'Pep E. LaPew',
     '100 Main Street',
     'Los Angles',
     'CA',
     '90046',
     'USA',
     'null',
     'null');**

    * **提高INSERT语句性能**：此技术可以提高数据库的性能，因为MySQL用单条INSERT语句处理多个插入比使用多条INSERT语句快。
 ### 19.4插入检索出的数据
 **INSERT SELECT**语句