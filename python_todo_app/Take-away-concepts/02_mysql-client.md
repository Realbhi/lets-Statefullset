## 1. What is Mysql Client ? 

A **MySQL client** is simply a program that talks to a MySQL server.

Think of it like:

```text id="s3h4rw"
MySQL Server  ← database engine storing data
MySQL Client  ← tool used to connect and send queries
```

##

Example architecture:

```text id="n9r1xj"

mysql client program
   ↓
network connection
   ↓
MySQL server
   ↓
database files/data
```

##

The MySQL server:

- stores databases
- stores tables
-  processes SQL queries

The MySQL client:

- connects to the server
- Sends SQL commands
- shows results

##

When you type:

```bash id="e1q0vz"
mysql -h localhost -u root -p
```

you are starting the **mysql client process**. The executable/program name itself is: mysql
That binary is the client application.

##

Then the client:

1. asks password
2. connects to MySQL server
3. opens interactive SQL shell

Example:

```sql id="m5x7pe"
mysql>
```

Now you can type SQL queries:

```sql id="bt2kg0"
SELECT * FROM users;
```

##

So:

| Thing    | Meaning                            |
| -------- | ---------------------------------- |
| `mysql`  | MySQL client executable            |
| `mysqld` | MySQL server daemon/process        |
| `mysql>` | Interactive SQL prompt from client |

---

### How to start mysql client process?**

**Interactive mode**

```bash id="4l18yo"
mysql -h localhost -u root -p
```

Starts client interactively.

##

**One-time query mode**

```bash id="g8n2vq"
mysql -h localhost -u root -psecret mydb -e "SELECT * FROM users;"
```

Client:

- connects
- executes query
- prints result
- exits

---



