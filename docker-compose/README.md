# Docker Compose

Como falamos anteriormente o docker-compose é um script que nos permite montar uma estrutura inteira de serviços em um só arquivo.

Como eu não tenho um projeto que eu possa utilizar para demonstrar o funcionamento completo, vou apenas criar o arquivo que vai subir ferramentas prontas mas sem o conteúdo que eu gostaria, mas nesse processo vou explicando também o funcionamento de tudo.

## Instalação docker-compose
Como foi falado anteriormente para toda e qualquer instalação, sempre seguir os passos da documentação oficial

* [ Link Documentação docker-compose ](https://docs.docker.com/compose/install/)

Vamos baixar o binário do docker compose e ele já vai ser alocado no local certo.
```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Dar a permissão necessária para conseguir executar o compose
```shell
sudo chmod +x /usr/local/bin/docker-compose
```

Criar o link simbólico para fazer o comando funcionar de qualquer pasta
```shell
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Vamos verificar se está funcionando o docker-compose
```shell
docker-compose --version
```
Retorno
```shell
[root@lab-docker cpu]# docker-compose --version
docker-compose version 1.29.2, build 5becea4c
```
Agora já temos nosso compose instalado e funcionando. Vamos desbravar como funciona o docker-compose.

Vamos começar por aquele comando mágico

```shell
[root@lab-docker cpu]# docker-compose --help
Define and run multi-container applications with Docker.

Usage:
  docker-compose [-f <arg>...] [--profile <name>...] [options] [--] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE             Specify an alternate compose file
                              (default: docker-compose.yml)
  -p, --project-name NAME     Specify an alternate project name
                              (default: directory name)
  --profile NAME              Specify a profile to enable
  -c, --context NAME          Specify a context name
  --verbose                   Show more output
  --log-level LEVEL           Set log level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  --ansi (never|always|auto)  Control when to print ANSI control characters
  --no-ansi                   Do not print ANSI control characters (DEPRECATED)
  -v, --version               Print version and exit
  -H, --host HOST             Daemon socket to connect to

  --tls                       Use TLS; implied by --tlsverify
  --tlscacert CA_PATH         Trust certs signed only by this CA
  --tlscert CLIENT_CERT_PATH  Path to TLS certificate file
  --tlskey TLS_KEY_PATH       Path to TLS key file
  --tlsverify                 Use TLS and verify the remote
  --skip-hostname-check       Don't check the daemon's hostname against the
                              name specified in the client certificate
  --project-directory PATH    Specify an alternate working directory
                              (default: the path of the Compose file)
  --compatibility             If set, Compose will attempt to convert keys
                              in v3 files to their non-Swarm equivalent (DEPRECATED)
  --env-file PATH             Specify an alternate environment file

Commands:
  build              Build or rebuild services
  config             Validate and view the Compose file
  create             Create services
  down               Stop and remove resources
  events             Receive real time events from containers
  exec               Execute a command in a running container
  help               Get help on a command
  images             List images
  kill               Kill containers
  logs               View output from containers
  pause              Pause services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pull service images
  push               Push service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  top                Display the running processes
  unpause            Unpause services
  up                 Create and start containers
  version            Show version information and quit
```

Agora vamos criar um arquivo de configuração do docker-compose

Antes de começar vamos criar 3 pastas para nosso teste

```shell
mkdir html database app
```
Agora vamos editar o arquivo e adicionar a configuração dentro dele.

```shell
vim docker-compose.yaml
```

```yaml
version: "3"
services:
    web-front:
      build:
        context: .
        dockerfile: dockerfile
      ports:
        - 8080:80
      volumes:
        - ./html/:/usr/local/apache2/htdocs/
      environment:
        - environment=staging
      networks:
        - web-infra
      links:
        - web-back
    web-back:
      image: node:14-slim
      ports:
        - 81:5060
      volumes:
        - ./app:/app
      command: ["sleep", "60000"]
      environment:
        - environment=staging
      networks:
        - web-infra
      links:
        - mysql
        - redis
        - mailcatcher
    mysql:
      image: mysql:5.7.29
      ports:
        - 3306:3306
      environment:
        - 'MYSQL_ROOT_PASSWORD=123456789'
      volumes:
        - ./database:/var/lib/mysql/
      networks:
        - web-infra
    redis:
      image: redis:alpine
      ports:
       - 6379:6379
      networks:
        - web-infra
    mailcatcher:
      image: schickling/mailcatcher
      ports:
        - 1025:1025
        - 1080:1080
      networks:
        - web-infra
networks:
    web-infra:
        driver: bridge
        ipam:
            driver: default
```

Agora vamos subir esse arquivo para analisarmos eles quando subir

```shell
docker-compose up
```
Podemos subir a aplicação dessa forma ou também em segundo plano e selecionando um arquivo diferente desta forma

```shell
docker-compose -f docker-compose.override.yaml up -d
```
Desta forma é escolhido um arquivo diferente com o -f e utiliza o up para subir tudo e o -d para rodar em segundo plano.

Após subir vamos conseguir ver que temos 5 serviços rodando

```shell
[root@lab-docker ~]# docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS          PORTS                                                                                  NAMES
9c7928805e6c   nginx                    "/docker-entrypoint.…"   34 seconds ago   Up 33 seconds   0.0.0.0:8080->80/tcp, :::8080->80/tcp                                                  root_web-front_1
76d7b4496eeb   node:14-slim             "docker-entrypoint.s…"   35 seconds ago   Up 33 seconds   0.0.0.0:81->5060/tcp, :::81->5060/tcp                                                  root_web-back_1
3fb2df882d5d   schickling/mailcatcher   "mailcatcher --no-qu…"   11 minutes ago   Up 34 seconds   0.0.0.0:1025->1025/tcp, :::1025->1025/tcp, 0.0.0.0:1080->1080/tcp, :::1080->1080/tcp   root_mailcatcher_1
b953255c6354   redis:alpine             "docker-entrypoint.s…"   11 minutes ago   Up 34 seconds   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp                                              root_redis_1
c4c798d22bb8   mysql:5.7.29             "docker-entrypoint.s…"   11 minutes ago   Up 34 seconds   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp                                   root_mysql_1

```

Para finalizar as máquinas caso não tenha utilizado o -d podemos apenas apertar ctrl + c, no caso de ter utilizado o -d podemos utilizar o comando.

```shell
docker-compose stop
```
Vai fazer apenas uma parada no serviço mas ainda será possível ver os containers com o ps -a, caso queira apagar definitivamente as imagens é necessário utilizar o comando:

```shell
docker-compose down
```

Após fazer isso ao utilizar o comando ps -a verá que não vai ter mais nenhum container.

* com o docker pode ser utilizado também o docker-swarm, mas então entramos em conceitos um pouco mais amplo e esse tutorial já abrange uma grande quantidade de conteúdo.

Mas se quiser saber mais sobre o docker swarm e continuar estudando a documentação oficial é uma ótima companheira para aprender mais.


## Dúvidas

Agora é o momento que trocamos uma ideia sobre os problemas que tiveram, sobre informações que não entenderam para na prática sanar as dúvidas.

## Arquivo docker-compose.yaml
* [ docker-compose.yaml ](https://github.com/ViniFTex/docker-compose/README.md?plain=179)

* [ Documentação V3 compose ](https://docs.docker.com/compose/compose-file/compose-file-v3/)

Descrição das opções do arquivo docker-compose que utilizamos no tutorial.

* Version - Deve ser informado a versão do compose que deseja utilizar
* Services - Abaixo dos services são informados todos os containers que deseja criar
    * web-front - Aqui é o nome que deseja definir para o serviço
        * image - Imagem que deseja utilizar no container
        * build - Build é definido quando deseja que o docker compose crie a imagem na hora de sugir
            * context - context é a pasta onde esta o docker file se utilizar . é no mesmo diretório do docker-compose
            * dockerfile dockerfile - arquivo dockerfile que deseja buildar
        * ports - essa é a configuração de portas abaixo podemos informar mais de uma porta
            * portaUtilizada - aqui é a porta que será utilizada se informar só a porta Ex. 80 ele ficara exposto somente internamente na rede criada para os container se informar porta:porta 8080:80 será a porta externa:interna
        * volumes: Aqui é chamado a configuração de volumes, aqui também podemos ter mais de um volume montado.
            * caminhoUtilizado -  Aqui informamos o caminho da montage, é aconselhavel informar o caminho inteiro local e o caminho inteiro dentro do container Ex. /mnt/data:/var/lib/docker.
        * command - Esta opção utilizamos quando precisamos executar alguma coisa dentro container assim que ele iniciar Ex para debug ["sleep","60000"]
        * environment - Aqui chamamos as variaveis e podemos ter mais de uma variavel informada
            * variavelUtilizada - A variavel deve seguir o padrão de chave valor 'environment=prod'
        * networks - aqui chamamos a configuração do network que foi configurado abaixo na area de network
            * nomeRedeUtilizada - informamos aqui o nome da rede que foi criada ou que necessita conexão
        * links - Chama os links de comunicação que precisa que tenha conexão podemos ter mais de um link
            * nomeServiçosLink - Essa configuração podemos informar o nome de serviço que queremos que esse serviço se comunique
    * networks -  Nesta configuração chamamos a configuração que vai criar a rede para utilização nessa estrutura
        * nomeNetwork - Informamos um nome para a rede que será chamada nos networks dentro dos serviços
            * driver - configuração que selecionamos varias formas de utilizar a rede do computador na documentação mostra todas as formas! Essa que estamos utilizando é a padrão
            * ipam -
                * driver -

