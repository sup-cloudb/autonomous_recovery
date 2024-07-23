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

