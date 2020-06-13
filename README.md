# Comandos para executar em um servidor utilizando SSH e aplicação Node com sucrase:

## Conexão SSH para usuário root Digital Ocean

- No terminal executar o comando:

```bash
ssh root@IP
```

- A Primeira coisa a fazer quando iniciar um servidor é realizar atualizações:

```bash
apt update
```

e

```bash
apt upgrade
```

---

## Adicionar novo usuário para não ter que utilizar sempre no root:

```bash
adduser NOMEUSUARIO
```

- Inserir uma senha
- Repetir senha

- depois inserir ou não as informações solicitadas

- Agora é necessário dar poderes de admin a esse usuário:

```bash
usermod -aG sudo NOMEUSUARIO
```

- Permitir que esse usuário seja acessado diretamente pelo ssh:

- Criar o diretorio necessario

```bash
cd /home/NOMEUSUARIO
mkdir .ssh
```

- Copiar o arquivo necessário

```bash
cp ~/.ssh/authorized_keys /home/NOMEUSUARIO/.ssh/
```

- Para verificar o dono dessa pasta utilizamos o comando dentro da pasta .ssh:

```bash
ls -la
```

- Precisamos alterar o dono do arquivo:

```bash
chown NOMEUSUARIO:NOMEUSUARIO authorized_keys
```

- Depois executar para conferir se deu certo:

```bash
ls -la
```

- Depois só conectar ao ssh utilizando o comando:

```bash
ssh NOMEUSUARIO@IP
```

---

## Instalar o node:

- Acessar o site do nodejs
- Ir em outros downloads
- acessar o link: `Installing Node.js via package manager`
- Acessar a opção: `Debian and Ubuntu based Linux distributions, Enterprise Linux/Fedora and Snap packages`
- Procurar por `Snap packages`
- Acessar o link para `Node.js binary distributions` [NodeSource Node.js Binary Distributions](https://github.com/nodesource/distributions/blob/master/README.md)

- Procurar a versão do lts

- E Executar a versão correspondente:

```bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```

```bash
sudo apt-get install -y nodejs
```

- O servidor já está pronto com o mínimo necessário

---

- Instalando a aplicação do git para o servidor...
- Podemos colocar dentro da pasta home do usuário que estamos acessando e clonar para pasta app

- Acessa a pasta

- Executa

```bash
npm install
```

- Outra coisa que pode ser configurada é o `.env`

- Uma das coisas importantes é colocar o `NODE_ENV` para `production`

---

## Docker

- O interessante ao escolher o tipo de instancia é obter no marketplace uma com docker já instalado, do contrário é necessário realizar a instalação do mesmo.

- Com o docker instalado é necessário realizar algumas configurações para não precisar executar utilizando o sudo na frente em usuários não root

- Para isso podemos buscar no google por `docker post installation for Linux` o qual podemos acessar essa url: [Post-installation steps for Linux](https://docs.docker.com/engine/install/linux-postinstall/)

- Ali traz mais detalhes do que fazer...

- podemos executar então os comandos:

```bash
sudo groupadd docker
```

```bash
sudo usermod -aG docker $USER
```

- Depois só sair da instancia e acessar novmante.

- Agora podemos executar normalmente os comandos docker...


- Instalar o `postgres`:

```bash
docker run --name postgres -e POSTGRES_PASSWORD=MINHA_SENHA_AQUI -p 5432:5432 -d -t postgress
```

- o `-e POSTGRES_PASSWORD` informamos que criamos uma veriavel de ambiente com o seu valor

- Instalar o mongo:

```bash
docker run --name mongo -p 27017:27017 -d -t mongo
```

- Instalar o redis:

```bash
docker run --name redis -p 6379:6379 -d -t redis:alpine
```

- Para criar a base de dados no postgres utilizando linha de comando no docker podemos utilizar:

```bash
docker exec -i -t NOME_DO_CONTAINER /bin/sh
```

- E agora entramos na distribuiçao do linux do docker do container:

- No caso do postgres precisamos acessar utilizando o usuário do postgres:

```bash
su postgres
```

- Estando ali dentro podemos criar a base de dados:

```bash
CREATE DATABASE nome_da_base_de_dados;
```

- Para sair execute:

```bash
\q
```

```bash
exit
exit
```

- Importante executar as migrations

---


## Sucrase:

- Não é interessante utilizar o nodemon e o sucrase como utilizamos no ambiente de desenvolvimento no ambiente de produção


- No arquivo `package.json` em `"scripts"` adicionamos mais dois comandos:

```json
"build": "sucrase ./src -d ./dist --transforms imports",
"start": "node dist/server.js",
```

- Mais detalhes sobre o transforms do sucrase [Transforms](https://github.com/alangpierce/sucrase#transforms)

- No comando do `sucrase ./src -d ./dist --transforms imports` informamos a pasta de origem, no caso ali é a `src` e a pasta destino onde o sucrase gerará o build

---

## Firewall

- Para liberar determinada porta utilizamos o comando:

```bash
sudo ufw allow PORTA
```

---

## SSH

- Manter o ssh vivo por mais tempo para não ficar caindo devido a inatividade, seguir isso: [Keep my SSH session alive](https://www.digitalocean.com/community/questions/keep-my-ssh-session-alive)

- Editar o arquivo:

```bash
sudo nano /etc/ssh/sshd_config
```

- No final do arquivo podemos inserir:

```txt
ClientAliveInterval 30
TCPKeepAlive yes
ClientAliveCountMax 99999
```

- Por fim restartamos o serviço do ssh executando:

```bash
service sshd restart
```

- Depois precisamos sair e acessar novamente o ssh


## Matar processo do node

- Parar o servidor node:

```bash
lsof -i :PORTA_ONDE_ESTA_RODANDO
```

- Pega o PID do processo e para matar o processo executa:

```bash
kill -9 NR_DO_PID
```

## NGINX

- Para colocar a aplicação de forma segura na porta 80 e permitir a utilização de subdominios utilizamos o nginx.

- Instalar o NGINX:

```bash
sudo apt install nginx
```

- Liberar no firewall a porta 80:

```bash
sudo ufw allow 80
```

- Se acessarmos o ip no browser ele já irá mostrar a msg do ngnix ok

- Para ajustarmos o ngnix para escolher qual pasta acessar quando o usuário acessar o nosso site/servidor precisamos configurar:

```bash
sudo nano /etc/nginx/sites-available/default
```

- podemos remover os comentários

- remover a parte de root do index

- agora em `location / ` ou seja rota raiz, no caso se fosse `location /api` seria referente a `/api`:

```txt
location / {
  proxy_pass http://localhost:3333;
  proxy_http_version 1.1; 
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection 'upgrade';
  proxy_set_header Host $host;
  proxy_cache_bypass $http_upgrade;
}
```

- Para surtir efeito precisamos dar restart do serviço:

```bash
sudo service nginx restart
```

- Para verificar se os arquivos de configuração estão ok podemos executar o comando:

```bash
sudo nginx -t
```

- Por fim podemos acessar a api normalmente pela porta 80


## Mantendo a aplicação sempre viva o com reinicio automatico

- Inicialmente instale o pm2:

```bash
sudo npm install -g pm2
```

- Depois startamos para aplicação

```bash
pm2 start dist/server.js
```

- Para verificar o que está rodando executamos o comando:

```bash
pm2 list
```

- Para startar o pm2 quando o servidor reiniciar precisamos fazer o seguinte:

```bash
pm2 startup systemd
```

- Seguir as instruçoes de copiar o comando e executar...

- Toda vez que executarmos o `pm2 start ...` só executar o `pm2 save` depois

- Para verificar o log da aplicação podeoms executar o `pm2 monit`