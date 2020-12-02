# SQL 大師速成包



## 基本語法

### 創建資料庫

* 語法

    ```mysql
    CREATE DATABASE `資料庫名稱` COLLATE `編碼`;
    ```

* 範例

    ```mysql
    CREATE DATABASE demo_shop COLLATE utf8_unicode_ci;
    ```

[回到目錄](#sql-大師速成包)

### 創建資料表

* 語法

    ```mysql
    CREATE TABLE `資料表名稱` (
        `欄位名稱1` `欄位屬性1`,
        `欄位名稱2` `欄位屬性2`,
        `欄位名稱3` `欄位屬性3`
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

[回到目錄](#sql-大師速成包)
