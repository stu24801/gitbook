---
description: 通用
---

# 常用DB 查詢

查詢資料表欄位資訊

```
//Oracle SQL
select lower(substr(replace(initcap(colCom.COLUMN_NAME),'_'),1,1))||substr(replace(initcap(colCom.COLUMN_NAME),'_'),2) camel, colCom.COLUMN_NAME, colCom.COMMENTS,DATA_TYPE
from user_tab_columns tabCol,user_col_comments colCom
where tabCol.TABLE_NAME = colCom.TABLE_NAME
and tabCol.COLUMN_NAME = colCom.COLUMN_NAME
and tabCol.TABLE_NAME ='TARGE TB_NAME';
```

查詢資料表欄位清單－撰寫StoredProcedure時，產出關聯表欄位

```
//Oracle SQL

select 'agnt.'||COLUMN_NAME ||',' || '--'||COMMENTS
from (
select colCom.TABLE_NAME, colCom.COLUMN_NAME, colCom.COMMENTS ,lower(substr(replace(initcap(colCom.COLUMN_NAME),'_'),1,1))||substr(replace(initcap(colCom.COLUMN_NAME),'_'),2) camel,DATA_TYPE, DATA_PRECISION, DATA_SCALE
from user_tab_columns tabCol,user_col_comments colCom
where tabCol.TABLE_NAME = colCom.TABLE_NAME
and tabCol.COLUMN_NAME = colCom.COLUMN_NAME
and tabCol.TABLE_NAME ='TARGET TB_NAME'
);
```

