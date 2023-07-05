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


### Configuración de Nginx con <code>PATH_INFO</code>:
<code>/etc/nginx/nginx.conf</code>
```nginx
server {
    listen       443 ssl http2;
    listen       [::]:443 ssl http2;

    server_name myapp.example.com;
    root       "/var/www/html/{proyecto}/public";
    ssl_certificate "/path/to/ssl/certificate.crt";
    ssl_certificate_key "/path/to/ssl/certificate.key";

    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers PROFILE=SYSTEM;
    ssl_prefer_server_ciphers on;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php;
    }

    location ~ \.php$ {
        fastcgi_pass  unix:/run/php-fpm/www.sock;
        # fastcgi_pass  127.0.0.1:9000;

        fastcgi_index /index.php;
        include /etc/nginx/fastcgi_params;
        fastcgi_split_path_info       ^(.+\.php)(/.+)$;
        fastcgi_param PATH_INFO       $fastcgi_path_info;
        fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
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

## Créditos
[<code>KumbiaPHP</code>](https://github.com/KumbiaPHP/Documentation/blob/master/es/to-install.md#configurar-nginx)
[<code>Computingforgeeks</code>](https://computingforgeeks.com/install-php-8-on-rocky-linux-almalinux-8/?expand_article=1)
