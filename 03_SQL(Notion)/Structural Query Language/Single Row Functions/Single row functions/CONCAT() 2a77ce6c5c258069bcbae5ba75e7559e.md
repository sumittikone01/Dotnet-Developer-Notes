# CONCAT()

Sr_no: 6

1. Concat :
    1. It is used to Merge given two strings or two columns.
    2. Can be passed in where Clouse. 
    3. It only accepts 2 arguments at a time, so for more argument concatenation nested concatenation can be used.
    4. Syntax:  CONCAT ( STRING1, STRING2) 
        
        <aside>
        ❓ EXAMPLE:
        
        ```sql
        SELECT CONCAT(ENAME, CONCAT(JOB, SAL)) 
        FROM EMP;
        ```
        
        ![image.png](CONCAT()/image.png)
        
        </aside>
        

[MOD()](MOD()%202a77ce6c5c2580f790b1e77e72a63642.md)