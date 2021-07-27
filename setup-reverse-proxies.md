# Nginx Reverse Proxies
To get access to the k8 subnet we used a reverse proxy

## Default Proxy settings (no websockets)

```nginx
server {
    listen 9000;

    location / {
        proxy_set_header   X-Forwarded-For $remote_addr;
        proxy_set_header Host $host;
        proxy_pass http://<external-ip-assigned-from-loadbalancer>;
    }
}
```

## Settings for JupyterHub (websockets)

JupyterHub is using websockets, so the settings have to be adjusted.

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80;

    location / {
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;

        # websocket headers
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header X-Scheme $scheme;

        proxy_buffering off;

        proxy_pass <external-ip-assigned-from-loadbalancer>;
    }
}
```
