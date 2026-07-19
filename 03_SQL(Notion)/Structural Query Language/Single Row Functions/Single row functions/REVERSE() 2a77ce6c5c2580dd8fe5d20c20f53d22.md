# REVERSE()

Sr_no: 16

- It is used to reverse a given string.
- You can only reverse string data not number or date.
- Syntax: REVERSE(’STRING’);

<aside>
❓

Example: 

```sql
SELECT ENAME, REVERSE(ENAME)
FROM EMP;
```

![image.png](REVERSE()/image.png)

OR 

```sql
SELECT REVERSE(MALYALAM) 
FROM DUAL 
WHERE REVRSE('MALYALAM') ='MALYALAM';
```

</aside>

<aside>
❓

WAQTD DETAILS OF EMP IF THEY HAVE PALINDROE NAMES

```sql
SELECT * FROM EMP
WHERE REVERSE(ENAME) = ENAME;
```

![image.png](REVERSE()/image%201.png)

</aside>

[JOINS](../../JOINS%202a77ce6c5c258059a947cd1b321a0ba2.md)