# autonomous_recovery

**Testes com DBRS**

# autonomous_recovery

PMs : Alex Goldblat 


**Documentacao**

1. > Log de backup FULL com ZDLRS : 

Caminho de logs para backup : /var/opt/oracle/log/DB26AI/dtrs/rman/bkup

<img width="1663" height="852" alt="Image" src="https://github.com/user-attachments/assets/88db7adb-d7b3-44fb-bdbf-0d9174464ffb" />

Atenção para COMPRESSION LOW : 

```
Version 23.26.0.0.0

Copyright (c) 1982, 2025, Oracle and/or its affiliates.  All rights reserved.

connected to target database: DB26AI (DBID=4146941561)
connected to recovery catalog database
PL/SQL package BRXJZKT504LXYAM9CG0IZJYMX.DBMS_RCVCAT version 23.04.00.23. in RCVCAT database is not current version 23.26.00.00
PL/SQL package BRXJZKT504LXYAM9CG0IZJYMX.DBMS_RCVMAN version 23.04.00.23 in RCVCAT database is not current version 23.26.00.00

RMAN> set echo on;
2> SET COMPRESSION ALGORITHM 'low' AS OF RELEASE 'DEFAULT' OPTIMIZE FOR LOAD TRUE;
3> set encryption ON;
4> show all;
5>
6> RUN
7> {
8> set command id to "c036fe77-4ff5-4b10-bd12-5797da0e";
9> ALLOCATE CHANNEL 'RA0' DEVICE TYPE SBT_TAPE PARMS  "SBT_LIBRARY=/u02/app/oracle/product/23.0.0.0/dbhome_2/lib/libra.so, ENV=(RA_WALLET='location=file:/var/opt/oracle/dbaas_acfs/DB26AI/wallet_root/server_seps credential_alias=DBRS',RA_FORMAT=TRUE)";
10>
11> BACKUP AS   BACKUPSET ARCHIVELOG UNTIL TIME 'SYSDATE + 1' NOT BACKED UP  FILESPERSET 1 ;
12> BACKUP AS  BACKUPSET CURRENT CONTROLFILE tag 'AL_AUTO_23_12_2025_101900'  SPFILE tag 'AL_AUTO_23_12_2025_101900'  ;
13> }
14>
echo set on

executing command: SET compression

executing command: SET encryption

RMAN configuration parameters for database with db_unique_name STDAA265 are:
CONFIGURE RETENTION POLICY TO REDUNDANCY 1; # default
CONFIGURE BACKUP OPTIMIZATION OFF;
CONFIGURE DEFAULT DEVICE TYPE TO 'SBT_TAPE';
CONFIGURE CONTROLFILE AUTOBACKUP ON;
CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE SBT_TAPE TO '%F'; # default
CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '%F'; # default
CONFIGURE DEVICE TYPE 'SBT_TAPE' PARALLELISM 2 BACKUP TYPE TO COMPRESSED BACKUPSET;
CONFIGURE DEVICE TYPE DISK PARALLELISM 1 BACKUP TYPE TO BACKUPSET; # default
CONFIGURE DATAFILE BACKUP COPIES FOR DEVICE TYPE SBT_TAPE TO 1; # default
CONFIGURE DATAFILE BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default
CONFIGURE ARCHIVELOG BACKUP COPIES FOR DEVICE TYPE SBT_TAPE TO 1; # default
CONFIGURE ARCHIVELOG BACKUP COPIES FOR DEVICE TYPE DISK TO 1; # default
CONFIGURE CHANNEL DEVICE TYPE 'SBT_TAPE' FORMAT   '%d_%I_%U_%T_%t' PARMS  "SBT_LIBRARY=/u02/app/oracle/product/23.0.0.0/dbhome_2/lib/libra.so, ENV=(RA_WALLET='location=file:/var/opt/oracle/dbaas_acfs/DB26AI/wallet_root/server_seps credential_alias=DBRS',RA_FORMAT=TRUE)";
CONFIGURE MAXSETSIZE TO UNLIMITED; # default
CONFIGURE ENCRYPTION FOR DATABASE ON;
CONFIGURE ENCRYPTION ALGORITHM 'AES256'; # default
CONFIGURE COMPRESSION ALGORITHM 'low' AS OF RELEASE 'DEFAULT' OPTIMIZE FOR LOAD TRUE;
CONFIGURE DB_UNIQUE_NAME 'DB26AI_ppb_yul' CONNECT IDENTIFIER  'DB26AI_ppb_yul';
CONFIGURE DB_UNIQUE_NAME 'stdaa265' CONNECT IDENTIFIER  'stdaa265';
CONFIGURE RMAN OUTPUT TO KEEP FOR 7 DAYS; # default
CONFIGURE ARCHIVELOG DELETION POLICY TO NONE; # default
CONFIGURE SNAPSHOT CONTROLFILE NAME TO '@q2Ln2trs/VMCND1-38DB22E9C357EF86BFF7858469BFF4E9/STDAA265/CONTROLFILE/snapcf_DB26AI.f';

executing command: SET COMMAND ID

```


Para o backup do archive temos isso : 

```


Recovery Manager: Release 23.26.0.0.0 - for Oracle Cloud and Engineered Systems on Tue Dec 23 10:19:08 2025
Version 23.26.0.0.0

Copyright (c) 1982, 2025, Oracle and/or its affiliates.  All rights reserved.

connected to target database: DB26AI (DBID=4146941561)
connected to recovery catalog database
PL/SQL package BRXJZKT504LXYAM9CG0IZJYMX.DBMS_RCVCAT version 23.04.00.23. in RCVCAT database is not current version 23.26.00.00
PL/SQL package BRXJZKT504LXYAM9CG0IZJYMX.DBMS_RCVMAN version 23.04.00.23 in RCVCAT database is not current version 23.26.00.00

RMAN> set echo on;
2> SET COMPRESSION ALGORITHM 'low' AS OF RELEASE 'DEFAULT' OPTIMIZE FOR LOAD TRUE;
3> set encryption ON;
4> show all;
5>
6> RUN
7> {
8> set command id to "c036fe77-4ff5-4b10-bd12-5797da0e";
9> ALLOCATE CHANNEL 'RA0' DEVICE TYPE SBT_TAPE PARMS  "SBT_LIBRARY=/u02/app/oracle/product/23.0.0.0/dbhome_2/lib/libra.so, ENV=(RA_WALLET='location=file:/var/opt/oracle/dbaas_acfs/DB26AI/wallet_root/server_seps credential_alias=DBRS',RA_FORMAT=TRUE)";
10>
11> BACKUP AS   BACKUPSET ARCHIVELOG UNTIL TIME 'SYSDATE + 1' NOT BACKED UP  FILESPERSET 1 ;
12> BACKUP AS  BACKUPSET CURRENT CONTROLFILE tag 'AL_AUTO_23_12_2025_101900'  SPFILE tag 'AL_AUTO_23_12_2025_101900'  ;
13> }
14>
echo set on

executing command: SET compression

executing command: SET encryption


```
1. > Verificar os pre-requisitos listados abaixo : 

      [https://blogs.oracle.com/infrastructure/post/autonomous-recovery-service-checklist](url)

2. > O Real Time Protection esta habilitado para todas as Editions ( Enterprise & Standard Edition) , o que seria requisito é ser 19.18 e preferencialmente > 19.20.

 Quando habilitamos o RTP, o parametro **redo_transport_user** é configurado, exemplo : 

> SQL> show parameter redo
> 
> NAME                                 TYPE        VALUE
> ------------------------------------ ----------- ------------------------------
> redo_transport_user                  string      WQQGOIN3ZFMDTGIGBXCGQWWXFCIO
> SQL>

Mesmo habilitando/desabilitando o RTP, o usuário permanece o mesmo. Fizemos um teste desativando o RTP e depois ativando novamente, verificamos que o usuário não mudou.

Alem desse parametro, outros tres parametros são configurados : **log_archive_dest_2/ log_archive_dest_3 e log_archive_config**

```
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
log_archive_dest_2                   string      service=DBRS_PRIMARY ENCRYPTION=ENABLE 
                                                                ASYNC NOAFFIRM db_unique_name=DBRS_PRIMARY valid_fo
                                                                r=(online_logfiles,all_roles) group=1 priority=1 net_timeout=8 max_failure=1

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
log_archive_dest_3                   string      service=DBRS_ALTERNATE ENCRYPTION=ENABLE ASYNC NOAFFIRM 
                                                                db_unique_name=DBRS_ALTERNATE 
                                                                valid_for=(online_logfiles,all_roles) group=1 priority=2 net_timeout=8 max_failure=1


log_archive_config                 string        dg_config=(DBRS_PRIMARY,DBRS_ALTERNATE,DB23AI_hv6_gru)
```
Dessa forma, vamos ter quase um Data Guard Max Performance (ASYNC + ENCRYPTION) para duas instances em Fault Domains diferentes:

Essas entradas de **DBRS_PRIMARY** & **DBRS_ALTERNATE** ja ficam salvas no arquivo dbrsnames.ora localizado no seguinte diretorio : 
```
[root@host23ai DB23AI_hv6_gru]# pwd
/opt/oracle/dcs/commonstore/dbrs/DB23AI_hv6_gru
[root@host23ai DB23AI_hv6_gru]#

[root@host23ai DB23AI_hv6_gru]# cat dbrsnames.ora
DBRS=(DESCRIPTION_LIST=(LOAD_BALANCE=off)(FAILOVER=on)(DESCRIPTION=(FAILOVER=on)(CONNECT_TIMEOUT=3)(RETRY_COUNT=3)(TRANSPORT_CONNECT_TIMEOUT=3)(ADDRESS_LIST=(LOAD_BALANCE=on)(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup004-1.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484))(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup004-3.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484))(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup004-2.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ZRCV_L6UZA3APPGKQHJJT12Q1))(SECURITY=(MY_WALLET_DIRECTORY=/opt/oracle/dcs/commonstore/wallets/DB23AI_hv6_gru/server_seps)))(DESCRIPTION=(FAILOVER=on)(CONNECT_TIMEOUT=3)(RETRY_COUNT=3)(TRANSPORT_CONNECT_TIMEOUT=3)(ADDRESS_LIST=(LOAD_BALANCE=on)(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup002-2.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484))(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup002-3.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484))(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup002-1.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ZRCV_NK37FF6HTIVEM2FFVEZSKZQWI3RV))(SECURITY=(MY_WALLET_DIRECTORY=/opt/oracle/dcs/commonstore/wallets/DB23AI_hv6_gru/server_seps))))
DBRS_PRIMARY=(DESCRIPTION=(FAILOVER=on)(CONNECT_TIMEOUT=3)(RETRY_COUNT=3)(TRANSPORT_CONNECT_TIMEOUT=3)(ADDRESS_LIST=(LOAD_BALANCE=on)(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup004-1.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484))(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup004-3.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484))(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup004-2.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ZRCV_L6UZA3APPGKQHJJT12Q1))(SECURITY=(MY_WALLET_DIRECTORY=/opt/oracle/dcs/commonstore/wallets/DB23AI_hv6_gru/server_seps)))
DBRS_ALTERNATE=(DESCRIPTION=(FAILOVER=on)(CONNECT_TIMEOUT=3)(RETRY_COUNT=3)(TRANSPORT_CONNECT_TIMEOUT=3)(ADDRESS_LIST=(LOAD_BALANCE=on)(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup002-2.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484))(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup002-3.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484))(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup002-1.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ZRCV_NK37FF6HTIVEM2FFVEZSKZQWI3RV))(SECURITY=(MY_WALLET_DIRECTORY=/opt/oracle/dcs/commonstore/wallets/DB23AI_hv6_gru/server_seps)))
[root@host23ai DB23AI_hv6_gru]#

```
  **Lembre-se de liberar  stateful ingress rules para as portas  2484 e 8005, mesmo que esteja na mesma subnet do banco de dados.**


Checklist for Security rules (Security List or NSG)

> 
> 1. Rule 1 - Ingress. Allow HTTPS traffic from anywhere
> 
>  
> 
>     Stateless: No (All rules must be stateful)
>     Source Type: CIDR
>     Source CIDR : CIDR of the VCN where the database resides
>     IP Protocol: TCP
>     Source Port Range: All
>     Destination Port Range: 8005
> 
>  
> 2. Rule 2 - Ingress. Allow SQLNet traffic from anywhere
> 
>  
> 
>     Stateless: No (All rules must be stateful)
>     Source Type: CIDR
>     Source CIDR : CIDR of the VCN where the database resides
>     IP Protocol: TCP
>     Source Port Range: All
>     Destination Port Range: 2484
> 
> NOTE: If your VCN restricts network traffic between subnets, ensure to add an egress rule for ports 2484, and 8005 from the database subnet to the Recovery Service subnet that you create.
> NOTE: You can use a public subnet, but it is not recommended for security reasons.
> NOTE: Nao é recomendado mas voce pode usar uma subnet publica para ser RECOVERY SUBNET, mas seria necessario adicionar uma rota ao Internet Gateway.


As alteracoes de RTP nao refletem nas backupconfigs : 

```
[root@host23ai ~]#  dbcli  describe-backupconfig -i  d96a2d7c-2647-458c-81cc-4fa96669a410
Backup Config details
----------------------------------------------------------------
                     ID: d96a2d7c-2647-458c-81cc-4fa96669a410
                   Name: ac5b32f6370745ccbb7147dd585608
      CrosscheckEnabled: true
         RecoveryWindow:
      BackupDestination: DBRS
         BackupLocation: buMrc3idCBe6ZxZl3Gci
          ObjectStoreId:
            CreatedTime: September 6, 2024 at 4:20:40 PM BRT
            UpdatedTime: December 20, 2024 at 11:46:00 AM BRT
       LocationAffinity:
            RecoveryTag:

```

 Qualquer alteração como essa, é criado um job novo e alimentado o log : 

```
[root@host23ai jobs]# pwd
/opt/oracle/dcs/log/jobs

[root@host23ai jobs]# ls -lrt
-rw-r--r-- 1 root root  102075 Dec 20 09:28 a15e3146-6fb0-4492-b707-0357de31cdfb.log
-rw-r--r-- 1 root root  138025 Dec 20 09:40 4cefabbd-8619-431f-8b35-e16734bb65c9.log
-rw-r--r-- 1 root root  589195 Dec 20 10:00 2cffdacf-f110-477d-aa99-a0cb6c1f1e9c.log
-rw-r--r-- 1 root root  589832 Dec 20 11:00 9eb67c00-9b1c-4255-97bd-091974368162.log
-rw-r--r-- 1 root root   45265 Dec 20 11:45 2c89b0cc-e10c-4c39-9b54-fce4dcf18804.log
-rw-r--r-- 1 root root   45265 Dec 20 11:46 e2e30ed5-8539-4a13-b8fe-3c7a6cbdd993.log
-rw-r--r-- 1 root root  221375 Dec 20 11:49 72f0d64a-e79b-4583-bba3-cc98b86e14e8.log
```

A configuracao default o rman crosscheck vem habilitado, porem é possivel desabilitar utilizando o dbcli : 

```
[root@host23ai ~]#  dbcli  update-backupconfig -i  d96a2d7c-2647-458c-81cc-4fa96669a410 --no-crosscheck
{
  "jobId" : "e2e30ed5-8539-4a13-b8fe-3c7a6cbdd993",
  "status" : "Created",
  "message" : "backup config",
  "errorCode" : "",
  "reports" : [ ],
  "createTimestamp" : "December 20, 2024 11:46:00 AM BRT",
  "resourceList" : [ {
    "resourceId" : "d96a2d7c-2647-458c-81cc-4fa96669a410",
    "resourceType" : "BackupConfig",
    "jobId" : "e2e30ed5-8539-4a13-b8fe-3c7a6cbdd993",
    "updatedTime" : "December 20, 2024 11:46:00 AM BRT"
  } ],
  "description" : "update backup config: ac5b32f6370745ccbb7147dd585608",
  "updatedTime" : "December 20, 2024 11:46:00 AM BRT",
  "percentageProgress" : "0%",
  "cause" : null,
  "action" : null
}
[root@host23ai ~]# dbcli list-jobs
```

**### Alterando o schedule dos backups de archives
****

```
[root@dbnte ~]# dbcli list-backupconfigs

ID                                       Name                 RecoveryWindow   CrosscheckEnabled   BackupDestination
---------------------------------------- -------------------- ---------------- ------------------- --------------------
dcs-spfile-backup                        dcs-spfile-backup                     true                Disk
abbce8ce-739f-425e-a57e-d73aff2e6dea     1f3812928e164818bb97b41c0399ba                  true                DBRS

```

Descreva o backup config


```
[root@dbnte ~]#  dbcli describe-backupconfig -i abbce8ce-739f-425e-a57e-d73aff2e6dea
Backup Config details
----------------------------------------------------------------
                     ID: abbce8ce-739f-425e-a57e-d73aff2e6dea
                   Name: 1f3812928e164818bb97b41c0399ba
      CrosscheckEnabled: true
         RecoveryWindow:
      BackupDestination: DBRS
         BackupLocation: bZTMtxJLVBbS7GcyshXA
          ObjectStoreId:
            CreatedTime: January 31, 2025 at 11:05:46 AM BRT
            UpdatedTime: January 31, 2025 at 11:05:46 AM BRT
       LocationAffinity:
            RecoveryTag:

```

**### Liste os schedules**

```

[root@dbnte ~]#  dbcli list-schedules

ID                                       Name                      Description                                        CronExpression                 Disabled
---------------------------------------- ------------------------- -------------------------------------------------- ------------------------------ --------
0139a309-2658-46e3-acef-fc3aaf61585e     backupreport maintenance  backup reports deletion                            0 0 0 1/3 * ? *                false
2184a808-00a5-4314-94f1-40b7ebafbaf3     Update PDB status         Update PDB status to metastore                     0 */30 * * * ? *               false
255bb294-4159-46c2-b52e-1549d329bf2a     AgentState metastore cleanup internal agentstateentry metastore maintenance     0 0 0 1/1 * ? *                false
27d1086b-274b-41f1-a06d-e55b6d1fac52     Big File Upload Cleanup   clean up expired big file uploads.                 0 0 1 ? * SUN *                false
28ac5ab8-d48e-4895-abaa-503626449053     dcs_agent_mysql_metadata_backup DCS Agent mysql metadata backup                    0 0 0 1/1 * ? *                false
97a66e64-5752-4b49-b408-f1c4250e7c6b     audit_files_auto_cleanup_DB0131_z2z_gru audit_files_auto_cleanup : DB0131                  0 0 0 1/1 * ? *                false
9afeaa05-761a-4e8e-bfd5-c63585b383d8     bom maintenance           bom reports generation                             0 0 1 ? * SUN *                false
c7be26db-45d8-4c09-b0b1-cafa109eef56     metastore maintenance     internal metastore maintenance                     0 0 0 1/15 * ? *               false
d6e5df24-102d-4252-8d78-bb0450322afd     log_files_auto_cleanup    log_files_auto_cleanup                             0 0 0 1/1 * ? *                false
df88e974-2607-46c3-8a87-b817b0673674     archive_log_backup_e18d66b5-76d1-46bc-b834-687c98071000 backup archive logs : DB0131                       0 0 0/2 1/1 * ? *              false
e34bc565-3a7a-4720-87ce-504577e4358e     auto_database_backup_e18d66b5-76d1-46bc-b834-687c98071000_SPFILE auto backup database: DB0131                       0 0 0 * * ?                    false

```


Ajuste o schedule do backup de archive para o intervalo que desejar : 

` dbcli update-schedule --cronExpression "0 0 0/2 1/1 * ? *" --scheduleid df88e974-2607-46c3-8a87-b817b0673674`




```
[root@dbnte ~]# dbcli list-jobs

ID                                       Description                                                                 Created                             Status
---------------------------------------- --------------------------------------------------------------------------- ----------------------------------- ----------
a9f34942-e3b6-4a8d-b62e-f5bef33d83cf     Provisioning service creation                                               Friday, January 31, 2025, 10:07:19 BRT Success
96c7dc61-d510-405a-884f-919c988136a0     SSH keys update                                                             Friday, January 31, 2025, 10:44:02 BRT Success
49b77a9c-e2fa-4c57-8e7a-579f60b63ec7     SSH key delete                                                              Friday, January 31, 2025, 10:45:42 BRT Success
1b163767-1a1e-45bb-85b1-9d6f63268e21     Resource Principle certification generation/rotation                        Friday, January 31, 2025, 10:45:47 BRT Success
1c035c27-c671-412a-99cf-dd82c0207833     Resource Principle certification generation/rotation                        Friday, January 31, 2025, 10:46:04 BRT Success
4ff19e3a-39e5-403c-9012-1bb313a88645     Register Resources OCID Request                                             Friday, January 31, 2025, 10:47:27 BRT Success
a5ae0456-481c-45d2-a410-c44f23381bd0     Authentication key update for DCS_ADMIN                                     Friday, January 31, 2025, 10:48:07 BRT Success
18d622ef-c4a6-421c-a692-38c96d14c010     Infra upgrade                                                               Friday, January 31, 2025, 10:48:11 BRT Success
dc7164b8-cfa3-435e-b5de-9e8274c9e194     Update DataPlane Diagnostic Settings                                        Friday, January 31, 2025, 10:51:06 BRT Success
dcbd90bc-4b39-4b2e-bc50-5b8ac428a76b     create object store:bZTMtxJLVBbS7GcyshXA                                    Friday, January 31, 2025, 11:04:19 BRT Success
a42bcbc6-bb75-4aca-becc-3cf48eb84bcd     Create DBRS: qs6yngus34vdtvnx7fanc_DBRS_CF                                  Friday, January 31, 2025, 11:05:29 BRT Success
e562c5d4-784f-42f2-9da8-7cd818def4ab     create backup config:1f3812928e164818bb97b41c0399ba                         Friday, January 31, 2025, 11:05:46 BRT Success
fbc94f2a-af36-417a-9d75-f14f84bfec4a     update database : DB0131                                                    Friday, January 31, 2025, 11:06:01 BRT Success
a35f15bc-8c90-483c-ab0b-da64cdedaef8     scheduleBackup_e18d66b5-76d1-46bc-b834-687c98071000_archivelog_(frequency:60)_Fri Jan 31 11:14:35 BRT 2025 Friday, January 31, 2025, 11:14:35 BRT Success
62923d7b-2aac-433f-a654-78770265384b     Create Regular-L0 Backup with TAG-DBTRegular-L01738331607381aqd for Db:DB0131 in recovery service Friday, January 31, 2025, 11:15:01 BRT Success
358e6b25-6485-408b-9cc4-d19b255a57bd     Create ARCHIVELOG Backup with TAG-Auto_Archive for Db:DB0131 in recovery service Friday, January 31, 2025, 12:00:00 BRT Success
5c9ee492-649b-4cb4-a56d-a9c93163f967     updateDBRS :qs6yngus34vdtvnx7fanc_DBRS_CF                                   Sunday, February 02, 2025, 10:59:46 BRT Success
f401a2bd-5714-4f42-81b9-7e782c73c1a7     Create ARCHIVELOG Backup with TAG-Auto_Archive for Db:DB0131 in recovery service Sunday, February 02, 2025, 11:00:00 BRT Success
5d51ecc4-abd1-4955-b9e5-f02112fd1b49     Create ARCHIVELOG Backup with TAG-Auto_Archive for Db:DB0131 in recovery service Sunday, February 02, 2025, 11:15:00 BRT Success
6943a099-22a0-48fc-b8a7-8c1077e24743     Create ARCHIVELOG Backup with TAG-Auto_Archive for Db:DB0131 in recovery service Sunday, February 02, 2025, 11:30:00 BRT Success
0476f580-d3a1-4f25-9d9f-56ff26d62c8c     Create ARCHIVELOG Backup with TAG-Auto_Archive for Db:DB0131 in recovery service Sunday, February 02, 2025, 11:45:00 BRT Success
a64c97fc-6946-427c-99a4-3842bbf8462c     Create ARCHIVELOG Backup with TAG-Auto_Archive for Db:DB0131 in recovery service Sunday, February 02, 2025, 12:00:00 BRT Success
4b4a6528-09fe-4b52-a1d2-434594878837     Create ARCHIVELOG Backup with TAG-Auto_Archive for Db:DB0131 in recovery service Sunday, February 02, 2025, 14:00:00 BRT Success
2ad512d5-a585-4041-8ace-d9817cc371d6     Create ARCHIVELOG Backup with TAG-Auto_Archive for Db:DB0131 in recovery service Sunday, February 02, 2025, 16:00:00 BRT Success
9841d187-be29-424a-a9bc-357ec5e8ebd7     Prepare encryption report for database - DB0131_z2z_gru                     Sunday, February 02, 2025, 17:06:29 BRT Success
74d69c75-8956-4f81-bf7b-aedd14db9f5c     Create ARCHIVELOG Backup with TAG-Auto_Archive for Db:DB0131 in recovery service Sunday, February 02, 2025, 18:00:00 BRT Success
5d9ed870-1902-4b25-81fc-72c9155794a8     Create ARCHIVELOG Backup with TAG-Auto_Archive for Db:DB0131 in recovery service Sunday, February 02, 2025, 20:00:00 BRT Success

```

NETWORK....

Apos a configuração de uma base, é feita automaticamente a rota da saida de backup para a Bondeth1


Antes a saida de backup é feita pelo Bondeth0 e é necessario liberar a client subnet de origem : 

```
[oracle@apollo1-oaric TFDB2]$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    100    0        0 bondeth0
100.105.0.0     0.0.0.0         255.255.0.0     U     103    0        0 stre0
100.105.0.0     0.0.0.0         255.255.0.0     U     104    0        0 stre1
100.107.0.0     0.0.0.0         255.255.0.0     U     101    0        0 clre0
100.107.0.0     0.0.0.0         255.255.0.0     U     102    0        0 clre1
169.254.200.0   0.0.0.0         255.255.255.252 U     106    0        0 eth0
172.16.0.0      0.0.0.0         255.255.255.224 U     100    0        0 bondeth0
172.16.0.32     0.0.0.0         255.255.255.224 U     105    0        0 bondeth1
```

DEPOIS Verificamos que a propria orquestracao faz a mudanca : 

```
[oracle@apollo1-oaric TFDB2]$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    100    0        0 bondeth0
100.105.0.0     0.0.0.0         255.255.0.0     U     103    0        0 stre0
100.105.0.0     0.0.0.0         255.255.0.0     U     104    0        0 stre1
100.107.0.0     0.0.0.0         255.255.0.0     U     101    0        0 clre0
100.107.0.0     0.0.0.0         255.255.0.0     U     102    0        0 clre1
169.254.200.0   0.0.0.0         255.255.255.252 U     106    0        0 eth0
172.16.0.0      0.0.0.0         255.255.255.224 U     100    0        0 bondeth0
172.16.0.32     0.0.0.0         255.255.255.224 U     105    0        0 bondeth1
172.16.0.128    _gateway        255.255.255.240 UG    0      0        0 bondeth1

```


**### Principais diferencas entre RCV ( Autonomous Recovery Service) e ZRCV ( Zero Data Loss Recovery Service)**
**Testes com DBRS**


> - Os backups de archives sao gerados a cada 60 minutos


[root@dbnorigem ~]# dbcli list-pendingjobs

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/265e69cc-9118-4384-b97c-4a430b3d51d4)


-  Foi feito um teste a partir do drop do banco e apos o restore/recovery : 

[oracle@dbnorigem DB0710]$ rman target / catalog /@dbrs
![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/90e82a66-d1c1-4f6c-b06f-e81f620a6f54)


> Restart na instance para ler o SPFILE restaurado : 

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/bf4fe1d7-6592-44e2-b33c-60d7c8f450f6)

> Restore do CONTROLFILE : 

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/d84eaeee-5178-40c3-b3b8-cde596586c68)

> RESTORE DATABASE : 

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/a6516803-44bb-4fb0-8389-63f4863bc29e)






 >- As backup pieces aparecem no rman list como media -->   Recovery Appliance (RAGRUP3) e nao como SBT_TAPE.

  Exemplo :

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/5929254e-458b-4259-9082-884fac7cadd8)


> Para recuperar o backup da wallet é necessario a utilizacao da senha da TDE :

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/d99f1ad4-527c-4a3e-8196-554eb1fc8954)



Tentei criar um outro banco a partir do backup ja realizado, porem estava em execucao um novo backup e por isso a console nao deixou eu prosseguir ( enquanto existia um backup em andamento )

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/1be62cbf-525d-46c1-b280-e99470995512)


> O primeiro backup leva mais tempo, porem o segundo "FULL" é bem rapido




[oracle@dbnorigin ~]$ ps -ef | grep rman
oracle   28993  4972  0 14:48 pts/0    00:00:08 rman
root     37234  4372  0 18:29 ?        00:00:00 su oracle -c /u01/app/oracle/product/19.0.0/dbhome_1/bin/rman target '"C##DBLCMUSER@DB0711_6rv_gru as sysbackup"' catalog /@DBRS log /opt/oracle/dcs/log/dbnorigin/rman/bkup/DB0711_6rv_gru/rman_configure_2024-07-22_18-29-56-9187285861858399689.log @/tmp/dcsserver/rman/rman2024-07-22_18-29-56-5354549677753518788.rman




connected to target database: DB0711 (DBID=1468339561)
connected to recovery catalog database
recovery catalog schema version 23.04.00.23. is newer than RMAN version

RMAN> CONFIGURE ARCHIVELOG DELETION POLICY TO NONE;
2>
old RMAN configuration parameters:
CONFIGURE ARCHIVELOG DELETION POLICY TO NONE;
new RMAN configuration parameters:
CONFIGURE ARCHIVELOG DELETION POLICY TO NONE;
new RMAN configuration parameters are successfully stored

Recovery Manager complete.
~

CONFIGURE ARCHIVELOG DELETION POLICY TO NONE;


quando convert to OSS :


[oracle@dbnorigin ~]$ cat /tmp/dcsserver/rman/rman2024-07-22_18-34-38-8079873537132614215.rman
configure controlfile autobackup on;
configure compression algorithm 'LOW';
configure encryption for database off;
configure backup optimization off;
CONFIGURE ENCRYPTION ALGORITHM 'AES256';



set command id to "68609b75-6f49-4b97-b0a7-1759409b";
report schema;
show all;
list incarnation of database;
set echo on;
set encryption on;
backup device type sbt as compressed backupset current controlfile tag 'auto' format 'auto_cf_%d_%I_%U_%T_%t_set%s'  ;
set encryption off;


set command id to "ecdc161c-9e22-4384-ac1a-f4fece14";
run{
ALLOCATE CHANNEL C0 DEVICE TYPE 'SBT_TAPE' PARMS 'SBT_LIBRARY=/opt/oracle/dcs/commonstore/oss/DB0711_6rv_gru/libopc.so, ENV=(OPC_PFILE=/opt/oracle/dcs/commonstore/oss/DB0711_6rv_gru/90dbc7e5-ef10-4305-adb7-0b02d29410c1/opc_DB0711_6rv_gru.ora)' FORMAT '%d_%I_%U_%T_%t';
ALLOCATE CHANNEL C1 DEVICE TYPE 'SBT_TAPE' PARMS 'SBT_LIBRARY=/opt/oracle/dcs/commonstore/oss/DB0711_6rv_gru/libopc.so, ENV=(OPC_PFILE=/opt/oracle/dcs/commonstore/oss/DB0711_6rv_gru/90dbc7e5-ef10-4305-adb7-0b02d29410c1/opc_DB0711_6rv_gru.ora)' FORMAT '%d_%I_%U_%T_%t';

backup as compressed backupset incremental level 0 force  database tag 'auto' format 'auto_df_%d_%I_%U_%T_%t_set%s';
}


configure device type 'SBT_TAPE' backup type to compressed backupset;
configure channel device type 'SBT_TAPE' maxpiecesize 2G format '%d_%I_%U_%T_%t' parms 'SBT_LIBRARY=/opt/oracle/dcs/commonstore/oss/DB0711_6rv_gru/libopc.so ENV=(OPC_PFILE=/opt/oracle/dcs/commonstore/oss/DB0711_6rv_gru/90dbc7e5-ef10-4305-adb7-0b02d29410c1/opc_DB0711_6rv_gru.ora)';
CONFIGURE ENCRYPTION FOR DATABASE ON;
CONFIGURE ARCHIVELOG DELETION POLICY TO NONE;




set command id to "01e3844e-0bae-4e86-ab8b-f080dc6b";
run{
ALLOCATE CHANNEL D0 DEVICE TYPE DISK;
DELETE NOPROMPT ARCHIVELOG ALL BACKED UP 2 TIMES TO SBT_TAPE;
}


[oracle@dbnorigin rman]$ cat rman2024-07-22_18-59-07-7016436141821091389.rman
CONFIGURE ARCHIVELOG DELETION POLICY TO BACKED UP 1 TIMES TO 'SBT_TAPE';







DBPWCSLA

![image](https://github.com/user-attachments/assets/8b215255-c4aa-424a-8a61-e85fb51e891a)

For any question, please contact the product management team:
 
Jun Jang – Outbound Product Management, Zero Data Loss Autonomous Recovery Service
Mark Comishock – Outbound Product Management, Government Markets
Kelly Smith – Technical Product Management, Zero Data Loss Autonomous Recovery Service
Alex Goldblatt – Technical Product Management, Zero Data Loss Autonomous Recovery Service

Seguem os Porques de se utilizar o DBRS : 




The Recovery Service offers the following major benefits:

•	Ransomware Resiliency
•	Operational Efficiency
•	Cloud Simplicity with Enhanced Observability and Cost Management

Ransomware Resiliency: Provides peace of mind to customer that the backups are resilient from ransomware attacks.
•	The database is fully encrypted from start to finish. And the keys are separate from the actual backup. Even if a ransomware intruder gets hold of the backup, he/she cannot do anything due to this encryption.
•	The backups are immutable. No one can make any changes to the backup. RCV/ZRCV enforces a 14 days of minimum retention period
•	RCV/ZRCV has the option for Zero Data Loss feature which provides sub-second RPO. This significantly minimizes the data loss in the event of a ransomware attack or a disaster.

Operational Efficiency: Deliver faster backup and restore at lower cost.
•	Faster backups. Autonomous Recovery Service eliminates the need to do full backups on weekly basis. Only small changes are backed up which significantly reduces the time required to do backups.
•	Faster restore. The Autonomous Recovery Service builds a virtual full backup with every incremental backup. Traditional backup methodology requires a full backup and some number of incremental to rebuild the database. The Autonomous Recovery Service eliminates this build process which provides faster and consistent restore performance.
•	Less overhead – All the backup validation is offloaded to Autonomous Recovery Service. Usually, database servers perform resource intensive backup validation which impacts the performance of the database and the application. With the elimination of full backup and offloading of validation to Autonomous Recovery Service, significantly reduces the overhead put on the database server which delivers increased application and database performance.

Cloud Simplicity with Enhanced Observability and Cost Management: On a per-database basis, quickly configure/manage/monitor key backup metrics.  
•	Simple and robust set of monitoring capabilities built into the OCI console.
•	Ability to monitor the health, usage, data loss exposure and many other metrics for each database backup.
•	No large upfront investment; capacity-based charges on Universal Credits, Microsoft Azure Consumption Commitment and Google Cloud Platform Credit.

The RCV/ZRCV delivers the best-in-class protection for Oracle databases. There is no other backup solution in the market today that has the capabilities offered by Recovery Service. 
 
It is strongly recommended that all Exadata Database Service-Dedicated, Exadata Database Service on Exascale Infrastructure and Base Database Service customers use the highly differentiated RCV/ZRCV as their backup target.



1. >  Ao preparar o ambiente para utilizacao do DBRS é necessario associar um SUBNET para ser a utilizada pelo servico : 
        Com isso ela consumira 6 IPs da subnet. No caso abaixo foi escolhida a propria subnet de backup do exa-xs : 

![image](https://github.com/user-attachments/assets/4b8d60fc-0480-40c7-832a-558892b17a18)

> Melhor zoom : 

![image](https://github.com/user-attachments/assets/e73fbac0-3dc1-4b27-a14b-f99f16155bc6)


![image](https://github.com/user-attachments/assets/57ee8616-e8c3-49d8-98b6-f0dfa9d581a1)


 Somente o jar do libopc.so vem no diretorio do commonstore 

1. >  As libs ja vem instaladas no DBCS :

![image](https://github.com/user-attachments/assets/57ee8616-e8c3-49d8-98b6-f0dfa9d581a1)

 Somente o jar do libopc.so vem no diretorio do commonstore 

3. >  Quando configura o backup automático, a automação configuração cria as pastas dbrs e oss :

![image](https://github.com/user-attachments/assets/be9184ce-8bde-4455-bd45-0f6833bcec7b)

Essa pasta dbrs 


![image](https://github.com/user-attachments/assets/86c3a77e-da28-4429-9ff7-855ec664b589)

4. >  O arquivo dbrsnames.ora, quando habilitamos o DBRS RTRT fica da seguinte forma : 

![image](https://github.com/user-attachments/assets/30521314-77b9-4a0e-9b67-2d502cb6bfd8)


![image](https://github.com/user-attachments/assets/fb5059f7-fc04-4ac8-a866-2f1d56d5bea6)


```
 DBRS_PRIMARY=(DESCRIPTION=(FAILOVER=on)(CONNECT_TIMEOUT=3)(RETRY_COUNT=3)(TRANSPORT_CONNECT_TIMEOUT=3)(ADDRESS_LIST=(LOAD_BALANCE=on)(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup002-2.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484))(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup002-3.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484))(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup002-1.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ZRCV_TJVGVHWVFB0EUMUGX0))(SECURITY=(MY_WALLET_DIRECTORY=/opt/oracle/dcs/commonstore/wallets/DB1029_658_gru/server_seps)))
DBRS_ALTERNATE=(DESCRIPTION=(FAILOVER=on)(CONNECT_TIMEOUT=3)(RETRY_COUNT=3)(TRANSPORT_CONNECT_TIMEOUT=3)(ADDRESS_LIST=(LOAD_BALANCE=on)(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup004-1.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484))(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup004-3.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484))(ADDRESS=(PROTOCOL=TCPS)(HOST=ragrup004-2.rs.br.sa-saopaulo-1.oraclecloud.com)(PORT=2484)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ZRCV_QHOF8YC2VLITADPHMCPANALXA2S9AI))(SECURITY=(MY_WALLET_DIRECTORY=/opt/oracle/dcs/commonstore/wallets/DB1029_658_gru/server_seps)))

```


3. >  Quando comentamos a entrada DBRS no arquivo dbrsnames.ora, o backup nao funciona mais : 

![image](https://github.com/user-attachments/assets/a50a20b3-b67d-44a4-82f0-c4a63b95b955)

![image](https://github.com/user-attachments/assets/81ee77c7-2016-445f-98f4-4872952ab59c)


4. >  A quantidade de OCPUs é a quantidade de PARALLEL que ele vai alocar , dessa forma se o DBCS tem 4 OCPUs ele vai criar um script de rman com 4 canais com sectionsize de 64GBs e FILESPERSET de 1.

![Image](https://github.com/user-attachments/assets/fcbaee75-3221-47ab-ae23-610f356be24b)


https://docs.oracle.com/en-us/iaas/exadatacloud/doc/ecs-managing-db-backup-and-recovery.html#GUID-F3967733-A31F-4CD7-9962-77F34EA54D02

No dbcs nao tem como alterar o bkup_channels_node, porem no EXAXS/EXACS pode-se alterar a configuracao para ter mais canais.

> [root@apollo1-oaric ~]#  dbaascli database backup  --dbname TFDB2 --getConfig  --configFile /tmp/configfile.txt

```
DBAAS CLI version 25.1.1.0.0
Executing command database backup --dbname TFDB2 --getConfig --configFile /tmp/configfile.txt
Session log: /var/opt/oracle/log/TFDB2/database/backup/dbaastools_2025-04-15_05-41-33-PM_89027.log
logfile:/var/opt/oracle/log/dtrs/dcs-dtrs.0.*.log
File /tmp/configfile.txt created


dbaascli execution completed
[root@apollo1-oaric ~]# cat  /tmp/configfile.txt
#########################
#RMAN Configs
#########################
bkup_filesperset_al=1
rmanFraCleanupChannels=1
bkup_section_size=64G
Compress_Archive_Logs=false
bkup_type=dbrs
bkup_channels_node=default
rmanArchiveLogChannels=default
bkup_disk_recovery_window=35
bkup_rman_retention=35
bkup_filesperset_regular=1
rmanBackupOptimization=OFF
bkup_archlog_fra_retention=1
bkup_nfs_recovery_window=35
bkup_oss_recovery_window=35
bkup_encryption=true
rmanEncryptionAlgorithm=AES256


#########################
#Schedule Configs
#########################
bkup_archlog_frequency=30
bkup_cron_entry=no
schedulesStartMode=SCHEDULE_TIME
bkup_l0_day=SUN
bkup_archlog_cron_entry=yes
bkup_daily_time=


#########################
#DBRS Destination Configs
#########################
bkup_dbrs_user=HCVCCORAUVT1E7P7MKAOIPXDCSG
```

Portanto, daria para configurar a compressão dos archivelogs, que por default não sao assim como o parallel ( channels) e algoritmo de criptografia.


[root@apollo1-oaric ~]#  dbaascli database backup --dbname TFDB2 --configure  --configFile /tmp/configfile.txt
```
DBAAS CLI version 25.1.1.0.0
Executing command database backup --dbname TFDB2 --configure --configFile /tmp/configfile.txt
Session log: /var/opt/oracle/log/TFDB2/database/backup/dbaastools_2025-04-15_05-47-54-PM_123543.log
logfile:/var/opt/oracle/log/dtrs/dcs-dtrs.0.*.log
UUID b0f6030b-20b0-4fc9-8b2a-0eeb985048fd for this backup

```


6. >  Quando comentamos as entradas DBRS_PRIMARY e DBRS_ALTERNATE, o Data Exposure Loss aumenta, porem no grafico nao mostra automaticamente, so depois de 30 minutos....

Descomentei as entradas, para funcionar novamente mas no dashboard continua com Data Loss Exposure 

![image](https://github.com/user-attachments/assets/1ffde591-e3f3-4746-8bdf-90936b7c2270)

Recomendo visualizar e colocar eventos nas metricas diretas de "DATA LOSS EXPOSURE", com intervalos de 1 minuto. Assim vai monitorar melhor que via dashboard do servico.

![image](https://github.com/user-attachments/assets/9c2e2d08-cff4-42a0-bec9-7dcd8689197d)

![image](https://github.com/user-attachments/assets/25759e4b-e492-4b43-a77a-6c03dbd2ead4)






1.  Monitorando o backup.
     Ele cria um processo de rman paralelizando de acordo com a quantidade de OCPU que o ambiente, possui. Nesse caso abaixo o ambiente esta com 04 OCPUs, portanto ele abriu 4 canais :

```
[oracle@dbn-orcl-hml-01 ~]$ ps -ef | grep rman
root     74206 43315  0 23:34 ?        00:00:00 su oracle -c /u01/app/oracle/product/19.0.0.0/dbhome_2/bin/rman target '"C##DBLCMUSER@DBHML19_6wc_gru as sysbackup"' catalog /@DBRS log /opt/oracle/dcs/log/dbn-orcl-hml-01/rman/bkup/DBHML19_6wc_gru/rman_backup_auto_2024-11-21_23-34-25-7412700671059484367.log @/tmp/dcsserver/rman/rman2024-11-21_23-34-25-5352681594182158448.rman
oracle   74207 74206  0 23:34 ?        00:00:05 rman app/oracle/product/19.0.0.0/dbhome_2/bin/rman
oracle   92885 63610  0 23:50 pts/0    00:00:00 grep --color=auto rman
[oracle@dbn-orcl-hml-01 ~]$ cat /tmp/dcsserver/rman/rman2024-11-21_23-34-25-5352681594182158448.rman
set command id to "bb1efb7b-ae31-4bab-ab42-b6580dd8";
run{
ALLOCATE CHANNEL C0 DEVICE TYPE 'SBT_TAPE' PARMS 'SBT_LIBRARY=/opt/oracle/dcs/commonstore/dbrs/DBHML19_6wc_gru/libra.so, ENV=(RA_WALLET=location=file:/opt/oracle/dcs/commonstore/wallets/DBHML19_6wc_gru/server_seps credential_alias=DBRS, RA_FORMAT=TRUE)' FORMAT '%U_%d';
ALLOCATE CHANNEL C1 DEVICE TYPE 'SBT_TAPE' PARMS 'SBT_LIBRARY=/opt/oracle/dcs/commonstore/dbrs/DBHML19_6wc_gru/libra.so, ENV=(RA_WALLET=location=file:/opt/oracle/dcs/commonstore/wallets/DBHML19_6wc_gru/server_seps credential_alias=DBRS, RA_FORMAT=TRUE)' FORMAT '%U_%d';
ALLOCATE CHANNEL C2 DEVICE TYPE 'SBT_TAPE' PARMS 'SBT_LIBRARY=/opt/oracle/dcs/commonstore/dbrs/DBHML19_6wc_gru/libra.so, ENV=(RA_WALLET=location=file:/opt/oracle/dcs/commonstore/wallets/DBHML19_6wc_gru/server_seps credential_alias=DBRS, RA_FORMAT=TRUE)' FORMAT '%U_%d';
ALLOCATE CHANNEL C3 DEVICE TYPE 'SBT_TAPE' PARMS 'SBT_LIBRARY=/opt/oracle/dcs/commonstore/dbrs/DBHML19_6wc_gru/libra.so, ENV=(RA_WALLET=location=file:/opt/oracle/dcs/commonstore/wallets/DBHML19_6wc_gru/server_seps credential_alias=DBRS, RA_FORMAT=TRUE)' FORMAT '%U_%d';

BACKUP AS BACKUPSET CUMULATIVE INCREMENTAL LEVEL 1 TAG 'auto' FORCE FILESPERSET 1 SECTION SIZE 64G DATABASE;
}
[oracle@dbn-orcl-hml-01 ~]$ sqlplus / as sysdba

SQL*Plus: Release 19.0.0.0.0 - Production on Thu Nov 21 23:50:21 2024
Version 19.24.0.0.0

Copyright (c) 1982, 2024, Oracle.  All rights reserved.


Connected to:
Oracle Database 19c EE High Perf Release 19.0.0.0.0 - Production
Version 19.24.0.0.0

SQL> show parameter cpu

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
cpu_count                            integer     8
cpu_min_count                        string      8
parallel_threads_per_cpu             integer     2
resource_manager_cpu_allocation      integer     0
SQL>

```

2.  Listando os backups :
```
[oracle@exa1brscan-8ilfa1 TEMP_TNS]$ export TNS_ADMIN=/var/opt/oracle/dbaas_acfs/BRSPROD/TEMP_TNS
[oracle@exa1brscan-8ilfa1 TEMP_TNS]$ rman target  / CATALOG "/@DBRS"

```

![image](https://github.com/user-attachments/assets/5c12a77c-5b0e-4714-a5e6-0a2d644f6283)



2.  A console deixa rodar um backup com retention policy e outro com LTR.
<img width="1680" height="452" alt="Image" src="https://github.com/user-attachments/assets/3a385aa6-7489-4341-bd1f-cdd88dc8bd82" />

2.  Com a mudanca de OSS para ZDLRS, as policies de rman sao automaticamente ajustadas para NONE
```

oracle@dbn-orcl-hml-01 ~]$ rman target /

Recovery Manager: Release 19.0.0.0.0 - Production on Thu Nov 21 23:29:19 2024
Version 19.24.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

connected to target database: DBSRVHML (DBID=225123752)

RMAN> show all ;

using target database control file instead of recovery catalog
RMAN configuration parameters for database with db_unique_name DBHML19_6WC_GRU are:
CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 15 DAYS;

```

PARA 
```

[oracle@dbn-orcl-hml-01 ~]$ rman target /

Recovery Manager: Release 19.0.0.0.0 - Production on Thu Nov 21 23:54:20 2024
Version 19.24.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

connected to target database: DBSRVHML (DBID=225123752)

RMAN> show all ;

using target database control file instead of recovery catalog
RMAN configuration parameters for database with db_unique_name DBHML19_6WC_GRU are:
CONFIGURE RETENTION POLICY TO NONE;
CONFIGURE BACKUP OPTIMIZATION OFF;
CONFIGURE DEFAULT DEVICE TYPE TO 'SBT_TAPE';
CONFIGURE CONTROLFILE AUTOBACKUP ON;
CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '%F'; # default
CONFIGURE CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE SBT_TAPE TO '%F'; # default

```

3.  Eu reparei que se nao limparmos o historico de backup que esta no outro device (OSS) apos mudarmos para RA, esses nao sao apagados ou submetidos a politica de delecao nem de backup, ou seja, o canal configurado do rman nao mais enxerga o OSS e por isso nao consegue DELETAR a imagem.

   Por isso antes de mudar de OSS para RA, copie as strings de configuracao de CHANNEL e reveja a backupconfig de OSS para deletar as pecas mais antigas e/ou fazer depois com a TAG especifica, fazendo esse DELETE MANUAL

```

run{
ALLOCATE CHANNEL C0 DEVICE TYPE 'SBT_TAPE' PARMS 'SBT_LIBRARY=/opt/oracle/dcs/commonstore/oss/DBHML19_6wc_gru/libopc.so, ENV=(OPC_PFILE=/opt/oracle/dcs/commonstore/oss/DBHML19_6wc_gru/6e4f607d-4beb-422e-9a03-034308e34181/opc_DBHML19_6wc_gru.ora)';
DELETE NOPROMPT BACKUP TAG "DBTREGULAR-L117313601617710RL" COMPLETED before 'sysdate-2';
}

```


Quando estamos configurando o ARS para uma base ORACLE :

> dbaascli database backup --showHistory --dbname BRSPROD

> [root@exa1brscan-8ilfa1 jobs]# dbaascli database backup --dbname BRSPROD --status --uuid dd958817-61c6-4314-a918-18d35b9f42ca


![image](https://github.com/user-attachments/assets/08963fee-47ce-4441-9620-66dda8b4931e)




# autonomous_recovery

**Testes com DBRS**

> -  Criando um banco DBCS na regiao de GRU com DBRS e a partir de um backup on-demand, criar um novo dbcs com shape distinto mantendo a mesma Edition(obrigatorio) na regiao de VCP.

> Na regiao de VCP o banco estaria numa subnet publica e por isso foi necessario adicionar uma rota ao Internet Gateway, mesmo a subnet do Recovery Service ser Privada em GRU.

> - Os backups de archives sao gerados a cada 60 minutos


[root@dbnorigem ~]# dbcli list-pendingjobs

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/265e69cc-9118-4384-b97c-4a430b3d51d4)


-  Foi feito um teste a partir do drop do banco e apos o restore/recovery : 

[oracle@dbnorigem DB0710]$ rman target / catalog /@dbrs
![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/90e82a66-d1c1-4f6c-b06f-e81f620a6f54)


> Restart na instance para ler o SPFILE restaurado : 

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/bf4fe1d7-6592-44e2-b33c-60d7c8f450f6)

> Restore do CONTROLFILE : 

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/d84eaeee-5178-40c3-b3b8-cde596586c68)

> RESTORE DATABASE : 

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/a6516803-44bb-4fb0-8389-63f4863bc29e)






 >- As backup pieces aparecem no rman list como media -->   Recovery Appliance (RAGRUP3) e nao como SBT_TAPE.

  Exemplo :

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/5929254e-458b-4259-9082-884fac7cadd8)


> Para recuperar o backup da wallet é necessario a utilizacao da senha da TDE :

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/d99f1ad4-527c-4a3e-8196-554eb1fc8954)



Tentei criar um outro banco a partir do backup ja realizado, porem estava em execucao um novo backup e por isso a console nao deixou eu prosseguir ( enquanto existia um backup em andamento )

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/1be62cbf-525d-46c1-b280-e99470995512)


> O primeiro backup leva mais tempo, porem o segundo "FULL" é bem rapido




[oracle@dbnorigin ~]$ ps -ef | grep rman
oracle   28993  4972  0 14:48 pts/0    00:00:08 rman
root     37234  4372  0 18:29 ?        00:00:00 su oracle -c /u01/app/oracle/product/19.0.0/dbhome_1/bin/rman target '"C##DBLCMUSER@DB0711_6rv_gru as sysbackup"' catalog /@DBRS log /opt/oracle/dcs/log/dbnorigin/rman/bkup/DB0711_6rv_gru/rman_configure_2024-07-22_18-29-56-9187285861858399689.log @/tmp/dcsserver/rman/rman2024-07-22_18-29-56-5354549677753518788.rman




connected to target database: DB0711 (DBID=1468339561)
connected to recovery catalog database
recovery catalog schema version 23.04.00.23. is newer than RMAN version

RMAN> CONFIGURE ARCHIVELOG DELETION POLICY TO NONE;
2>
old RMAN configuration parameters:
CONFIGURE ARCHIVELOG DELETION POLICY TO NONE;
new RMAN configuration parameters:
CONFIGURE ARCHIVELOG DELETION POLICY TO NONE;
new RMAN configuration parameters are successfully stored

Recovery Manager complete.
~

CONFIGURE ARCHIVELOG DELETION POLICY TO NONE;


quando convert to OSS :


[oracle@dbnorigin ~]$ cat /tmp/dcsserver/rman/rman2024-07-22_18-34-38-8079873537132614215.rman
configure controlfile autobackup on;
configure compression algorithm 'LOW';
configure encryption for database off;
configure backup optimization off;
CONFIGURE ENCRYPTION ALGORITHM 'AES256';



set command id to "68609b75-6f49-4b97-b0a7-1759409b";
report schema;
show all;
list incarnation of database;
set echo on;
set encryption on;
backup device type sbt as compressed backupset current controlfile tag 'auto' format 'auto_cf_%d_%I_%U_%T_%t_set%s'  ;
set encryption off;


set command id to "ecdc161c-9e22-4384-ac1a-f4fece14";
run{
ALLOCATE CHANNEL C0 DEVICE TYPE 'SBT_TAPE' PARMS 'SBT_LIBRARY=/opt/oracle/dcs/commonstore/oss/DB0711_6rv_gru/libopc.so, ENV=(OPC_PFILE=/opt/oracle/dcs/commonstore/oss/DB0711_6rv_gru/90dbc7e5-ef10-4305-adb7-0b02d29410c1/opc_DB0711_6rv_gru.ora)' FORMAT '%d_%I_%U_%T_%t';
ALLOCATE CHANNEL C1 DEVICE TYPE 'SBT_TAPE' PARMS 'SBT_LIBRARY=/opt/oracle/dcs/commonstore/oss/DB0711_6rv_gru/libopc.so, ENV=(OPC_PFILE=/opt/oracle/dcs/commonstore/oss/DB0711_6rv_gru/90dbc7e5-ef10-4305-adb7-0b02d29410c1/opc_DB0711_6rv_gru.ora)' FORMAT '%d_%I_%U_%T_%t';

backup as compressed backupset incremental level 0 force  database tag 'auto' format 'auto_df_%d_%I_%U_%T_%t_set%s';
}


configure device type 'SBT_TAPE' backup type to compressed backupset;
configure channel device type 'SBT_TAPE' maxpiecesize 2G format '%d_%I_%U_%T_%t' parms 'SBT_LIBRARY=/opt/oracle/dcs/commonstore/oss/DB0711_6rv_gru/libopc.so ENV=(OPC_PFILE=/opt/oracle/dcs/commonstore/oss/DB0711_6rv_gru/90dbc7e5-ef10-4305-adb7-0b02d29410c1/opc_DB0711_6rv_gru.ora)';
CONFIGURE ENCRYPTION FOR DATABASE ON;
CONFIGURE ARCHIVELOG DELETION POLICY TO NONE;




set command id to "01e3844e-0bae-4e86-ab8b-f080dc6b";
run{
ALLOCATE CHANNEL D0 DEVICE TYPE DISK;
DELETE NOPROMPT ARCHIVELOG ALL BACKED UP 2 TIMES TO SBT_TAPE;
}


[oracle@dbnorigin rman]$ cat rman2024-07-22_18-59-07-7016436141821091389.rman
CONFIGURE ARCHIVELOG DELETION POLICY TO BACKED UP 1 TIMES TO 'SBT_TAPE';




  
**Documentacao**
1. > Para O LTR é necessario configurar os tres buckets: 

![image](https://github.com/user-attachments/assets/b7bf5cb8-5d2f-452b-bfcd-49abedb74167)


2. > Criamos os diretorios : 

```
[oracle@dbsysh1 ~]$ pwd
/home/oracle
[oracle@dbsysh1 ~]$ mkdir archive_backups
[oracle@dbsysh1 ~]$ cd archive_backups/

[oracle@dbsysh1 archive_backups]$ mkdir module rman_config keys logs scripts

[oracle@dbsysh1 archive_backups]$ ls -lrt
total 24
drwxr-xr-x 4 oracle oinstall 4096 Jan 14 20:25 module
drwxr-xr-x 4 oracle oinstall 4096 Jan 14 20:59 rman_config
-rw------- 1 oracle oinstall  324 Jan 14 21:21 config
drwxr-xr-x 2 oracle oinstall 4096 Jan 14 21:31 keys
drwxr-xr-x 2 oracle oinstall 4096 Jan 14 21:46 logs
drwxr-xr-x 2 oracle oinstall 4096 Jan 14 21:50 scripts

```

Tomamos o erro abaixo devido a um erro no final do arquivo de private_key : 

3. Instalacao do modulo RMAN usando o oci_installer.jar

https://docs.oracle.com/en/cloud/paas/db-backup-cloud/csdbb/installing-oracle-database-cloud-backup-module-oci.html#GUID-4CB8F021-3B10-4E44-950C-4F799C66CC9D

Um ponto importante é que devemos usar o JAVA do $ORACLE_HOME/jdk conforme setamos abaixo no .bashrc

```
[oracle@dbsysh1 ~]$ cat .bashrc
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
. /etc/bashrc
fi

# User specific environment
if ! [[ "$PATH" =~ "$HOME/.local/bin:$HOME/bin:" ]]
then
PATH="$HOME/.local/bin:$HOME/bin:$PATH"
fi
export PATH

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions
ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1; export ORACLE_HOME
PATH=$PATH:/u01/app/oracle/product/19.0.0/dbhome_1/bin; export PATH
LD_LIBRARY_PATH=/u01/app/oracle/product/19.0.0/dbhome_1/lib; export LD_LIBRARY_PATH
ORACLE_UNQNAME=DB0102_nsr_gru;export ORACLE_UNQNAME
ORACLE_SID=DB0102; export ORACLE_SID
JAVA_HOME=/u01/app/oracle/product/19.0.0/dbhome_1/jdk
export PATH=$JAVA_HOME/bin:$PATH
## WARNING!! Modifying this file can cause failures in API/CLI provided by Cloud Tooling!!

```


```
run
{
ALLOCATE CHANNEL ch1 DEVICE TYPE 'SBT_TAPE' PARMS  'SBT_LIBRARY=/home/oracle/archive_backups/config/lib/libopc.so,SBT_PARMS=(OPC_PFILE=/home/oracle/archive_backups/config/oci_config.ora)';
}

RMAN Command Id : 2025-01-14T21:04:27
RMAN-00571: ===========================================================
RMAN-00569: =============== ERROR MESSAGE STACK FOLLOWS ===============
RMAN-00571: ===========================================================
RMAN-03009: failure of allocate command on ch1 channel at 01/14/2025 21:04:32
ORA-19554: error allocating device, device type: SBT_TAPE, device name:
ORA-27023: skgfqsbi: media manager protocol error
ORA-19511: non RMAN, but media manager or vendor specific failure, error text:
   KBHS-00715: HTTP error occurred 'operation-unauthorized'

```

E o erro estava no final arquivo private **OCI_API_KEY:**  

```
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQDKBIhkk2YhaIxR
gjaNmoKbldqUVqcBS4SoAc68yIje4rs+rRat3tPRAzVDwtx6R9qTRTvW5X5iiYq5
HHSmVmNGJ4WudP7c/hXg7uUZT+FQ9PIUVbMOxf7Kv2cVLHFmZVTy17vJdtch8Wjk

1Il9Bl3BnEFudTkJFiMNjI4=
-----END PRIVATE KEY-----
OCI_API_KEY
```



Para deletar uma peca de backup é necessario abrir um channel 

run
 {
 ALLOCATE CHANNEL ch1 DEVICE TYPE 'SBT_TAPE' PARMS  'SBT_LIBRARY=/home/oracle/archive_backups/rman_config/lib/libopc.so,SBT_PARMS=(OPC_PFILE=/home/oracle/archive_backups/rman_config/oci_config.ora)';
  delete backupset tag 'KEEP_DB0102_NSR_GRU_20250114' ;
 }
 





![image](https://github.com/user-attachments/assets/c6ff687e-726b-4c20-b94a-3cca9edb1792)




![image](https://github.com/user-attachments/assets/b7b412f7-cc14-451b-a609-39bec3aa6c5d)



 


![image](https://github.com/user-attachments/assets/a22922b6-4816-4cd5-8b00-34060824802d)

![image](https://github.com/user-attachments/assets/7c875967-58a2-45f2-838e-c63315fa250f)


![image](https://github.com/user-attachments/assets/8206a31a-32fe-4de1-95ed-f4c8ccad9de3)

![image](https://github.com/user-attachments/assets/3657dc35-dc36-4eb1-a13f-a58e6ca7b887)

![image](https://github.com/user-attachments/assets/0cba4cd1-435a-4957-9cc1-e3615223a373)


![image](https://github.com/user-attachments/assets/abf5820e-80d6-4fb6-b070-0fd5ca823401)
![image](https://github.com/user-attachments/assets/16efcfe1-37b8-4052-b554-971c328c5169)

![image](https://github.com/user-attachments/assets/3c9dd59f-0624-49fd-a57a-4787d8ef06d2)

![image](https://github.com/user-attachments/assets/a27c72bc-6bff-418f-8284-998ee5288945)

![image](https://github.com/user-attachments/assets/688a0a09-ae97-488d-8005-e60501cc5085)

![image](https://github.com/user-attachments/assets/1ade8012-1d41-4672-b5d8-d0d813935387)

Na configuracao do OEDA, os diskgroups do ZDLRA  sao 2 : CATALOG e  DELTA, as imagens de backup ficam no DELTA com redundancia NORMAL.
![image](https://github.com/user-attachments/assets/4a278ebd-7a17-45eb-939f-335e7d88f0e6)





**PERGUNTAS**
1. > Todas as Oracle Policies estao configuradas para armazenagem em disco ? ou funciona semelhantemente ao RA 

![Image](https://github.com/user-attachments/assets/af56a1f2-2659-4dce-bc85-7ed4236459f4)
