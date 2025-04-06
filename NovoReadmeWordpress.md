# Infraestrutura AWS para WordPress: Escalabilidade e Alta Disponibilidade

![image](https://github.com/user-attachments/assets/1c5aa93b-7461-454f-933a-a90ccda5ab49)


## Descrição do Projeto

Este projeto tem como objetivo implementar uma infraestrutura escalável e resiliente na Amazon Web Services (AWS) para efetuar o deploy de uma aplicação WordPress. A arquitetura consiste em duas instâncias EC2 localizadas em zonas diferentes, ambas conectadas a um banco de dados RDS e suportadas por um Load Balancer Classic. Utilizando serviços como EC2, RDS, EFS, Auto Scaling e Load Balancer, a infraestrutura é projetada para suportar picos de tráfego, garantindo alta disponibilidade e desempenho para a aplicação WordPress.

## Objetivo

O objetivo deste projeto é criar uma arquitetura de nuvem que permita o deploy eficiente de uma aplicação WordPress, garantindo escalabilidade automática para lidar com variações na carga de trabalho. Além disso, o projeto visa proporcionar um ambiente seguro e eficiente para o armazenamento de dados e arquivos, utilizando os serviços da AWS.

## Arquitetura

A arquitetura do projeto é composta pelos seguintes componentes:

- **VPC (Virtual Private Cloud)**: Uma rede isolada onde todos os recursos da AWS serão criados.
- **Sub-redes**: Duas sub-redes públicas para as instâncias EC2 e duas sub-redes privadas para o banco de dados RDS.
- **EC2 (Elastic Compute Cloud)**: Duas instâncias que executam a aplicação web, localizadas em zonas diferentes para garantir alta disponibilidade.
- **RDS (Relational Database Service)**: Um banco de dados gerenciado que armazena os dados da aplicação, acessado por ambas as instâncias EC2.
- **EFS (Elastic File System)**: Um sistema de arquivos escalável que permite o compartilhamento de arquivos entre as instâncias EC2. 
- **Auto Scaling**: Um grupo que ajusta automaticamente o número de instâncias EC2 com base na demanda.
- **Load Balancer**: Um balanceador de carga que distribui o tráfego entre as instâncias EC2, garantindo alta disponibilidade.

## Pré-requisitos

Antes de iniciar a implementação, certifique-se de que você possui:

- Uma conta na AWS com permissões adequadas para criar e gerenciar os recursos necessários para execução desta aplicação.
- Conhecimento básico em AWS.

## Tecnologias Utilizadas

- **Amazon Web Services (AWS)**: Plataforma de nuvem que fornece os serviços necessários para a implementação da infraestrutura.
- **EC2 (Elastic Compute Cloud)**: Para executar instâncias de servidores.
- **RDS (Relational Database Service)**: Para gerenciar o banco de dados.
- **EFS (Elastic File System)**: Para armazenamento de arquivos.
- **Auto Scaling**: Para escalabilidade automática.
- **Application Load Balancer**: Para balanceamento de carga.

## Funcionalidades

- **Escalabilidade Automática**: O Auto Scaling ajusta automaticamente o número de instâncias EC2 com base na carga de trabalho.
- **Balanceamento de Carga**: O Load Balancer distribui o tráfego entre as instâncias EC2, garantindo que nenhuma instância fique sobrecarregada.
- **Armazenamento Persistente**: O EFS fornece um sistema de arquivos compartilhado que pode ser acessado por várias instâncias EC2.
- **Banco de Dados Gerenciado**: O RDS oferece um banco de dados relacional gerenciado, facilitando a configuração, operação e escalabilidade.
- **Segurança**: A infraestrutura é configurada com grupos de segurança para controlar o acesso aos recursos.

# Instalação da Infraestrutura na AWS

## Parte 1: Configuração Inicial

### Etapa 1. Criação de uma VPC
- Acesse o Console AWS e vá para a seção **VPC**. 
- Clique em **Create VPC** e selecione a opção **VPC and more**. 
- Crie uma VPC com **2 sub-redes públicas** e **2 sub-redes privadas**.
- Selecione duas **Availability Zones (AZs)** e um **NAT Gateway**.
- Finalize a criação clicando em **Create VPC**.

   ![VPC](https://github.com/user-attachments/assets/652498ab-0d6d-4f13-a878-7152ebbd7708)

### Etapa 2. Criação dos Security Groups
  - Em **Security groups** selecione **Create security group** e crie quatro grupos.
  - **web**: acessso às instâncias.
  - **clb**: acesso para o Load Balancer.
  - **rds**: acesso ao banco de dados.
  - **efs**: acesso ao sistema de arquivos.

  ![image](https://github.com/user-attachments/assets/f0d7bdc3-3ad8-4a8a-bd62-555963a0e45b)

- Após a criação, edite:

1. **Security Group Web**
  - **Regras de entrada**: configure uma regra HTTP na porta 80, permitindo tráfego de origem do grupo de segurança do Load Balancer.
  - **Regras de saída**: 
  - **All Traffic** com o destino `0.0.0.0/0`.
  - **HTTP** na porta 80, com o destino para o grupo de segurança do Load Balancer.
  - **MYSQL/Aurora** na porta 3306, com o destino para o grupo de segurança do RDS.
  - **NFS** na porta 2049, com o destino para o grupo de segurança do EFS.

2. **Security Group do Load Balancer**
  - **Regras de entrada**: configure uma regra **HTTP** na porta 80, permitindo tráfego de origem `0.0.0.0/0`.  
  - **Regras de saída**: configure uma regra **HTTP** na porta 80, com o destino para o grupo de segurança Web.

3. **Security Group do RDS**
  - **Regras de entrada**: configure uma regra **MYSQL/Aurora** na porta 3306, permitindo tráfego de origem do grupo de segurança Web.
  - **Regras de saída**: configure uma regra **MYSQL/Aurora** na porta 3306, com o destino para o grupo de segurança Web.

4. **Security Group do EFS**
  - **Regras de entrada**: configure uma regra **NFS** na porta 2049, permitindo tráfego de origem do grupo de segurança Web.
  - **Regras de saída**: configure uma regra **NFS** na porta 2049, com o destino para o grupo de segurança Web.


### Etapa 3: Criação de uma Instância RDS 
1. **Inicie o Processo de Criação**: 
   - Procure por "RDS" na barra de pesquisa e selecione **Aurora and RDS**. 
   - Clique em **Databases**.
   - Clique no botão **Create database**.

2. **Escolha o Tipo de Banco de Dados**:
   - Selecione o tipo de banco de dados que deseja usar.
   - Selecione "Standard Create".

3. **Configurações da Instância**:
   - Dê um nome à sua instância de banco de dados.
   - Insira o nome de usuário do administrador.
   - Em "Configurações de credenciais", selecione **Autogerenciada**, crie e confirme uma senha para o usuário administrador.

4. **Configurações de Banco de Dados**:
   - Escolha o tipo de instância que você quer utilizar.
   
5. **Configurações de Rede e Segurança**:
   - Escolha a VPC (Virtual Private Cloud) criada na Etapa 1 - (parte-1).
   - Selecione o grupo de segurança RDS criado na Etapa 2 - (parte-1).
   - Em **Configuração adicional**, dê um nome para o banco de dados.

6. **Criar a Instância**:
   - Revise suas configurações e clique em **Create database** para iniciar a criação da instância.

7. **Aguarde a Criação**:
   - A criação da instância pode levar alguns minutos. 
   - Copie o endpoint da instância criada e cole no `script_Wordpress` fornecido neste repositório.

   ![image](https://github.com/user-attachments/assets/e94a35ce-60c5-4d72-b583-bc134161156c)
   

### Etapa 4: Criando um Sistema de Arquivos EFS
1. **Inicie o Processo de Criação**: 
   - Procure por "EFS" na barra de pesquisa e selecione **EFS**, depois clique no botão **Create file system**.

2. **Configurações do Sistema de Arquivos**:
   - Dê um nome ao seu sistema de arquivos.
   - Escolha a VPC (Virtual Private Cloud) criada na Etapa 1 - (parte-1).
   - Clique em Personalizar.
   
3. **Configurações de Rede**:
   - Selecione duas Zonas de disponibilidade.
   - Selecione duas sub-redes privadas para cada Zona. 
   - Selecione o grupo de segurança EFS criado na Etapa 2 - (parte-1).

4. **Criar o Sistema de Arquivos**:
   - Revise suas configurações e clique em **Create file system** para iniciar a criação do sistema de arquivos.

5. **Aguarde a Criação**:
   - A criação do sistema de arquivos pode levar alguns minutos. 
   - Copie o ID do sistema de arquivos e cole no `script_Wordpress` fornecido neste repositório.

    ![image](https://github.com/user-attachments/assets/a1d7fee2-560f-4a35-8698-136b5aa91b6e)


### Etapa 5: Creation of a Launch Template
1. **Inicie o Processo de Criação**: 
   - Navegue até a seção **EC2** em **Launch instances** e clique em **Create Launch instances**.
   - Dê um nome ao seu template.
   - Descreva a versão do template.
   - Escolha a imagem.
   - Escolha o tipo de instância.
   - Selecione o grupo de segurança WEB criado na Etapa 2 - (parte-1).
   - Em **Advanced Details**, no campo **User  Data (opcional)**, cole o arquivo: `script_Wordpress` fornecido neste repositório.
   - Clique em **Create launch template**.

## Parte 2: Configuração de Auto Scaling e Load Balancer na AWS

### Etapa 1: Criação do Classic Load Balancer
1. **Acesse o Auto Scaling Group**:
   - Navegue até a seção **EC2** em **Load Balancer** e clique em **Create Load Balancer**.
- Configure as opções:
   - Selecione **Internet-facing**.
   - Escolha a VPC (Virtual Private Cloud) criada na Etapa 1 - (parte-1).
   - Em **Availability Zones** selecione duas zonas de disponibilidade e escolha sub-redes públicas para elas.
   - Selecione o grupo de segurança CLB criado na Etapa 2.
   - Em **Listeners and routing**, defina HTTP na porta 80.
   - Em **Health Checks**, altere o **Ping path** para: `/wp-admin/install.php`.

     ![image](https://github.com/user-attachments/assets/d20216b2-a96d-4daa-8efd-7d1cced609e6)
     

### Etapa 2: Criação de um Auto Scaling Group
1. **Inicie o Processo de Criação**: 
   - Navegue até a seção **EC2** em **Auto Scaling Groups** e clique em **Create Auto Scaling group**.
   - Dê um nome ao seu Auto Scaling Group.
   - Selecione o Launch Template que você criou anteriormente.
   - Clique em **Next**.

2. **Configurações de Rede**:
   - Escolha a VPC (Virtual Private Cloud) criada na Etapa 1 - (parte-1).
   - Escolha as duas sub-redes privadas.
   - Clique em **Next**.
   - Associe o Classic Load Balancer criado na Etapa 2 - (parte-2).
   - Selecione **Ative as verificações de integridade do Elastic Load Balancing**.
   - Clique em **Next**.

3. **Configurações de Escalonamento**:
   - Defina a Capacidade Desejada, Capacidade mínima desejada e Capacidade máxima desejada de instâncias.
   - Selecione **Nenhuma política de escalabilidade**.
   - Selecione **Habilitar coleta de métricas de grupo no CloudWatch**.
   - Clique em **Next**.

4. **Revisão e Criação**:
   - Revise suas configurações e clique em **Crear Grupo do Auto Scaling**.

     ![image](https://github.com/user-attachments/assets/de04b954-c252-448d-a0f4-88a162050947)
     

### Etapa 3: Acesso ao WordPress
   - Navegue até a seção **EC2** em **Instances**, aguarde a criação das instâncias.
   - Navegue até a seção **Load Balancer**, copie o **DNS name** e cole no navegador para acessar a aplicação WordPress.
   - Faça a configuração e o login para ter acesso ao WordPress.

     ![image](https://github.com/user-attachments/assets/3b92a1a6-6364-4a8d-bde7-772d70f47d5f)


### Etapa 4: Criação de Alarme no CloudWatch Adicionar
1. **Criação da Escalabilidade automática**
  - Navegue até a seção **Auto Scaling Group**, em **Escalabilidade Automatica** clique em **Criar política de Escalabilidade**.
  - Em tipos de política, escolha **Escalabilidade Simples**.
  - Dê um nome da política de escalabilidade.
  - Selecione **Adicionar**, com valor : **Desejado**.
  - Clique em **Criar**.

2. **Criação do Alarme**
  - Navegue até a seção **ClouWatch**, em **Alarmes** clique em **Criar alarmes**.
  - Clique em **Selecionar métricas**, em **EC2** e depois em **By Auto Scaling Group**.
  - Selecione a métrica **CPUUtilization** e clique em **Select metric**.
  - Em **Sempre que CPUUtilization for...**, selecione **Maior** e digite valor **Desejado** em **que...**.
  - Em **Notificação**, clique em **Remover**. 
  - Clique em **Ação do Auto Scaling**, selecione o Auto Scaling Group criado na Etapa 3 - (parte-2).
  - Clique em **Next**.
  - Dê um nome para o alarme.
  - Clique em **Next**.
  - Clique em **Criar alarme**.

    ![image](https://github.com/user-attachments/assets/b0ebefa2-080b-447c-a487-c40a04f6c8ec)
    

### Etapa 5: Criação de Alarme no CloudWatch Remover
1. **Criação Auto Scaling Group**
  - Navegue até a seção **Auto Scaling Group**, em **Escalabilidade Automatica** clique em **Criar política de Escalabilidade**.
  - Em tipos de política, escolha **Escalabilidade Simples**.
  - Dê um nome da política de escalabilidade.
  - Selecione **Remover**, com valor : **Desejado**.
  - Clique em **Criar**.

2. **Criação do Alarme**
  - Navegue até a seção **ClouWatch**, em **Alarmes** clique em **Criar alarmes**.
  - Clique em **Selecionar métricas**, em **AutoScaling** e depois em **Group Metrics**.
  - Selecione a métrica **GroupTotalCapacity** e clique em **Select metric**.
  - Em **Sempre que GroupTotalCapacity for...**, selecione **Maior** e digite valor **Desejado** em **que...**.
  - Clique em **Next**.
  - Em **Notificação**, clique em **Remover**. 
  - Clique em **Ação do Auto Scaling**, selecione o Auto Scaling Group criado na Etapa 3 - (parte-2)..
  - Clique em **Next**.
  - Dê um nome para o alarme.
  - Clique em **Next**.
  - Clique em **Criar alarme**.

   ![image](https://github.com/user-attachments/assets/d0cc4180-94f4-45f8-9a4f-8ae7fc9a4c77)


## Conclusão

Este projeto implementou uma infraestrutura escalável e resiliente na Amazon Web Services (AWS) para uma aplicação WordPress. Com instâncias EC2, um banco de dados RDS, EFS, Auto Scaling e Load Balancer, criamos um ambiente robusto que suporta picos de tráfego e garante alta disponibilidade, além de práticas de segurança para proteger dados.

O Auto Scaling permite que a aplicação se adapte automaticamente à carga de trabalho, enquanto o Classic Load Balancer distribui o tráfego de forma eficiente, aumentando a resiliência do sistema.

Assim, o projeto demonstra a eficácia da AWS em fornecer soluções de infraestrutura que atendem às demandas de aplicações modernas e seguras.
