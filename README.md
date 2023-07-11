# Configurar Nginx

### Ajustar php-fpm:
<code>/etc/php-fpm.d/www.conf</code>
```ini
[www]
user = nginx
group = nginx
listen = /run/php-fpm/www.sock
; listen = 127.0.0.1:9000
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
listen.acl_users = nginx
listen.allowed_clients = 127.0.0.1
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
slowlog = /var/log/php-fpm/www-slow.log
```


### Configuración básica de Nginx con <code>PATH_INFO</code>:
<code>/etc/nginx/nginx.conf</code>
```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name demo.localhost;

    # Prevent nginx HTTP Server Detection
    server_tokens off;

    # Enforce HTTPS
    return 301 https://$server_name$request_uri;
	#return 301 https://$host$request_uri;
}

server {
    listen       443 ssl http2;
    listen       [::]:443 ssl http2;

    server_name             demo.localhost;
    root                    "/var/www/html/proyect/public";
    ssl_certificate         "/ssl/certificate.crt";
    ssl_certificate_key     "/ssl/certificate.key";
    ssl_trusted_certificate "/ssl/CA.crt";

    server_tokens off; # Prevent nginx HTTP Server Detection

    index index.php index.html /index.php$request_uri;

    location ~ \.php(?:$|/) {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        set $path_info $fastcgi_path_info;
        try_files $fastcgi_script_name =404;

        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_index /index.php;

        include /etc/nginx/fastcgi_params;
        fastcgi_param PATH_INFO       $path_info;
        fastcgi_param PATH_TRANSLATED $document_root$path_info;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location / {
        try_files $uri $uri/ /index.php$request_uri;
    }

    location ~ /\. {
        deny all;
    }
}
```

## Obtener la URI en PHP <code><i>^8.1</i></code>
```php
$uri = $_SERVER['PATH_INFO'] ?? '/';
echo $uri;
```

### Reiniciar
```shell
systemctl restart nginx php-fpm
```

## Créditos
[<code>Nextcloud</code>](https://docs.nextcloud.com/server/latest/admin_manual/installation/nginx.html#nextcloud-in-the-webroot-of-nginx)
[<code>Mozilla</code>](https://ssl-config.mozilla.org/#server=nginx&version=1.20.1&config=modern&openssl=3.0.7&guideline=5.7)
[<code>KumbiaPHP</code>](https://github.com/KumbiaPHP/Documentation/blob/master/es/to-install.md#configurar-nginx)
[<code>Computingforgeeks</code>](https://computingforgeeks.com/install-php-8-on-rocky-linux-almalinux-8/?expand_article=1)
[<code>Nginx</code>](https://www.nginx.com/resources/wiki/start/topics/examples/phpfcgi)

