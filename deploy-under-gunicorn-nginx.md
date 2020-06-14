# Deploy A Python app for production with gunicorn and nginx

* **OS**: Ubuntu 18.04LTS
* **Python**: 3.6.9
* **App Name**: apis, developed under /path/to/dev/apis/v1
```
env/
main.py
module1/
module2/
```

## Set up for production
```
cd /var/www/
cp -r /path/to/dev/apis .
rm -rf apis/v1/env
chown -R www-data:users apis
cd apis/v1
python -m venv env
source env/bin/activate
pip install flask
pip install flask-restful
pip install requests
```
Test the app
```
python main.py
```
Type Ctrl+c, then
```
deactivate
```
## Create /etc/systemd/system/apis.service
```
[Unit]
Description=apis daemon
Requires=apis.socket
After=network.target

[Service]
Type=notify
# the specific user that our service will run as
User=www-data
Group=users
# another option for an even more restricted service is
# DynamicUser=yes
# see http://0pointer.net/blog/dynamic-users-with-systemd.html
RuntimeDirectory=gunicorn
WorkingDirectory=/var/www/apis/v1
ExecStart=/var/www/apis/v1/env/bin/gunicorn --workers=3 main:app
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=mixed
TimeoutStopSec=5
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

## Create /etc/systemd/system/apis.socket 
```
[Unit]
Description=apis socket

[Socket]
ListenStream=/run/apis.sock
# Our service won't need permissions for the socket, since it
# inherits the file descriptor by socket activation
# only the nginx daemon will need access to the socket
User=www-data
# Optionally restrict the socket permissions even more.
# Mode=600

[Install]
WantedBy=sockets.target
```
## Start apis as a service under systemd
```
systemctl enable --now apis.socket
service apis start
```
## Create /etc/nginx/sites-available/apis
### Deploy under root
```
# Desired URL: http://1.1.1.1:3000/texts/counts
server {
    listen 3000;
    server_name 1.1.1.1;

    location / {
        proxy_pass http://unix:/run/apis.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
### Deploy under a subdirectory
```
# Desired URL: http://1.1.1.1:3000/apis/v1/texts/counts
server {
    listen 3000;
    server_name 1.1.1.1;

    location /apis/v1/ {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        rewrite /apis/v1/([^/]+) /$1 break;
        proxy_pass http://unix:/run/apis.sock;
    }
}
```
## Go live
```
cd /etc/nginx/sites-enabled/
ln -s ../sites-available/apis .
sudo nginx -t
sudo nginx -s reload
```

# References:
* https://docs.gunicorn.org/en/stable/deploy.html
* https://www.linode.com/docs/development/python/flask-and-gunicorn-on-ubuntu/
* http://nginx.org/en/docs/http/ngx_http_proxy_module.html
* https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
* https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/
