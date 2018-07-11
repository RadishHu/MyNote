# 根据B表中的数据修改原始表A中的数据

表A中为原始数据，表B为新增数据，表A和表B通过字段key关联。如果表B的key在表A中存在，则修改表A该行的数据；如果不存在，添加该数据

实现一:

```sql
CREATE TABLE A_TMP LIKE A;

#从表A中剔除表B中存在的数据
INSERT INTO TABLE A_TMP
SELECT
A.*
FROM
A
LEFT JOIN B
ON A.key=B.key
WHERE
A.key IS NULL;

#添加表B的数据
INSERT INTO TABLE A_TMP
SELECT
*
FROM
B

#删除表A，修改名称
DROP TABLE A;
ALTER A_TMP RENAME TO A;
```

