Here are some notes regarding Oracle. The motivation is https://github.com/RotherOSS/otobo/issues/873.

Starting with no Oracle at all.

# Server

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

# Client

Get the instantclient from 21.1.0.0.0. 
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
care must be taken to log on to ORCLPDB1 and not to ORCLCDB. While we are at it,
let's create schemas for _otrs_ and _otobo_.

    bes:/opt/otrs/scripts/database $ sqlplus sys/Oradoc_db1@ORCLPDB1 as sysdba

    SQL*Plus: Release 21.0.0.0.0 - Production on Mon Mar 22 17:15:25 2021
    Version 21.1.0.0.0

    Copyright (c) 1982, 2020, Oracle.  All rights reserved.

    Last Successful login time: Mon Mar 22 2021 17:05:42 +01:00

    Connected to:
    Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

    SQL> create user otrs identified by otrs
    2  ;

    User created.

    SQL> create user znuny identified by znuny   
    2  ;

    User created.


# DBD::Oracle

    sudo     cpanm DBD    ::Ora    cle

# InstOTRS 6 ((Community Edition))

https://doc.znuny.org/doc/manual/admin/6.0/en/html/manual-installation-of-otrs.html
