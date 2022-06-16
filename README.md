# Aplicação rotten-potatoes

Nesta questão do Desafio Docker, foi criada uma aplicação web que realiza avaliações em filmes cadastrados com o Flask, Nginx e MongoDB dentro de containers Docker. 
Foi definida a configuração da pilha em um arquivo docker-compose.yml, juntamente com os arquivos de configuração para Python, MongoDB e Nginx. 
O Flask requer um servidor web para atender solicitações HTTP, portanto foi usado o Gunicorn, que é um servidor HTTP Python WSGI, para atender à aplicação. 
O Nginx atua como um servidor de proxy reverso que encaminha solicitações ao Gunicorn para processamento.


## Aplicação escrita em Python utilizando Flask - Desafio Docker - Questão 4

- Endereço do repositório:
<https://github.com/fernandomullerjr/rotten-potatoes>

- Para criação da imagem Docker foram usadas algumas boas práticas aprendidas no curso até o momento, como:

* Nomear a imagem Docker
* Evitar usar imagens não-oficiais de origem duvidosa
* Sempre especificar a tag nas imagens
* Um processo por Container
* Aproveitamento das camadas de imagem
* Utilização de gitignore e dockerignore

- A versão final do Dockerfile da aplicação ficou assim:

~~~Dockerfile
FROM python:3.10.1-slim

LABEL MAINTAINER="Fernando Müller <fernandomj90@gmail.com>"

RUN pip install --upgrade pip

WORKDIR /app

COPY ./requirements.txt ./requirements.txt
RUN pip3 install -r requirements.txt
RUN pip install gunicorn

COPY . .

EXPOSE 5000

ENV FLASK_APP=./app.py
ENV FLASK_ENV=development

CMD ["gunicorn", "-c", "config.py", "--bind", "0.0.0.0:5000", "app:app"]
~~~


- Também foi criado um Dockerfile para o NGINX:

~~~Dockerfile
FROM alpine:3.15.4

LABEL MAINTAINER="Fernando Müller <fernandomj90@gmail.com>"

RUN apk --update add nginx && \
    ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log && \
    mkdir /etc/nginx/sites-enabled/ && \
    mkdir -p /run/nginx && \
    rm -rf /etc/nginx/conf.d/default.conf && \
    rm -rf /var/cache/apk/*

COPY conf.d/app.conf /etc/nginx/conf.d/app.conf

EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]
~~~


## Ignorando arquivos desnecessários na criação da imagem

No diretório da aplicação podem existir arquivos indesejados e/ou desnecessários que seriam copiados com o comando `COPY` e que podem ser ignorados durante o processo de criação da imagem *Docker*.

Para isso foi criado no mesmo diretório onde se encontram os arquivos do projeto o arquivo `.dockerignore` que possui uma lista (com o mesmo padrão do arquivo `.gitignore`) com os diretórios/arquivos que serão ignorados no momento da execução da cópia dos arquivos para dentro da imagem.


## Criando a aplicação em uma única linha de comando

Para criação da aplicação (site e banco de dados) em uma única linha de comando precisamos utilizar o *Docker Compose*, onde vamos gerar a imagem *Docker* a ser utilizada para criar e executar o container da aplicação, um servidor web e outro container com o banco de dados da aplicação.

Foi criado, dentro do diretório da aplicação, um arquivo chamado `docker-compose` contendo o script *yaml* de criação dos objetos necessários para execução da aplicação, como mostrado abaixo:

~~~~yaml
version: '3'
services:

  flask:
    build:
      context: app
      dockerfile: Dockerfile
    container_name: flask
    image: fernandomj90/app-rotten-potatoes:v1
    ports:
      - 5000:5000
    environment:
      APP_PORT: 5000
      MONGODB_DB: admin
      MONGODB_DATABASE: admin
      MONGODB_USERNAME: mongouser
      MONGODB_PASSWORD: mongopwd
      MONGODB_HOSTNAME: mongodb
      MONGODB_HOST: mongodb
      MONGODB_PORT: 27017
#    volumes:
#      - appvolume:/app
    depends_on:
      - mongodb
    networks:
      - frontend
      - backend

  mongodb:
    image: mongo:4.0.8
    container_name: mongodb
    restart: unless-stopped
    command: mongod --auth
    environment:
      MONGO_INITDB_ROOT_USERNAME: mongouser
      MONGO_INITDB_ROOT_PASSWORD: mongopwd
      MONGO_INITDB_DATABASE: admin
      MONGODB_DATABASE: admin
      MONGODB_USER: mongouser
      MONGODB_PASS: mongopwd
      MONGODB_DATA_DIR: /data/db
      MONDODB_LOG_DIR: /dev/null
    ports:
      - "27017:27017"
    volumes:
      - mongodbdata:/data/db
    networks:
      - backend

  webserver:
    build:
      context: nginx
      dockerfile: Dockerfile
    image: fernandomj90/nginx-alpine-desafio-docker:3.15.4
    container_name: webserver
    restart: unless-stopped
    tty: true
    environment:
      APP_ENV: "prod"
      APP_NAME: "webserver"
      APP_DEBUG: "true"
      SERVICE_NAME: "webserver"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - nginxdata:/var/log/nginx
    depends_on:
      - flask
    networks:
      - frontend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge

volumes:
  mongodbdata:
#  appvolume:
  nginxdata:
~~~~


## Configuração das variáveis de ambiente

MONGODB_DB => Nome do database

MONGODB_HOST => Host do MongoDB

MONGODB_PORT => Posta de acesso ao MongoDB

MONGODB_USERNAME => Usuário do MongoDB

MONGODB_PASSWORD => Senha do MongoDB



## Realizando teste com o *Docker Compose*

Para realização dos testes com script configurado no arquivo `docker-compose`, precisamos:

1. Entrar no diretório onde está o arquivo docker-compose.yml
2. Executar o seguinte comando:
    docker-compose up -d

> Esse comando é para criar todos os objetos configurados no arquivo `docker-compose`.

Após a execução do comando teremos os seguintes containers em execução:

~~~~bash
fernando@debian10x64:~/cursos/kubedev/aula56-Desafio-Docker/questao4/rotten-potatoes$ docker ps
CONTAINER ID   IMAGE                                             COMMAND                  CREATED          STATUS          PORTS                                                                      NAMES
2accb746e68a   fernandomj90/nginx-alpine-desafio-docker:3.15.4   "nginx -g 'daemon of…"   27 seconds ago   Up 27 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp   webserver
5683c5724dad   fernandomj90/app-rotten-potatoes:v1               "gunicorn -c config.…"   28 seconds ago   Up 27 seconds   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp                                  flask
3517bc995b64   mongo:4.0.8                                       "docker-entrypoint.s…"   29 seconds ago   Up 28 seconds   0.0.0.0:27017->27017/tcp, :::27017->27017/tcp                              mongodb
fernando@debian10x64:~/cursos/kubedev/aula56-Desafio-Docker/questao4/rotten-potatoes$
~~~~

Com todos os Containers rodando corretamente, podemos acessar a aplicação através da URL:
<http://localhost:5000>


# Material adicional - Troubleshooting sobre problemas encontrados durante o projeto

Durante a criação do projeto, ocorreram problemas para executar a aplicação com sucesso, o principal deles será explicado melhor abaixo.

## ERRO:

- Erro no Container do Flask, utilizando NGINX, Gunicorn e Flask.
- Erro ocorre na subida do Gunicorn.
- É como se o COPY do Dockerfile não tivesse efeito, não existisse o módulo dentro do container, consequentemente não conseguia efetuar o start no serviço do GUNICORN conforme o esperado.

- Erro:
ModuleNotFoundError: No module named 'app'

~~~~bash
[2022-06-05 21:48:12 +0000] [10] [INFO] Worker exiting (pid: 10)
[2022-06-05 21:48:12 +0000] [1] [INFO] Shutting down: Master
[2022-06-05 21:48:12 +0000] [1] [INFO] Reason: Worker failed to boot.
[2022-06-05 21:48:16 +0000] [1] [INFO] Starting gunicorn 20.0.4
[2022-06-05 21:48:16 +0000] [1] [INFO] Listening at: http://0.0.0.0:5000 (1)
[2022-06-05 21:48:16 +0000] [1] [INFO] Using worker: sync
[2022-06-05 21:48:16 +0000] [10] [INFO] Booting worker with pid: 10
[2022-06-05 21:48:16 +0000] [10] [ERROR] Exception in worker process
Traceback (most recent call last):
  File "/usr/local/lib/python3.6/site-packages/gunicorn/arbiter.py", line 583, in spawn_worker
    worker.init_process()
  File "/usr/local/lib/python3.6/site-packages/gunicorn/workers/base.py", line 119, in init_process
    self.load_wsgi()
  File "/usr/local/lib/python3.6/site-packages/gunicorn/workers/base.py", line 144, in load_wsgi
    self.wsgi = self.app.wsgi()
  File "/usr/local/lib/python3.6/site-packages/gunicorn/app/base.py", line 67, in wsgi
    self.callable = self.load()
  File "/usr/local/lib/python3.6/site-packages/gunicorn/app/wsgiapp.py", line 49, in load
    return self.load_wsgiapp()
  File "/usr/local/lib/python3.6/site-packages/gunicorn/app/wsgiapp.py", line 39, in load_wsgiapp
    return util.import_app(self.app_uri)
  File "/usr/local/lib/python3.6/site-packages/gunicorn/util.py", line 358, in import_app
    mod = importlib.import_module(module)
  File "/usr/local/lib/python3.6/importlib/__init__.py", line 126, in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
  File "<frozen importlib._bootstrap>", line 978, in _gcd_import
  File "<frozen importlib._bootstrap>", line 961, in _find_and_load
  File "<frozen importlib._bootstrap>", line 948, in _find_and_load_unlocked
ModuleNotFoundError: No module named 'app'
[2022-06-05 21:48:16 +0000] [10] [INFO] Worker exiting (pid: 10)
[2022-06-05 21:48:16 +0000] [1] [INFO] Shutting down: Master
[2022-06-05 21:48:16 +0000] [1] [INFO] Reason: Worker failed to boot.
fernando@debian10x64:~/cursos/kubedev/aula56-Desafio-Docker/questao4/rotten-potatoes$
~~~~


## SOLUÇÃO

- O problema ocorria devido o Docker-compose estar setado para criar um volume para o service "app", este volume ao ser criado na subida via Docker-compose estava limpando o que havia sido feito via build do Dockerfile(COPY), fazendo com que o Gunicorn não encontrasse o módulo que ele precisava.

- Foram comentadas as linhas referentes ao volume do app no arquivo docker-compose.yml:

~~~~yaml
#    volumes:
#      - appvolume:/app

volumes:
  mongodbdata:
#  appvolume:
  nginxdata:
~~~~

- Sites que ajudaram a chegar nesta solução:

1. <https://forums.docker.com/t/dockerfile-copy-command-not-copying-files/81520/4>
2. <https://stackoverflow.com/questions/72277348/docker-is-not-copying-file-on-a-shared-volume-during-build>