# Novoga  

## Instalação

### Criando a VM no Proxmox  
  
Geral: Configs padrões   
  
SO: Imagem ISO -> Debian-12.5.0, resto padrão  
  
Sistema: selecionar Agente Qemu, resto padrão 
  
Discos: Configs padrões, pode diminuir o espaço de disco (20GB)  
  
CPU: Configs padrões  
  
Memória: Configs padrões
  
Rede: Tag da VLAN -> 200, resto padrão  
  
### Instalando o Debian na VM  
Selecionar a linguagem: ptBr (58) -> Localidade   
  
Configuração do teclado: ptBr (11)  
  
Hostname: novo (ou qualquer outro nome)  
  
Domínio: ebserhnet.ebserh.gov.br  
  
Senha do root:   
Nome de um usuário (obrigatório):    
Login do usuário:  
Senha do usuário:  
  
Configuração do relógio: MT (14)   
  
Partição de disco: usar disco inteiro -> Selecionar o disco -> Todos os arquivos em uma partição -> Finalizar e escrever (12) -> Sim ->  não ler mídias adicionais   
  
Gerenciador de pacotes: Brasil (7) -> deb.debian.org -> deixar em branco a próxima opção   
  
*Seleção de software -> deixar apenas os dois últimos marcados (11 12)(servidor SSH e utilitário de sistema padrão) -> marcar sim para a opção que irá aparecer durante essa ultima etapa e selecionar o dispositivo da instalação (/dev/sda...)   

### Rodando o script

```bash
#!/bin/bash
echo "Digite sua senha:"
read -s suasenha
senha=$suasenha
apt-get install sudo -y

sudo apt install docker.io docker-compose <<EOF
y
EOF
sudo systemctl enable --now docker docker.socket containerd
#Conteiner simples
# aplicação base
#sudo docker run --rm -p 80:80 -e DATABASE_URL="mysql://novosga:MySQL_App_P4ssW0rd@mysqldb:3306/novosga2?charset=utf8mb4&serverVersion=5.7" novosga/novosga:2.1


# serviço mercure para troca de mensagens
#docker run --rm -it -p 3000:3000 -e 'SERVER_NAME=:3000' -e 'MERCURE_PUBLISHER_JWT_KEY=!ChangeMe!' -e 'MERCURE_EXTRA_DIRECTIVES=anonymous 1; cors_origins *' novosga/mercure:v0.11
docker run --rm -d -p 3000:3000 -e 'SERVER_NAME=:3000' -e 'MERCURE_PUBLISHER_JWT_KEY=!ChangeMe!' -e 'MERCURE_EXTRA_DIRECTIVES=anonymous 1; cors_origins *' novosga/mercure:v0.11

#---------------------------------------------------------------------
#Criar um arquivo chamado docker-compose.yml e colocar esses dados: 
cat <<EOF > docker-compose.yml
version: '2'

services:
  novosga:
    image: novosga/novosga:2.1
    restart: always
    depends_on:
      - mysqldb
    ports:
      - "80:80"
      - "2020:2020"
    environment:
      APP_ENV: 'prod'
      # database connection
      DATABASE_URL: 'mysql://novosga:MySQL_App_P4ssW0rd@mysqldb:3306/novosga2?charset=utf8mb4&serverVersion=5.7'
      # default admin user
      NOVOSGA_ADMIN_USERNAME: 'admin'
      NOVOSGA_ADMIN_PASSWORD: '$suasenha'
      NOVOSGA_ADMIN_FIRSTNAME: 'Administrador'
      NOVOSGA_ADMIN_LASTNAME: 'Global'
      # default unity
      NOVOSGA_UNITY_NAME: 'Minha unidade'
      NOVOSGA_UNITY_CODE: 'U01'
      # default no-priority
      NOVOSGA_NOPRIORITY_NAME: 'Normal'
      NOVOSGA_NOPRIORITY_DESCRIPTION: 'Atendimento normal'
      # default priority
      NOVOSGA_PRIORITY_NAME: 'Prioridade'
      NOVOSGA_PRIORITY_DESCRIPTION: 'Atendimento prioritário'
      # default place
      NOVOSGA_PLACE_NAME: 'Guichê'
      # Set TimeZone and locale
      TZ: 'America/Cuiaba'
      LANGUAGE: 'pt_BR'
      # Endereço Mercure para publicar mensagem (onde "mercure" é o nome do host)
      # esse endereço será chamado internamente via o PHP
      MERCURE_PUBLIC_URL: http://mercure:3000/.well-known/mercure
      # Endereço Mercure para consumir mensagem
      # esse endereço será chamado via o navegador web
      MERCURE_CONSUMER_URL: http://127.0.0.1:3000/.well-known/mercure
  mercure:
    image: novosga/mercure:v0.11
    restart: always
    ports:
      - "3000:3000"
    environment:
      # same value from ports
      SERVER_NAME: ":3000"
      # default publish key, must be changed
      MERCURE_PUBLISHER_JWT_KEY: "!ChangeMe!"
      MERCURE_EXTRA_DIRECTIVES:  "anonymous 1; cors_origins *"
  mysqldb:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_USER: 'novosga'
      MYSQL_DATABASE: 'novosga2'
      MYSQL_ROOT_PASSWORD: '$suasenha'
      MYSQL_PASSWORD: '$suasenha'
      # Set TimeZone
      TZ: 'America/Cuiaba'
EOF
#---------------------------------------------------------------------

sudo docker-compose up -d
```

