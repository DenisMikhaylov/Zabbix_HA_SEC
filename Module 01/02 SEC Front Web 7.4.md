
Конфигурация

1. Генерация сертификатов и ключей

Создайте рабочую директорию:

```bash
sudo mkdir -p /etc/zabbix/ssl && cd /etc/zabbix/ssl
```
Создайте сертификат удостоверяющего центра

```bash
sudo openssl genrsa -out ca.key 4096
sudo openssl req -new -x509 -days 3650 -key ca.key -out ca.crt -subj "/CN=ZabbixCA/"
```
Генерируйте приватный ключ и сертификат для сервера Zabbix (корректируйте значение `server.corp.local` согласно вашему фактическому общему имени):

```bash
sudo openssl genrsa -out server.key 2048
sudo openssl req -new -key server.key -out server.csr -subj "/CN=server.corp.local/"
sudo openssl x509 -req -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -sha256 -out server.crt
```
Генерируйте приватный ключ и сертификат для фронтенда Zabbix :

```bash
sudo openssl genrsa -out frontend.key 2048
sudo openssl req -new -key frontend.key -out frontend.csr -subj "/CN=server/"
sudo openssl x509 -req -days 365 -in frontend.csr -CA ca.crt -CAkey ca.key -CAcreateserial -sha256 -out frontend.crt
```
2. Установите правильные разрешения

Для сервера Zabbix:

```bash
sudo chown root:zabbix /etc/zabbix/ssl/server.{crt,key} /etc/zabbix/ssl/ca.crt
sudo chmod 640 /etc/zabbix/ssl/server.key
sudo chmod 644 /etc/zabbix/ssl/server.crt /etc/zabbix/ssl/ca.crt
```
Для фронтенда (установите владельца и группу согласно используемому вами веб-серверу):

```bash
sudo chown root:www-data /etc/zabbix/ssl/frontend.{crt,key}
sudo chmod 640 /etc/zabbix/ssl/frontend.key
sudo chmod 644 /etc/zabbix/ssl/frontend.crt
```
3. Настройте сервер Zabbix

В файле `zabbix_server.conf` добавьте строки:

```ini
TLSFrontendAccept=cert
TLSCertFile=/etc/zabbix/ssl/server.crt
TLSKeyFile=/etc/zabbix/ssl/server.key
TLSCAFile=/etc/zabbix/ssl/ca.crt
# Дополнительно можете добавить (если необходимо):
# TLSFrontendCertIssuer=/CN=ZabbixCA/
# TLSFrontendCertSubject=/CN=server/
```
Затем перезагрузите сервер:

```bash
sudo systemctl restart zabbix-server
```
4. Настройка фронтенда Zabbix.
На существующих установках отредактируйте следующие поля в файле `zabbix.conf.php`:

```php
$ZBX_SERVER_TLS['ACTIVE'] = '1';
$ZBX_SERVER_TLS['CA_FILE'] = '/etc/zabbix/ssl/ca.crt';
$ZBX_SERVER_TLS['KEY_FILE'] = '/etc/zabbix/ssl/frontend.key';
$ZBX_SERVER_TLS['CERT_FILE'] = '/etc/zabbix/ssl/frontend.crt';
// Дополнительно можно задать (при необходимости):
// $ZBX_SERVER_TLS['CERTIFICATE_ISSUER']  = '/CN=ZabbixCA/';
// $ZBX_SERVER_TLS['CERTIFICATE_SUBJECT'] = '/CN=server.corp.local/';
```
5. Проверьте правильность шифрования, убедившись в отсутствии ошибок в журнале сервера Zabbix или журнала фронтенда Zabbix:

```bash
tail -f /var/log/zabbix/zabbix_server.log
```
