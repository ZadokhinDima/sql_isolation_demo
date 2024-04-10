# Database Isolation Level Comparison

This document compares the isolation levels in PostgreSQL and Percona databases, specifically focusing on the issues of lost update, dirty read, non-repeatable read, and phantom read.

## PostgreSQL

| Isolation Level | Lost Update | Dirty Read | Non-Repeatable Read | Phantom Read |
|-----------------|-------------|------------|---------------------|--------------|
| Read Uncommitted|             |            |                     |              |
| Read Committed  |             |            |                     |              |
| Repeatable Read |             |            |                     |              |
| Serializable    |             |            |                     |              |


## PerconaDB

| Isolation Level | Lost Update | Dirty Read | Non-Repeatable Read | Phantom Read |
|-----------------|-------------|------------|---------------------|--------------|
| Read Uncommitted|             |            |                     |              |
| Read Committed  |             |            |                     |              |
| Repeatable Read |             |            |                     |              |
| Serializable    |             |            |                     |              |


## Problems

### Dirty Read
Dirty read refers to reading uncommitted data from another transaction. It occurs when a transaction reads data that has been modified by another transaction but not yet committed.

Steps to reproduce:

| Transaction #1 | Transaction #2 |
|----------|----------|
| `begin;` | `begin;` |
| `select a lot of columns from awesome table;` |  |        
|          | `insert some data to awesome table;`  |

### Lost Update
Lost update occurs when two or more transactions concurrently access and modify the same data, resulting in one transaction overwriting the changes made by another transaction

Steps to reproduce:

| Transaction #1 | Transaction #2 |
|----------|----------|
| `begin;` | `begin;` |
| `select a lot of columns from awesome table;` |  |        
|          | `insert some data to awesome table;`  |

### Non-Repeatable Read
Non-repeatable read occurs when a transaction reads the same data multiple times within the same transaction, but the data changes between the reads.

Steps to reproduce:

| Transaction #1 | Transaction #2 |
|----------|----------|
| `begin;` | `begin;` |
| `select a lot of columns from awesome table;` |  |        
|          | `insert some data to awesome table;`  |

### Phantom Read
Phantom read refers to a situation where a transaction reads a set of rows that satisfy a certain condition, but when it tries to read the same set of rows again, it finds additional rows that were inserted by another transaction.

Steps to reproduce:

| Transaction #1 | Transaction #2 |
|----------|----------|
| `begin;` | `begin;` |
| `select a lot of columns from awesome table;` |  |        
|          | `insert some data to awesome table;`  |