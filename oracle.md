Here are some notes regarding Oracle. The motivation is https://github.com/RotherOSS/otobo/issues/873.

Starting with no Oracle at all under Ubuntu 20.04.

# Oracle Server

- The obvious solution would be Oracle Cloud Free Tier, https://www.oracle.com/cloud/free/, but that does not accept my credit card.
- Fallback https://hub.docker.com/_/oracle-database-enterprise-edition 
  - this Oracle 12c R2 from 2017
  - License for development is given via OTN https://www.oracle.com/downloads/licenses/standard-license.html
  - Documentation can only be seen after 'Proceed to checkout'
  - also: https://github.com/oracle/docker-images/tree/main/OracleDatabase/SingleInstance
  - ` docker run -d -it --name oracle_otobo_1  -p 1521:1521 -p 5500:5500 store/oracle/database-enterprise:12.2.0.1-slim` 
    - downloading about 1.5 G.
    - keeping the ports used in the image
    - do not bother to keep setup and data in Oracle persistent
    - takes a bit to start up, check health with `docker ps`

# Oracle Client

Get the instantclient 21.1.0.0.0 from https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html. 
- oracle-instantclient-basic-21.1.0.0.0-1.x86_64.rpm
- oracle-instantclient-sqlplus-21.1.0.0.0-1.x86_64.rpm.
- oracle-instantclient-devel-21.1.0.0.0-1.x86_64.rpm

Follow instructions from https://www.rosehosting.com/blog/how-to-install-rpm-packages-on-ubuntu/.

    bernhard@bernhard-HP-250-G6-Notebook-PC:~$ cat /etc/profile.d/oracle.sh 
    # added by bernhard 2021-03-22
    export ORACLE_HOME=/usr/lib/oracle/21/client64
    export LD_LIBRARY_PATH=$ORACLE_HOME/lib
    export TNS_ADMIN=$ORACLE_HOME/network/admin
    bernhard@bernhard-HP-250-G6-Notebook-PC:~$ sqlplus -v

    SQL*Plus: Release 21.0.0.0.0 - Production
    Version 21.1.0.0.0
    
Tell the instant client about the running database:

    bernhard@bernhard-HP-250-G6-Notebook-PC:~$ cat $TNS_ADMIN/tnsnames.ora
    ORCLCDB=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=127.0.0.1)(PORT=1521))
        (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLCDB.localdomain)))
    ORCLPDB1=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=127.0.0.1)(PORT=1521))
        (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB1.localdomain)))
        
Oracle 12 has a feature called multitenant container database (CDB). Whatever this is,
care must be taken to log on to ORCLPDB1 and not to ORCLCDB.


    bes:/opt/otrs/scripts/database $ sqlplus sys/Oradoc_db1@ORCLPDB1 as sysdba

    SQL*Plus: Release 21.0.0.0.0 - Production on Mon Mar 22 17:15:25 2021
    Version 21.1.0.0.0

    Copyright (c) 1982, 2020, Oracle.  All rights reserved.

    Last Successful login time: Mon Mar 22 2021 17:05:42 +01:00

    Connected to:
    Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

    SQL> SELECT TO_CHAR(SYSDATE, 'MM-DD-YYYY HH24:MI:SS') FROM dual;

    TO_CHAR(SYSDATE,'MM
    -------------------
    03-24-2021 09:09:26

    SQL> exit
    Disconnected from Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production


# Set up Users and Schemas _otrs_ and _otobo_

Create the file _oracle_setup.sql_ and run the commands:

    bes:~/devel/OTOBO/otobo (issue-#873-oracle_migration)$ cat oracle_setup.sql 
    CREATE USER otobo IDENTIFIED BY otobo;
    GRANT CREATE SESSION TO otobo;
    GRANT CREATE TABLE TO otobo;
    GRANT CREATE SEQUENCE TO otobo;
    GRANT CREATE TRIGGER TO otobo;
    ALTER USER otobo QUOTA UNLIMITED ON users;

    CREATE USER otrs IDENTIFIED BY otrs;
    GRANT CREATE SESSION TO otrs;
    GRANT CREATE TABLE TO otrs;
    GRANT CREATE SEQUENCE TO otrs;
    GRANT CREATE TRIGGER TO otrs;
    ALTER USER otrs QUOTA UNLIMITED ON users;

    sqlplus sys/Oradoc_db1@ORCLPDB1 as sysdba < oracle_setup.sql
    
Doublecheck with: `sqlplus otrs/otrs@ORCLPDB1`

# DBD::Oracle

    sudo cpanm DBD::Oracle
    
# Install OTRS 6 ((Community Edition))

Follow instructions in https://doc.znuny.org/doc/manual/admin/6.0/en/html/manual-installation-of-otrs.html   

## Find the correct service name

`docker exec -it oracle_otobo_1 bash ` and then `lsnrctl services` look for the **pdb1** entry.

## Connection settings

Open http://localhost/otrs/installer.pl, choose Oracle, and supply these settings:

    Host: 127.0.0.1
    SID: orclpdb1.localdomain
    Port: 1521
    Benutzer: otrs
    Passwort: otrs

# Install OTOBO

Follow more or less the instructions in https://doc.otobo.org/manual/installation/stable/en/content/installation.html . In this case link /opt/otobo to a OTOBO sandbox.

## Connection settings

Open http://localhost/otobo/installer.pl, choose Oracle, and supply these settings:

    Host: 127.0.0.1
    SID: orclpdb1.localdomain
    Port: 1521
    Benutzer: otobo
    Passwort: otobo
