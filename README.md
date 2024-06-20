# Novoga
## Instalação

###Criando a VM no Proxmox
Geral: Configs padrões 
SO: Imagem ISO -> Debian-12.5.0, resto padrão
Sistema: selecionar Agente Qemu, resto padrão
Discos: Configs padrões, pode diminuir o espaço de disco (20GB)
CPU: Configs padrões 
Memória: Configs padrões
Rede: Tag da VLAN -> 200, resto padrão

###Instalando o Debian na VM
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

