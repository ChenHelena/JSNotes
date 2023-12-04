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
  * <> : 不等於
```js
select * from heroes
where hero_level <> 'S'
//where hero_level not in ('S')
```
