# Atividade AWS - Docker - Compass

## Sobre o Projeto

Este trabalho tem como objetivo consolidar os conhecimentos em Docker e serviços AWS, por meio da instalação, configuração e implantação de uma aplicação WordPress em um ambiente baseado em EC2.

<p align="center">
  <img src="https://github.com/user-attachments/assets/eda31610-d0d1-421b-a9d8-6eabf5d78d92" alt="Descrição da imagem" width="600">
</p>

### Principais Objetivos
- Instalação e Configuração:
  - Instalar e configurar Docker na instância EC2
  - Realizar instalação automatizada utilizar arquivo user_data.sh
- Deploy da Aplicação WordPress
  - Implementar o container da aplicação utilizando Docker Compose
  - Configurar um banco de dados RDS MySQL para armazenar os dados do WordPress
- Configuração de Armazenamento e Balanceamento de Carga
  - Utilizar EFS (Elastic File System) para armazenar arquivos estáticos do WordPress.
  - Configurar um Load Balancer (Classic Load Balancer da AWS) para distribuir o tráfego de entrada.

### Serviços AWS utilizados no Projeto
- VPC
- RDS (Banco de Dados MySQL)
- EFS
- Instâncias EC2
- Load Balancer
- Auto Scaling

### Ferramentas
- AWS
- Shell Script
- Linux
- Docker

## Início

O primeiro passo do nosso projeto, é a criação de uma VPC.

- Bloco CIDR IPv4: 10.0.0.0/16
- Número de Zonas de Disponibilidade (AZs): 2
- Sub-redes: 2 públicas e 2 privadas
- Gateway NAT: 1 por AZ

## Grupo de Segurança
Para o projeto, será necessário criar Grupos de segurança onde será definido regras de entrada e saida.

- sgGroup-loadbalancer
  - HTTP / HTTPS => IPV4
- sgGroup-ec2
  - HTTP / HTTPS => Load Balancer
  - SSH => Qualquer IP
- sgGroup-rds
  - MySQL/Aurora => sgGroup-ec2
- sgGroup-efs
  - NFS => sgGroup-ec2

## RDS
O Amazon RDS (Relational Database Service) facilita a configuração, manutenção e escalabilidade de bancos de dados relacionais. Para aumentar a segurança, é essencial utilizar grupos de sub-redes em sub-redes privadas, impedindo o acesso direto à internet e restringindo conexões apenas a instâncias autorizadas. Por esse motivo, o primeiro passo será a criação do grupo de sub-redes privadas.

### Grupo de Sub-redes Privadas
- Vá em serviço RDS e acesse a aba "Grupos de sub-redes"
- Clicar em Criar Grupo de sub-redes
- Informações
  - Nome do Grupo: ___________
  - Descrição: _____________
  - VPC: Selecione a VPC que você criou
- Selecionar as zonas de disponibilidas, em seguida, selecionar sub-redes privadas
- Criar Grupo

### Configurações RDS
- Tipo de banco de dados: MySQL (Nível gratuito).
- Preencher Identificador da instância
- Preencher nome do usuário Principal
- Senha
- Selecionar instância: db.t3.micro
- Desative Backup e Cripografia para testes
- Selecionar VPC Criada
- Selecionar Grupo de sub-redes já criado
- Não permitir acesso público
- Adicionar Grupo de Segurança: sgGroup-rds
- Nome do Banco de dados inicial: wordpress
- Desmarcar escalabilidade automática de armazenamento

Ao criar o RDS, será gerado um IP, salve o IP para acessar o banco para adicionar no nosso arquivo user_data.sh

### EFS

- Nome: meuEFS
- Selecionar VPC criada
- Zonas de disponibilidade: selecionar sub-redes privadas 1 e 2
- Selecionar grupo de segurança: sgGroup-efs

1. Após a criação, você vai acessar o comando de Anexar e "Usando o cliente do NFS"
2. Você vai ter que copiar e salvar o comando de montagem do sistema de arquivo Amazon EFS
```
sudo mkdir -p /mnt/efs

sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport <id>:/ /mnt/efs
```
































