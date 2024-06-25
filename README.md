# Instalação do novoga


### Criando a VM no Proxmox

1. **Geral:** Configurações padrões
2. **SO:** Selecionar imagem ISO: Debian-12.5.0. Restante das configurações padrões.
3. **Sistema:** Deixar selecionado a seguinte checkbox: Agente Qemu. Restante das configurações padrões.
4. **Discos:** Configurações padrões. Reduzir espaço de disco para 20GB
5. **CPU:** Configurações padrões.
6. **Memória:** Configurações padrões.
7. **Rede:** Definir tag da VLAN como 200. Restante das configurações padrões.

### Instalação do Debian

1. **Selecionar a linguagem e localidade:** ptBr (58) e Brasil

2. **Configuração do teclado:** ptBr (11)

3. **Hostname:** novo (ou qualquer outro nome)

4. **Domínio:** ebserhnet.ebserh.gov.br

5. **Senha do root:**
   
6. **Nome de um usuário (obrigatório):**
   
7. **Login do usuário:**
   
8. **Senha do usuário:**

9. **Configuração do relógio:** MT (14)

10. **Partição de disco:** 
    - Usar disco inteiro
    - Selecionar o disco
    - Todos os arquivos em uma partição
    - Finalizar e escrever (12)
    - Sim
    - Não ler mídias adicionais

11. **Gerenciador de pacotes:**
    - Brasil (7)
    - deb.debian.org
    - Deixar em branco a próxima opção

12. **Seleção de software:**
    - Deixar apenas os dois últimos marcados (11 12) (servidor SSH e utilitário de sistema padrão)
    - Marcar sim para a opção que irá aparecer durante essa última etapa
    - Selecionar o dispositivo da instalação (/dev/sda...)
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

#---------------------------------------------------------------------
#Criar um arquivo chamado docker-compose.yml e colocando os dados abaixo: 
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

#Executando docker-compose:
sudo docker-compose up -d


#################### escrever separadamente na documentação
sudo docker-compose exec mysqldb sh -c  'mysql -uroot -p'

GRANT ALL ON novosga2.* TO 'novosga'@'%' IDENTIFIED BY 'MySQL_App_P4ssW0rd';
quit
```

