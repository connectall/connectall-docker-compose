# ConnectAll

ConnectAll is a commercial and licensed application integration platform based on MuleSoft, Tomcat, and MySql.

## Getting Started

ConnectALL is an enterprise application integration solution that connects multiple tools and applications, enabling a company’s development and management teams to collaborate efficiently and flawlessly across multiple development platforms. Easy-to-buy, easy-to-install, and easy-to-use, ConnectALL meets strict enterprise governance requirements. It leverages commercial Enterprise Service Bus (ESB) technology to achieve an enterprise-grade infrastructure with clustering, multi-tenancy architecture, multiple server support both in the cloud and on-premise, traceability, and audit trails.

More details on the ConnectAll product including a complete documentation set can be found here: https://www.connectall.com.

To start the container first install Docker and docker-compose for your machine and download the contents of this Git project. To get access to the ConnectAll license, please fill out the form here: https://www.connectall.com/contact/.

## Prerequisite

- Docker (https://docs.docker.com/get-docker/)
- docker-compose (https://docs.docker.com/compose/install/)
- ConnectAll License (https://support.connectall.com/)

# Usage

This docker-compose file requires a database to be setup before it can be used. A sample database that can be added:

```yaml
db:
  image: mysql:5.7
  environment:
    - MYSQL_ROOT_PASSWORD=root
  entrypoint: sh -c "
    echo 'CREATE DATABASE IF NOT EXISTS connectall;' > /docker-entrypoint-initdb.d/init.sql;
    /usr/local/bin/docker-entrypoint.sh --bind-address=0.0.0.0
    "
  ports:
    - "3306:3306"
  networks:
    - cg-connectall
```

There are 3 images that ConnectAll will provide:

- `ca-cg-dbupdater:tag`
  - image that will prepoulate the database as a new database or perform and updates as nessassry
- `ca-cg-mule:tag`
  - image that runs the Mule portion of the product
- `ca-cg-tomcat:tag`
  - image that runs the Tomcat portion of the product

**When running for the first time :**

- `docker-compose up -d`

**For upgrades**

Edit the images with the new versions before redeploying the containers.

**Volumes**

- connectall_db_conf - Where the encrypted database password for the database is stored
- connectall_current_sql - Where the current batch of .sql statements are stored
- connectall_database - ConnectAll's internal storage
- connectall_logs - ConnectAll's logs

# Notes

- If Tomcat refuses to startup, check if this error appears in the `localhost.yyy-mm-dd.log` log:

```log
08-Jan-2020 10:32:26.755 SEVERE [localhost-startStop-1] org.apache.catalina.core.StandardContext.listenerStart Exception sending context initialized event to listener instance of class [org.springframework.web.context.ContextLoaderListener]
 java.lang.NoSuchMethodError: org.slf4j.spi.LocationAwareLogger.log(Lorg/slf4j/Marker;Ljava/lang/String;ILjava/lang/String;Ljava/lang/Throwable;)V
        at org.apache.commons.logging.impl.SLF4JLocationAwareLog.error(SLF4JLocationAwareLog.java:173)
        at org.springframework.web.context.ContextLoader.initWebApplicationContext(ContextLoader.java:312)
        at org.springframework.web.context.ContextLoaderListener.contextInitialized(ContextLoaderListener.java:103)
        at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:4792)
        at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5256)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
        at org.apache.catalina.core.ContainerBase.addChildInternal(ContainerBase.java:754)
        at org.apache.catalina.core.ContainerBase.addChild(ContainerBase.java:730)
        at org.apache.catalina.core.StandardHost.addChild(StandardHost.java:734)
        at org.apache.catalina.startup.HostConfig.deployWAR(HostConfig.java:985)
        at org.apache.catalina.startup.HostConfig$DeployWar.run(HostConfig.java:1857)
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)

08-Jan-2020 10:32:29.014 INFO [localhost-startStop-1] org.apache.catalina.core.ApplicationContext.log Closing Spring root WebApplicationContext
08-Jan-2020 10:32:29.020 SEVERE [localhost-startStop-1] org.apache.catalina.core.StandardContext.listenerStop Exception sending context destroyed event to listener instance of class [org.springframework.web.context.ContextLoaderListener]
 java.lang.IllegalStateException: BeanFactory not initialized or already closed - call 'refresh' before accessing beans via the ApplicationContext
        at org.springframework.context.support.AbstractRefreshableApplicationContext.getBeanFactory(AbstractRefreshableApplicationContext.java:177)
        at org.springframework.context.support.AbstractApplicationContext.destroyBeans(AbstractApplicationContext.java:1035)
        at org.springframework.context.support.AbstractApplicationContext.doClose(AbstractApplicationContext.java:1011)
        at org.springframework.context.support.AbstractApplicationContext.close(AbstractApplicationContext.java:961)
        at org.springframework.web.context.ContextLoader.closeWebApplicationContext(ContextLoader.java:516)
        at org.springframework.web.context.ContextLoaderListener.contextDestroyed(ContextLoaderListener.java:112)
        at org.apache.catalina.core.StandardContext.listenerStop(StandardContext.java:4839)
        at org.apache.catalina.core.StandardContext.stopInternal(StandardContext.java:5478)
        at org.apache.catalina.util.LifecycleBase.stop(LifecycleBase.java:226)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:154)
        at org.apache.catalina.core.ContainerBase.addChildInternal(ContainerBase.java:754)
        at org.apache.catalina.core.ContainerBase.addChild(ContainerBase.java:730)
        at org.apache.catalina.core.StandardHost.addChild(StandardHost.java:734)
        at org.apache.catalina.startup.HostConfig.deployWAR(HostConfig.java:985)
        at org.apache.catalina.startup.HostConfig$DeployWar.run(HostConfig.java:1857)
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)
```

If this error exists, please remove the following file from the Tomcat container:

- `/usr/local/tomcat/webapps/ConnectAll/WEB-INF/lib/jcl104-over-slf4j-1.4.3.jar`

After the file has been removed from the container, it can be restarted.

# Variables

| Variable              | Description                                            | Vaule(s)                                                                                                                                                                     | Example                                                                                                                                                                                                                                                                                              |
| --------------------- | ------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CA_NEW_OR_UPGRADE     | Used in `dbupdater` to setup or update the database    | `new` or `update`                                                                                                                                                            |                                                                                                                                                                                                                                                                                                      |
| CA_DATABASE_TYPE      | Database representation                                | - `0` = ORACLE<br/>- `1` = MSSQL<br/>- `2` = MYSQL<br/>- `3` = POSTGRES                                                                                                      |                                                                                                                                                                                                                                                                                                      |
| CA_JDBC_DRIVER        | Name of the database driver to connect to the database | - `oracle.jdbc.driver.OracleDriver` = ORACLE<br/>- `net.sourceforge.jtds.jdbc.Driver` = MSSQL<br/>- `com.mysql.jdbc.Driver` = MYSQL<br/>- `org.postgresql.Driver` = POSTGRES |                                                                                                                                                                                                                                                                                                      |
| CA_DATABASE_JDBC      | JDBC URL to connect to the database                    |                                                                                                                                                                              | - `jdbc:oracle:thin:@db:1521/ORCL`= ORACLE<br/>- `"jdbc:sqlserver://db:1433;databaseName=connectall"` = MSSQL<br/>- `"jdbc:mysql://db:3306/connectall?useUnicode=true&characterEncoding=UTF-8&characterSetResults=UTF-8&useSSL=false"` = MYSQL<br/>- `"jdbc:postgresql://db:5432/jiradb"` = POSTGRES |
| CA_DATABASE_USERNAME  | Username to connect to the database                    |                                                                                                                                                                              |                                                                                                                                                                                                                                                                                                      |
| CA_DATABASE_PASSSWORD | Password to connect to the database                    |                                                                                                                                                                              |                                                                                                                                                                                                                                                                                                      |
|                       |                                                        |                                                                                                                                                                              |                                                                                                                                                                                                                                                                                                      |
