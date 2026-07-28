# xcdriver ddl支持情况

## xcdriver ddl语法支持范围

### 列(column)操作

#### 1. 添加列

```sql
ALTER TABLE table_name ADD COLUMN column_name column_definition;
```

在表中添加新列，并定义列的类型、约束等信息。

#### 2. 修改列

**修改列的数据类型或定义（列名不变）**

```sql
ALTER TABLE table_name MODIFY COLUMN column_name column_definition;
```

注意：
- 支持bigint转varchar，但不支持varchar转bigint；
- 修改主键字段的类型时，必须指定not null;
- 不支持修改主键自增字段;
- text类型不支持索引;

**修改列名及其定义（包括数据类型、约束等）**

```sql
ALTER TABLE table_name CHANGE COLUMN old_column_name new_column_name column_definition;
```

注意：
- 不支持修改主键自增字段（低版本oceanbase不支持）；
- 修改主键字段的类型时，必须指定not null;

#### 3. 删除列

```sql
ALTER TABLE table_name DROP COLUMN column_name;
```

#### 4. 重命名列

```sql
ALTER TABLE table_name RENAME COLUMN old_column_name TO new_column_name;
```

注意：若列的定义中存在 on update属性，例如：`field timestamp on update current_timestamp`，不支持使用该语法重命名列名，推荐使用 change column语法。

### 索引(index)操作

#### 1. 添加普通索引

```sql
ALTER TABLE table_name ADD INDEX index_name(column1, column2);
CREATE INDEX index_name ON table_name(column1, column2);
```

注意：create index 不支持 if not exists 判断（低版本达梦不支持）

#### 2. 添加唯一索引

```sql
ALTER TABLE table_name ADD UNIQUE INDEX index_name(column1, column2);
CREATE UNIQUE INDEX index_name ON table_name(column1, column2);
```

注意：CREATE UNIQUE INDEX不支持 if not exists 判断（低版本达梦不支持）

#### 3. 删除索引

```sql
ALTER TABLE table_name DROP INDEX index_name;
```

### 主键操作

#### 1. 添加主键

```sql
ALTER TABLE table_name ADD PRIMARY KEY (column1, column2);
```

#### 2. 删除主键

```sql
ALTER TABLE table_name DROP PRIMARY KEY;
```

### 表操作

#### 重命名表

```sql
ALTER TABLE table_name RENAME TO new_table_name;
```

注意：不支持修改索引名

### 公文库ddl支持情况

AI公文索引丢失表变更记录.sql 支持情况：

- 支持datetime转timestamp;
- 支持bigint转varchar，但不支持varchar转bigint；
- 主键字段从bigint改成varchar，需要指定not null;
- 不支持修改主键自增字段;
- 不支持set类型;
- 支持varchar类型改text类型，但varchar字段若存在索引，则不支持修改为text；

---

## 类型变更支持情况(表中已有数据的情况)

> **已验证的信创数据库**：达梦V8.6.2.2、达梦V8.1.4.6、人大金仓V008R006C006B0013、oceanbase4.2.1、oceanbase3.2.1、opengauss。
>
> **未包含**：vastbase、tdsql、tidb

**注意**：
1. 所有类型变更都要考虑数值大小是否兼容，例如bigint转tinyint（低版本oceanbase不支持），数值溢出也不支持。
2. 类型修改后，已有的数据可能会发生变更，需要业务侧去评估影响。
3. oceanbase3.2.3版本限制很严格，需特殊处理，表格中的支持情况不包含oceanbase3.2.3。

### 兼容类型相互转换

#### 整型类型相互转换

除oceanbase3.2.3以外，其余信创数据库均支持。

| 源类型/目标类型 | tinyint | smallint | int | bigint | tinyint unsigned | smallint unsigned | int unsigned | bigint unsigned |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| tinyint | - | | | | | | | |
| smallint | | - | | | | | | |
| int | | | - | | | | | |
| bigint | | | | - | | | | |
| tinyint unsigned | | | | | - | | | |
| smallint unsigned | | | | | | - | | |
| int unsigned | | | | | | | - | |
| bigint unsigned | | | | | | | | - |

> oceanbase3.2.3仅支持unsigned类型之间互相转换，以及有符号之间想转换，且只能小转大；

#### 小数类型转换

除oceanbase3.2.3以外，其余信创数据库均支持。

| 源类型/目标类型 | double | float | decimal | numeric |
| --- | --- | --- | --- | --- |
| double | - | 支持 | 支持 | 支持 |
| float | 支持 | - | 支持 | 支持 |
| decimal | 支持 | 支持 | - | 支持 |
| numeric | 支持 | 支持 | 支持 | - |

> oceanbase3.2.3仅支持decimal和numeric互相转换，其余均不支持

#### 时间类型转换

| 源类型/目标类型 | date | time | datetime | timestamp |
| --- | --- | --- | --- | --- |
| date | - | 不支持 | 支持 | 支持 |
| time | 不支持 | - | 不支持 | 不支持 |
| datetime | 支持 | 支持 | - | 支持 |
| timestamp | 支持 | 支持 | 支持 | - |

> oceanbase3.2.3不支持任意两个时间类型互相转换

#### 二进制类型转换

| 源类型/目标类型 | binary | varbinary | blob | mediumblob |
| --- | --- | --- | --- | --- |
| binary | - | 支持 | 支持 | 支持 |
| varbinary | 支持 | - | 支持 | 支持 |
| blob | 不支持 | 不支持 | - | 支持 |
| mediumblob | 不支持 | 不支持 | 支持 | - |

> oceanbase3.2.3不支持修改binary类型；不支持将varbinary类型修改为binary类型；不允许修改blob/text类型；

#### 字符类型转换

| 源类型/目标类型 | char | varchar | text | mediumtext | longtext |
| --- | --- | --- | --- | --- | --- |
| char | - | 不支持 | 不支持 | 不支持 | 不支持 |
| varchar | 支持 | - | 有索引或默认值时不支持 | 有索引或默认值时不支持 | 有索引或默认值时不支持 |
| text | 支持 | 支持 | - | 支持 | 支持 |
| mediumtext | 支持 | 支持 | 支持 | - | 支持 |
| longtext | 支持 | 支持 | 支持 | 支持 | - |

> oceanbase3.2.3不支持修改 text/lob 列、不允许修改char字段、不允许将varchar改为固定长度字段；

### 不兼容类型相互转换

#### 整型和字符类型转换

| 源类型/目标类型 | char | varchar | text | mediumtext | longtext |
| --- | --- | --- | --- | --- | --- |
| tinyint | 支持 | 支持 | 不支持 | 不支持 | 不支持 |
| smallint | 支持 | 支持 | 不支持 | 不支持 | 不支持 |
| int | 支持 | 支持 | 不支持 | 不支持 | 不支持 |
| bigint | 支持 | 支持 | 不支持 | 不支持 | 不支持 |
| int unsigned | 支持 | 支持 | 不支持 | 不支持 | 不支持 |
| tinyint unsigned | 支持 | 支持 | 不支持 | 不支持 | 不支持 |
| bigint unsigned | 支持 | 支持 | 不支持 | 不支持 | 不支持 |
| smallint unsigned | 支持 | 支持 | 不支持 | 不支持 | 不支持 |

> oceanbase3.2.3不支持任意两个整型和字符类型转换

#### 整型和二进制类型转换

| 源类型/目标类型 | binary | varbinary | blob | mediumblob |
| --- | --- | --- | --- | --- |
| tinyint | 支持 | 支持 | 不支持 | 不支持 |
| smallint | 支持 | 支持 | 不支持 | 不支持 |
| int | 支持 | 支持 | 不支持 | 不支持 |
| bigint | 支持 | 支持 | 不支持 | 不支持 |
| int unsigned | 支持 | 支持 | 不支持 | 不支持 |
| tinyint unsigned | 支持 | 支持 | 不支持 | 不支持 |
| bigint unsigned | 支持 | 支持 | 不支持 | 不支持 |
| smallint unsigned | 支持 | 支持 | 不支持 | 不支持 |

> oceanbase3.2.3不支持任意两个整型和二进制类型转换

#### 整型和小数类型转换

除oceanbase外其余信创数据库均支持。

| 源类型/目标类型 | double | float | decimal |
| --- | --- | --- | --- |
| tinyint | 支持 | 支持 | 支持 |
| smallint | 支持 | 支持 | 支持 |
| int | 支持 | 支持 | 支持 |
| bigint | 支持 | 支持 | 支持 |
| int unsigned | 支持 | 支持 | 支持 |
| tinyint unsigned | 支持 | 支持 | 支持 |
| bigint unsigned | 支持 | 支持 | 支持 |
| smallint unsigned | 支持 | 支持 | 支持 |

> oceanbase3.2.3不支持任意两个整型和小数类型转换

#### 字符类型和二进制类型转换

| 源类型/目标类型 | binary | varbinary | blob | mediumblob |
| --- | --- | --- | --- | --- |
| char | 支持 | 支持 | 不支持 | 不支持 |
| varchar | 支持 | 支持 | 不支持 | 不支持 |
| text | 不支持 | 不支持 | 不支持 | 不支持 |
| mediumtext | 不支持 | 不支持 | 不支持 | 不支持 |
| longtext | 不支持 | 不支持 | 不支持 | 不支持 |

> oceanbase3.2.3和opengauss不支持任意两个字符型和二进制类型转换

---

## 类型变更导致索引缺失影响范围

```sql
create table test_index_2510(
    field1 int,
    field2 varchar(100),
    field3 varchar(100),
    field4 varchar(100),
    index inx_f1(field1),
    index inx_f1_f2(field1,field2),
    unique index inx_f3(field3),
    unique index inx_f3_f4(field3,field4)
);

alter table test_index_2510 drop column field1; -- 普通索引和联合索引
alter table test_index_2510 drop column field3; -- 唯一普通索引和联合索引
```

| 数据库 | 唯一索引-联合 | 唯一索引-非联合 | 普通索引-联合 | 普通索引-非联合 |
| --- | --- | --- | --- | --- |
| 达梦(dm) | 索引缺失 | 索引缺失 | 索引缺失 | 索引缺失 |
| 人大金仓(kingbase) | 索引缺失 | 索引缺失 | 索引缺失 | 索引缺失 |
| opengauss | 索引缺失 | 索引缺失 | 索引缺失 | 索引缺失 |
| oceanbase3.2.3 | 不允许删除索引列 | 不允许删除索引列 | 不允许删除索引列 | 不允许删除索引列 |
| oceanbase4.2.1 | 索引字段缺失（无冲突数据） | 索引字段缺失（无冲突数据） | 索引字段缺失 | 索引字段缺失 |
| vastbase | 索引缺失 | 索引缺失 | 索引缺失 | 索引缺失 |
| mysql 8.0.42 | 索引字段缺失（无冲突数据） | 索引字段缺失（无冲突数据） | 索引字段缺失 | 索引字段缺失 |
| mysql(goldendb, tidb) | 待补充 | 待补充 | 待补充 | 待补充 |
