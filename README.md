# nginx-config
Nginx configuration


## Simple proxy server
Suppose you running the app and listen on port 6000.

Create a config file, using domain name for naming is recommended
`sudo vi /etc/nginx/sites-enabled/tvux.me`

**/etc/nginx/sites-enabled/tvux.me**
```
server {
   server_name tvux.me;
   listen 80;
   location / {
      proxy_pass http://localhost:6000;
   }
}
```

## Https server
To enabled https for nginx, install certbot & certbot nginx plugin (tutorial in ...)
`sudo certbot --nginx`
then select config file you want to make a https -> select 2 to redirect all http request to https

## IPv6 support
**/etc/nginx/sites-enabled/tvux.me**
```
server {
   server_name tvux.me;
   listen 80;
   listent [::]:80;
   location / {
      proxy_pass http://localhost:6000;
   }
}
```

## Load balancer
**/etc/nginx/sites-enabled/tvux.me**
```
upstream backend {
  server localhost:6000 fail_timeout=5s max_fails=3;
  server localhost:6001 backup;
}

server {
   server_name tvux.me;
   listen 80;
   listent [::]:80;
   location / {
      proxy_pass http://backend;
   }
}
```

## Inspect request IP address
**/etc/nginx/sites-enabled/tvux.me**
```
server {
   server_name tvux.me;
   listen 80;

   location / {
      proxy_pass http://backend;
      proxy_set_header X-Real-IP  $remote_addr;
      proxy_set_header X-Forwarded-For $remote_addr;
      proxy_set_header Host $host;
   }
}
```
Now you can get request IP address from `request.headers['X-Real-IP']`

## CORS
**/etc/nginx/sites-enabled/tvux.me**
```
server {
   server_name tvux.me;
   listen 80;

   location / {
      proxy_pass http://backend;
      add_header Access-Control-Allow-Origin *;
      add_header Access-Control-Allow-Headers *;
   }
}
```

## Worker connections
By default, Nginx use 768 worker_connections. It'll fail to serve your app if there is a massive amount of requests at the same time. A simple solution is increase worker_connections to a larger number.

**/etc/nginx/nginx.conf**
```
events {
    worker_connections 20000;
}
```

## Increase maximum request body size
**/etc/nginx/nginx.conf**
```
http {
   client_max_body_size 300M;
}
```

