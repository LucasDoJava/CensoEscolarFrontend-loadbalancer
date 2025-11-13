# CensoEscolarFrontend-loadbalancer

#Crie um network para os nós e o loadbalancer

    docker network create webnet

# Instalação dos serviços

    docker run -itd --network webnet --name loadbalancer -p 82:80 nginx:1.29.3-alpine
    docker run -itd --network webnet --name node1 -v caminho/do/projeto:usr/share/nginx/html nginx:1.29.3-alpine
    docker run -itd --network webnet --name node2 -v caminho/do/arquivo:usr/share/nginx/html nginx:1.29.3-alpine
    docker run -itd --network webnet --name node3 -v caminho/do/arquivo:usr/share/nginx/html nginx:1.29.3-alpine
    docker run -itd --network webnet --name node4 -v caminho/do/arquivo:usr/share/nginx/html nginx:1.29.3-alpine
    docker run -itd --network webnet --name node5 -v caminho/do/arquivo:usr/share/nginx/html nginx:1.29.3-alpine

# Acesse o bash do loadbalancer

    docker exec -it loadbalancer sh

# Acesse o arquivo default.conf

    cd etc/nginx/conf.d
    nano default.conf

# Edite o arquivo da seguinte forma

    upstream webfront {
      server node1:80;
      server node2:80;
      server node3:80;
      server node4:80;
      server node5:80;
    }

    server {
        listen 80;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://webfront;
      }
    }

# Acessando esse mesmo arquivo nos nós, faça a seguinte confguração

    server {
        listen 80;
        server_name _;
    
       
        root /usr/share/nginx/html;
        index index.html;
    
    
        location / {
            try_files $uri /index.html;
        }
    
        
        location /static/ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    
     
        location = /favicon.ico { access_log off; log_not_found off; }
        location = /manifest.json { access_log off; log_not_found off; }
    }

# Reinicie todos os container

    docker restart loadbalancer node1 node2 node3 node4 node5

# No host, acesse seu navegador e passe a url 

    localhost:82


