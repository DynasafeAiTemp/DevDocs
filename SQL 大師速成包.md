# SQL 大師速成包

*參考來源：[簡明 SQL 資料庫語法入門教學 (techbridge.cc)](https://blog.techbridge.cc/2020/02/09/sql-basic-tutorial/)*

> * [基本語法](#基本語法)
>   * [創建資料庫](#創建資料庫)
>   * [創建資料表](#創建資料表)
>   * [刪除資料庫 / 資料表](#刪除資料庫--資料表)
>   * [新增 / 刪除欄位](#新增--刪除欄位)
>   * [插入資料](#插入資料)
>   * [查詢資料](#查詢資料)

## 基本語法

### 創建資料庫

* 語法

    ```mysql
    CREATE DATABASE 資料庫名稱
    COLLATE 編碼;
    ```

* 範例

    ```mysql
    CREATE DATABASE demo_shop
    COLLATE utf8_unicode_ci;
    ```

[返回目錄](#sql-大師速成包)

---

### 創建資料表

* 語法

    ```mysql
    CREATE TABLE 資料表名稱 (
        欄位名稱一 欄位屬性一,
        欄位名稱二 欄位屬性二,
        欄位名稱三 欄位屬性三
    );
    ```

* 範例

    ```mysql
    CREATE TABLE users (
        username VARCHAR(12) NOT NULL,
        age INTEGER,
        gender VARCHAR(6)
    );
    ```

[返回目錄](#sql-大師速成包)

---

### 刪除資料庫 / 資料表

* 語法

  ```mysql
  DROP DATABASE/TABLE 資料庫/資料表名稱;
  ```

* 範例

  ```mysql
  DROP DATABASE demo_shop;
  ```

  ```mysql
  DROP TABLE users;
  ```

[返回目錄](#sql-大師速成包)

---

### 新增 / 刪除欄位

* 語法

  ```mysql
  ALTER TABLE 資料表名稱 ADD/DROP 欄位名稱;
  ```

* 範例

  ```mysql
  ALTER TABLE users ADD profile TEXT;
  ```

  ```mysql
  ALTER TABLE users DROP age;
  ```

[返回目錄](#sql-大師速成包)

---

### 插入資料

* 語法

  ```mysql
  INSERT INTO 資料表名稱 VALUES (
      數值一, 數值二, 數值三
  );
  ```

* 範例

  ```mysql
  INSERT INTO users VALUES (
      'Jack', 20, 'Male'
  );
  ```

[返回目錄](#sql-大師速成包)

---

### 查詢資料

