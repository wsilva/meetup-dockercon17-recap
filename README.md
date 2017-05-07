# Demos apresentadas no 12º Meetup Docker SP

## Multi-Stage Build

Em uma pasta de preferência vamos clonar o repositório Dockercraft, mas não o oficial mas sim o fork feito pelo Victor Vieux onde temos o exemplo com o novo Dockerfile.

```bash
$ git clone https://github.com/vieux/dockercraft.git
```

Vamos acessar a pasta clonada

```bash
$ cd dockercraft
```

Vamos construir uma imagem do dockercraft como fazíamos anteriormente, neste caso chamei a imagem de dockercraft-single:

```bash
$ docker build -t dockercraft-single .
```

Vamos mudar nosso branch para o branch onde o Dockerfile tem as diretivas de multi-stage:

```bash
$ git checkout -b dockerfile origin/dockerfile
```

Vamos construir a imagem usando o Dockerfile com multi stage build

```bash
$ docker build -t dockercraft-multi .
```

Vamos comparar os tamanhos

```bash
$ docker image ls | grep dockercraft
dockercraft-single   latest   70a80029d369   1 days ago   838MB
dockercraft-multi    latest   7caf75d46ed2   1 days ago   158MB
``` 

Legal mas essa imagem não temos o cuberite nem os fontes do docker, se eu precisar entrar no container e debugar ou até precisar instalar uma nova versão desses componentes?
Basta você informar qual stage onde você quer que o build pare:

```bash
$ docker build --tag dockercraft-dev --target dockercraft .
```

Comparando as imagens:

```bash
$ docker image ls | grep dockercraft
dockercraft-single   latest   70a80029d369   1 hour ago   838MB
dockercraft-multi    latest   7caf75d46ed2   1 hour ago   158MB
dockercraft-dev      latest   0dc2e33ad89c   1 hour ago   728MB
``` 

## Linuxkit / Moby

Para rodar esse demo temos que ter Go instalado e configurado na máquina. Para instalação do Go recomendamos as instruções da página oficial https://golang.org/doc/install#install

Vamos construir os binários do linuxkit e do moby. Em uma pasta qualquer do sistema vamos clonar o código fonte e compilar os binários.

```bash
git clone https://github.com/linuxkit/linuxkit
cd linuxkit
make
```

Vamos colocar os binários na pasta do sistema.

```bash
sudo cp ./bin/moby /usr/local/bin  
sudo cp ./bin/linuxkit /usr/local/bin 
```

Voltando a pasta de nossas demos vamos construir uma máquina linux com base no exemplo do redis-os.yml (similar aos exemplos que temos em https://github.com/linuxkit/linuxkit/tree/master/examples):

```bash
moby build ./redis-os.yml
```

Vamos rodar a máquina criada. No OSX podemos usar o `hyperkit`:

```bash
linuxkit run hyperkit -ip 192.168.65.107 redis-os
```

No Linux podemos utilizar o `qemu`:

```bash
linuxkit run hyperkit -ip 192.168.65.107 redis-os
```

Podemos espiar quais processos estão rodando nessa máquina.

```bash
/ # pstree
init-+-containerd
     |-containers-+-runc---dhcpcd
     |            |-runc---redis-server
     |            `-runc---tini---rngd
     |-sh---pstree
     `-sh
/ #
```

Como inicializamos utilizando a rede do docker preciso rodar um container na mesma rede para poder mandar comandos.

```bash
$ docker run --rm -it alpine sh
```

Dentro do container utilizamos o netcat (nc) para conectarmos no redis-os que acabamos de subir e enviar comandos.

```bash
/ # nc 192.168.65.107 6379
ping
+PONG

set foo bar
+OK

get foo
$3
bar

keys *
*1
$3
foo
```

Para desligar a máquina criada utilizamos o comando `halt`.

Se tiver o VMWare instalado podemos utilizá-lo também:

```bash
linuxkit run vmware redis-os
```

Atenção. Esses repositórios estão em constantes mudanças, por exemplo há algumas semanas não era ainda possível rodar a máquina criada com o linuxkit, utilizávamos o próprio moby. 

Baixe o repositório do linuxkit e recompile com make para gerar novos binários sempre que quiser utilizar as versões mais recentes do moby e do linuxkit.

Para listar todos os hypervisors disponíveis rode o comando:

```bash
linuxkit run --help
USAGE: linuxkit run [backend] [options] [prefix]

'backend' specifies the run backend.
If not specified the platform specific default will be used
Supported backends are (default platform in brackets):
  gcp
  hyperkit [macOS]
  qemu [linux]
  vmware
  packet

'options' are the backend specific options.
See 'linuxkit run [backend] --help' for details.

'prefix' specifies the path to the VM image.
It defaults to './image'.
```