# 03-Desafio-Santander-Code-Girls-Implementando-sua-Primeira-Stack-com-AWS-CloudFormation
# 🏗️ Implementando sua Primeira Stack com AWS CloudFormation

## 🎯 Objetivo
Este projeto tem como objetivo demonstrar, na prática, como criar e gerenciar recursos de infraestrutura na AWS por meio do **AWS CloudFormation**, aplicando o conceito de **Infraestrutura como Código (IaC)**.  
Ao final, você será capaz de:
- Compreender o funcionamento do CloudFormation  
- Criar uma Stack completa  
- Implementar um Firewall (Security Group) associado a uma instância EC2  

---

## ☁️ O que é o AWS CloudFormation
O **AWS CloudFormation** é um serviço que permite **definir, provisionar e gerenciar recursos da AWS de forma automatizada**.  
A partir de um **template em YAML ou JSON**, você descreve sua infraestrutura — servidores, redes, firewalls, bancos de dados e mais — e o CloudFormation cria tudo automaticamente.

### 🔧 Benefícios
- Automação completa da criação e remoção de recursos  
- Consistência entre ambientes (dev, teste, produção)  
- Versionamento e rastreabilidade da infraestrutura  
- Redução de erros humanos  
- Integração com Git, CodePipeline, Terraform e outras ferramentas  

---

## 🧱 Criando uma Stack com AWS CloudFormation na AWS

### Etapas para criar sua primeira Stack:
1. Acesse o **Console da AWS** → **CloudFormation**  
2. Clique em **Create Stack** → **With new resources (standard)**  
3. Escolha **Upload a template file** e envie seu arquivo `.yaml` (modelo abaixo)  
4. Dê um nome à Stack (ex: `MinhaPrimeiraStack`)  
5. Configure permissões (IAM Role, se necessário)  
6. Revise as configurações e clique em **Create Stack**  

Após alguns minutos, sua infraestrutura será criada automaticamente.

---

## 🔥 Criando um Firewall e uma Instância EC2 com AWS CloudFormation

Abaixo está um **exemplo de template completo** que cria:

- Uma **VPC**  
- Uma **sub-rede pública**  
- Um **Security Group (Firewall)** permitindo SSH e HTTP  
- Uma **instância EC2** usando Amazon Linux 2  

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: Template para criar uma Stack com EC2 e Firewall (Security Group)

Resources:
  # Criação da VPC
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: MinhaVPC

  # Criação da Sub-rede Pública
  MySubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: true
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: MinhaSubnetPublica

  # Internet Gateway
  MyInternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: MeuInternetGateway

  # Anexar o Internet Gateway à VPC
  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref MyVPC
      InternetGatewayId: !Ref MyInternetGateway

  # Tabela de Roteamento
  MyRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref MyVPC
      Tags:
        - Key: Name
          Value: MinhaRouteTable

  # Rota para acesso à Internet
  MyRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref MyRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref MyInternetGateway

  # Associação da Sub-rede à Tabela de Roteamento
  MySubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref MySubnet
      RouteTableId: !Ref MyRouteTable

  # Security Group (Firewall)
  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Permitir SSH (22) e HTTP (80)
      VpcId: !Ref MyVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      SecurityGroupEgress:
        - IpProtocol: -1
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: MeuFirewall

  # Instância EC2
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0c55b159cbfafe1f0  # Amazon Linux 2 (substitua conforme a região)
      SubnetId: !Ref MySubnet
      SecurityGroupIds:
        - !Ref MySecurityGroup
      KeyName: MeuParDeChaves  # Altere para o nome do seu par de chaves existente
      Tags:
        - Key: Name
          Value: MinhaInstanciaEC2

Outputs:
  EC2PublicIP:
    Description: Endereço IP público da instância EC2
    Value: !GetAtt MyEC2Instance.PublicIp
  SecurityGroupID:
    Description: ID do Security Group criado
    Value: !Ref MySecurityGroup
```

---

## ✅ Conclusão
Com esse exemplo, o que eu aprendi:
- Criar sua primeira **Stack** no CloudFormation  
- Provisionar uma **VPC, Sub-rede, Firewall e Instância EC2**  
- Utilizar a abordagem de **Infraestrutura como Código (IaC)** para automatizar e versionar recursos AWS  

O CloudFormation simplifica a gestão da nuvem e torna os ambientes **reprodutíveis e escaláveis**.

---

## 📚 Pontos para Estudo
- Conceitos de **Templates, Stacks e StackSets**  
- Parâmetros, Mappings e Outputs avançados  
- Uso do **AWS CloudFormation Designer** (interface visual)  
- Criação de **Lambda Functions** com CloudFormation  
- Integração com **AWS CodePipeline** e **GitHub Actions**  
- Templates modulares com **Nested Stacks**  
