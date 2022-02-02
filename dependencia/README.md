# Instalações para o LAB
Para este tutorial irei utilizar a ferramenta boxes do fedora, mas podem utilizar o virtualizador de sua preferência.

## Download

* [ Link Download VirtualBox ](https://www.virtualbox.org/wiki/Downloads)

* [ Link Download Centos7 ](http://mirror.ufscar.br/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-Minimal-2009.iso)

### Processo de instalação default da Virtualização

Pode realizar a instalação tudo de forma padrão até conseguir o acesso ao sistema operacional via linha de comando. (Como o intuito desse laboratório é o Docker não explicarei como realizar a instalação destas ferramentas)

Para este laboratório será utilizado o CentOS por ser um sistema operacional mais utilizado geralmente em servidores.

* Um ponto importante seria deixar a máquina virtual em modo bridge para que consigamos acessar ela da nossa máquina principal via SSH, vai facilitar copiar e colar os passos do tutorial

### Configuração do CentOS

Vamos fazer algumas instalações de dependências que iremos precisar para o funcionamento do laboratório, já que a versão no link acima é a versão mínima do centos para ficar mais leve e instalarmos apenas o que iremos utilizar. 

* Antes de tudo vamos pegar o ip que a máquina virtual pegou para acessarmos este novo servidor via SSH
```yaml
ip addr
```
Retorno

```yaml
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:6b:72:e2 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.10/24 brd 192.168.122.255 scope global noprefixroute dynamic eth0
       valid_lft 2840sec preferred_lft 2840sec
    inet6 fe80::902a:c9db:a9dd:159a/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever

```
Teremos aqui o ip para acessarmos via SSH o servidor

```yaml
ssh root@192.168.1.10
```
Retorno

```yaml
The authenticity of host '192.168.1.10 (192.168.1.10)' can't be established.
ED25519 key fingerprint is SHA256:d5Be3qv4vYHBCx4RlAXcUCcKNq3h1FJMTYUoRqRlukQ.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.10' (ED25519) to the list of known hosts.
root@192.168.1.10's password: 
Last login: Tue Feb  1 15:44:45 2022
[root@lab-docker ~]# 
```

é preciso aceitar para ser confirmar a conexão com o servidor e ser adicionado no know_hosts que essa maquina conhece esse servidor e após informar a senha que foi cadastrada na instalação do sistema operacional

* Vamos atualizar para sempre buscar os pacotes mais atuais e instalar o vim

```yaml
yum update -y && yum install vim -y 
```
* vim - para editarmos textos via linha de comando

### Instalação do Docker

Primeiro passo é ir diretamente à documentação do docker, sempre buscar a instalação diretamente na documentação do docker.


* [ Doc Install Docker CentOS ](https://docs.docker.com/engine/install/centos/)

Como estamos utilizando uma versão limpa do CentOS vamos pular diretamente para a parte de configuração das dependências para o docker (Caso tenha pego um servidor que não tenha conhecimento se já teve uma instalação do docker faça todos os passos da documentação do docker.)

* Instalando dependências

```yaml
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
Primeiro comando vai instalar utilitários do yum

Segundo vai adicionar o repositório do docker no caminho /etc/yum.repos.d/docker-ce.repo. Para quando for chamar a instalação do docker ele conseguir encontrar nesse caminho os links de instalação do docker.

```yaml
sudo yum install -y docker-ce docker-ce-cli containerd.io
```

Este comando vai instalar o docker community edition e as ferramentas necessárias para seu pleno funcionamento.

Após instalado vamos iniciar o serviço e ativar ele permanentemente na inicialização do docker

```yaml
systemctl start docker && systemctl enable docker
```
Retorno

```yaml
[root@lab-docker yum.repos.d]# systemctl start docker && systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
```

Informa que foi criado o link simbólico para inicializar com o sistema.

Agora vamos testar o funcionamento do docker 

```yaml
sudo docker run hello-world
```
Retorno

```yaml
[root@lab-docker yum.repos.d]# sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete 
Digest: sha256:507ecde44b8eb741278274653120c2bf793b174c06ff4eaa672b713b3263477b
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```
Se chegamos até aqui o Docker já está instalado e funcionando.

* Lembrando que todos esses passos foram retirados da documentação oficial, é sempre importante buscar informação na documentação oficial.