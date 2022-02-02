# Aprendendo sobre docker
Aqui vamos utilizar a prática para ilustrar mais ou menos o que foi dito na apresentação.

## Copy-On-Write (COW) e Camadas Docker
Para iniciarmos vamos fazer uma pequena amostra sobre como funcionam as camadas de uma imagem e vamos aprender a ver as imagens no docker hub

Iniciaremos vendo como funcionam e como encontramos a imagem que baixamos do docker hub e entender como funcionam as camadas das imagens.

* [ Docker Hub Imagem nginx ](https://hub.docker.com/_/nginx)

Desta forma vamos fazer o download da imagem latest do nginx
```shell
 docker pull nginx
```
Também podemos procurar lá no docker hub uma versão diferente para baixarmos e utilizarmos

```shell
docker pull nginx:alpine
```
Após baixar as duas vamos dar uma olhada nelas baixadas

```shell
docker images
```
Retorno

```shell
[root@lab-docker yum.repos.d]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    c316d5a335a5   6 days ago     142MB
nginx         alpine    bef258acf10d   7 days ago     23.4MB
hello-world   latest    feb5d9fea6a5   4 months ago   13.3kB
```

Temos a imagem nginx:latest, nginx:alpine e hello-world que usamos para testar o docker quando instalamos.

* Podemos notar também a diferença de tamanho das imagens e isso geralmente porque uma tem menos camadas e menos dependências do que as outras versões e é assim que devemos pensar na hora de construir nossas imagens.

Para analisarmos melhor vamos utilizar um comando para compararmos o que tem em cada camada de imagens e ver por que a diferença de tamanho

```shell
docker history nginx:latest && docker history hello-world
```
Retorno
```shell
[root@lab-docker yum.repos.d]# docker history nginx:latest && docker history hello-world
IMAGE          CREATED      CREATED BY                                      SIZE      COMMENT
c316d5a335a5   6 days ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B        
<missing>      6 days ago   /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B        
<missing>      6 days ago   /bin/sh -c #(nop)  EXPOSE 80                    0B        
<missing>      6 days ago   /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B        
<missing>      6 days ago   /bin/sh -c #(nop) COPY file:09a214a3e07c919a…   4.61kB    
<missing>      6 days ago   /bin/sh -c #(nop) COPY file:0fd5fca330dcd6a7…   1.04kB    
<missing>      6 days ago   /bin/sh -c #(nop) COPY file:0b866ff3fc1ef5b0…   1.96kB    
<missing>      6 days ago   /bin/sh -c #(nop) COPY file:65504f71f5855ca0…   1.2kB     
<missing>      6 days ago   /bin/sh -c set -x     && addgroup --system -…   61.1MB    
<missing>      6 days ago   /bin/sh -c #(nop)  ENV PKG_RELEASE=1~bullseye   0B        
<missing>      6 days ago   /bin/sh -c #(nop)  ENV NJS_VERSION=0.7.2        0B        
<missing>      6 days ago   /bin/sh -c #(nop)  ENV NGINX_VERSION=1.21.6     0B        
<missing>      6 days ago   /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B        
<missing>      6 days ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      6 days ago   /bin/sh -c #(nop) ADD file:90495c24c897ec479…   80.4MB  
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
feb5d9fea6a5   4 months ago   /bin/sh -c #(nop)  CMD ["/hello"]               0B        
<missing>      4 months ago   /bin/sh -c #(nop) COPY file:50563a97010fd7ce…   13.3kB 
```

Conseguimos ver a diferença entre as camadas do hello-word que só tem 2 contra as camadas com mais execuções que são as do nginx

Conseguimos comparar o tamanho, o que está ocupando muito espaço e retirar o que não tem necessidade de ser executado.


## Namespaces para que serve?

Namespaces é a funcionalidade do sistema operacional com que permite que o docker consiga utilizar o kernel separando seu seus processo e outras atividades das atividades da servidor host

É como se ele criasse um compartimento separado e executasse os processos, outro para executar processos de rede outro para processos de storage, tudo com ID separados e separados do kernel do host.

Vamos fazer alguns exemplos para tentar explicar melhor

Para ilustrar vamos criar um Dockerfile já para entendermos as camadas e brincarmos com os processos separados do Docker e do host.

Vamos criar o arquivo que gera a imagem do docker
```shell
vim dockerfile
```
Adicionamos essas duas camadas, uma vai usar a imagem que já baixamos e a segunda vai adicionar o index no caminho /usr/share/nginx/html
```yaml
FROM nginx
COPY index.html /usr/share/nginx/html
```
Agora vamos criar o arquivo index.html
```shell
vim index.html
```
E adicionar essa informação dentro dele, essa informação é a que vai aparecer no navegador.
```html
<!DOCTYPE html>
<html>
<head>
<title>Docker LAB!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Bem vindo ao Docker LAB!</h1>
</body>
</html>
```
Vamos criar a imagem agora para rodar ela em um container
```shell
docker build -t docker-lab .
```
Retorno

```shell
[root@lab-docker ~]# docker build -t docker-lab .
Sending build context to Docker daemon  14.34kB
Step 1/2 : FROM nginx:alpine
 ---> bef258acf10d
Step 2/2 : COPY index.html /usr/share/nginx/html
 ---> 4d0282dcc86d
Successfully built 4d0282dcc86d
Successfully tagged docker-lab:latest
```

O comando build é o que vai gerar a imagem, o -t vai dar um nome uma tag para essa imagem que se chamará docker-lab e o ponto depois do espaço esta se referindo para pegar o dockerfile dentro do diretório que você esta. 
* (isso só funcionará para o arquivo chamado dockerfile que é o default caso queira chamar um dockerfile com outro nome pode ser utilizado o -f (-f é referênte a file) e o nome do arquivo.)

agora para avaliarmos vamos rodar essa nova imagem expondo a porta dela para acessarmos e vermos se está tudo funcionando

```shell
docker run -p 8080:80 -d docker-lab
```
Vamos executar docker run para iniciar o funcionamento deste novo serviço informando o -p (referência porta) para redirecionarmos todo o tráfego que bater na 8080 ser redirecionado para 80 e o -d (referência daemon) vai deixar o container rodando em segundo plano 
* (Temos a possíbilidade também de utilizar no lugar do -d ou até junto com ele o -it (referência -i interaction) e (referência -t terminal) isso permitirá que consigas acessar o container e mexer no conteiner em um terminal bash)

Exemplo

```shell
docker run -p 8080:80 -it docker-lab bash
```
Como aplicamos primeiramente o -d conseguiremos ver ele em execução com o seguinte comando

```shell
docker ps
```
Retorno
```shell
[root@lab-docker ~]# docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS         PORTS                                   NAMES
0d2864ea41e1   docker-lab   "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   condescending_mirzakhani
```
Também temos a interação -a que adicionada podemos ver containers que já finalizaram suas atividades mas ainda estão habilitados a serem ativados novamente.

Com essa informação que temos no docker ps também utilizamos para o id do container para diversas verificações que veremos adiante.

Agora com o container rodando vamos para o próximo passo.

## Entendendo Namespaces

* PID Namespace

para vermos na prática podemos fazer o seguinte teste

Como temos um container rodando com nginx podemos ver os processos de dentro e de fora do container.

Agora seremos apresentados ao comando exec, ele nos permite conectar no container que foi criado pelo run com o -it informado na instrução anterior.
```shell
docker exec -it 0d2864ea41e1 bash
```
Ao entrar nesse container será necessário instalar o serviço que nos mostra os processos e para isso será necessário o seguinte comando

```shell
apt update -y && apt install procps -y
```
após vamos utilizar o comando ps para vermos os processos e os id dos processos que estão rodando.

```shell
ps -ef | grep nginx
```
Retorno
```shell
root@0d2864ea41e1:/# ps -ef | grep nginx
root         1     0  0 01:41 ?        00:00:00 nginx: master process nginx -g daemon off;
nginx       32     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       33     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       34     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       35     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       36     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       37     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       38     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       39     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       40     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       41     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       42     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       43     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       44     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       45     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       46     1  0 01:41 ?        00:00:00 nginx: worker process
nginx       47     1  0 01:41 ?        00:00:00 nginx: worker process
root       413    48  0 01:58 pts/0    00:00:00 grep nginx
```
Comando ps mostra na tela os processos que estão sendo executados no host -e seleciona todos os processos e o -f mostra todas a informações dos processos. o | grep é para filtrar somente processos que tenham o nginx no nome.

Agora vamos executar no host que está executando o docker para verificarmos a diferença dos pids

```shell
ps -ef | grep nginx
```
Retorno
```shell
[root@lab-docker ~]# ps -ef | grep nginx
root     26305 26284  0 20:41 ?        00:00:00 nginx: master process nginx -g daemon off;
101      26370 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26371 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26372 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26373 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26374 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26375 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26376 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26377 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26378 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26379 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26380 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26381 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26382 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26383 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26384 26305  0 20:41 ?        00:00:00 nginx: worker process
101      26385 26305  0 20:41 ?        00:00:00 nginx: worker process
root     26387 25480  0 20:42 pts/1    00:00:00 grep --color=auto nginx
```

Desta forma conseguimos verificar a separação de processos que o docker consegue fazer. esses id de processamento são isolados.

Vamos agora para o próximo namespace

* NET Namespace

Aqui vamos fazer mais uma vez o mesmo processo para verificarmos como é dentro do container e como é fora

Para que seja possível a comunicação entre os containers, é necessário criar dois Net Namespaces diferentes, um responsável pela interface do container (normalmente utilizamos o mesmo nome das interfaces convencionais do Linux, por exemplo, a eth0) e outro responsável por uma interface do host, normalmente chamada de veth* (veth + um identificador aleatório). Essas duas interfaces estão linkadas através da bridge Docker0 no host, que permite a comunicação entre os containers através de roteamento de pacotes.

para realizar esse processo ainda dentro do container que executamos o teste vamos instalar mais uma ferramenta necessária para analisar esse próximo namespace

```shell
apt install iproute2 -y
```
após instalar vamos executar o seguinte comando dentro do container

```shell
ip addr
```
Retorno

```shell
root@0d2864ea41e1:/# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
42: eth0@if43: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```
Verificamos como foi informado na descrição do processo, que é criado dentro do container uma interface eth0 que se comunicará com a bridge e ligará a interface veth que será mostrada a seguir com o seguinte comando

No host que está executando o docker executar o mesmo comando que foi executado dentro do container
```shell
ip addr
```
Retorno
```shell
[root@lab-docker ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:6b:72:e2 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.10/24 brd 192.168.0.255 scope global noprefixroute dynamic eth0
       valid_lft 2986sec preferred_lft 2986sec
    inet6 fe80::902a:c9db:a9dd:159a/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:0b:eb:37:e8 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:bff:feeb:37e8/64 scope link 
       valid_lft forever preferred_lft forever
43: vethc0c0e55@if42: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 96:26:eb:29:c2:26 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::9426:ebff:fe29:c226/64 scope link 
       valid_lft forever preferred_lft forever

```

Dessa forma conseguimos ver duas formas de como os namespaces funcionam agora vamos verificar quais os namespaces estamos utilizando

```shell
lsns
```
Retorno
```shell
[root@lab-docker ~]# lsns
        NS TYPE  NPROCS   PID USER   COMMAND
4026531836 pid      213     1 root   /usr/lib/systemd/systemd --system --deserialize 16
4026531837 user     231     1 root   /usr/lib/systemd/systemd --system --deserialize 16
4026531838 uts      213     1 root   /usr/lib/systemd/systemd --system --deserialize 16
4026531839 ipc      213     1 root   /usr/lib/systemd/systemd --system --deserialize 16
4026531840 mnt      209     1 root   /usr/lib/systemd/systemd --system --deserialize 16
4026531856 mnt        1    88 root   kdevtmpfs
4026531956 net      213     1 root   /usr/lib/systemd/systemd --system --deserialize 16
4026532506 mnt        1   866 chrony /usr/sbin/chronyd
4026532507 mnt        2   913 root   /usr/sbin/NetworkManager --no-daemon
4026532517 mnt       18 26305 root   nginx: master process nginx -g daemon off
4026532518 uts       18 26305 root   nginx: master process nginx -g daemon off
4026532519 ipc       18 26305 root   nginx: master process nginx -g daemon off
4026532520 pid       18 26305 root   nginx: master process nginx -g daemon off
4026532522 net       18 26305 root   nginx: master process nginx -g daemon off
```
Esse comando lista os namespaces que estão em execução e podemos ver vários dos namespaces que falamos durante a apresentação.

O processo do nginx tem namespace de 
* MNT - controla os sistemas de arquivo e volumes de cada namespace nesse caso ele é quem está cuidando das montagens do nginx
* UTS - Responsável por prover isolamento de hostnames, nomes de domínios entre outros.
* IPC - Responsável por prover um SystemV IPC isolado com mensageria, compartilhamento de memória entre outros.
* PID que já vimos que controla os processos isoladamente
* NET que é o responsável por fazer a comunicação de rede.

Assim concluímos a demonstração sobre Namespaces do docker

## Cgroup
* O cgroups é o responsável por permitir a limitação da utilização de recursos do host pelos containers. Com o cgroups você consegue gerenciar a utilização de CPU, memória, dispositivos, I/O, etc

Nesse caminho que vamos acessar agora ficam as configurações do cgroup onde podemos ver os valores setados
```shell
cd /sys/fs/cgroup && ls
```
Retorno

```shell
blkio  cpu  cpuacct  cpu,cpuacct  cpuset  devices  freezer  hugetlb  memory  net_cls  net_cls,net_prio  net_prio  perf_event  pids  systemd
```

Agora vamos fazer um teste no docker e veremos que foi modificado como informado na inicialização do docker

vamos dar um cat no nosso container que estamos utilizando para o teste

```shell
cat /sys/fs/cgroup/cpu/docker/0d2864ea41e1.../cpu.cfs_quota_us
```
Retorno
```shell
[root@lab-docker]# cat /sys/fs/cgroup/cpu/docker/0d2864ea41e1.../cpu.cfs_quota_us 
-1
```

Neste momento vamos parar e subir novamente o container agora com o parâmetro informando esse limite

```shell
docker stop 0d && docker rm $(docker ps -a)
```
Nesse comando vamos parar a imagem que estamos utilizando e em seguida vamos remover qualquer container que esteja parado.

Agora executando...
```shell
docker ps -a
```
Verificaremos que não tem mais nenhum container em execução ou parado.

Vamos executar agora o comando com a limitação que vamos querer impor ao docker
```shell
docker run -p 8080:80 --cpu-quota=20000 -d docker-lab
```
Mesmo comando que utilizamos antes mas agora adicionamos --cpu-quota para esse container em específico.

Retorno
```shell
[root@lab-docker cpu]# docker run -p 8080:80 --cpu-quota=20000 -d docker-lab 
ae58c5c0b8393af447449745134453c5daa8ba127ef607ae857b769f3d2a7150
```
Agora vamos ter um novo hash de container para procurar no caminho

```shell
cat /sys/fs/cgroup/cpu/docker/ae58c5c0b839.../cpu.cfs_quota_us
```
Retorno
```shell
[root@lab-docker cpu]# cat /sys/fs/cgroup/cpu/docker/ae58c5c0b839.../cpu.cfs_quota_us
20000
```
Agora temos nosso container com limitação de cpu

Isso é somente uma demonstração do que se é possível fazer, daria para alterar diretamente nos arquivos, via comandos e muitas outras formas, mas para isso é necessário entender tudo o que pode ser feito e seu impacto depois de feito.

## Netfilter

O Netfilter é o serviço que cuida do iptables e o iptables é muito importante para o funcionamento do docker então vamos fazer o seguinte comando para ver as regras que são configuradas.

```shell
[root@lab-docker cpu]# iptables -t nat -L | grep docker
POST_docker  all  --  anywhere             anywhere            [goto] 
Chain POST_docker (1 references)
POST_docker_log  all  --  anywhere             anywhere            
POST_docker_deny  all  --  anywhere             anywhere            
POST_docker_allow  all  --  anywhere             anywhere            
Chain POST_docker_allow (1 references)
Chain POST_docker_deny (1 references)
Chain POST_docker_log (1 references)
PRE_docker  all  --  anywhere             anywhere            [goto] 
Chain PRE_docker (1 references)
PRE_docker_log  all  --  anywhere             anywhere            
PRE_docker_deny  all  --  anywhere             anywhere            
PRE_docker_allow  all  --  anywhere             anywhere            
Chain PRE_docker_allow (1 references)
Chain PRE_docker_deny (1 references)
Chain PRE_docker_log (1 references)
```
Desta forma conseguimos ver várias regras criadas pelo Docker fazendo com que seja possível a comunicação entre os container e para a internet.

## Utilizando o Docker
Agora vamos para algumas formas de utilização do docker e comandos uteis


* A primeira forma que vou mencionar aqui é o docker multi stage muito útil para desenvolvimento de imagens e que facilita para não adicionar dependências desnecessárias em uma imagem.

Esse formato descrito abaixo é só um exemplo de como utilizar
```yaml
# syntax=docker/dockerfile:1
FROM node:14-slim AS builder
WORKDIR /app
RUN yarn install
RUN yarn build
RUN yarn lint  

FROM httpd:alpine  
RUN apk update --no-cache \
 && apk add --no-cache shadow
WORKDIR /usr/local/apache2/htdocs
COPY --from=builder dist ./usr/local/apache2/htdocs
CMD [httpd-foreground] 
```

Desta forma na primeira interação o docker ele vai gerar o build do frontend e na segunda interação ele só vai copiar essa pasta dist que foi gerado pelo build para dentro de um container alpine com apache fazendo assim que só tenha a camada do alpine com os arquivos necessários, como eu não tinha um código de exemplo essa demonstração é meramente ilustrativa.

* comandos docker
Comando mais importante que facilita a vida.

```shell
[root@lab-docker cpu]# docker --help

Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/root/.docker")
  -c, --context string     Name of the context to use to connect to the daemon (overrides DOCKER_HOST env var and default context set with "docker context use")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/root/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/root/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/root/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  app*        Docker App (Docker Inc., v0.9.1-beta3)
  builder     Manage builds
  buildx*     Docker Buildx (Docker Inc., v0.7.1-docker)
  config      Manage Docker configs
  container   Manage containers
  context     Manage contexts
  image       Manage images
  manifest    Manage Docker image manifests and manifest lists
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  scan*       Docker Scan (Docker Inc., v0.12.0)
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.
```

Executando um container com o docker e utilizando tudo que é necessário

```shell
docker run -d --rm --name nginx-teste -v caminho_host:/caminho_container -p 8080:80 -e "Variavel=Valor" docker-lab:latest nginx
```

* -d - Daemon deixa rodando o processo e desconecta da container
* --rm - Remove o container automaticamente quando ela é parada
* -v - Volume persistente, quando o container for removido os dados ainda continuaram nesse diretório
* -p - Porta externa > para porta interna expõem o container para ser acessível pelo host do docker
* -e - Variável que vai aparecer dentro do container com chave e valor.
* docker-lab:latest - é a imagem que deseja chamar para rodar o container
* nginx é o comando necessário para que o processo se inicie dentro do container.

Com esses comandos já é possível subir um container com praticamente tudo que é necessário.
