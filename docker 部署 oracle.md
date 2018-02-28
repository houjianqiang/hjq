# docker 部署 oracle

标签（空格分隔）： docker oracle

---

首先要安装docker。

接下来，我需要获取新的Oracle数据库容器。你有两个选择：

1. 回到Docker Store并搜索“oracle数据库”，它将返回一个拥有12.1.0.2数据库但不是持久映像的Docker容器。这意味着如果你删除容器，数据库也会消失。也无法从容器内拔出数据库并将其插入其他任何地方。

2. 转到Github并获取Oracle Docker构建文件，然后转到Oracle.com并获取12.2数据库软件。这种组合可以让你创建一个拥有最新的12.2数据库的Docker容器并且持久化。也就是说，如果容器被丢弃，您仍然拥有数据库，并且可以拔出数据库并将其插入其他位置。

我选择了选项2，这是一个稍微多一点的参与，但拥有12.2和持久的形象对我来说都非常重要。而且在你问之前，是否会在Docker Store中提供12.2，并且它会持久。这只是还没有。

`1`. 转到https://github.com/oracle/docker-images并下载Oracle Docker构建文件。
![docker-images](https://i2.wp.com/sqlmaria.com/wp-content/uploads/2017/04/Github.png?resize=768%2C390&ssl=1)
`2`.转到http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html并下载用于Linux x86-64的Oracle Database 12c第2版。
![oracle](https://i2.wp.com/sqlmaria.com/wp-content/uploads/2017/04/oracle_download.png?w=626&ssl=1)
`3`. 您现在应该有俩个文件：
    
    - i.docker-images-master.zip 
    - ii. linuxx64_12201_database.zip

解压docker-images-master.zip文件，这将产生一个名为docker-images-master的新目录。该目录包含12个子目录，这是Docker支持的每个Oracle产品之一。

    unzip docker-images-master.zip
    
    Archive: docker-images-master.zip
    ce91c58275d24df32b3f5d3b8a68000ade61d562
    creating: docker-images-master/
    extracting: docker-images-master/.gitattributes
    inflating: docker-images-master/.gitignore
    inflating: docker-images-master/.gitmodules
    ...
    ...
    ...
    inflating: docker-images-master/README.md
    
`4`.将12.2软件（linuxx64_12201_database.zip）复制到/docker-images-master/OracleDatabase/dockerfiles/12.2.0.1子目录中。
`5`.接下来，您需要创建Docker镜像。我采用了简单的路线，并使用了docker-images-master.zip中包含的buildDockerImage.sh shell脚本。它将使用oraclelinux创建一个映像：7-slim以及您指定的任何数据库版本。我使用以下命令来安装Oracle Database 12.2软件：

``buildDockerImage.sh 脚本的文件位置： /docker-images-master/OracleDatabase/dockerfiles/``
    
    ./buildDockerImage.sh -v 12.2.0.1 -e
    Checking if required packages are present and valid...
    linuxamd64_12201_database.zip: OK
    =====================
    Building image 'oracle/database:12.2.0.1-ee' ...
    Sending build context to Docker daemon 2.688 GB
    
`请注意`,安装程序通过互联网获得oraclelinux：7-slim并在容器内部对操作系统进行了yum更新，因此如果您在企业防火墙后执行此操作，则需要明确设置环境变量https_proxy。buildDockerImage.sh脚本会自动检测http_proxy和https_proxy，并将其传递给Docker以用于图像构建.

`6`.图像生成后，您可以通过运行“docker images”命令来检查您的图像.
    
    docker images
|REPOSITORY| TAG |IMAGEID| CREATED |SIZE|
|:------:|:------:|:------:|:------:|:------:|
|oracle/database  |12.2.0.1-ee |8d82cbde2781| About an hour ago|   13.3GB|

`7`.为了获得一个实际的数据库，我们需要创建我们的第一个容器。我使用下面的“docker run”命令来执行此操作，并为映射提供了一些附加参数，因为即使删除了容器，我也想保留数据库文件。
    
    docker run --name oracle -p 1521:1521 -p 5500:5500 -v /Users/mcolgan-mac/oradata:/opt/oracle/oradata oracle/database:12.2.0.1-ee
    LSNRCTL for Linux: Version 12.2.0.1.0 - 
    Production on 26-APR-2017 18:04:36
     
    Copyright (c) 1991, 2016, Oracle.  All rights reserved.
     
    Starting /opt/oracle/product/12.2.0.1/dbhome_1/bin/tnslsnr: 
    please wait...
     
    TNSLSNR for Linux: Version 12.2.0.1.0 - Production
    System parameter file is /opt/oracle/product/12.2.0.1/dbhome_1/network/admin/listener.ora
    Log messages written to /opt/oracle/diag/tnslsnr/75ca99d5275d/listener/alert/log.xml
    Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=75ca99d5275d)(PORT=1521)))
     
    Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
    STATUS of the LISTENER
    ------------------------
`-name`将新容器的名称命名为“oracle”（如果省略，将自动生成）
`-p`笔记本计算机上的端口1521和5500映射到容器内的相应端口
`-v`映射我的本地目录（/Users/mcolgan-mac/oradata）复制到数据文件将被存储的默认位置（:/opt/oracle/oradata），以确保文件在我的容器外保存。
``(如果oracle没有权限的话会失败，sudo chmod -R +w /opt/oracle;或者直接将oracle文件夹映射在你当前的用户文件夹下，docker run 命令改成: docker run --name oracle -p 1521:1521 -p 5500:5500 -v /Users/mcolgan-mac/oradata:/home/$USER/oracle/oradata oracle/database:12.2.0.1-ee)``

    如果过程中出现问题也可以去github上讨论 https://github.com/oracle/docker-images/issues

`8`.“docker run”命令实际上调​​用Oracle DBCA并自动创建一个数据库。由于这是一个12c数据库，它会自动创建一个容器数据库ORCLCDB和一个可插入数据库ORCLPDB1（如果你愿意，它们都可以被覆盖）。
我没有在我的命令中为SYS，SYSTEM或PDBADMIN帐户指定密码，所以我自动为我生成了一个密码并显示在命令的输出中。请务必从此默认值更改密码。数据库启动并运行后，您可以使用“docker exec”命令执行此操作。
    
    docker exec oracle ./setPassword.sh XXXXXX

`9`.我们可以确认我们的容器已成功创建并正在使用“docker ps”命令运行。
    
    docker ps -a 
|CONTAINER ID|IMAGE|COMMAND| CREATED|STATUS|PORTS|NAMES|
|:------:|:------:|:------:|:------:|:------:|:------:|:------:|
|75ca99d5275d| oracle/database:12.2.0.1-ee|"/bin/sh -c 'exec ..."   |8 hours ago | Up 5 hours  | 0.0.0.0:1521/tcp|oracle|

`10`.通过使用docker log命令查看docker日志，您可以精确地监控这些命令期间发生的情况。

    docker start oracle
    docker stop oracle

通过使用docker log命令查看docker日志，您可以精确地监控这些命令期间发生的情况。
   
    docker logs oracle
    
    LSNRCTL for Linux: Version 12.2.0.1.0 - Production on 27-APR-2017 18:58:05
     
    Copyright (c) 1991, 2016, Oracle.  All rights reserved.
     
    Starting /opt/oracle/product/12.2.0.1/dbhome_1/bin/tnslsnr: please wait...
     
    TNSLSNR for Linux: Version 12.2.0.1.0 - Production
    System parameter file is /opt/oracle/product/12.2.0.1/dbhome_1/network/admin/listener.ora
    Log messages written to /opt/oracle/diag/tnslsnr/75ca99d5275d/listener/alert/log.xml
    Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=75ca99d5275d)(PORT=1521)))
     
    Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
    STATUS of the LISTENER
    ------------------------
    Alias                     LISTENER
    Version                   TNSLSNR for Linux: Version 12.2.0.1.0 - Production
    Start Date                27-APR-2017 18:58:05
    Uptime                    0 days 0 hr. 0 min. 2 sec
    Trace Level               off
    Security                  ON: Local OS Authentication
    SNMP                      OFF
    Listener Parameter File   /opt/oracle/product/12.2.0.1/dbhome_1/network/admin/listener.ora
    Listener Log File         /opt/oracle/diag/tnslsnr/75ca99d5275d/listener/alert/log.xml
    Listening Endpoints Summary...
      (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=75ca99d5275d)(PORT=1521)))
    The listener supports no services
    The command completed successfully
     
    SQL*Plus: Release 12.2.0.1.0 Production on Thu Apr 27 18:58:08 2017
     
    Copyright (c) 1982, 2016, Oracle.  All rights reserved.
     
    Connected to an idle instance.
     
    SQL&gt; ORACLE instance started.
     
    Total System Global Area 1610612736 bytes
    Fixed Size		    8793304 bytes
    Variable Size		  520094504 bytes
    Database Buffers	 1073741824 bytes
    Redo Buffers		    7983104 bytes
    Database mounted.
    Database opened.
    SQL&gt; Disconnected from Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
    #########################
    DATABASE IS READY TO USE!
    #########################
    ORCLPDB1(3):Database Characterset for ORCLPDB1 is AL32UTF8
    2017-04-27T18:58:20.468112+00:00
    ORCLPDB1(3):Opatch validation is skipped for PDB ORCLPDB1 (con_id=0)
    2017-04-27T18:58:21.545187+00:00
    ORCLPDB1(3):Opening pdb with no Resource Manager plan active
    Pluggable database ORCLPDB1 opened read write
    Starting background process CJQ0
    Completed: ALTER DATABASE OPEN
    2017-04-27T18:58:22.163089+00:00
    CJQ0 started with pid=40, OS id=262 
    2017-04-27T18:58:22.589640+00:00
    Shared IO Pool defaulting to 64MB. Trying to get it from Buffer Cache for process 81.
    ===========================================================
    Dumping current patch information
    ===========================================================
    No patches have been applied
    ===========================================================
    2017-04-27T18:58:23.381214+00:00
    db_recovery_file_dest_size of 12780 MB is 0.00% used. This is a
    user-specified limit on the amount of space that will be used by this
    database for recovery-related files, and does not reflect the amount of
    space available in the underlying filesystem or ASM diskgroup.

因此，您可以通过十个简单的步骤在Docker上启动并运行Oracle Database 12.2。如果您想了解更多有关构建Oracle数据库Docker镜像的细节，请查看[Gerald][1]的博客。


  [1]: https://geraldonit.com/2017/08/21/creating-an-oracle-database-docker-image/