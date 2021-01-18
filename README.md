iTop
=========

This role installs iTop (assuming the database is already installed elsewhere)

Requirements
------------

Role Variables
--------------

```
itop_download_from_sourceforge: [YES|no] Download the zip file?
itop_owner: User owner of itop application files
itop_group: Group owner of itop application files
itop_zipfile: URL or path name of the file. Default: Sourceforge download url
itop_webroot: path name of webroot for itop. Default: /srv/itop

itop_setup_mysql: [yes|NO] Install and configure MySQL
itop_mysql_control_socket: Path name of MySQL control socket
itop_mysql_db_name: name of itop database
itop_mysql_host: itop DB server
itop_mysql_user: itop db user
itop_mysql_password: itop db password

itop_mysql_apt_packages: .deb packages for MySQL
itop_php_apt_packages: .deb packages for PHP modules needed

itop_mysql_yum_packages: .rpm packages for MySQL
itop_php_yum_packages: .rpm packages for PHP modules needed
```


Dependencies
------------

Geerlingguy.nginx role is recommended

Example Playbook
----------------

This example uses geerlingguy.nginx to do the nginx setup.

    - hosts: servers
      become: yes
      roles:
        - role: itop
        - role: geerlingguy.nginx
      vars:
        itop_mysql_password: "Your Password Goes Here"

        nginx_vhosts:
          - listen: "[::]:443 ssl"
            server_name: "{{ ansible_fqdn }}"
            state: "present"
            filename: "{{ ansible_fqdn }}.443.conf"
            extra_parameters: |
              root {{ itop_webroot }};
              location ~ ^/.well-known/acme-challenge/* {
                allow all;
                root /var/www/html;
              }
              location / {
                try_files $uri $uri/ =404;
              }
              index index.php;
              location ~ \.(php|phar)(/.*)?$ {
                   fastcgi_split_path_info ^(.+\.(?:php|phar))(/.*)$;

                   fastcgi_intercept_errors on;
                   fastcgi_index  index.php;
                   include        fastcgi_params;
                   fastcgi_read_timeout 300;
                   fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                   fastcgi_param  PATH_INFO $fastcgi_path_info;
                   fastcgi_pass   php-fpm;
              }
              ssl_certificate /etc/letsencrypt/live/{{ ansible_fqdn }}/fullchain.pem;
              ssl_certificate_key /etc/letsencrypt/live/{{ ansible_fqdn }}/privkey.pem;
              ssl_protocols TLSv1.3 TLSv1.2;
              ssl_ciphers 'TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';
              add_header Strict-Transport-Security "max-age=15768000";
              proxy_buffering off;
          - listen: "[::]:80"
            server_name: "{{ ansible_fqdn }}"
            return: "301 https://{{ ansible_fqdn }}$request_uri"
            filename: "{{ ansible_fqdn }}.80.conf"

        itop_setup_mysql: yes
        itop_fastcgi_socket: "php-fpm"


License
-------

BSD

