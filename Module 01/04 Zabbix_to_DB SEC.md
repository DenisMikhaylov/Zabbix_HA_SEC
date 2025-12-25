Создайте пользователей и роли для основных компонентов:
```bash

```
```
mysql> CREATE USER   
        'zbx_srv'@'%' IDENTIFIED WITH mysql_native_password BY '<strong_password>',   
        'zbx_web'@'%' IDENTIFIED WITH mysql_native_password BY '<strong_password>'
        REQUIRE SSL   
        PASSWORD HISTORY 5; 

       mysql> CREATE ROLE 'zbx_srv_role', 'zbx_web_role'; 

       mysql> GRANT SELECT, UPDATE, DELETE, INSERT, CREATE, DROP, ALTER, INDEX, REFERENCES ON zabbix.* TO 'zbx_srv_role'; 
       mysql> GRANT SELECT, UPDATE, DELETE, INSERT ON zabbix.* TO 'zbx_web_role'; 

       mysql> GRANT 'zbx_srv_role' TO 'zbx_srv'@'%'; 
       mysql> GRANT 'zbx_web_role' TO 'zbx_web'@'%'; 

       mysql> SET DEFAULT ROLE 'zbx_srv_role' TO 'zbx_srv'@'%'; 
       mysql> SET DEFAULT ROLE 'zbx_web_role' TO 'zbx_web'@'%';
```

Проверка подключения

```bash
mysql -u zbx_srv -p -h 10.211.55.9 --ssl-mode=REQUIRED
```
Изминение строки в настройках zabbix server 
```
nano /etc/zabbix/zabbix_server.conf
```
```bash
DBTLSConnect=required
```

Настройка шибровани фронт веб к базе данных

```
/etc/zabbix/web/zabbix.conf.php
```
```
$DB['ENCRYPTION'] = true;
```
