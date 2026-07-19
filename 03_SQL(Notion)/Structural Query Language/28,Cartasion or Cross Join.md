# CARTASION/CROSS JOIN

- In cartasian join all records of first table will merge with all records of second table.
- The number of columns present in result table will be equals to summation of columns present in both tables
- The number of rows present in result table will be equal to the product of rows present in both the tables.

| ANSI(Ameriacan national standard institute)                           | ORACLE                                                                |
| --------------------------------------------------------------------- | --------------------------------------------------------------------- |
| SELECT COLUMN-NAME/EXPRESSION FROM TABLE-NAME1 CROSS JOIN TABLE_NAME2 | SELECT COLUMN_NAME/EXPRESSION FROM TABLE TABLE_NAME1, TABLE_NAME2,…; |
|                                                                       |                                                                       |

<aside>
❓

WAQTD EMPLOYEES AND THEIR DEPT DETAILS

```sql
SELECT *  --(2)
FROM EMP,DEPT;  --(1)
```

```sql
SELECT * 
FROM EMP,DEPT
WHERE EMP.DEPTNO=DEPT.DEPTNO;
```

![image.png](<CARTASION%20CROSS%20JOIN/image.png>)

```sql
```

</aside>

[INNER JOIN](<INNER%20JOIN%202a77ce6c5c25804f9bd0fc474108d10f.md>)
