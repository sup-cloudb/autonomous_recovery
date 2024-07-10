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

