## How to connect to mysql server as a client/or by MySql-driver :

To connect to mysql server as client , we need to start mysql client process that is 'mysql' binary . 

##

### one of the way is :

Full command:

```bash id="md7ms0"
kubectl run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never  mysql -h mysql-normal -uroot -proot todo -e "SELECT * from todo;"
```

This command is interesting because here:

```bash id="x7v0bw"
mysql -h mysql-normal -uroot -proot todo -e "SELECT * from todo;"
```

- so here there are starting a dummy container called mysql-client , image being mysql:5.7 and the above command 
- is **NOT executed by a shell**. Instead, it is passed directly as the command for the container process.

So a random container is where mysql binary is made to run and mysql client process is started and made to connect to mysql server.

##

When a dummy container is a mysql client as above , Conceptually Kubernetes interprets it like:

```text id="i4s3t2"
Create pod:
  image = mysql:5.7

Run inside container:
  mysql
    -h mysql-normal
    -uroot
    -proot
    todo
    -e "SELECT * from todo;"
```

- here the hostname ( -h ), is the service through which we can reach the pod backed by mysql service.
- So instead of reaching pod directly which is non realiable way , we use service as hostname.


The important part:

There is NO:

```bash id="jwl35s"
/bin/sh -c
```

So:

- no shell parsing
- no shell expansion
- no pipes/redirection interpretation

Kubernetes directly launches the `mysql` executable. Options:

| Part                       | Meaning                                         |
| -------------------------- | ----------------------------------------------- |
| `-h mysql-normal`          | connect to MySQL server hostname `mysql-normal` |
| `-uroot`                   | username = root                                 |
| `-proot`                   | password = root                                 |
| `todo`                     | database name                                   |
| `-e "SELECT * from todo;"` | execute SQL query and exit                      |

##

So this:

```sql id="m0x7en"
SELECT * from todo;
```

means:

```text id="f8e8w7"
Fetch all rows from table `todo`
```

Here `-e` belongs to the **mysql client**, not shell. It means:

```text id="8chfx2"
"execute this SQL query"
```

##

Flow:

```text id="uqf5f7"
Your terminal
   ↓
kubectl
   ↓
Temporary pod created
   ↓
mysql client runs inside container
   ↓
connects to mysql-normal service
   ↓
executes SQL query
   ↓
prints result
   ↓
pod deleted because --rm
```

---

### second way 

The configMap configuration :


<img width="746" height="196" alt="image" src="https://github.com/user-attachments/assets/63ffa05f-86a7-4692-bdc8-e7bc50bc7542" />

##

The starfullset configuration :


<img width="353" height="237" alt="image" src="https://github.com/user-attachments/assets/aeb2f926-29fd-44e5-8c30-ef8904491ddb" />


##

So it is going this way :

- The frontend application want to reach the mysql server , Application uses a MySQL driver/library to connect to the MySQL server. The URI helps for that.
- We passed the needed info the is hostname , username ,password and db-name as env variables to the frontend app . The URI mentioned in the frontend application yaml spec passed as env variable is :

  ~~~
   env:
          - name: DATABASE_URI
            value: 'mysql+pymysql://root:root@mysql-normal.default.svc.cluster.local:3306/todo'
  ~~~
  
- The frontend application logic uses these env variable and connect to mysql server. 
- then the mysql binary ( in mysql container ) will automatically run the init.sql as show in picuture-1 as init script. Its also important to understand that init script only run if the /var/lib/mysql folder is empty or if its
  the first intialization of the db. Else if the /var/lib/mysql is persistant and has data already during initalization then the init script running is skipped.
- The init script is always within /docker-entrypoint-initdb.d folder as its a init.sql file with sql extension the mysql program /binary run its automatically

**so with the help of URI the frontend application as mysql client is connect to the mysql server.**

---

**Earlier the init script was passed as configMap data mounted at /docker-entrypoint-initdb.d of mysql container , the mysql binary uses that and run the init script creating tables and databases.
Other way use here is where they have just passes a shell script and mapped it to the variable my_init_script.sh .Inside this they have actuall database init script which needs mysql binary to run.**

Path to values.yaml : https://github.com/Realbhi/lets-Statefullset/blob/main/python_todo_app/kubernetes/02/mysql/values.yaml


<img width="821" height="237" alt="image" src="https://github.com/user-attachments/assets/e3e49eb9-fa2f-42c9-982f-25045b34b427" />

##

- In script file the mysql binary is invoked and made to run init db script.
- Its not at .sql type file to run it automatically , so we need to invoke the mysql binary by ouselves and make it to run the init script in there.

   ~~~
   mysql -P 3306 -uroot -proot todo -e "
   ~~~
   
- mysql binary is invoked in the script itself, as its a mysql client we need to provide db name , username and password as well so connect to mysql server. Hostname is not provided as mysql client and server are in same host
- With provided info it will connect to sql server and then execute the the init script that is sql query after -e "..." . This creates the needed tables and databases.

##

### Final conclusion :

- In second case discussed ,  frontend application is reaching the mysql server and fetching data to render on frontend. The ways to start the database and run the inti script is different.
- the fronend just needs URI to reach database through driver and then the frontend logic fetches and reder the fetched.
