# Collabora online ansible role

For integration on Nextcloud

Works on debian 9

WARNING, I don't use SSL, because in my case, loolswd is on a private LAN, behind a reverse proxy (an other VM) witch uses HTTPS.

On defaults/mail.yml change the value by your own Nextcloud TLD.


Here the a sample of the reverse proxy configuration for the vhost collabora.domaine.com.
With 
* 192.168.100.16 collabora server local IP
* collabora.domaine.com the subdomain pointing on the reverse proxy

    server {

        server_name collabora.domaine.com;

        location / {
            include proxy_params;
                    proxy_pass http://192.168.100.16:9980;
        }
        
        listen [::]:443 ssl; 
        listen 443 ssl; 
        ssl_certificate /etc/letsencrypt/live/collabora.domaine.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/collabora.domaine.com/privkey.pem;
        include /etc/letsencrypt/options-ssl-nginx.conf;
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

        # static files
        location ^~ /loleaflet {
            proxy_pass http://192.168.100.16:9980;
            proxy_set_header Host $http_host;
        }
        # WOPI discovery URL
        location ^~ /hosting/discovery {
            proxy_pass http://192.168.100.16:9980;
            proxy_set_header Host $http_host;
        }
        # main websocket
        location ~ ^/lool/(.*)/ws$ {
            proxy_pass http://192.168.100.16:9980;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
            proxy_set_header Host $http_host;
            proxy_read_timeout 36000s;
        }
        # download, presentation and image upload
        location ~ ^/lool {
            proxy_pass http://192.168.100.16:9980;
            proxy_set_header Host $http_host;
        }
        # Admin Console websocket
        location ^~ /lool/adminws {
            proxy_pass http://192.168.100.16:9980;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
            proxy_set_header Host $http_host;
            proxy_read_timeout 36000s;
        }


    }
