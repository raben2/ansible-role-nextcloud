Ansible Role: Nextcloud
=======================

Installs and configures [nextcloud](https://www.nextcloud.com) on Ubuntu based servers using Nginx and PostgreSQL

## Requirements
* [raben2.nginx](https://github.com/raben2/ansible-role-nginx)
* [raben2.postgres](https://github.com/raben2/ansible-role-postgres)  

## Role Variables

please refer to `defaults/main.yml`

## Example Playbook

Define your group vars for postges and nginx 
This example will install nextcloud according to this
[nginx-example](https://docs.nextcloud.com/server/11/admin_manual/installation/nginx_examples.html)

```yaml

postgres_host_address: 127.0.0.1
nextcloud_database_host: "{{ postgres_host_address }}"

nginx_ppa_use: true
nginx_install_php_fpm: true
nginx_remove_default_vhost: true
nginx_php_dependencies: 
    - gd
    - dom
    - ctype
    - iconv
    - json
    - xml
    - mbstring
    - posix
    - simplexml
    - xmlwriter
    - zip
    - curl
    - pgsql
    - intl
    - imagick

nginx_upstreams:
    - name: "{{ nginx_php_fpm_upstream_name }}"
      servers:
        - "unix:{{ nginx_php_socket }}"
nginx_vhosts: 
  - listen: '0.0.0.0'
    server_name: "cloud.example.com"
    port: 80
    index: "index.php index.html index.htm"
    root: "{{ nginx_vhost_root }}"
    ssl: false
    gzip: 'off'
    upload_size: 512M
    cgi_params:
      - 'fastcgi_buffers 64 4K'
    add_header:
     - 'X-Content-Type-Options nosniff'
     - 'X-Frame-Options "SAMEORIGIN"'
     - 'X-XSS-Protection "1; mode=block"'
     - 'X-Robots-Tag none'
     - 'X-Download-Options noopen'
     - 'X-Permitted-Cross-Domain-Policies none'
    location:
     - uri: '/'
       modifier: none
       regex: none
       extra_params:
           - 'rewrite ^ /index.php$uri'
     - uri: '^/'
       modifier: '~'
       regex: '(?:build|tests|config|lib|3rdparty|templates|data)/'
       extra_params: 
           - 'deny all'
     - uri: '^/'
       modifier: '~'
       regex: '/(?:\.|autotest|occ|issue|indie|db_|console)'
       extra_params: 
           - 'deny all'
     - uri: '^/'
       modifier: '~'
       regex: '(?:index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|ocs-provider/.+|core/templates/40[34])\.php(?:$|/)'
       extra_params:
         - 'fastcgi_split_path_info ^(.+\.php)(/.*)$' 
         - 'include fastcgi_params'
         - 'fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name'
         - 'fastcgi_param PATH_INFO $fastcgi_path_info'
         - 'fastcgi_param HTTPS off'
         - 'fastcgi_param modHeadersAvailable true'
         - 'fastcgi_param front_controller_active true'
         - 'fastcgi_pass {{ nginx_php_fpm_upstream_name }}'
         - 'fastcgi_intercept_errors on'
         - 'fastcgi_request_buffering off'
     - uri: '^/'
       modifier: '~'
       regex: '(?:updater|ocs-provider)(?:$|/)'
       extra_params:
           - 'try_files $uri/ =404'
           - 'index index.php'
     - uri: '\.'
       modifier: '~*'
       regex: '(?:css|js|woff|svg|gif)$'
       add_header: 
         - 'Cache-Control "public, max-age=7200'
         - 'X-Content-Type-Options nosniff"'
         - 'X-Frame-Options "SAMEORIGIN"'
         - 'X-XSS-Protection "1; mode=block"'
         - 'X-Robots-Tag none'
         - 'X-Download-Options noopen'
         - 'X-Permitted-Cross-Domain-Policies none'
       extra_params:
         - 'try_files $uri /index.php$uri$is_args$args'
         - 'access_log off'
     - uri: '\.'
       modifier: '~*'
       regex: '(?:png|html|ttf|ico|jpg|jpeg)$'
       extra_params: 
         - 'try_files $uri /index.php$uri$is_args$args'
         - 'access_log off'
````

## Example Playbook
```yaml
    - hosts: server
      roles:
      	- { role: raben2.postgres }
        - { role: raben2.nginx }
        - { role: raben2.nextcloud }
```
## ToDo
* Add support for mariadb database
* add different distributions


## License

LGPL

## Author Information

This role was created in 2017 by [Georg Rabenstein](https://github.com/raben2)