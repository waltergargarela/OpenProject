# IAC - Infraestrutura como Código

Infraestrutura como código ou IaC, é uma forma e capacidade de provisionar, 
configurar e manter uma infraestrutura computacional através de códigos ao 
invés de processos e configurações manuais. 

Quando se desenvolve uma aplicação precisamos de um ambiente computacional com 
muitos componentes de infraestrutura como redes, servidores, sistemas 
operacionais, conexões com banco de dados e sistemas de armazenamento. 

Provisionar todo o ambiente manualmente pode gastar muito tempo e se tornar uma 
tarefa maçante, principalmente se estas tarefas precisam ser feitas 
diversas vezes. 

A Infraestrutura como Código, IaC, é uma tecnologia que nos ajuda a criar 
componentes computacionais, através de códigos, de forma que agilize os processos 
de criação ou provisionamento de um ambiente em questões de segundos. 
Claro que gastamos um tempo para criar esses códigos ou templates, mas uma vez 
criado um modelo ou template, basta apenas customizar conforme a necessidade ou 
o caso de uso.

Imagine que você já tenha um ambiente funcional, para uma aplicação, e precisa 
criar um ambiente para testes iniciais de desenvolvimento, outro igual para 
homologação e outro para produção, criar estes ambientes de forma manual levará 
muito tempo e podendo até gerar alguns erros no momento da criação. Com IaC, 
podemos criar um primeiro template, codificado, com toda a infraestrutura 
necessária para implementar e replicar para quantos ambientes quiser, de forma 
mais rápida e com poucas chances de erros, permitindo ainda uma customização 
pontual para cada ambiente ou caso de uso, reaproveitando o código original.

Para isso precisamos de ferramentas que interprete esses códigos e execute em 
algum provedor de serviços computacionais. 
Os provedores mais comuns são provedores de nuvem pública como AWS, Azure e 
Google Cloud Platform (GCP), porém ainda podemos usar em ambientes de nuvem 
privada como Openstack. 
Outros como Proxmox, que não é um serviço de nuvem, estão começando a ter a 
possibilidade de usar ferramentas de IaC. 


Terraform é a ferramenta de IaC mais conhecida e usada no mercado. 
O Terraform começou como uma ferramenta open-source e uso de forma gratuita, porém 
nos últimos tempos começou a ter algumas formas de cobrança, com isso, a comunidade 
open-source fez o chamado “fork” criando o OpenTofu, que funciona igual ao Terraform.


Outras ferramentas que ajudam nesses processos além do Terraform são: Ansible, Puppet e Chef. 

Provedores de nuvem também tem suas ferramentas próprias como o Cloud Formation 
da AWS, a Azure tem o Resource Manager e o CGP tem Cloud Deployment Manager.

Mesmo que os provedores citados tenham suas próprias ferramentas, podemos também 
provisionar serviços e infraestrutura com o Terraform e o OpenTofu, em seus 
ambientes de nuvem, inclusive podendo ter a vantagem de criar os mesmos recursos 
em diferentes provedores, apenas customizando o código IaC. 

---


## Nosso exemplo

Apesar de mencionar mais  o Terraform como exemplo, isso por conta de sua 
versatilidade e vantagens, vamos falar um pouco mais do AWS Cloud Formation, 
isso porque um projeto específico em estudo aqui no trabalho, deverá ser usado o 
Cloud Formation como ferramenta de IaC.

Este projeto visa a criação de alguns recursos básicos na AWS como uma VPC e 
subredes, além de VPN e recursos computacionais. 

Estes códigos foram criados, implementados e testatos para uma oficina de projeto do curso de pós 
graduação em Cloud Computing que participei, o objetivo dessa oficina é a implementação 
do sistema OpenProject. 

Para nosso exemplo aqui, vou abordar a criação de uma VPC (Virtual Private Cloud) 
que nada mais é que o provisionamento de um ambiente de rede, 
com subredes privadas e públicas, tabelas de rotas para as subredes, gateway de 
internet e controle de acessos de redes e suas associações. 
Vou colocar um exemplo da criação de recuros computacionais mais complexos, apenas 
para ilustrar o contexto, mas não vamos abordar esse código em detalhes. 

### Contexto e definições: 

Códigos IaC criados para o Cloud Formation, no momento em que executamos estes 
códigos eles são chamados de pilhas ou “stacks”, muitas vezes podemos criar mais 
de uma “stack” para diferentes tipos de recursos. 

Por exemplo, posso ter uma “stack” para a criação de recursos de rede, como eu 
vou mostrar abaixo, e criar uma outra “stack” com recursos computacionais do tipo 
máquina virtual ou instâncias, que depende dos recursos de rede criada na 
primeira “stack”.  Isso é chamado de “Cross-Stacks”, onde uma “stack” depende 
de recursos de uma outra. 

Vamos criar uma stack, modelo ou template, como preferir chamar, para a criação 
e provisionamento de recursos de rede __VPC_CF_Template.yaml__, que acaba sendo a 
base para a maioria dos projetos em nuvem.

Depois, depois para esta oficina de projetos da pós graduação, precisamos configurar 
um sistema de banco de dados( __RDS_Template.yaml__), um sistema para uso do Memcached 
(__Memcached_Template.yaml__) e por fim todos os recursos computacionais usando 
Containers (ECR,ECS e Fargate), balanceadores de carga, demais itens como configuração
do Route53, ACL e outras coisas foram feitas na mão. 

Não vou abordar a configuração destes outros códigos, me concentando apenas no básico
para a VPC. 


### Primeiros passos para criação da VPC

Primeiro precisamos definir uma região da AWS, neste exemplo usaremos a região de 
Ohio (us-east-2), criaremos a VPC como um nome e definimos uma classe de endereço 
IP. 

Em seguida vamos usar duas zonas de disponibilidade (Data Centers) desta região, 
criaremos duas subredes públicas e duas subredes privadas em cada região. 

Precisamos criar as tabelas de rotas e as listas de controle de acesso de rede 
para cada tipo de subrede, criaremos o gateway de internet para as subredes 
públicas e faremos as devidas associações. 

No final vamos criar algumas saídas para podemos associar os recursos criados 
neste template (stack) com outras stacks. Essas saídas servirão de parâmetros, 
como variáveis, para ser uasada em outras stacks. 

Lembrete, para o Cloud Formation podemos usar tanto o formato JSON como YAML, 
usaremos YAML por padrão, mas devemos ter muita atenção com a indentação e 
formatação em ambos os casos. 


<img src="/images/Layout_VPC.png" alt="VPC Layout Image" width="800px">  

---

#### Parâmetros seguindo os requisitos apresentados na imagem acima: 


- VPC: Definir a classe de endereço IP, para simplificar vamos usar IPv4 e o nome. 
	- CIDR: 10.0.0.0/16 
	- VPC Name: OpenProject

- Subredes públicas: Definir os nomes e endereços IP. 
	- Public01
		- CIDR: 10.0.1.0/24
		- VpcId: OpenProject ou   !Ref VPC
		
	- Public02
		- CIDR: 10.0.2.0/24
		- VpcId: OpenProject ou   !Ref VPC

- Subredes privadas: Definir os nomes e endereços IP. 
	- Private01
		- CIDR: 10.0.3.0/24
		- VpcId: OpenProject ou   !Ref VPC
	- Private02
		- CIDR: 10.0.4.0/24
		- VpcId: OpenProject ou   !Ref VPC
- Gateway de Internet (Internet Gateway)
	- Nome: InternetGateway
	- Attach
		- VpcId: OpenProject ou   !Ref VPC


- Tabela de rotas para subredes publicas: Definir os nomes, rotas e associação 
	- PublicRoute
		- Destination: 0.0.0.0/0  (regra bem aberta)
		- VpcId: OpenProject ou   !Ref VPC
		- GatewayId: InternetGateway  ou  !Ref InternetGateway

- Tabela de rotas para subredes privadas: Definir os nomes, rotas e associação 
	- PublicRoute
		- Destination: 0.0.0.0/0  (regra bem aberta)
		- VpcId: OpenProject ou   !Ref VPC


Nosso exemplo de código não cria uma regra de lista de controle de acesso e usa 
um NAT Gateway (Opcional), mas é funcional para este exemplo. 
O código todo estará incluso neste repositório: [VPC_CF_Template.yaml](VPC_CF_Template.yaml)

---

<br>

## Descrevendo o código

Nosso código de exemplo não cria regras de lista de controle de acesso (ACL) e 
usa um NAT Gateway (Opcional), mas é funcional para este exemplo, permitindo que
instâncias (VMs) instaladas em rede privada possam acessar a internet. 

> Separei em 3 partes para explicar um pouco melhor a estrutura do código. 
>
> - 1a parte - Versão do template e parâmetros / variáveis. 
> - 2a parte - Declaração dos recursos. 
> - 3a parte - Declaração das saídas/outputs. 
 

</br></br></br>

Na __primeira parte__  do código precisamos declarar a versão do template e uma 
descrição (opcional). 

Em seguida declaramos os principais parâmetros, que podemos entender como 
variáveis, como o nome da VPC e os blocos de endereços IP, dessa forma não 
precisamos ficar alterando estas informações diversas vezes no meio do código. 

```
AWSTemplateFormatVersion: '2010-09-09'
Description: Template para criar uma VPC com 2 subnets públicas e 2 subnets privadas

Parameters:
  VPCName:
    Type: String
    Default: "OpenProjectVPC"
    Description: Nome da VPC
  VPCCidrBlock:
    Type: String
    Default: "10.0.0.0/16"
    Description: CIDR Block da VPC
  public01Cidr:
    Type: String
    Default: "10.0.1.0/24"
    Description: CIDR Block da primeira subnet pública
  public02Cidr:
    Type: String
    Default: "10.0.2.0/24"
    Description: CIDR Block da segunda subnet pública
  private01Cidr:
    Type: String
    Default: "10.0.3.0/24"
    Description: CIDR Block da primeira subnet privada
  private02Cidr:
    Type: String
    Default: "10.0.4.0/24"
    Description: CIDR Block da segunda subnet privada
```
</br> </br> </br> 
Na __segunda parte__ do código já entramos com a declaração para a criação ou provisionamento 
dos recursos, ou seja, criação de recursos de rede, rotas e etc. 

Reparem que sempre entramos com uma nomenclatura do recurso, em alguns casos 
colocamos o nome de um recursos como uma “tag”, depois declaramos as propriedades 
de cada recurso. 

As propriedades estão sempre atreladas aos tipos e características de cada recurso. 
Para maiores detalhes das propriedades de todos os recursos da AWS para o 
Cloud Formation, consulte o link abaixo: 
[User Guide - Template Resources Cloud Formation](https://docs.aws.amazon.com/pt_br/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)

```
Resources:
  # VPC
  VPC:
    Type: "AWS::EC2::VPC"
    Properties: 
      CidrBlock: !Ref VPCCidrBlock
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags: 
        - Key: Name
          Value: !Ref VPCName
  
  # Internet Gateway
  InternetGateway:
    Type: "AWS::EC2::InternetGateway"
    Properties: 
      Tags: 
        - Key: Name
          Value: "InternetGateway"

  AttachGateway:
    Type: "AWS::EC2::VPCGatewayAttachment"
    Properties: 
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  # public01
  public01:
    Type: "AWS::EC2::Subnet"
    Properties: 
      VpcId: !Ref VPC
      CidrBlock: !Ref public01Cidr
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      MapPublicIpOnLaunch: true
      Tags: 
        - Key: Name
          Value: "public01"

  # public02
  public02:
    Type: "AWS::EC2::Subnet"
    Properties: 
      VpcId: !Ref VPC
      CidrBlock: !Ref public02Cidr
      AvailabilityZone: !Select [ 1, !GetAZs '' ]
      MapPublicIpOnLaunch: true
      Tags: 
        - Key: Name
          Value: "public02"

  # private01
  private01:
    Type: "AWS::EC2::Subnet"
    Properties: 
      VpcId: !Ref VPC
      CidrBlock: !Ref private01Cidr
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      Tags: 
        - Key: Name
          Value: "private01"

  # private02
  private02:
    Type: "AWS::EC2::Subnet"
    Properties: 
      VpcId: !Ref VPC
      CidrBlock: !Ref private02Cidr
      AvailabilityZone: !Select [ 1, !GetAZs '' ]
      Tags: 
        - Key: Name
          Value: "private02"


 # Route Tables
  PublicRouteTable:
    Type: "AWS::EC2::RouteTable"
    Properties: 
      VpcId: !Ref VPC
      Tags: 
        - Key: Name
          Value: "Public Route Table"

  PrivateRouteTable:
    Type: "AWS::EC2::RouteTable"
    Properties: 
      VpcId: !Ref VPC
      Tags: 
        - Key: Name
          Value: "Private Route Table"

  # Routes
  PublicRoute:
    Type: "AWS::EC2::Route"
    Properties: 
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: "0.0.0.0/0"
      GatewayId: !Ref InternetGateway

  # Subnet Associations
  public01RouteTableAssociation:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties: 
      SubnetId: !Ref public01
      RouteTableId: !Ref PublicRouteTable

  public02RouteTableAssociation:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties: 
      SubnetId: !Ref public02
      RouteTableId: !Ref PublicRouteTable

  private01RouteTableAssociation:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties: 
      SubnetId: !Ref private01
      RouteTableId: !Ref PrivateRouteTable

  private02RouteTableAssociation:
    Type: "AWS::EC2::SubnetRouteTableAssociation"
    Properties: 
      SubnetId: !Ref private02
      RouteTableId: !Ref PrivateRouteTable

  # NAT Gateway
  EIPForNAT:
    Type: "AWS::EC2::EIP"

  NATGateway:
    Type: "AWS::EC2::NatGateway"
    Properties: 
      AllocationId: !GetAtt EIPForNAT.AllocationId
      SubnetId: !Ref public01
      Tags: 
        - Key: Name
          Value: "NAT Gateway"

  PrivateRoute:
    Type: "AWS::EC2::Route"
    Properties: 
      RouteTableId: !Ref PrivateRouteTable
      DestinationCidrBlock: "0.0.0.0/0"
      NatGatewayId: !Ref NATGateway

```

</br> </br> </br> 
Na __terceira parte__ declaramos as saídas (Outputs), estas saídas servem para 
declararmos os nomes dos principais recursos criados e que precisaremos usar 
como entradas/variáveis para outras “stacks”. 

Repare que usamos sempre o “!Ref”  no campo de valor (“Value”), para referenciar 
o recurso criado, logo em seguida declaramos a exportação e o nome que queremos usar. 

```
Outputs:
  VPCId:
    Description: "ID da VPC criada"
    Value: !Ref VPC
    Export:
      Name: OpenProjectVPC
      
  public01Id:
    Description: "ID da primeira subnet pública"
    Value: !Ref public01
    Export:
      Name: PublicSubnet01ID
  public02Id:
    Description: "ID da segunda subnet pública"
    Value: !Ref public02
    Export:
      Name: PublicSubnet02ID
  private01Id:
    Description: "ID da primeira subnet privada"
    Value: !Ref private01
    Export:
      Name: PrivateSubnet01ID
  private02Id:
    Description: "ID da segunda subnet privada"
    Value: !Ref private02
    Export:
      Name: PrivateSubnet02ID


```



