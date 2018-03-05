# Ubuntu14.04或16 安装Oracle SQL Developer

标签（空格分隔）： oracle sql-developer

---
``1``.下载
首先保证你装的JDK在1.8版本以上。
然后下载sqlDeveloper的安装包。
``链接``： http://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html
选择 Linux RPM

    (0e5be16b84af29cd19dfc2f8d3aaa1da)
    Installation Notes, JDK 8 or above required Download
``2``.安装

    1. 将rpm转成deb
        
        sudo apt-get install tar
        sudo apt-get install alien
    
如果tar，alien安装完成了,就可以开始转换下载好的rpm安装包。
我的包名为:
        ``sqldeveloper-17.4.0.355.2349-1.noarch.rpm``
        
        开始转换：
            sudo alien -cv sqldeveloper-17.4.0.355.2349-1.noarch.rpm
        得到一个deb包：
            ``sqldeveloper_17.4.0.355.2349-2_all.deb``
    
    2.开始安装
    sudo chmod +x sqldeveloper_17.4.0.355.2349-2_all.deb
    sudo dpkg -i sqldeveloper_17.4.0.355.2349-2_all.deb
    
    静静等待安装成功...
2.解决sqldeveloper 乱码
    
    在SQL Developer中，打开显示的是系统默认的语言。若想使用其他语言又该怎么办呢？
    首先，找到SQL Developer的安装目录。
    随后，在“ide\bin"目录下打开ide.conf文件。
    然后，将AddVMOption -Duser.language=en 与 AddVMOption -Duser.country=US 各占一行写入。
    最后，保存文件，重启SQL Developer.
    注释：
    上面显示的是中文简体的配置，若想使用其他语言需要调整配置。具体值，请参考下表。
    
    AddVMOption -Duser.language    
    AddVMOption -Duser.country

|语言名称 	|语系（language） |	国家（country）|
|:-----:|:-----:|:-----:|
|中文简体 |	zh |	CN|
|中文繁体（台湾） 	|zh |	TW|
|中文繁体（香港） |	zh |	HK|
|英文（美国） |	en 	|US|
|日文 |	ja 	|JA|
|韩文 |	ko 	|KO|
|俄文 |	ru 	|RU|


``重启sqldeveloper``

3.打开Oracle SQL Developer
    
    打开命令：
    sqldeveloper
    
    Oracle SQL Developer
    Copyright (c) 1997, 2017, Oracle and/or its affiliates. All rights reserved.
    
    .......
    
4.软件安装路径
    
    /opt/sqldeveloper

5.软件语言设置
    
    /opt/sqldeveloper/ide/bin 
    
增加

    AddVMOption -Duser.language=en  
    AddVMOption -Duser.country=US 
    
``将 en US替换成需要的语言 ``

``官网安装教程：``

    This download does not include the JDK. You can connect to and use any JDK 1.8 or above.

    To install and run:

    - Ensure you have a JDK installed, if not, download here
    - rpm -Uhv sqldeveloper-(build number)-1.noarch.rpm (install the package)
    - cd sqldeveloper (go to sqldeveloper folder)
    - ./sqldeveloper.sh (run sqldeveloper.sh file)
    - You will be prompted to enter a jdk path. (ie usr/java/jdk1.8.0_25)
    - SQL Developer will automatically launch once jdk location is provided




