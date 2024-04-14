# Database Isolation Level Comparison

This document compares the isolation levels in PostgreSQL and Percona databases, specifically focusing on the issues of lost update, dirty read, non-repeatable read, and phantom read.

## PostgreSQL

In PostgreSQL READ UNCOMMITTED is treated as READ COMMITTED.

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Lost Update |
|-----------------|------------|---------------------|--------------|-------------|
| Read Uncommitted|     no     |        yes          |      yes     |     no      |
| Read Committed  |     no     |        yes          |      yes     |     no      |
| Repeatable Read |     no     |         no          |       no     |     no      |
| Serializable    |     no     |         no          |       no     |     no      |


## PerconaDB

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Lost Update |
|-----------------|------------|---------------------|--------------|-------------|
| Read Uncommitted|     yes    |        yes          |      yes     |     no      |
| Read Committed  |     no     |        yes          |      yes     |     no      |
| Repeatable Read |     no     |         no          |       no     |     no      |
| Serializable    |     no     |         no          |       no     |     no      |


Couldn't reproduce lost updates - second transaction update is locked and timed out on any level.

## Problems

### Dirty Read
Dirty read refers to reading uncommitted data from another transaction. It occurs when a transaction reads data that has been modified by another transaction but not yet committed.

Steps to reproduce:

```
begin;
select email from users where id = 9;
// admin@localhost
                                                begin;
                                                UPDATE users SET email = 'johnathan@localhost' WHERE id = 9;
select email from users where id = 9;
// johnathan@localhost
commit;
                                                rollback;
```

### Non-Repeatable Read
Non-repeatable read occurs when a transaction reads the same data multiple times within the same transaction, but the data changes between the reads.

Steps to reproduce:

```
begin;
select email from users where id = 9;
// admin@localhost
                                                begin;
                                                UPDATE users SET email = 'johnathan@localhost' WHEREid = 9;
                                                commit;
select email from users where id = 9;
// johnathan@localhost
commit;                                        
```


### Phantom Read
Phantom read refers to a situation where a transaction reads a set of rows that satisfy a certain condition, but when it tries to read the same set of rows again, it finds additional rows that were inserted by another transaction.

Steps to reproduce:
```
begin;
select name, email from users where 
birthday between '1990-01-01' and '1990-12-31';
// Only John
                                                begin;
                                                INSERT INTO users (name, email, birthday) VALUES ('Fred', 'fred@email.com', '1990-03-03');
                                                commit;
begin;
select name, email from users where 
birthday between '1990-01-01' and '1990-12-31';
// John and Fred
commit;                                        
```

### Lost Update
Lost update occurs when two or more transactions concurrently access and modify the same data, resulting in one transaction overwriting the changes made by another transaction

Steps to reproduce:

```
begin;
SELECT balance FROM users WHERE id = 12; // 10
UPDATE users SET balance = balance + 50 
    WHERE id = 12;
                                                begin;
                                                SELECT balance FROM users WHERE id = 12;
                                                UPDATE users SET balance = balance + 70 WHERE id = 12;
                                                commit;
commit;                                        
```
In the end Fred`s balance is 60 instead of 130.