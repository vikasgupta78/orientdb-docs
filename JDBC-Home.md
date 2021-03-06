---
search:
   keywords: ['Java API', 'JDBC', 'JDBC driver']
---

# JDBC Driver

The JDBC driver for OrientDB allows to connect to a remote server using the standard and consolidated way of interacting with database in the Java world.

## Include in your projects

To be used inside your project, simply add the dependency to your pom:

```xml
<dependency>
  <groupId>com.orientechnologies</groupId>
  <artifactId>orientdb-jdbc</artifactId>
  <version>ORIENTDB_VERSION</version>
</dependency>
```
_NOTE: to use SNAPSHOT version remember to add the Snapshot repository to your ```pom.xml```._

## How can be used in my code?

The driver is registered to the Java SQL DriverManager and can be used to work with all the OrientDB database types:
- memory,
- plocal and
- remote

The driver's class is ```com.orientechnologies.orient.jdbc.OrientJdbcDriver```. Use your knowledge of JDBC API to work against OrientDB.

## First get a connection

```java
Properties info = new Properties();
info.put("user", "admin");
info.put("password", "admin");

Connection conn = (OrientJdbcConnection) DriverManager.getConnection("jdbc:orient:remote:localhost/test", info);
```

Then execute a Statement and get the ResultSet:

```java
Statement stmt = conn.createStatement();

ResultSet rs = stmt.executeQuery("SELECT stringKey, intKey, text, length, date FROM Item");

rs.next();

rs.getInt("@version");
rs.getString("@class");
rs.getString("@rid");

rs.getString("stringKey");
rs.getInt("intKey");

rs.close();
stmt.close();
```

The driver retrieves OrientDB metadata (@rid,@class and @version) only on direct queries. Take a look at tests code to see more detailed examples.

## Advanced features

### Connection pool
By default a new database instance is created every time you ask for a JDBC connection. OrientDB JDBC driver provides a Connection Pool out of the box. Set the connection pool parameters before to ask for a connection:

```java
Properties info = new Properties();
info.put("user", "admin");
info.put("password", "admin");

info.put("db.usePool", "true"); // USE THE POOL
info.put("db.pool.min", "3");   // MINIMUM POOL SIZE
info.put("db.pool.max", "30");  // MAXIMUM POOL SIZE

Connection conn = (OrientJdbcConnection) DriverManager.getConnection("jdbc:orient:remote:localhost/test", info);
```

### Spark compatibility (from 2.2.7)

[Apache Spark](http://spark.apache.org/) allows reading and writing of DataFrames from JDBC data sources. 
The driver offers a compatibility mode to enable load of data frame from an OrientDb's class or query. 

```java
Map<String, String> options = new HashMap<String, String>() {{
    put("url", "jdbc:orient:remote:localhost/sparkTest");
    put("user", "admin");
    put("password", "admin");
    put("spark", "true"); // ENABLE Spark compatibility
    put("dbtable", "Item");
}};

SQLContext sqlCtx = new SQLContext(ctx);

DataFrame jdbcDF = sqlCtx.read().format("jdbc").options(options).load();
```

## Custom drivers

OrientDB JDBC driver is compatible with most of the tools that support the JDBC standard. Even if we have tested the OrientDB JDBC driver against the most popular BI/Reporting tools, some tool could use a feature not supported. If you have problems with your tool and the OrientDB JDBC driver, please [create an issue](https://github.com/orientechnologies/orientdb/issues?q=is%3Aopen+is%3Aissue).

For some tool, instead, in order to use OrientDB JDBC driver, you need an additional connector:

QlickView: https://www.tiq-solutions.de/en/solutions/qlik-solutions/jdbc-connector/
