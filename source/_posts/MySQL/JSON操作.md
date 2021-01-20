---
title: SQL语句操作JSON字段
tags:
  - 数据库
  - MySql
  - JSON
copyright: true
abbrlink: 26307
date: 2020-09-09 23:35:15
---



Mysql5.7版本以后新增的功能，Mysql提供了一个原生的Json类型，Json值将不再以字符串的形式存储，而是采用一种允许快速读取文本元素（document elements）的内部二进制（internal binary）格式。在Json列插入或者更新的时候将会自动验证Json文本，未通过验证的文本将产生一个错误信息。Json文本采用标准的创建方式，可以使用大多数的比较操作符进行比较操作，例如：=, <, <=, >, >=, <>, != 和 <=>。

# MySQL JSON相关函数

MySQL官方列出json相关的函数，完整列表如下:

| 分类                                                         | 函数                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| [创建json](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-creation-functions.html) | [json_array](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-creation-functions.html#function_json-array) | 创建json数组                                                 |
|                                                              | [json_object](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-creation-functions.html#function_json-object) | 创建json对象                                                 |
|                                                              | [json_quote](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-creation-functions.html#function_json-quote) | 将json转成json字符串类型                                     |
| [查询json](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html) | [json_contains](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html#function_json-contains) | 判断是否包含某个json值                                       |
|                                                              | [json_contains_path](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html#function_json-contains-path) | 判断某个路径下是否包json值                                   |
|                                                              | [json_extract](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html#function_json-extract) | 提取json值                                                   |
|                                                              | [column->path](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html#operator_json-column-path) | json_extract的简洁写法，MySQL 5.7.9开始支持                  |
|                                                              | [column->>path](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html#operator_json-inline-path) | json_unquote(column -> path)的简洁写法                       |
|                                                              | [json_keys](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html#function_json-keys) | 提取json中的键值为json数组                                   |
|                                                              | [json_search](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html#function_json-search) | 按给定字符串关键字搜索json，返回匹配的路径                   |
| [修改json](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html) | [json_append](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-append) | 废弃，MySQL 5.7.9开始改名为json_array_append                 |
|                                                              | [json_array_append](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-array-append) | 末尾添加数组元素，如果原有值是数值或json对象，则转成数组后，再添加元素 |
|                                                              | [json_array_insert](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-array-insert) | 插入数组元素                                                 |
|                                                              | [json_insert](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-insert) | 插入值（插入新值，但不替换已经存在的旧值）                   |
|                                                              | [json_merge](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-merge) | 合并json数组或对象                                           |
|                                                              | [json_remove](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-remove) | 删除json数据                                                 |
|                                                              | [json_replace](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-replace) | 替换值（只替换已经存在的旧值）                               |
|                                                              | [json_set](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-set) | 设置值（替换旧值，并插入不存在的新值）                       |
|                                                              | [json_unquote](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-unquote) | 去除json字符串的引号，将值转成string类型                     |
| [返回json属性](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-attribute-functions.html) | [json_depth](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-attribute-functions.html#function_json-depth) | 返回json文档的最大深度                                       |
|                                                              | [json_length](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-attribute-functions.html#function_json-length) | 返回json文档的长度                                           |
|                                                              | [json_type](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-attribute-functions.html#function_json-type) | 返回json值得类型                                             |
|                                                              | [json_valid](https://link.jianshu.com/?t=http://dev.mysql.com/doc/refman/5.7/en/json-attribute-functions.html#function_json-valid) | 判断是否为合法json文档                                       |

在Mysql5.7版本及之后的版本可以使用column->path作为JSON_EXTRACT(column, path)的快捷方式。这个函数可以作为列数据的别名出现在SQL语句中的任意位置，包括WHERE,ORDER BY,和GROUP BY语句。同样包含SELECT， UPDATE， DELETE，CREATE TABLE和其他SQL语句。->左边的参数为JSON数据的列名而不是一个表达式，其右边参数JSON数据中的某个路径表达式。

<!--more-->

# MySQL JOSN相关函数语法

## 一、创建JSON值的函数

### json_array  创建json数组

计算（可能为空）值列表并返回包含这些值的JSON数组。

```language-sql
mysql> SELECT JSON_ARRAY(1, "abc", NULL, TRUE, CURTIME());
+---------------------------------------------+
| JSON_ARRAY(1, "abc", NULL, TRUE, CURTIME()) |
+---------------------------------------------+
| [1, "abc", null, true, "11:30:24.000000"]   |
+---------------------------------------------+
```

### JSON_OBJECT  创建json对象

计算（可能为空）键 - 值对列表,并返回包含这些对的JSON对象.如果任何键名称`NULL`或参数数量为奇数,则会发生错误。

```language-sql
mysql> SELECT JSON_OBJECT('id', 87, 'name', 'carrot');
+-----------------------------------------+
| JSON_OBJECT('id', 87, 'name', 'carrot') |
+-----------------------------------------+
| {"id": 87, "name": "carrot"}            |
+-----------------------------------------+
```

###  JSON_QUOTE  将json转成json字符串类型

通过用双引号字符包装并转义内部引号和其他字符，然后将结果作为`utf8mb4`字符串返回，将字符串引用为JSON值 。`NULL`如果参数是，则 返回 `NULL`。

此函数通常用于生成有效的JSON字符串文字以包含在JSON文档中。

```
mysql> SELECT JSON_QUOTE('null'), JSON_QUOTE('"null"');
+--------------------+----------------------+
| JSON_QUOTE('null') | JSON_QUOTE('"null"') |
+--------------------+----------------------+
| "null"             | "\"null\""           |
+--------------------+----------------------+
mysql> SELECT JSON_QUOTE('[1, 2, 3]');
+-------------------------+
| JSON_QUOTE('[1, 2, 3]') |
+-------------------------+
| "[1, 2, 3]"             |
+-------------------------+
```

## 二、搜索JSON值的函数

### JSON_CONTAINS()   判断是否包含某个json值

- 语法： 

  `JSON_CONTAINS(json_doc, val[, path]) `

- 说明： 

  - 返回0或者1来表示目标JSON文本中是否包含特定值，或者JSON文本的指定路径下是否包含特定值。
  - 以下情况将返回NULL: 
    - 目标JSON文本或者特定值为NULl
    - 指定路径非目标JSON文本下的路径
  - 以下情况将报错： 
    - 目标JSON文本不合法
    - 指定路径不合法
    - 包含* 或者 ** 匹配符
  - 若仅检查路径是否存在，使用JSON_CONTAINS_PATH()代替
  - 这个函数中做了以下约定： 
    - 当且仅当两个标量可比较而且相等时，约定目标表标量中包含候选标量。两个标量的JSON_TYPE()值相同时约定他们是可比较的，另外类型分别为INTEGER和DECEMAL的两个标量也是可比较的
    - 当且仅当目标数组中包含所有的候选数组元素，约定目标数组包含候选数组
    - 当且仅当目标数组中某些元素包含空数组，约定目标数组包含空数组
    - 当且仅当候选对象中所有的键值都能在目标对象中找到相同名称的键而且候选键值被目标键值包含，约定目标对象包含候选对象
    - 其他的情况均为目标文本不包含候选文本

- 示例： 

  ```
  mysql> SET @j = '{"a": 1, "b": 2, "c": {"d": 4}}';
  mysql> SET @j2 = '1';
  mysql> SELECT JSON_CONTAINS(@j, @j2, '$.a');
  +-------------------------------+
  | JSON_CONTAINS(@j, @j2, '$.a') |
  +-------------------------------+
  |                             1 |
  +-------------------------------+
  mysql> SELECT JSON_CONTAINS(@j, @j2, '$.b');
  +-------------------------------+
  | JSON_CONTAINS(@j, @j2, '$.b') |
  +-------------------------------+
  |                             0 |
  +-------------------------------+
   
  mysql> SET @j2 = '{"d": 4}';
  mysql> SELECT JSON_CONTAINS(@j, @j2, '$.a');
  +-------------------------------+
  | JSON_CONTAINS(@j, @j2, '$.a') |
  +-------------------------------+
  |                             0 |
  +-------------------------------+
  mysql> SELECT JSON_CONTAINS(@j, @j2, '$.c');
  +-------------------------------+
  | JSON_CONTAINS(@j, @j2, '$.c') |
  +-------------------------------+
  |                             1 |
  +-------------------------------+
  ```

  ### JSON_CONTAINS_PATH()   判断某个路径下是否包json值

- 语法： 

  `JSON_CONTAINS_PATH(json_doc, one_or_all, path[, path] ...) `

- 说明： 

  - 返回0或者1表示JSON文本的指定的某个路径或者某些路径下是否包含特定值。
  - 当某些参数为NULL是否返回NULL
  - 以下情况将报错： 
    - 参数json_doc为不合法JSON文本
    - path参数中包含不合法的路径
    - one_or_all参数为非’one’或者’all’的值
  - 检测某个路径中是否包含某个特定值，使用 JSON_CONTAINS()代替
  - 目标文本中如果没有指定的路径，则返回0。否则，返回值依赖于one_or_all值： 
    - ’one’: 文本中存在至少一个指定路径则返回1，否则返回0
    - ‘all’: 文本中包含所有指定路径则返回1， 否则返回0

- 示例： 

  ```
  mysql> SET @j = '{"a": 1, "b": 2, "c": {"d": 4}}';
  mysql> SELECT JSON_CONTAINS_PATH(@j, 'one', '$.a', '$.e');
  +---------------------------------------------+
  | JSON_CONTAINS_PATH(@j, 'one', '$.a', '$.e') |
  +---------------------------------------------+
  |                                           1 |
  +---------------------------------------------+
  mysql> SELECT JSON_CONTAINS_PATH(@j, 'all', '$.a', '$.e');
  +---------------------------------------------+
  | JSON_CONTAINS_PATH(@j, 'all', '$.a', '$.e') |
  +---------------------------------------------+
  |                                           0 |
  +---------------------------------------------+
  mysql> SELECT JSON_CONTAINS_PATH(@j, 'one', '$.c.d');
  +----------------------------------------+
  | JSON_CONTAINS_PATH(@j, 'one', '$.c.d') |
  +----------------------------------------+
  |                                      1 |
  +----------------------------------------+
  mysql> SELECT JSON_CONTAINS_PATH(@j, 'one', '$.a.d');
  +----------------------------------------+
  | JSON_CONTAINS_PATH(@j, 'one', '$.a.d') |
  +----------------------------------------+
  |                                      0 |
  +----------------------------------------+
  ```

  ### JSON_EXTRACT()   提取json值

- 语法： 

  `JSON_EXTRACT(json_doc, path[, path] ...) `

- 说明： 

  - 返回json_doc中与path参数相匹配的数据。当有参数为NULl或者文本中未找到指定path时将返回NULL。当参数不合法时将报错。
  - 返回结果包含所有与path匹配的值。如果返回多个值，则将自动包装为数组，其顺序为匹配顺序；相反则返回单个匹配值。
  - MySQL5.7.9及之后的版本将支持’->’操作符作为本函数两个参数时的便捷写法。->左边的参数为JSON数据的列名而不是一个表达式，其右边参数JSON数据中的某个路径表达式。详细使用方法将在文末详细阐述。

- 示例： 

  ```
  mysql> SELECT JSON_EXTRACT('[10, 20, [30, 40]]', '$[1]');
  +--------------------------------------------+
  | JSON_EXTRACT('[10, 20, [30, 40]]', '$[1]') |
  +--------------------------------------------+
  | 20                                         |
  +--------------------------------------------+
  mysql> SELECT JSON_EXTRACT('[10, 20, [30, 40]]', '$[1]', '$[0]');
  +----------------------------------------------------+
  | JSON_EXTRACT('[10, 20, [30, 40]]', '$[1]', '$[0]') |
  +----------------------------------------------------+
  | [20, 10]                                           |
  +----------------------------------------------------+
  mysql> SELECT JSON_EXTRACT('[10, 20, [30, 40]]', '$[2][*]');
  +-----------------------------------------------+
  | JSON_EXTRACT('[10, 20, [30, 40]]', '$[2][*]') |
  +-----------------------------------------------+
  | [30, 40]                                      |
  +-----------------------------------------------+
  ```

   

 

### CLOUMN`->`PATH  json_extract的简洁写法，MySQL 5.7.9开始支持

- 语法： 

  CLOUMN`->`PATH`(`c->"$.id"`) `

- 说明： 

  - 当与两个参数一起使用时， `->` 运算符用作[`JSON_EXTRACT()`](https://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html#function_json-extract)函数的别名， 左边是列标识符，右边是JSON路径，根据JSON文档（列值）计算。您可以在SQL语句中的任何位置使用此类表达式代替列标识符
  - 返回结果包含所有与path匹配的值。如果返回多个值，则将自动包装为数组，其顺序为匹配顺序；相反则返回单个匹配值。
  - MySQL5.7.9及之后的版本将支持’->’操作符作为本函数两个参数时的便捷写法。->左边的参数为JSON数据的列名而不是一个表达式，其右边参数JSON数据中的某个路径表达式。详细使用方法将在文末详细阐述。

- `SELECT`

  这里显示 的两个语句产生相同的输出：

  ```
  mysql> SELECT c, JSON_EXTRACT(c, "$.id"), g
       > FROM jemp
       > WHERE JSON_EXTRACT(c, "$.id") > 1
       > ORDER BY JSON_EXTRACT(c, "$.name");
  +-------------------------------+-----------+------+
  | c                             | c->"$.id" | g    |
  +-------------------------------+-----------+------+
  | {"id": "3", "name": "Barney"} | "3"       |    3 |
  | {"id": "4", "name": "Betty"}  | "4"       |    4 |
  | {"id": "2", "name": "Wilma"}  | "2"       |    2 |
  +-------------------------------+-----------+------+
  3 rows in set (0.00 sec)
   
  mysql> SELECT c, c->"$.id", g
       > FROM jemp
       > WHERE c->"$.id" > 1
       > ORDER BY c->"$.name";
  +-------------------------------+-----------+------+
  | c                             | c->"$.id" | g    |
  +-------------------------------+-----------+------+
  | {"id": "3", "name": "Barney"} | "3"       |    3 |
  | {"id": "4", "name": "Betty"}  | "4"       |    4 |
  | {"id": "2", "name": "Wilma"}  | "2"       |    2 |
  +-------------------------------+-----------+------+
  3 rows in set (0.00 sec)
  ```

   

- 此功能不限 `SELECT`于此，如下所示：

  ```
  mysql> ALTER TABLE jemp ADD COLUMN n INT;
  Query OK, 0 rows affected (0.68 sec)
  Records: 0  Duplicates: 0  Warnings: 0
   
  mysql> UPDATE jemp SET n=1 WHERE c->"$.id" = "4";
  Query OK, 1 row affected (0.04 sec)
  Rows matched: 1  Changed: 1  Warnings: 0
   
  mysql> SELECT c, c->"$.id", g, n
       > FROM jemp
       > WHERE JSON_EXTRACT(c, "$.id") > 1
       > ORDER BY c->"$.name";
  +-------------------------------+-----------+------+------+
  | c                             | c->"$.id" | g    | n    |
  +-------------------------------+-----------+------+------+
  | {"id": "3", "name": "Barney"} | "3"       |    3 | NULL |
  | {"id": "4", "name": "Betty"}  | "4"       |    4 |    1 |
  | {"id": "2", "name": "Wilma"}  | "2"       |    2 | NULL |
  +-------------------------------+-----------+------+------+
  3 rows in set (0.00 sec)
   
  mysql> DELETE FROM jemp WHERE c->"$.id" = "4";
  Query OK, 1 row affected (0.04 sec)
   
  mysql> SELECT c, c->"$.id", g, n
       > FROM jemp
       > WHERE JSON_EXTRACT(c, "$.id") > 1
       > ORDER BY c->"$.name";
  +-------------------------------+-----------+------+------+
  | c                             | c->"$.id" | g    | n    |
  +-------------------------------+-----------+------+------+
  | {"id": "3", "name": "Barney"} | "3"       |    3 | NULL |
  | {"id": "2", "name": "Wilma"}  | "2"       |    2 | NULL |
  +-------------------------------+-----------+------+------+
  2 rows in set (0.00 sec)
  ```

   

- 这也适用于JSON数组值，如下所示：

  ```
  mysql> CREATE TABLE tj10 (a JSON, b INT);
  Query OK, 0 rows affected (0.26 sec)
   
  mysql> INSERT INTO tj10
       > VALUES ("[3,10,5,17,44]", 33), ("[3,10,5,17,[22,44,66]]", 0);
  Query OK, 1 row affected (0.04 sec)
   
  mysql> SELECT a->"$[4]" FROM tj10;
  +--------------+
  | a->"$[4]"    |
  +--------------+
  | 44           |
  | [22, 44, 66] |
  +--------------+
  2 rows in set (0.00 sec)
   
  mysql> SELECT * FROM tj10 WHERE a->"$[0]" = 3;
  +------------------------------+------+
  | a                            | b    |
  +------------------------------+------+
  | [3, 10, 5, 17, 44]           |   33 |
  | [3, 10, 5, 17, [22, 44, 66]] |    0 |
  +------------------------------+------+
  2 rows in set (0.00 sec)
  ```

   

- 支持嵌套数组。使用的表达式 `->`计算`NULL` 好像在目标JSON文档中找不到匹配的键，如下所示：

  ```
  mysql> SELECT * FROM tj10 WHERE a->"$[4][1]" IS NOT NULL;
  +------------------------------+------+
  | a                            | b    |
  +------------------------------+------+
  | [3, 10, 5, 17, [22, 44, 66]] |    0 |
  +------------------------------+------+
  mysql> SELECT a->"$[4][1]" FROM tj10;
  +--------------+
  | a->"$[4][1]" |
  +--------------+
  | NULL         |
  | 44           |
  +--------------+
  2 rows in set (0.00 sec)
  ```

   

- 这与使用时的情况相同 `JSON_EXTRACT()`：

  ```
  mysql> SELECT JSON_EXTRACT(a, "$[4][1]") FROM tj10;
  +----------------------------+
  | JSON_EXTRACT(a, "$[4][1]") |
  +----------------------------+
  | NULL                       |
  | 44                         |
  +----------------------------+
  2 rows in set (0.00 sec)
  ```

  ### COLUMN->>PATH   json_unquote(column -> path)的简洁写法

  这是一个改进的，不引用的提取操作符，可在MySQL 5.7.13及更高版本中使用。而 `->`操作者简单地提取的值时， `->>`在加法运算unquotes所提取的结果。换句话说，给定 [`JSON`](https://dev.mysql.com/doc/refman/5.7/en/json.html)列值 *`column`*和路径表达式 *`path`*，以下三个表达式返回相同的值：

- [`JSON_UNQUOTE(`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-unquote) [`JSON_EXTRACT(*`column`*, *`path`*) )`](https://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html#function_json-extract)

- `JSON_UNQUOTE(*`column`*` [`->`](https://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html#operator_json-column-path) `*`path`*)`

- `->>`只要`JSON_UNQUOTE(JSON_EXTRACT())`允许 ，操作员就可以使用 。这包括（但不限于） `SELECT`列表，`WHERE`和 `HAVING`条款，并`ORDER BY`和`GROUP BY`条款。

- 演示`->>在**mysql**`客户端中其他表达式的一些 运算符等价：

  ```
  mysql> SELECT * FROM jemp WHERE g > 2;
  +-------------------------------+------+
  | c                             | g    |
  +-------------------------------+------+
  | {"id": "3", "name": "Barney"} |    3 |
  | {"id": "4", "name": "Betty"}  |    4 |
  +-------------------------------+------+
  2 rows in set (0.01 sec)
   
  mysql> SELECT c->'$.name' AS name
      ->     FROM jemp WHERE g > 2;
  +----------+
  | name     |
  +----------+
  | "Barney" |
  | "Betty"  |
  +----------+
  2 rows in set (0.00 sec)
   
  mysql> SELECT JSON_UNQUOTE(c->'$.name') AS name
      ->     FROM jemp WHERE g > 2;
  +--------+
  | name   |
  +--------+
  | Barney |
  | Betty  |
  +--------+
  2 rows in set (0.00 sec)
   
  mysql> SELECT c->>'$.name' AS name
      ->     FROM jemp WHERE g > 2;
  +--------+
  | name   |
  +--------+
  | Barney |
  | Betty  |
  +--------+
  2 rows in set (0.00 sec)
  ```

   

- 此运算符也可以与JSON数组一起使用，如下所示：

  ```
  mysql> CREATE TABLE tj10 (a JSON, b INT);
  Query OK, 0 rows affected (0.26 sec)
   
  mysql> INSERT INTO tj10 VALUES
      ->     ('[3,10,5,"x",44]', 33),
      ->     ('[3,10,5,17,[22,"y",66]]', 0);
  Query OK, 2 rows affected (0.04 sec)
  Records: 2  Duplicates: 0  Warnings: 0
  mysql> SELECT a->"$[3]", a->"$[4][1]" FROM tj10;
  +-----------+--------------+
  | a->"$[3]" | a->"$[4][1]" |
  +-----------+--------------+
  | "x"       | NULL         |
  | 17        | "y"          |
  +-----------+--------------+
  2 rows in set (0.00 sec)
   
  mysql> SELECT a->>"$[3]", a->>"$[4][1]" FROM tj10;
  +------------+---------------+
  | a->>"$[3]" | a->>"$[4][1]" |
  +------------+---------------+
  | x          | NULL          |
  | 17         | y             |
  +------------+---------------+
  2 rows in set (0.00 sec)
  ```

   

- 与此同时 [`->`](https://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html#operator_json-column-path)，`->>`运算符总是在输出中展开[`EXPLAIN`](https://dev.mysql.com/doc/refman/5.7/en/explain.html)，如下例所示：

  ```
  mysql> EXPLAIN SELECT c->>'$.name' AS name
      ->     FROM jemp WHERE g > 2\G
  *************************** 1. row ***************************
             id: 1
    select_type: SIMPLE
          table: jemp
     partitions: NULL
           type: range
  possible_keys: i
            key: i
        key_len: 5
            ref: NULL
           rows: 2
       filtered: 100.00
          Extra: Using where
  1 row in set, 1 warning (0.00 sec)
   
  mysql> SHOW WARNINGS\G
  *************************** 1. row ***************************
    Level: Note
     Code: 1003
  Message: /* select#1 */ select
  json_unquote(json_extract(`jtest`.`jemp`.`c`,'$.name')) AS `name` from
  `jtest`.`jemp` where (`jtest`.`jemp`.`g` > 2)
  1 row in set (0.00 sec)
  ```

  这类似于MySQL [`->`](https://dev.mysql.com/doc/refman/5.7/en/json-search-functions.html#operator_json-column-path) 在相同情况下扩展 运算符的方式。

  该`->>`操作符已添加到MySQL 5.7.13中。

### JSON_KEYS()  提取json中的键值为json数组

- 语法： 

  `JSON_KEYS(json_doc[, path]) `

- 说明： 

  - 返回JSON对象的顶层目录的所有key值或者path指定路径下的顶层目录的所有key所组成的JSON数组。
  - 以下情况返回NULL 
    - 必填参数为NULL
    - json_doc非对象（为数组等）
    - 当给定path，但是在JSON中未找到
  - 以下情况报错 
    - json_doc为不合法的JSON文本
    - path为不合法的路径表达
    - 包含 * 或者 ** 通配符
  - 当目标对象为空时，返回值为空。返回结果不包含顶层目录下的嵌套的目录中的key

- 示例： 

  ```
  mysql> SELECT JSON_KEYS('{"a": 1, "b": {"c": 30}}');
  +---------------------------------------+
  | JSON_KEYS('{"a": 1, "b": {"c": 30}}') |
  +---------------------------------------+
  | ["a", "b"]                            |
  +---------------------------------------+
  mysql> SELECT JSON_KEYS('{"a": 1, "b": {"c": 30}}', '$.b');
  +----------------------------------------------+
  | JSON_KEYS('{"a": 1, "b": {"c": 30}}', '$.b') |
  +----------------------------------------------+
  | ["c"]                                        |
  +----------------------------------------------+
  ```

  ### JSON_SEARCH()   按给定字符串关键字搜索json，返回匹配的路径

- 语法： 

  `JSON_SEARCH(json_doc, one_or_all, search_str[, escape_char[, path] ...]) `

- 说明： 

  - 返回JSON包含指定字符串的路径。
  - 以下情况将返回NULL 
    - json_doc, search_str 或path 为NULL
    - 文本中不包含path
    - search_str未找到
  - 以下情况将报错 
    - json_doc不合法
    - path不合法
    - one_or_all 不是one 或者all
    - escape_char 不是一个常量表达式
  - one_or_all的作用 
    - ’one’:当查找操作找到第一个匹配对象，并将结果路径返回后就停止查找。
    - ‘all’:将返回所有的匹配结果路径，结果中不包含重复路径。如果返回结果集中包含多个字符串，将自动封装为一个数组，元素的排序顺序无意义。
  - 在search_str中，通配符’%’和’*‘可以如同LIKE操作上一样运行。’%’可以匹配多个字符（包括0个字符），’*‘则仅可匹配一个字符。
  - ‘%’或’_’作为特殊字符出现时，需要使用转义字符进行转义。当escape_char参数为NULL或者不存在的情况下，系统默认使用’\’作为转义字符。escape_char参数必须要常量（为空或者一个字符）
  - 对于通配符的处理上与LIKE操作不同之处在于，JSON_SEARCH()中的通配符在编译时的计算结果必须要是常量，而不像LIKE仅需在执行时为常量。例如：在prepared Statement中使用JSON_SEARCH(), escape_char参数使用’?’作为参数，那么这个参数在执行时可能是常量，而不是在编译的时候。（这句话自己也没怎么懂，想了很久没想到该怎么翻译）

- 示例： 

  ```
  mysql> SET @j = '["abc", [{"k": "10"}, "def"], {"x":"abc"}, {"y":"bcd"}]';
   
  mysql> SELECT JSON_SEARCH(@j, 'one', 'abc');
  +-------------------------------+
  | JSON_SEARCH(@j, 'one', 'abc') |
  +-------------------------------+
  | "$[0]"                        |
  +-------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', 'abc');
  +-------------------------------+
  | JSON_SEARCH(@j, 'all', 'abc') |
  +-------------------------------+
  | ["$[0]", "$[2].x"]            |
  +-------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', 'ghi');
  +-------------------------------+
  | JSON_SEARCH(@j, 'all', 'ghi') |
  +-------------------------------+
  | NULL                          |
  +-------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', '10');
  +------------------------------+
  | JSON_SEARCH(@j, 'all', '10') |
  +------------------------------+
  | "$[1][0].k"                  |
  +------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', '10', NULL, '$');
  +-----------------------------------------+
  | JSON_SEARCH(@j, 'all', '10', NULL, '$') |
  +-----------------------------------------+
  | "$[1][0].k"                             |
  +-----------------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', '10', NULL, '$[*]');
  +--------------------------------------------+
  | JSON_SEARCH(@j, 'all', '10', NULL, '$[*]') |
  +--------------------------------------------+
  | "$[1][0].k"                                |
  +--------------------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', '10', NULL, '$**.k');
  +---------------------------------------------+
  | JSON_SEARCH(@j, 'all', '10', NULL, '$**.k') |
  +---------------------------------------------+
  | "$[1][0].k"                                 |
  +---------------------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', '10', NULL, '$[*][0].k');
  +-------------------------------------------------+
  | JSON_SEARCH(@j, 'all', '10', NULL, '$[*][0].k') |
  +-------------------------------------------------+
  | "$[1][0].k"                                     |
  +-------------------------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', '10', NULL, '$[1]');
  +--------------------------------------------+
  | JSON_SEARCH(@j, 'all', '10', NULL, '$[1]') |
  +--------------------------------------------+
  | "$[1][0].k"                                |
  +--------------------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', '10', NULL, '$[1][0]');
  +-----------------------------------------------+
  | JSON_SEARCH(@j, 'all', '10', NULL, '$[1][0]') |
  +-----------------------------------------------+
  | "$[1][0].k"                                   |
  +-----------------------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', 'abc', NULL, '$[2]');
  +---------------------------------------------+
  | JSON_SEARCH(@j, 'all', 'abc', NULL, '$[2]') |
  +---------------------------------------------+
  | "$[2].x"                                    |
  +---------------------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', '%a%');
  +-------------------------------+
  | JSON_SEARCH(@j, 'all', '%a%') |
  +-------------------------------+
  | ["$[0]", "$[2].x"]            |
  +-------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', '%b%');
  +-------------------------------+
  | JSON_SEARCH(@j, 'all', '%b%') |
  +-------------------------------+
  | ["$[0]", "$[2].x", "$[3].y"]  |
  +-------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', '%b%', NULL, '$[0]');
  +---------------------------------------------+
  | JSON_SEARCH(@j, 'all', '%b%', NULL, '$[0]') |
  +---------------------------------------------+
  | "$[0]"                                      |
  +---------------------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', '%b%', NULL, '$[2]');
  +---------------------------------------------+
  | JSON_SEARCH(@j, 'all', '%b%', NULL, '$[2]') |
  +---------------------------------------------+
  | "$[2].x"                                    |
  +---------------------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', '%b%', NULL, '$[1]');
  +---------------------------------------------+
  | JSON_SEARCH(@j, 'all', '%b%', NULL, '$[1]') |
  +---------------------------------------------+
  | NULL                                        |
  +---------------------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', '%b%', '', '$[1]');
  +-------------------------------------------+
  | JSON_SEARCH(@j, 'all', '%b%', '', '$[1]') |
  +-------------------------------------------+
  | NULL                                      |
  +-------------------------------------------+
   
  mysql> SELECT JSON_SEARCH(@j, 'all', '%b%', '', '$[3]');
  +-------------------------------------------+
  | JSON_SEARCH(@j, 'all', '%b%', '', '$[3]') |
  +-------------------------------------------+
  | "$[3].y"                                  |
  +-------------------------------------------+
  ```

## 三、修改JSON值的函数

- ### JSON_APPEND  废弃，MySQL 5.7.9开始改名为json_array_append

  将值附加到JSON文档中指定数组的末尾并返回结果。这个函数[`JSON_ARRAY_APPEND()`](https://blog.csdn.net/szxiaohe/article/details/82772881#function_json-array-append) 在MySQL 5.7.9中被重命名为; 该别名`JSON_APPEND()`现已在MySQL 5.7中弃用，并在MySQL 8.0中删除。

### JSON_ARRAY_APPEND()  末尾添加数组元素，如果原有值是数值或json对象，则转成数组后，再添加元素

- 语法：

  JSON_ARRAY_APPEND(json_doc, path, val[, path, val] ...)

- 说明：

  - 在指定的数组末尾以JSON文本形式追加指定的值并返回。当参数中包含NULL时，返回NULL。
  - 以下情况将报错 
    - json_doc不合法
    - path 不合法
    - 包含* 或者 ** 通配符
  - 键值对采用自左到右的顺序进行追加。追加一对键值后的新值将成为下一对键值追加的目标。
  - 如果指定目录下为标量或者对象值，则会被封装为数组，然后将新的值加入到数组中。对于不包含任何值得键值对将直接忽略。

- 示例：

  ```
  mysql> SET @j = '["a", ["b", "c"], "d"]';
  mysql> SELECT JSON_ARRAY_APPEND(@j, '$[1]', 1);
  +----------------------------------+
  | JSON_ARRAY_APPEND(@j, '$[1]', 1) |
  +----------------------------------+
  | ["a", ["b", "c", 1], "d"]        |
  +----------------------------------+
  mysql> SELECT JSON_ARRAY_APPEND(@j, '$[0]', 2);
  +----------------------------------+
  | JSON_ARRAY_APPEND(@j, '$[0]', 2) |
  +----------------------------------+
  | [["a", 2], ["b", "c"], "d"]      |
  +----------------------------------+
  mysql> SELECT JSON_ARRAY_APPEND(@j, '$[1][0]', 3);
  +-------------------------------------+
  | JSON_ARRAY_APPEND(@j, '$[1][0]', 3) |
  +-------------------------------------+
  | ["a", [["b", 3], "c"], "d"]         |
  +-------------------------------------+
   
  mysql> SET @j = '{"a": 1, "b": [2, 3], "c": 4}';
  mysql> SELECT JSON_ARRAY_APPEND(@j, '$.b', 'x');
  +------------------------------------+
  | JSON_ARRAY_APPEND(@j, '$.b', 'x')  |
  +------------------------------------+
  | {"a": 1, "b": [2, 3, "x"], "c": 4} |
  +------------------------------------+
  mysql> SELECT JSON_ARRAY_APPEND(@j, '$.c', 'y');
  +--------------------------------------+
  | JSON_ARRAY_APPEND(@j, '$.c', 'y')    |
  +--------------------------------------+
  | {"a": 1, "b": [2, 3], "c": [4, "y"]} |
  +--------------------------------------+
   
  mysql> SET @j = '{"a": 1}';
  mysql> SELECT JSON_ARRAY_APPEND(@j, '$', 'z');
  +---------------------------------+
  | JSON_ARRAY_APPEND(@j, '$', 'z') |
  +---------------------------------+
  | [{"a": 1}, "z"]                 |
  +---------------------------------+
  ```

  ### JSON_ARRAY_INSERT()   插入数组元素

- 语法： 

  `JSON_ARRAY_INSERT(json_doc, path, val[, path, val] ...) `

- 说明： 

  - 更新一个JSON文本，向文本中插入一个数组，然后返回修改后的文本。如果参数为NULL，则返回NULL。
  - 以下情况报错 
    - json_doc参数不合法
    - path不合法
    - 包含 * 或者 ** 通配符
    - 不是以数组标示结尾
  - 键值对采用自左到右的顺序进行插入，插入一对后的新值将作为下一对插入的目标。
  - 对于不包含任何值得键值对将直接忽略。如果path指定的是一个数组元素，则其对应的值将插入到该元素右边的任意位置；如果path指定的是数组的末尾，则其值将插入到该数组末尾。
  - 执行插入操作后，其元素位置将发生变化，也将影响后续插入数据的位置定义。如最后的示例中，第二个插入数据并未出现数数据库中，是因为第一次插入操作后，原语句中定义的位置在新数据中未找到指定的元素，从而被忽略。

- 示例： 

  ```
  mysql> SET @j = '["a", {"b": [1, 2]}, [3, 4]]';
  mysql> SELECT JSON_ARRAY_INSERT(@j, '$[1]', 'x');
  +------------------------------------+
  | JSON_ARRAY_INSERT(@j, '$[1]', 'x') |
  +------------------------------------+
  | ["a", "x", {"b": [1, 2]}, [3, 4]]  |
  +------------------------------------+
  mysql> SELECT JSON_ARRAY_INSERT(@j, '$[100]', 'x');
  +--------------------------------------+
  | JSON_ARRAY_INSERT(@j, '$[100]', 'x') |
  +--------------------------------------+
  | ["a", {"b": [1, 2]}, [3, 4], "x"]    |
  +--------------------------------------+
  mysql> SELECT JSON_ARRAY_INSERT(@j, '$[1].b[0]', 'x');
  +-----------------------------------------+
  | JSON_ARRAY_INSERT(@j, '$[1].b[0]', 'x') |
  +-----------------------------------------+
  | ["a", {"b": ["x", 1, 2]}, [3, 4]]       |
  +-----------------------------------------+
  mysql> SELECT JSON_ARRAY_INSERT(@j, '$[2][1]', 'y');
  +---------------------------------------+
  | JSON_ARRAY_INSERT(@j, '$[2][1]', 'y') |
  +---------------------------------------+
  | ["a", {"b": [1, 2]}, [3, "y", 4]]     |
  +---------------------------------------+
  mysql> SELECT JSON_ARRAY_INSERT(@j, '$[0]', 'x', '$[2][1]', 'y');
  +----------------------------------------------------+
  | JSON_ARRAY_INSERT(@j, '$[0]', 'x', '$[2][1]', 'y') |
  +----------------------------------------------------+
  | ["x", "a", {"b": [1, 2]}, [3, 4]]                  |
  +----------------------------------------------------+
  ```

  较早的修改会影响数组中以下元素的位置，因此同一[`JSON_ARRAY_INSERT()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-array-insert)调用中的后续路径 应考虑到这一点。在最后一个示例中，第二个路径不插入任何内容，因为路径在第一次插入后不再匹配任何内容。

### JSON_INSERT  插入值（插入新值，但不替换已经存在的旧值）

将数据插入JSON文档并返回结果。`NULL`如果有任何参数，则 返回`NULL`。如果发生错误 *`json_doc`*的参数是不是一个有效的JSON文档或任何*`path`*参数是不是有效的路径表达式或包含一个 `*`或`**`通配符。

路径值对从左到右进行评估。通过评估一对产生的文档成为评估下一对的新值。

将忽略文档中现有路径的路径值对，并且不会覆盖现有文档值。如果路径标识以下类型的值之一，则文档中不存在路径的路径值对会将值添加到文档中：

不存在于现有对象中的成员。该成员将添加到对象并与新值关联。

位于现有数组末尾的位置。该数组使用新值进行扩展。如果现有值不是数组，则将其作为数组自动包装，然后使用新值进行扩展。

为了进行比较 [`JSON_INSERT()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-insert)， [`JSON_REPLACE()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-replace)以及 [`JSON_SET()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-set)，看到的讨论[`JSON_SET()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-set)。

否则，将忽略文档中不存在路径的路径 - 值对，但不起作用。

```
mysql> SET @j = '{ "a": 1, "b": [2, 3]}';
mysql> SELECT JSON_INSERT(@j, '$.a', 10, '$.c', '[true, false]');
+----------------------------------------------------+
| JSON_INSERT(@j, '$.a', 10, '$.c', '[true, false]') |
+----------------------------------------------------+
| {"a": 1, "b": [2, 3], "c": "[true, false]"}        |
+----------------------------------------------------+
```

结果中列出的第三个也是最后一个值是带引号的字符串，而不是像第二个那样的数组（在输出中没有引用）; 不会将值转换为JSON类型。要将数组作为数组插入，必须显式执行此类强制转换，如下所示：

```
mysql> SELECT JSON_INSERT(@j, '$.a', 10, '$.c', CAST('[true, false]' AS JSON));
+------------------------------------------------------------------+
| JSON_INSERT(@j, '$.a', 10, '$.c', CAST('[true, false]' AS JSON)) |
+------------------------------------------------------------------+
| {"a": 1, "b": [2, 3], "c": [true, false]}                        |
+------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### JSON_MERGE  合并json数组或对象

合并两个或多个JSON文档。同义词 `JSON_MERGE_PRESERVE()`; 在MySQL 5.7.22中已弃用，并且在将来的版本中将被删除。

```
mysql> SELECT JSON_MERGE('[1, 2]', '[true, false]');
+---------------------------------------+
| JSON_MERGE('[1, 2]', '[true, false]') |
+---------------------------------------+
| [1, 2, true, false]                   |
+---------------------------------------+
1 row in set, 1 warning (0.00 sec)
 
mysql> SHOW WARNINGS\G
*************************** 1. row ***************************
  Level: Warning
   Code: 1287
Message: 'JSON_MERGE' is deprecated and will be removed in a future release. \
 Please use JSON_MERGE_PRESERVE/JSON_MERGE_PATCH instead
1 row in set (0.00 sec)
```

[`JSON_MERGE_PATCH(*`json_doc`*, *`json_doc`*[, *`json_doc`*\] ...)`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-merge-patch)

执行 符合[RFC 7396](https://tools.ietf.org/html/rfc7396)的两个或多个JSON文档的合并，并返回合并的结果，而不保留具有重复键的成员。如果至少有一个作为参数传递给此函数的文档无效，则引发错误。

***\*注意\****

有关此函数与之间差异的解释和示例`JSON_MERGE_PRESERVE()`，请参阅与 [JSON_MERGE_PRESERVE（）相比较的JSON_MERGE_PATCH（）](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#json-merge-patch-json-merge-preserve-compared)。

`JSON_MERGE_PATCH()` 执行合并如下：

1. 如果第一个参数不是对象，则合并的结果与将空对象与第二个参数合并的结果相同。
2. 如果第二个参数不是对象，则合并的结果是第二个参数。
3. 如果两个参数都是对象，则合并的结果是具有以下成员的对象：
   - 第一个对象的所有成员没有在第二个对象中具有相同键的相应成员。
   - 第二个对象的所有成员在第一个对象中没有对应的键，其值不是JSON `null`文字。
   - 具有在第一个和第二个对象中存在的键的所有成员，并且其在第二个对象中的值不是JSON `null` 文字。这些成员的值是以递归方式将第一个对象中的值与第二个对象中的值合并的结果。

```
mysql> SELECT JSON_MERGE_PATCH('[1, 2]', '[true, false]');
+---------------------------------------------+
| JSON_MERGE_PATCH('[1, 2]', '[true, false]') |
+---------------------------------------------+
| [true, false]                               |
+---------------------------------------------+
 
mysql> SELECT JSON_MERGE_PATCH('{"name": "x"}', '{"id": 47}');
+-------------------------------------------------+
| JSON_MERGE_PATCH('{"name": "x"}', '{"id": 47}') |
+-------------------------------------------------+
| {"id": 47, "name": "x"}                         |
+-------------------------------------------------+
 
mysql> SELECT JSON_MERGE_PATCH('1', 'true');
+-------------------------------+
| JSON_MERGE_PATCH('1', 'true') |
+-------------------------------+
| true                          |
+-------------------------------+
 
mysql> SELECT JSON_MERGE_PATCH('[1, 2]', '{"id": 47}');
+------------------------------------------+
| JSON_MERGE_PATCH('[1, 2]', '{"id": 47}') |
+------------------------------------------+
| {"id": 47}                               |
+------------------------------------------+
 
mysql> SELECT JSON_MERGE_PATCH('{ "a": 1, "b":2 }',
     >     '{ "a": 3, "c":4 }');
+-----------------------------------------------------------+
| JSON_MERGE_PATCH('{ "a": 1, "b":2 }','{ "a": 3, "c":4 }') |
+-----------------------------------------------------------+
| {"a": 3, "b": 2, "c": 4}                                  |
+-----------------------------------------------------------+
 
mysql> SELECT JSON_MERGE_PATCH('{ "a": 1, "b":2 }','{ "a": 3, "c":4 }',
     >     '{ "a": 5, "d":6 }');
+-------------------------------------------------------------------------------+
| JSON_MERGE_PATCH('{ "a": 1, "b":2 }','{ "a": 3, "c":4 }','{ "a": 5, "d":6 }') |
+-------------------------------------------------------------------------------+
| {"a": 5, "b": 2, "c": 4, "d": 6}                                              |
+-------------------------------------------------------------------------------+
```

您可以使用此函数通过`null`在seond参数中指定相同成员的值来删除成员 ，如下所示：

```
mysql> SELECT JSON_MERGE_PATCH('{"a":1, "b":2}', '{"b":null}');
+--------------------------------------------------+
| JSON_MERGE_PATCH('{"a":1, "b":2}', '{"b":null}') |
+--------------------------------------------------+
| {"a": 1}                                         |
+--------------------------------------------------+
```

这个例子表明该函数以递归方式运行; 也就是说，成员的值不仅限于标量，而是它们本身可以是JSON文档：

```
mysql> SELECT JSON_MERGE_PATCH('{"a":{"x":1}}', '{"a":{"y":2}}');
+----------------------------------------------------+
| JSON_MERGE_PATCH('{"a":{"x":1}}', '{"a":{"y":2}}') |
+----------------------------------------------------+
| {"a": {"x": 1, "y": 2}}                            |
+----------------------------------------------------+
```

`JSON_MERGE_PATCH()` MySQL 5.7.22及更高版本支持。

**JSON_MERGE_PATCH（）与JSON_MERGE_PRESERVE（）进行比较。** 的行为`JSON_MERGE_PATCH()`是一样的是[`JSON_MERGE_PRESERVE()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-merge-preserve)，有以下两种情况例外：

- `JSON_MERGE_PATCH()`使用第二个对象中的匹配键删除第一个对象中的任何成员，前提是与第二个对象中的键关联的值不是JSON `null`。
- 如果第二个对象的成员具有与第一个对象中的成员匹配的键，则将第一个对象中 的值`JSON_MERGE_PATCH()` *替换*为第二个对象中的值，而 `JSON_MERGE_PRESERVE()` *将*第二个值*附加*到第一个值。

此示例比较了将相同的3个JSON对象（每个对象具有匹配的密钥`"a"`）与这两个函数中的每一个进行合并的结果：

```
mysql> SET @x = '{ "a": 1, "b": 2 }',
     >     @y = '{ "a": 3, "c": 4 }',
     >     @z = '{ "a": 5, "d": 6 }';
 
mysql> SELECT  JSON_MERGE_PATCH(@x, @y, @z)    AS Patch,
    ->         JSON_MERGE_PRESERVE(@x, @y, @z) AS Preserve\G
*************************** 1. row ***************************
   Patch: {"a": 5, "b": 2, "c": 4, "d": 6}
Preserve: {"a": [1, 3, 5], "b": 2, "c": 4, "d": 6}
```

[`JSON_MERGE_PRESERVE(*`json_doc`*, *`json_doc`*[, *`json_doc`*\] ...)`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-merge-preserve)

合并两个或多个JSON文档并返回合并的结果。`NULL`如果有任何参数，则 返回`NULL`。如果任何参数不是有效的JSON文档，则会发生错误。

合并根据以下规则进行。有关其他信息，请参阅 [JSON值的规范化，合并和自动包装](https://dev.mysql.com/doc/refman/5.7/en/json.html#json-normalization)。

- 相邻阵列合并为单个阵列。

- 相邻对象合并为单个对象。

- 标量值作为数组自动包装并合并为数组。

- 通过将对象自动包装为数组并合并两个数组来合并相邻的数组和对象

  ```
  mysql> SELECT JSON_MERGE_PRESERVE('[1, 2]', '[true, false]');
  +------------------------------------------------+
  | JSON_MERGE_PRESERVE('[1, 2]', '[true, false]') |
  +------------------------------------------------+
  | [1, 2, true, false]                            |
  +------------------------------------------------+
   
  mysql> SELECT JSON_MERGE_PRESERVE('{"name": "x"}', '{"id": 47}');
  +----------------------------------------------------+
  | JSON_MERGE_PRESERVE('{"name": "x"}', '{"id": 47}') |
  +----------------------------------------------------+
  | {"id": 47, "name": "x"}                            |
  +----------------------------------------------------+
   
  mysql> SELECT JSON_MERGE_PRESERVE('1', 'true');
  +----------------------------------+
  | JSON_MERGE_PRESERVE('1', 'true') |
  +----------------------------------+
  | [1, true]                        |
  +----------------------------------+
   
  mysql> SELECT JSON_MERGE_PRESERVE('[1, 2]', '{"id": 47}');
  +---------------------------------------------+
  | JSON_MERGE_PRESERVE('[1, 2]', '{"id": 47}') |
  +---------------------------------------------+
  | [1, 2, {"id": 47}]                          |
  +---------------------------------------------+
   
  mysql> SELECT JSON_MERGE_PRESERVE('{ "a": 1, "b": 2 }',
       >    '{ "a": 3, "c": 4 }');
  +--------------------------------------------------------------+
  | JSON_MERGE_PRESERVE('{ "a": 1, "b": 2 }','{ "a": 3, "c":4 }') |
  +--------------------------------------------------------------+
  | {"a": [1, 3], "b": 2, "c": 4}                                |
  +--------------------------------------------------------------+
   
  mysql> SELECT JSON_MERGE_PRESERVE('{ "a": 1, "b": 2 }','{ "a": 3, "c": 4 }',
       >    '{ "a": 5, "d": 6 }');
  +----------------------------------------------------------------------------------+
  | JSON_MERGE_PRESERVE('{ "a": 1, "b": 2 }','{ "a": 3, "c": 4 }','{ "a": 5, "d": 6 }') |
  +----------------------------------------------------------------------------------+
  | {"a": [1, 3, 5], "b": 2, "c": 4, "d": 6}                                         |
  +----------------------------------------------------------------------------------+
  ```

  这个函数在MySQL 5.7.22中作为同义词添加 [`JSON_MERGE()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-merge)。该 `JSON_MERGE()`函数现已弃用，并且将在MySQL的未来版本中删除。

  该功能与[`JSON_MERGE_PATCH()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-merge-patch)重要方面类似但不同 ; 有关详细信息，请参阅 [JSON_MERGE_PATCH（）与JSON_MERGE_PRESERVE（）](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#json-merge-patch-json-merge-preserve-compared)进行比较。

### JSON_REMOVE  删除json数据

从JSON文档中删除数据并返回结果。`NULL`如果有任何参数，则 返回`NULL`。如果*`json_doc`*参数不是有效的JSON文档或任何*`path`*参数不是有效的路径表达式或者是`$`或包含`*`或`**` 通配符，则会发生错误 。

该*`path`*参数进行评估从左到右。通过评估一条路径生成的文档将成为评估下一条路径的新值。

如果文档中不存在要删除的元素，则不是错误; 在这种情况下，路径不会影响文档。

```
mysql> SET @j = '["a", ["b", "c"], "d"]';
mysql> SELECT JSON_REMOVE(@j, '$[1]');
+-------------------------+
| JSON_REMOVE(@j, '$[1]') |
+-------------------------+
| ["a", "d"]              |
+-------------------------+
```

JSON_REPLACE  替换值（只替换已经存在的旧值）

替换JSON文档中的现有值并返回结果。`NULL`如果有任何参数，则 返回`NULL`。如果发生错误 *`json_doc`*的参数是不是一个有效的JSON文档或任何*`path`*参数是不是有效的路径表达式或包含一个 `*`或`**`通配符。

路径值对从左到右进行评估。通过评估一对产生的文档成为评估下一对的新值。

文档中现有路径的路径值对使用新值覆盖现有文档值。文档中不存在路径的路径 - 值对将被忽略，并且不起作用。

为了进行比较 [`JSON_INSERT()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-insert)， [`JSON_REPLACE()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-replace)以及 [`JSON_SET()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-set)，看到的讨论[`JSON_SET()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-set)

```
mysql> SET @j = '{ "a": 1, "b": [2, 3]}';
mysql> SELECT JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]');
+-----------------------------------------------------+
| JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]') |
+-----------------------------------------------------+
| {"a": 10, "b": [2, 3]}                              |
+-----------------------------------------------------+
```

 

### JSON_SET  设置值（替换旧值，并插入不存在的新值）

在JSON文档中插入或更新数据并返回结果。返回`NULL`如果任何参数是 `NULL`或*`path`*，如果给，不定位的对象。如果发生错误 *`json_doc`*的参数是不是一个有效的JSON文档或任何*`path`*参数是不是有效的路径表达式或包含一个 `*`或`**`通配符。

路径值对从左到右进行评估。通过评估一对产生的文档成为评估下一对的新值。

文档中现有路径的路径值对使用新值覆盖现有文档值。如果路径标识以下类型的值之一，则文档中不存在路径的路径值对会将值添加到文档中：

- 不存在于现有对象中的成员。该成员将添加到对象并与新值关联。
- 位于现有数组末尾的位置。该数组使用新值进行扩展。如果现有值不是数组，则将其作为数组自动包装，然后使用新值进行扩展。

否则，将忽略文档中不存在路径的路径 - 值对，但不起作用。

的[`JSON_SET()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-set)， [`JSON_INSERT()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-insert)和 [`JSON_REPLACE()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-replace)功能的关系：

- [`JSON_SET()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-set) 替换现有值并添加不存在的值。
- [`JSON_INSERT()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-insert) 插入值而不替换现有值。
- [`JSON_REPLACE()`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-replace)*仅*替换 现有值。

以下示例说明了这些差异，使用了文档（`$.a`）中存在的一个路径和另一个不存在的路径（`$.c`）：

```
mysql> SET @j = '{ "a": 1, "b": [2, 3]}';
mysql> SELECT JSON_SET(@j, '$.a', 10, '$.c', '[true, false]');
+-------------------------------------------------+
| JSON_SET(@j, '$.a', 10, '$.c', '[true, false]') |
+-------------------------------------------------+
| {"a": 10, "b": [2, 3], "c": "[true, false]"}    |
+-------------------------------------------------+
mysql> SELECT JSON_INSERT(@j, '$.a', 10, '$.c', '[true, false]');
+----------------------------------------------------+
| JSON_INSERT(@j, '$.a', 10, '$.c', '[true, false]') |
+----------------------------------------------------+
| {"a": 1, "b": [2, 3], "c": "[true, false]"}        |
+----------------------------------------------------+
mysql> SELECT JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]');
+-----------------------------------------------------+
| JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]') |
+-----------------------------------------------------+
| {"a": 10, "b": [2, 3]}                              |
+-----------------------------------------------------+
```

### JSON_UNQUOTE  去除json字符串的引号，将值转成string类型

取消引用JSON值并将结果作为`utf8mb4`字符串返回 。`NULL`如果参数是，则 返回 `NULL`。如果值以双引号结束但不是有效的JSON字符串文字，则会发生错误。

在字符串中，除非[`NO_BACKSLASH_ESCAPES`](https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sqlmode_no_backslash_escapes)启用SQL模式，否则某些序列具有特殊含义。这些序列中的每一个都以反斜杠（`\`）开头，称为 *转义字符*。MySQL识别 [表12.21“JSON_UNQUOTE（）特殊字符转义序列”中显示的转义序列](https://blog.csdn.net/szxiaohe/article/details/82772881#json-unquote-character-escape-sequences)。对于所有其他转义序列，将忽略反斜杠。也就是说，转义字符被解释为好像它没有被转义。例如，`\x`就是`x`。这些序列区分大小写。例如， `\b`被解释为退格，但 `\B`被解释为`B`。



**表12.21 JSON_UNQUOTE（）特殊字符转义序列**

| 逃脱序列     | 由Sequence表示的字符          |
| :----------- | :---------------------------- |
| `\"`         | 双引号（`"`）字符             |
| `\b`         | 退格字符                      |
| `\f`         | 一个换文字符                  |
| `\n`         | 换行符（换行符）              |
| `\r`         | 回车符                        |
| `\t`         | 标签字符                      |
| `\\`         | 反斜杠（`\`）字符             |
| `\u*`XXXX`*` | Unicode值的UTF-8字节 *`XXXX`* |

 

这里显示了使用此函数的两个简单示例：

```
mysql> SET @j = '"abc"';
mysql> SELECT @j, JSON_UNQUOTE(@j);
+-------+------------------+
| @j    | JSON_UNQUOTE(@j) |
+-------+------------------+
| "abc" | abc              |
+-------+------------------+
mysql> SET @j = '[1, 2, 3]';
mysql> SELECT @j, JSON_UNQUOTE(@j);
+-----------+------------------+
| @j        | JSON_UNQUOTE(@j) |
+-----------+------------------+
| [1, 2, 3] | [1, 2, 3]        |
+-----------+------------------+
```

以下示例显示了如何 `JSON_UNQUOTE`在[`NO_BACKSLASH_ESCAPES`](https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sqlmode_no_backslash_escapes) 禁用和启用时转义句柄 ：

```
mysql> SELECT @@sql_mode;
+------------+
| @@sql_mode |
+------------+
|            |
+------------+
 
mysql> SELECT JSON_UNQUOTE('"\\t\\u0032"');
+------------------------------+
| JSON_UNQUOTE('"\\t\\u0032"') |
+------------------------------+
|       2                           |
+------------------------------+
 
mysql> SET @@sql_mode = 'NO_BACKSLASH_ESCAPES';
mysql> SELECT JSON_UNQUOTE('"\\t\\u0032"');
+------------------------------+
| JSON_UNQUOTE('"\\t\\u0032"') |
+------------------------------+
| \t\u0032                     |
+------------------------------+
 
mysql> SELECT JSON_UNQUOTE('"\t\u0032"');
+----------------------------+
| JSON_UNQUOTE('"\t\u0032"') |
+----------------------------+
|       2                         |
+----------------------------+
```

## 四、返回JSON值属性的函数

### JSON_DEPTH  返回json文档的最大深度

返回JSON文档的最大深度。`NULL`如果参数是，则 返回 `NULL`。如果参数不是有效的JSON文档，则会发生错误。

空数组，空对象或标量值具有深度1.仅包含深度为1的元素的非空数组或仅包含深度为1的成员值的非空对象具有深度2.否则，JSON文档的深度大于2。

```
mysql> SELECT JSON_DEPTH('{}'), JSON_DEPTH('[]'), JSON_DEPTH('true');
+------------------+------------------+--------------------+
| JSON_DEPTH('{}') | JSON_DEPTH('[]') | JSON_DEPTH('true') |
+------------------+------------------+--------------------+
|                1 |                1 |                  1 |
+------------------+------------------+--------------------+
mysql> SELECT JSON_DEPTH('[10, 20]'), JSON_DEPTH('[[], {}]');
+------------------------+------------------------+
| JSON_DEPTH('[10, 20]') | JSON_DEPTH('[[], {}]') |
+------------------------+------------------------+
|                      2 |                      2 |
+------------------------+------------------------+
mysql> SELECT JSON_DEPTH('[10, {"a": 20}]');
+-------------------------------+
| JSON_DEPTH('[10, {"a": 20}]') |
+-------------------------------+
|                             3 |
+-------------------------------+
```

### JSON_LENGTH  返回json文档的长度

返回JSON文档的长度，或者，如果*`path`*给出参数，则返回 由路径标识的文档中的值的长度。返回`NULL`如果任何参数 `NULL`或*`path`* 参数不文档中确定的值。如果*`json_doc`*参数不是有效的JSON文档或 *`path`*参数不是有效的路径表达式或包含`*`或 `**`通配符，则会发生错误。

文件的长度确定如下：

- 标量的长度为1。

- 数组的长度是数组元素的数量。

- 对象的长度是对象成员的数量。

- 长度不计算嵌套数组或对象的长度。

  ```
  mysql> SELECT JSON_LENGTH('[1, 2, {"a": 3}]');
  +---------------------------------+
  | JSON_LENGTH('[1, 2, {"a": 3}]') |
  +---------------------------------+
  |                               3 |
  +---------------------------------+
  mysql> SELECT JSON_LENGTH('{"a": 1, "b": {"c": 30}}');
  +-----------------------------------------+
  | JSON_LENGTH('{"a": 1, "b": {"c": 30}}') |
  +-----------------------------------------+
  |                                       2 |
  +-----------------------------------------+
  mysql> SELECT JSON_LENGTH('{"a": 1, "b": {"c": 30}}', '$.b');
  +------------------------------------------------+
  | JSON_LENGTH('{"a": 1, "b": {"c": 30}}', '$.b') |
  +------------------------------------------------+
  |                                              1 |
  +------------------------------------------------+
  ```

   

### JSON_TYPE  返回json值得类型

返回`utf8mb4`表示JSON值类型的字符串。这可以是对象，数组或标量类型，如下所示：

```
mysql> SET @j = '{"a": [10, true]}';
mysql> SELECT JSON_TYPE(@j);
+---------------+
| JSON_TYPE(@j) |
+---------------+
| OBJECT        |
+---------------+
mysql> SELECT JSON_TYPE(JSON_EXTRACT(@j, '$.a'));
+------------------------------------+
| JSON_TYPE(JSON_EXTRACT(@j, '$.a')) |
+------------------------------------+
| ARRAY                              |
+------------------------------------+
mysql> SELECT JSON_TYPE(JSON_EXTRACT(@j, '$.a[0]'));
+---------------------------------------+
| JSON_TYPE(JSON_EXTRACT(@j, '$.a[0]')) |
+---------------------------------------+
| INTEGER                               |
+---------------------------------------+
mysql> SELECT JSON_TYPE(JSON_EXTRACT(@j, '$.a[1]'));
+---------------------------------------+
| JSON_TYPE(JSON_EXTRACT(@j, '$.a[1]')) |
+---------------------------------------+
| BOOLEAN                               |
+---------------------------------------+
```

[`JSON_TYPE()`](https://dev.mysql.com/doc/refman/5.7/en/json-attribute-functions.html#function_json-type)返回 `NULL`如果参数为 `NULL`：

```
mysql> SELECT JSON_TYPE(NULL);
+-----------------+
| JSON_TYPE(NULL) |
+-----------------+
| NULL            |
+-----------------+
```

如果参数不是有效的JSON值，则会发生错误：

```
mysql> SELECT JSON_TYPE(1);
ERROR 3146 (22032): Invalid data type for JSON data in argument 1
to function json_type; a JSON string or JSON type is required.
```

对于`NULL`非非错误结果，以下列表描述了可能的 [`JSON_TYPE()`](https://dev.mysql.com/doc/refman/5.7/en/json-attribute-functions.html#function_json-type)返回值：

- 纯JSON类型：
  - `OBJECT`：JSON对象
  - `ARRAY`：JSON数组
  - `BOOLEAN`：JSON真假文字
  - `NULL`：JSON null文字
- 数字类型：
  - `INTEGER`：MySQL的 [`TINYINT`](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html)， [`SMALLINT`](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html)， [`MEDIUMINT`](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html)和 [`INT`](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html)和 [`BIGINT`](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html)标量
  - `DOUBLE`：MySQL 标量 [`DOUBLE`](https://dev.mysql.com/doc/refman/5.7/en/floating-point-types.html) [`FLOAT`](https://dev.mysql.com/doc/refman/5.7/en/floating-point-types.html)
  - `DECIMAL`：MySQL [`DECIMAL`](https://dev.mysql.com/doc/refman/5.7/en/fixed-point-types.html)和 [`NUMERIC`](https://dev.mysql.com/doc/refman/5.7/en/fixed-point-types.html)标量
- 时间类型：
  - `DATETIME`：MySQL [`DATETIME`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html)和 [`TIMESTAMP`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html)标量
  - `DATE`：MySQL [`DATE`](https://dev.mysql.com/doc/refman/5.7/en/datetime.html)标量
  - `TIME`：MySQL [`TIME`](https://dev.mysql.com/doc/refman/5.7/en/time.html)标量
- 字符串类型：
  - `STRING`：MySQL的 `utf8`字符类型标量： [`CHAR`](https://dev.mysql.com/doc/refman/5.7/en/char.html)， [`VARCHAR`](https://dev.mysql.com/doc/refman/5.7/en/char.html)， [`TEXT`](https://dev.mysql.com/doc/refman/5.7/en/blob.html)， [`ENUM`](https://dev.mysql.com/doc/refman/5.7/en/enum.html)，和 [`SET`](https://dev.mysql.com/doc/refman/5.7/en/set.html)
- 二进制类型：
  - `BLOB`：MySQL二进制类型标量： [`BINARY`](https://dev.mysql.com/doc/refman/5.7/en/binary-varbinary.html)， [`VARBINARY`](https://dev.mysql.com/doc/refman/5.7/en/binary-varbinary.html)， [`BLOB`](https://dev.mysql.com/doc/refman/5.7/en/blob.html)
  - `BIT`：MySQL [`BIT`](https://dev.mysql.com/doc/refman/5.7/en/bit-type.html)标量
- 所有其他类型：
  - `OPAQUE` （原始位）

### JSON_VALID  判断是否为合法json文档

返回0或1以指示值是否为有效JSON。`NULL`如果参数是，则 返回`NULL`。

```
mysql> SELECT JSON_VALID('{"a": 1}');
+------------------------+
| JSON_VALID('{"a": 1}') |
+------------------------+
|                      1 |
+------------------------+
mysql> SELECT JSON_VALID('hello'), JSON_VALID('"hello"');
+---------------------+-----------------------+
| JSON_VALID('hello') | JSON_VALID('"hello"') |
+---------------------+-----------------------+
|                   0 |                     1 |
+---------------------+-----------------------+
```

 



官方文档有更详细的说明的举例，请参考https://dev.mysql.com/doc/refman/5.7/en/json-function-reference.html