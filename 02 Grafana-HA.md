# How to install Grafa in HA mode with the help of Docker Compose
![image](https://github.com/user-attachments/assets/6dbce0f5-e96f-4d55-9144-923fbdf2a7fd)

### create the direcotry
```
mkdir /data/conf/
```

### Grafana Config file
```
cat <<EOF>> /data/conf/grafana.ini
# instance name, defaults to HOSTNAME environment variable value or hostname if HOSTNAME var is empty
instance_name = ${HOSTNAME}
[smtp]
;enabled = false
;host = <host>:25
;user = Grafana
;# If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""
;password =
;cert_file =
;key_file =
;skip_verify = true
;from_address = admin@grafana-container.com
;from_name = Grafana
;# EHLO identity in SMTP dialog (defaults to instance_name)
;;ehlo_identity = dashboard.example.com
;# SMTP startTLS policy (defaults to 'OpportunisticStartTLS')
;;startTLS_policy = NoStartTLS

[database]
user = grafana
password = grafana123
url = postgres://grafana:grafana123@grafana-db:5432/grafana

[users]
# disable user signup / registration
allow_sign_up = false


[security]
# disable creation of admin user on first start of grafana
;disable_initial_admin_creation = false

;# default admin user, created on startup
admin_user = admin
;
;# default admin password, can be changed before first start of grafana,  or in profile settings
admin_password = admin123
EOF
```

## NGINX CONFIG FILE
```
cat <<EOF>> /data/conf/nginx.conf
events {
    worker_connections  1024;
}

http {

  upstream grafana-HA {
    server grafana1:3000 weight=5;
    server grafana2:3000 weight=5;
  }

  

  server {
    listen 80;
      server_name localhost

      client_max_body_size 0;

      chunked_transfer_encoding on;

    location / {
      

#      auth_basic "Admin Area";
#      auth_basic_user_file /etc/nginx/conf.d/nginx.htpasswd;


      proxy_pass                          http://grafana-HA;
      proxy_set_header  Host              $http_host;         proxy_set_header  X-Real-IP         $remote_addr;       proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
      proxy_set_header  X-Forwarded-Proto $scheme;
      proxy_read_timeout                  900;
    }
  }
}
EOF
```

### Docker Compose file
```
cat <<EOF>> /data/docker-compose.yml
version: "3"
volumes:
  db_data:
networks:
  monitor:
    driver: bridge
services:
  proxy:
    image: nginx:alpine
    container_name: nginx
    ports:
      - 8080:80
    networks:
      - monitor
    volumes:
      - ./conf/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - grafana1
      - grafana2
  grafana-db:
    container_name: grafana-db
    image: postgres:12-alpine
    restart: unless-stopped
    volumes:
      - db_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: grafana123
      POSTGRES_USER: grafana
      POSTGRES_DB: grafana
      PGDATA: /var/lib/postgresql/data/pgdata
    networks:
      - monitor
  grafana1:
    container_name: grafana1
    image: grafana/grafana:7.4.1
    networks:
      - monitor
    expose:
      - 3000
    volumes:
      - ./plugins:/var/lib/grafana/plugins
      - ./conf/grafana.ini:/etc/grafana/grafana.ini
    depends_on:
      - grafana-db

  grafana2:
    container_name: grafana2
    image: grafana/grafana:7.4.1
    networks:
      - monitor
    expose:
      - 3000
    volumes:
      - ./plugins:/var/lib/grafana/plugins
      - ./conf/grafana.ini:/etc/grafana/grafana.ini
    depends_on:
      - grafana-db
EOF
```
### Now use the Docker compose up command.
```
docker compose up -d
```

### Check the conatiners
```
docker ps
```

## Check the VM IP
```
ip a
```

### Check the Grafana URL
```
192.168.1.32:8080
```

### How to delete this setup?
```
docker compose down
```
### Check the conatiners
```
docker ps
```
