# autonomous_recovery
Testes com DBRS.

**Testes com DBRS**
1------- 
> Criando um banco DBCS na regiao de GRU com DBRS e a partir de um backup on-demand, criar um novo dbcs com shape distinto mantendo a mesma Edition(obrigatorio) na regiao de VCP.

> Na regiao de VCP o banco estaria numa subnet publica e por isso foi necessario adicionar uma rota ao Internet Gateway, mesmo a subnet do Recovery Service ser Privada em GRU.

2------- Os backups de archives sao gerados a cada 60 minutos


[root@dbnorigem ~]# dbcli list-pendingjobs

![image](https://github.com/sup-cloudb/autonomous_recovery/assets/72585042/265e69cc-9118-4384-b97c-4a430b3d51d4)

