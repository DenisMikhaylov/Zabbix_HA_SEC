Настройка подключение к Zabbix по HTTPS

1. Генерация сертификата
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/zabbix.key -out /etc/ssl/certs/zabbix.crt
```

2. Внесение корретикоровк в файл nginx

```bash
nano /etc/zabbix/nginx.conf
```
Заменить настройки
```bash
server {
    listen          80;
    server_name     zabbix.corp1.ru;
    return          301 https://$host$request_uri;
}


server {
    listen          443 ssl;
    server_name     zabbix.corp1.ru;
    root            /usr/share/zabbix;

    index           index.php;

    ssl_certificate /etc/ssl/certs/zabbix.crt; # путь к сертификату
    ssl_certificate_key /etc/ssl/private/zabbix.key; # путь к приватному ключу
    ssl_protocols   TLSv1.2; # минимальный уровень протокола

    location = /favicon.ico {
        log_not_found   off;
    }

    location / {
        try_files       $uri $uri/ =404;
    }

    location /assets {
        access_log      off;
        expires         10d;
    }

    location ~ /\.ht {
        deny            all;
    }

    location ~ /(api\/|conf[^\.]|include|locale) {
        deny            all;
        return          404;
    }

    location /vendor {
        deny            all;
        return          404;
    }

    location ~ [^/]\.php(/|$) {
        fastcgi_pass    unix:/var/run/php/zabbix.sock;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_index   index.php;

        fastcgi_param   DOCUMENT_ROOT   /usr/share/zabbix;
        fastcgi_param   SCRIPT_FILENAME /usr/share/zabbix$fastcgi_script_name;
        fastcgi_param   PATH_TRANSLATED /usr/share/zabbix$fastcgi_script_name;

        include fastcgi_params;
        fastcgi_param   QUERY_STRING    $query_string;
        fastcgi_param   REQUEST_METHOD  $request_method;
        fastcgi_param   CONTENT_TYPE    $content_type;
        fastcgi_param   CONTENT_LENGTH  $content_length;

        fastcgi_intercept_errors        on;
        fastcgi_ignore_client_abort     off;
        fastcgi_connect_timeout         60;
        fastcgi_send_timeout            180;
        fastcgi_read_timeout            180;
        fastcgi_buffer_size             128k;
        fastcgi_buffers                 4 256k;
        fastcgi_busy_buffers_size       256k;
        fastcgi_temp_file_write_size    256k;
    }
}
```

```
systemctl restart zabbix-server zabbix-agent nginx php8.2-fpm
```
