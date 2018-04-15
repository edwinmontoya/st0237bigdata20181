# Universidad EAFIT
# Curso ST0237 Big Data 20181-1
# Profesor: Edwin Montoya M. – emontoya@eafit.edu.co

# hive

## TABLAS SENCILLAS EN HIVE

1. Se usarán los datos de Human Development Report (HDR) (http://hdr.undp.org/en/statistics/data)

o usar los datos en [datasets](datasets) de este repositorio.

2. Conectarse al cluster Hadoop (192.168.10.75)

entrar al cluster:

```
$ ssh username@192.168.10.75
username@192.168.10.75's password: ******
[username@hdplabmaster ~]$
```

3. copiar el archivo hdi-data.csv al HOME LOCAL del usuario:

```
/home/<username>/datasets
```

debe haber una archivo /home/<username>/datasets/hdi-data.csv para el trabajo a continuación.   

4. ejecutar Hive:

```
$ beeline
beeline>
!connect jdbc:hive2://hdpmaster.dis.eafit.edu.co:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2     
```

5. Crear la tabla HDI en Hive:
```
beeline> CREATE TABLE HDI (id INT, country STRING, hdi FLOAT, lifeex INT, mysch INT, eysch INT, gni INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE;
```

Nota: Esta tabla la crea en una BASE DE DATOS ‘default’
```
beeline> use default;
beeline> show tables;
beeline> describe hdi;
```

6. cargar los datos LOCALES a la tabla HDI:

```
beeline> load data local inpath 'hdi-data.csv' into table HDI;
```

7. hacer consultas y cálculos sobre la tabla HDI:
```
beeline> select * from hdi;
beeline>
beeline> select country, gni from hdi where gni > 2000;    
beeline>
```

## EJECUTAR UN JOIN CON HIVE:

1. Obtener los datos base: export-data.csv

usar los datos en 'datasets' de este repositorio.

2. Iniciar hive y crear la tabla EXPO:

```
$ hive
beeline> CREATE TABLE EXPO (country STRING, expct FLOAT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ‘,’ STORED AS TEXTFILE;
```

3. Carga los datos:export-data.csv
```
beeline>LOAD DATA LOCAL INPATH ‘export-data.csv’ INTO TABLE EXPO;
```

4. EJECUTAR EL JOIN DE 2 TABLAS:
```
beeline> SELECT h.country, gni, expct FROM HDI h JOIN EXPO e ON (h.country = e.country) WHERE gni > 2000;
```
## WORDCOUNT EN HIVE:
```
beeline>CREATE TABLE docs (line STRING);
```
// carga incremental
```
beeline>LOAD DATA INPATH ‘docs’ INTO TABLE docs;
```
// carga desde cero
```
beeline>LOAD DATA INPATH ‘docs’ OVERWRITE INTO TABLE docs;
```
// ordenado por palabra
```
beeline>SELECT word, count(1) AS count FROM
    (SELECT explode(split(line,' ')) AS word FROM docs) w
    GROUP BY word
    ORDER BY word;
```
// ordenado por frecuencia de menor a mayor
```
beeline>SELECT word, count(1) AS count FROM
    (SELECT explode(split(line,' ')) AS word FROM docs) w
    GROUP BY word
    ORDER BY count;
```

RETO:

¿cómo llenar una tabla con los resultados de un Query? por ejemplo, como almacenar en una tabla el diccionario de frecuencia de palabras en el wordcount?

// TENER EN CUENTA LA CARGA LOCAL o DESDE EL HDFS de los DATOS a HIVE:

Carga desde archivos locales:
```
beeline>LOAD DATA LOCAL INPATH …
```
Carga desde archivos del HDFS:
```
beeline>LOAD DATA INPATH …
```

# Apache Sqoop

## Datos en MySQL

```
En 192.168.10.75, se tiene Mysql con:
Base de datos: “curso”
Tabla: “username_employee”
User: curso/curso
$ mysql –u curso –p
Enter password: ******
mysql> use curso;
```
//ingreso de datos
```
$ mysql -u curso -p
Enter password: curso
mysql> use curso;
mysql> insert into username_employee values (101, 'name1', 1800);
mysql> insert into username_employee values (102, 'name2', 1500);
mysql> insert into username_employee values (103, 'name3', 1000);
mysql> insert into username_employee values (104, 'name4', 2000);
mysql> insert into username_employee values (105, 'name5', 1600);
Query OK, 1 row affected (0.00 sec)
mysql> 
```
## Comandos Sqoop

//Transferir datos de una base de datos (tipo mysql) hacia HDFS:
```
$ sqoop import --connect jdbc:mysql://192.168.10.75:3306/curso --username curso -P --table username_employee --target-dir /user/username/mysqlOut -m 1
```

// listar archivos:
```
$ hadoop fs -ls /user/username/mysqlOut
```

// Crear tabla HIVE a partir de definición tabla Mysql:
```
$ sqoop create-hive-table --connect jdbc:mysql://192.168.10.75:3306/curso --username curso -P --table username_employee --hive-table username_emps --mysql-delimiters
```

// Transferir datos de una base de datos (tipo mysql) hacia HIVE vía HDFS:

```
$ sqoop import --connect jdbc:mysql://192.168.10.75:3306/curso --username curso -P --table username_employee --hive-import --hive-table username_emps -m 1
```

// Crear la tabla Hive Manualmente:
```
$ beeline
beeline> CREATE TABLE username_emps (empid INT, name  STRING, salary INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ','  LINES TERMINATED BY ‘\n’ STORED AS TEXTFILE;
beeline>
```
// Cargar datos a Hive Manualmente:
```
beeline> load data inpath '/user/username/mysqlOut/part-m-00000' into table username_emps;
OK                          
beeline> select * from username_emps;
OK
101 name1 1800
102 name2 1500
103 name3 1000
104 name4 2000
105 name5 1600
taken: 0.269 seconds, Fetched: 5 row(s) Time
beeline> 
```

//Sqoop export hacia mysql:

// Crear una Tabla ‘username_employee2’ en Mysql con los mismos atributos de ‘username_employee’
```
mysql> CREATE TABLE `curso`.`username_employee2` (  `emp_id` INT NOT NULL,  `name` VARCHAR(45),  `salary` INT,  PRIMARY KEY (`emp_id`));
```

// Asumiendo datos separados por ”,” en HDFS en:

/user/username/mysql_in/*

```
$ sqoop export --connect jdbc:mysql://192.168.10.75:3306/curso --username curso -P --table username_employee2 --export-dir /user/username/mysql_in
```

# TAREA:

1. Compare el rendimiento del wordcount en hive vs mapreduce (en java o python) manteniendo el mismo dataset (gutenberg-txt-es)

# REALIZAR LOS SIGUIENTES TALLERES:

1. HORTONWORKS: HOW TO PROCESS DATA WITH APACHE HIVE:

https://es.hortonworks.com/tutorial/how-to-process-data-with-apache-hive/

2. CLOUDERA: Exercise 1: Ingest and query relational data

https://www.cloudera.com/developers/get-started-with-hadoop-tutorial/exercise-1.html
