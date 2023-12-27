---
description: 資料庫、資料表
---
# 資料庫
一個資料庫( database )裡面有很多資料表( table )

## MySQL
MySQL 是一個**關聯式資料庫管理系統（RDBMS)**，可以操作一個或多個資料庫，每個資料庫可以包含多個資料表，資料庫是 MySQL 中的一個實例，而資料表是資料庫中的一個組成部分

## 資料定義語言 DDL
* create
* drop
* alter

### 建立資料庫
建立一個名叫 hello-world 的資料庫（ workbench介面新增出來的會是大寫，自己手動新增的會是小寫 ）

```js
CREATE SCHEMA `hello-world` ;
create database `hello-world
```

#### 使用資料庫 
`use hello-world`
#### 刪除資料庫 
`drop database hello-world`

### 建立資料表
建立一個名叫 heros 的資料表(資料表為複數型態名稱)（rails 的慣例）
```js
CREATE TABLE `hello-world`.`heros` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(100) NOT NULL,
  `gender` CHAR(1) NULL,
  `age` INT NULL,
  `hero_level` CHAR(1) NOT NULL,
  `hero_rank` INT NULL,
  `description` TEXT NULL,
  PRIMARY KEY (`id`));
```

#### 修改資料表
在 hello-world 的資料庫裡的 heros 資料表，增加 other 的欄位
```js
ALTER TABLE `hello-world`.`heros` 
ADD COLUMN `other` VARCHAR(45) NULL AFTER `description`;
```

#### 刪除資料表
`drop table heros`


## 資料型別
* DECIMAL(10,2) - 資料庫中用來儲存數值的資料型別
(這個例子：總共能夠儲存 10 位數的數字，其中小數點後有 2 位)
### CHAR & VARCHAR 差異
* CHAR(10)
  * 不管放入多少數字（10以內），剩下的會填入空白，長度依舊是 10
* VARCHAR(10) - 可變動的數字型態
  * 不管放入多少數字（含10），會額外增加 1byte 存入長度（例如：7 + 1 byte）




## 資料操作語言 DML
* insert
* delete
* update

## 資料查詢語言 DＱL
* select

## CRUD 
### - C 新增資料

* 新增資料 insert：欄位如果要省略的話， values 一定全部都要填入
```js
insert into heros
(name, gender, age, hero_level) //可自訂
values
('helena', 'F', 18, 1)//對應上方欄位
```

### - R 查詢資料
  * 查詢全部欄位
```js
select * from heroes
where hero_level = 'S' and gender = 'F';// 篩選
```

 * 計算欄位數量
```js
select count(*) from heroes
where hero_level = 'S' // 11
```

 * 計算年紀總和
```js
select sum(age) from heroes
where hero_level = 'S' and age is not null;//排除空值
```

 * 計算最大年紀
```js
select max(age) from heroes
```

 * 計算最小年紀
```js
select min(age) from heroes
```

 * 分組 : group by
```js
select hero_level, sum(age) from heroes
//前面會顯示 hero_level 的標題，後面顯示年紀總和
group by hero_level;
```

* 計算欄位種類 : distinct
```js
select distinct hero_level * from heroes; // S, A, B, C
```

* 照字母或數字順序排序 : order by
```js
select distinct hero_level * from heroes
order by hero_level desc //排序 , desc:反向
limit 2 //抓出前兩個查詢欄位
```
* 查詢部分欄位
```js
select name, gender from heroes;
```

* 查詢**未填寫**欄位
```js
select * from heroes
where age is null;
```

* 查詢**關鍵字**
  * 百分比：放前面 - 代表前面可以有字，放後面 - 代表後面可以有字
```js
select * from heroes
where name like '%背心%';
```

* 查詢區間
  * 建議使用 between
```js
select * from heroes
where age >= 10 and age <=25
//where age between 10 and 25
```

* 特定查詢
```js
select * from heroes
where hero_level in ('S','A')
//where hero_level = 'S' or hero_level = 'A'
```

* 排除特定查詢
  * SQL 語法  `<>` : 不等於
```js
select * from heroes
where hero_level <> 'S'
//where hero_level not in ('S')
```

* 查詢兩種資料**關聯**
  * 查詢 monsters 資料表裡面的 kill_by 欄位，對應上 heroes 資料表的 id 欄位是 埼玉 的人
```js
select * from monsters
where kill_by in(
	select id from heroes
    where name in ('埼玉')
)
```

* 交集 : join
  * 查詢 t1, t2 的資料表，name 是一樣的
```js
select * from t1, t2
where t1.username = t2.name
```
等於（這時 t1 在左邊，t2 在右邊，保留左邊表格（t1）的所有記錄，並且與右邊表格（t2）中匹配的記錄）
```js
select * from t1
left join t2
on t1.username = t2.name
```
* 名稱太長，可使用 as 建立縮寫
```js
select m.name,m.danger_level
from monsters as m
```

### - U 更新資料
把編號 20 的資料，改成年齡 10 歲，並且等級變成 A 級
```js
update heros
set age = 10, hero_level = 'A'
where id = 20
```
* 大範圍更新
  * 全部一次更新會有 MySQL 的安全模式，把安全模式關掉就可以了
```js
set sql_safe_updates = 0;
update heros 
set age = age + 1;
```

### - D 刪除資料
```js
delete from heros
where hero_level = 'A' 
```
*  大範圍刪除
   * 一樣也會有 MySQL 安全模式：`set sql_safe_updates = 0;`

## 正規化
將資料表細分成更小的資料表，以減少多餘的數據及重複儲存，提高資料庫的性能和維護性

未正規化劃的表格：

| 學生ID | 學生姓名 | 課程 | 課程描述 | 老師ID | 老師姓名 |
|----------|----------|----------|----------|----------|----------|
| 1 | 張三	| 數學, 物理	| 數學描述、物理描述 | A001、A002 | 張老師、王老師 |
| 2 | 李四 | 英語, 地理 | 英語描述、地理描述 | A003、A002 | 林老師、王老師 |
| 3 | 王五 | 數學 | 數學描述 | A001 | 張老師 |
	

#### 第一正規化
表中的每一個欄位都應該是單一的值，而不是包含多個值的集合


| 學生ID | 學生姓名 | 課程 | 分數 | 課程描述 | 老師ID | 老師姓名 |
|----------|----------|----------|----------|----------|----------|----------|
| 1 | 張三 | 數學 | 88 | 數學描述 | A001 | 張老師 |
| 1 | 張三 | 物理 | 76 | 物理描述 | A002 | 王老師 |
| 2 | 李四 | 英語 | 50 | 英語描述 | A001 | 張老師 |
| 2 | 李四 | 地理 | 80 | 地理描述 | A002 | 王老師 |
| 3 | 王五 | 數學 | 100 | 數學描述 | A001 | 張老師 |

#### 第二正規化
建立在第一正規化的基礎上，其目標是消除部分依賴

學生表：

| 學生ID | 學生姓名 |
|----------|----------|
| 1 | 張三 |
| 2 | 李四 | 
| 3 | 王五 | 

課程表：

| ID | 課程 | 課程描述 | 老師ID | 老師姓名 |
|----------|----------|----------|----------|----------|
| 1 | 數學 | 數學描述 | A001 | 張老師 |
| 2 | 物理 | 物理描述 | A002 | 王老師 |
| 3 | 英語 | 英語描述 | A001 | 張老師 |


| ID | 學生ID | 課程ID |  分數 |
|----------|----------|----------|----------|
| 1 | 1 | 1 | 88 |
| 2 | 1 | 2 | 76 |
| 3 | 2 | 2 | 50 |
| 4 | 2 | 3 | 80 |
| 5 | 3 | 1 | 100 |


#### 第三正規化
建立在第二正規化的基礎上，其目標是消除對非主鍵列的**遞移依賴**(老師ID及老師姓名的關係)

| ID | 課程 | 課程描述 | 老師ID | 老師姓名 |
|----------|----------|----------|----------|----------|
| 1 | 數學 | 數學描述 | A001 | 張老師 |
| 2 | 物理 | 物理描述 | A002 | 王老師 |
| 3 | 英語 | 英語描述 | A001 | 張老師 |

把老師ID及老師姓名抽取出來

| ID | 老師ID | 
|----------|----------|
| A001 | 張老師 |
| A002 | 王老師 |
| A001 | 張老師 |