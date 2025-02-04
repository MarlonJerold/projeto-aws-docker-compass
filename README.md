# Atividade AWS - Docker - Compass

## Sobre o Projeto

Este trabalho tem como objetivo consolidar os conhecimentos em Docker e serviços AWS, por meio da instalação, configuração e implantação de uma aplicação WordPress em um ambiente baseado em EC2.

<p align="center">
  <img src="https://github.com/user-attachments/assets/eda31610-d0d1-421b-a9d8-6eabf5d78d92" alt="Descrição da imagem" width="600">
</p>

# Sumário

1. [Sobre o Projeto](#sobre-o-projeto)
2. [Principais Objetivos](#principais-objetivos)
3. [Serviços AWS Utilizados no Projeto](#serviços-aws-utilizados-no-projeto)
4. [Ferramentas](#ferramentas)
5. [Início](#início)
6. [Grupo de Segurança](#grupo-de-segurança)
7. [RDS](#rds)
   - 7.1. [Grupo de Sub-redes Privadas](#grupo-de-sub-redes-privadas)
   - 7.2. [Configurações RDS](#configurações-rds)
8. [EFS](#efs)
9. [EC2](#ec2)
10. [Load Balancer](#load-balancer)
11. [Auto Scaling](#auto-scaling)
12. [Fim](#fim)

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

### EC2

- Nome e tags: Seguir o padrão da equipe.
- Sistema operacional: Ubuntu.
- Tipo de instância: Padrão.
- Par de chaves: Criar ou reutilizar um existente.
- Sub-redes:
  - Instância 1: Sub-rede privada 1.
  - Instância 2: Sub-rede privada 2.
  - Atribuir IP público automaticamente: Habilitado.
- Grupo de segurança: sgGroup-ec2

Em Configurações avançadas, adicione o user_data.sh.

### Load Balancer
- Tipo: Classic Load Balancer.
- Nome: MyLoadBalancer.
- Mapeamento de rede: Sub-redes públicas.
- Grupo de segurança: sgGroup-loadbalancer
- Caminho de ping: /wp-admin/install.php (espera-se retorno com status 200).
- Selecionar as duas instâncias que criamos privadas que criamos no tópico de EC2

### Auto Scaling
Modelo de Execução (Template):
- Tipo de instância: t2.micro
- Tags e User Data: Mesmos das instâncias EC2 anteriores
- Zonas de disponibilidade: Sub-redes privadas
- Integração: Load Balancer existente
- Demais configurações: Padrão

Após configurar o Auto Scaling, uma nova instância será criada automaticamente, confirmando que o processo foi concluído com sucesso

### Fim
Acesse o DNS do Load Balancer para se conectar ao projeto agora






















