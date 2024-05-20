# Continuos deploy with Github actions and AWS

## **1. A importância de CI/CD no desenvolvimento de software e como ele melhora a eficiência dos times de desenvolvimento.**

Integração Contínua (CI) é uma prática de desenvolvimento onde os desenvolvedores combinam mudanças de código em um repositório. Cada integração é verificada por uma build automatizada, permitindo a detecção precoce de erros com a realização de testes e preparando uma espécie de pacote para deploy, já que o output do CI é o Input do CD. Por outro lado a entrega Contínua (CD) é uma extensão da esteira CI, onde o código é automaticamente implantado em um ambiente de produção após passar por uma série de testes.

A integração contínua (CI) e entrega contínua (CD) são práticas essenciais no desenvolvimento de software já que elas permitem que os times responsáveis pelo sistema integrem o código novo com frequência e entreguem as novas versões de software de maneira rápida eficiente, principalmente no cenário atual onde a rapidez e a eficiência são cruciais para o sucesso de qualquer projeto. Essa metodologia transformou a forma como as equipes de desenvolvimento trabalham, entregam valor e mantêm a qualidade do software que está sendo entregue; A integração dessas duas práticas fazem com que o software esteja sempre pronto para a produção com uma entrega mais ágil e confiável do que se fosse realizada manualmente. Podemos melhorar a eficiência do software com CI e CD da seguinte forma:

- **1. Detecção Precoce de Erros**: Com CI, cada commit de código é automaticamente testado. Isso permite a identificação e correção de erros mais rápida e no início do ciclo de desenvolvimento.
- **2. Automatização de Processos**: CI/CD automatiza muitas das tarefas repetitivas envolvidas no desenvolvimento e na implantação de software, como testes unitários, integração de código e deploys. Isso além de economizar tempo da equipe, também reduz o risco de erro humano.
- **3. Feedback Rápido**: A integração contínua fornece feedback imediato sobre o impacto das mudanças de código. Isso é garantido com features de notificações sobre resultados de builds e testes, permitindo uma resposta mais rápida.
- **4. Entrega Frequente**: A entrega contínua permite que seja lançado  novas funcionalidades e correções de bugs de forma contínua e frequente.

A adoção de práticas de CI/CD é fundamental para qualquer equipe de desenvolvimento que busca eficiência, qualidade e rapidez na entrega de software.

## **2. A estrutura e os componentes principais de um workflow do GitHub Actions.**

GitHub Actions é uma ferramenta de automação de CI/CD que permite automatizar fluxos de trabalho diretamente a partir dos repositórios do GitHub. Uma das características mais importântes do GitHub Actions é a sua flexibilidade e capacidade de integração com uma vasta gama de serviços e ferramentas. A exemplificação da estrutura e dos componentes principais de um workflow do GitHub Actions será feita com base em um exemplo simples:

Um workflow no GitHub Actions é um conjunto de instruções definidas em um arquivo YAML, que descreve um processo automatizado. Esse arquivo é armazenado no diretório `.github/workflows` do repositório do github. A estrutura básica de um workflow inclui:

- **name**: O nome do workflow.
- **on**: Especifica os triggers (gatilhos) que iniciam o workflow, como eventos de push (em uma branch especifica, por exemplo), pull request ou um cronograma (diario ou de 12 em 12 horas, por exemplo).
- **jobs**: Um conjunto de trabalhos que descrevem as tarefas a serem executadas. Cada job é composto por uma série de steps (passos).

Exemplo simples de um workflow básico:

```yaml
name: Deploy to Amazon EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Check out the code
        uses: actions/checkout@v2
      
      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '14'
      
      - name: Install dependencies
        run: npm install
      
      - name: Build the project
        run: npm run build
      
      - name: Deploy to EC2
        uses: appleboy/ssh-action@v0.1.4
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_KEY }}
          script: |
            cd /var/www/myapp
            git pull
            npm install
            npm run start
```

### Componentes Principais:

1. **Gatilhos (Triggers)**

Os triggers são os eventos que iniciam a execução do workflow. Eles são definidos na chave `on` do arquivo YAML. Eles podem ser baseados em eventos de repositório, como `push` ou `pull_request`, ou podem ser agendados.

```yaml
on:
  push:
    branches:
      - main
```

Neste exemplo, o workflow é acionado sempre que há um push na branch `main`.

2. **Jobs**

Os jobs são unidades de trabalho que contêm uma série de passos. Cada job é executado em um ambiente separado e pode ser executado em paralelo, em sequência, ou em dependência de outro job, dependendo da configuração.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
```

O job `deploy` está configurado para ser executado em uma máquina virtual com o sistema operacional Ubuntu mais recente.

3. **Steps**

Os steps são as ações individuais dentro de um job. Cada step pode executar um comando de shell ou usar uma ação predefinida do GitHub Actions.

```yaml
steps:
  - name: Check out the code
    uses: actions/checkout@v2

  - name: Set up Node.js
    uses: actions/setup-node@v2
    with:
      node-version: '14'
```

Neste exemplo, os steps incluem a verificação do código do repositório e a configuração do Node.js.

4. **Usando Ações (Uses)**

As ações são tarefas reutilizáveis que podem ser incluídas nos steps. Elas são definidas usando a chave `uses` e podem ser ações da comunidade ou personalizadas.

```yaml
- name: Check out the code
  uses: actions/checkout@v2
```

A ação `actions/checkout@v2` é uma ação oficial do GitHub que verifica o código do repositório.

5. **Segredos e Variáveis de Ambiente**

Os segredos github e as variáveis de ambiente são usados para armazenar as informações sensíveis e configuráveis, como credenciais de API e chaves SSH. Eles são referenciados dentro do workflow usando a sintaxe `${{ secrets.NOME_DO_SEGREDO }}`.

```yaml
with:
  host: ${{ secrets.EC2_HOST }}
  username: ${{ secrets.EC2_USER }}
  key: ${{ secrets.EC2_KEY }}
```

Aqui, `EC2_HOST`, `EC2_USER` e `EC2_KEY` são segredos armazenados no repositório GitHub, garantindo que informações sensíveis não sejam expostas no código.

## **3. A função e a importância do AWS CloudFormation na automação da infraestrutura.**

A gestão de infraestrutura em nuvem pode ser complexa e demorada, já que geralmente a gente lida com múltiplos recursos interconectados. A AWS CloudFormation oferece uma solução para simplificar e automatizar isso, ele é um serviço de IaC utilizado para gerenciar a infraestrutura da AWS através de código. Com o CloudFormation, podemos definir, provisionar e gerenciar toda a infraestrutura necessária para as aplicações em arquivos de texto yaml, conhecidos como templates, esses templates servem como um plano automatizado para a construção de ambientes em nuvem, permitindo a automatização da criação e a configuração de recursos de maneira consistente.

CloudFormation permite a descrição da infraestrutura como código (IaC), que traz inúmeros benefícios como:

- **1. Automação e Repetibilidade**: Com um template para o CloudFormation, podemos provisionar os mesmos recursos de forma consistente em diferentes ambientes (desenvolvimento, homologação, produção) sem a necessidade de configuração manual.
- **2. Controle de Versão**: Os templates podem ser versionados, permitindo rastrear mudanças, reverter para versões anteriores e colaborar de forma eficiente entre a equipe de desenvolvimento de software.
- **3. Orquestração de Recursos**: O CloudFormation gerencia dependências entre recursos automaticamente, garantindo que eles sejam criados, atualizados ou excluídos na ordem correta.
- **4. Gerenciamento de Estado**: O CloudFormation também mantém o controle do estado dos recursos gerenciados, permitindo atualizações e exclusões seguras e consistentes.

### Explorando o Template do CloudFormation do projeto

Esse template pode ser acessado [aqui!](https://github.com/brun0meira/continuos-deploy-github/blob/main/cloudformation/template.yaml)

Vou detalhar o template, que descreve a criação de uma infraestrutura completa para uma aplicação web, incluindo VPC, subnets, instâncias EC2, grupos de segurança e uma instância RDS.

#### Definição dos Parâmetros

```yaml
Parameters:
  EnvironmentType:
    Description: "Specify the Environment type of the stack."
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - uat
      - prod
  AmiID:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Description: "The ID of the AMI."
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2
  KeyPairName:
    Type: String
    Description: The name of an existing Amazon EC2 key pair in this region to use to SSH into the Amazon EC2 instances.
    Default: KP-GEN
  DBInstanceIdentifier:
    Type: String
    Default: "postgres-db"
  DBUsername:
    Type: String
    MinLength: "1"
    MaxLength: "16"
    AllowedPattern: "[a-zA-Z][a-zA-Z0-9]*"
    Default: "postgres"
  DBPassword:
    Type: String
    MinLength: "8"
    MaxLength: "41"
    ConstraintDescription: Must have minimum length of 8 and maximum length of 41
```

Aqui definimos os parâmetros que tornam o template reutilizável e flexível. Esses parâmetros permitem especificar o tipo de ambiente (`dev`, `uat`, `prod`), o ID da AMI (Imagem de Máquina da Amazon), o nome do par de chaves para SSH, e as credenciais para a instância RDS PostgreSQL.

#### Mapeamentos

```yaml
Mappings:
  EnvironmentToInstanceType:
    dev:
      InstanceType: t2.micro
    uat:
      InstanceType: t2.micro
    prod:
      InstanceType: t2.micro
```

O mapeamento `EnvironmentToInstanceType` associa tipos de instância EC2 com base no tipo de ambiente. Neste exemplo, todos os ambientes usam `t2.micro` pois utilizarmos AWS Learn Lab onde os recursos são free tier, mas este mapeamento pode ser expandido para utilizar diferentes tipos de instância conforme necessário em outros projetos.

#### Criação da VPC e Subnets

```yaml
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
      - Key: Name
        Value: !Join ["-", [vpc, !Ref EnvironmentType]]
  
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
      - Key: Name
        Value: !Join ["-", [internet-gateway, !Ref EnvironmentType]]

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: us-east-1a
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Join ["-", [public-subnet-1, !Ref EnvironmentType]]

  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.3.0/24
      AvailabilityZone: us-east-1b
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Join ["-", [public-subnet-2, !Ref EnvironmentType]]
```

Este trecho do template define a criação de uma VPC (Virtual Private Cloud) com duas subnets públicas. Também inclui a configuração de um gateway de internet e a associação deste gateway ao VPC, permitindo que as instâncias dentro do VPC se comuniquem com a internet.

#### Configuração da Tabela de Roteamento

```yaml
PublicRouteTable:
  Type: AWS::EC2::RouteTable
  Properties:
    VpcId: !Ref VPC
    Tags:
      - Key: Name
        Value: !Join ["-", [public-route-table, !Ref EnvironmentType]]

PublicRoute:
  Type: AWS::EC2::Route
  Properties:
    RouteTableId: !Ref PublicRouteTable
    DestinationCidrBlock: 0.0.0.0/0
    GatewayId: !Ref InternetGateway

PublicSubnetRouteTableAssociation1:
  Type: AWS::EC2::SubnetRouteTableAssociation
  Properties:
    SubnetId: !Ref PublicSubnet1
    RouteTableId: !Ref PublicRouteTable

PublicSubnetRouteTableAssociation2:
  Type: AWS::EC2::SubnetRouteTableAssociation
  Properties:
    SubnetId: !Ref PublicSubnet2
    RouteTableId: !Ref PublicRouteTable
```

A configuração da tabela de roteamento define uma rota para todo o tráfego de saída (`0.0.0.0/0`) para passar pelo gateway de internet. As subnets públicas são associadas a esta tabela de roteamento para garantir que o tráfego de rede seja roteado corretamente para fora da VPC e seja acessado corretamente.

#### Instâncias EC2

```yaml
EC2Instance1:
  Type: AWS::EC2::Instance
  Properties:
    ImageId: !Ref AmiID
    InstanceType:
      !FindInMap [
        EnvironmentToInstanceType,
        !Ref EnvironmentType,
        InstanceType,
      ]
    SubnetId: !Ref PublicSubnet1
    KeyName: !Ref KeyPairName
    SecurityGroupIds:
      - !Ref EC2SecurityGroup
    UserData:
      Fn::Base64: !Sub |
        #!/bin/bash
        LOG_FILE="/var/log/userdata_execution.log"
        yum update -y &>> $LOG_FILE
        amazon-linux-extras install docker -y &>> $LOG_FILE
        systemctl start docker &>> $LOG_FILE
        touch home/ec2-user/script.sh &>> $LOG_FILE
        echo "#\!/bin/bash" > home/ec2-user/script.sh
        echo "release_type=\$1" >> home/ec2-user/script.sh
        echo "AWS_DEFAULT_REGION="us-east-1"" >> home/ec2-user/script.sh
        echo "export AWS_ACCESS_KEY_ID=\$(aws ssm get-parameter --name '/my-app/aws-access-key-id' --with-decryption --query 'Parameter.Value' --output text)" >> home/ec2-user/script.sh
        echo "export AWS_SECRET_ACCESS_KEY=\$(aws ssm get-parameter --name '/my-app/aws-secret-access-key' --with-decryption --query 'Parameter.Value' --output text)" >> home/ec2-user/script.sh
        echo "export AWS_SESSION_TOKEN=\$(aws ssm get-parameter --name '/my-app/aws-session-token' --with-decryption --query 'Parameter.Value' --output text)" >> home/ec2-user/script.sh
        echo "export AWS_DEFAULT_REGION=\$(aws ssm get-parameter --name '/my-app/aws-region' --query 'Parameter.Value' --output text)" >> home/ec2-user/script.sh
        echo "" >> home/ec2-user/script.sh
        echo "AWS_ECR_PASSWORD=\$(aws ecr get-login-password --region $AWS_DEFAULT_REGION)" >> home/ec2-user/script.sh
        echo "" >> home/ec2-user/script.sh
        echo "if [[ $(docker ps -q) ]]; then" >> home/ec2-user/script.sh
        echo "  sudo docker stop $(docker ps -q)" >> home/ec2-user/script.sh


        echo "  sudo docker rm $(docker ps -aq)" >> home/ec2-user/script.sh
        echo "fi" >> home/ec2-user/script.sh
        echo "" >> home/ec2-user/script.sh
        echo "echo \$AWS_ECR_PASSWORD | sudo docker login -u AWS --password-stdin 111111111111.dkr.ecr.us-east-1.amazonaws.com" >> home/ec2-user/script.sh
        echo "echo \$release_type" >> home/ec2-user/script.sh
        echo "" >> home/ec2-user/script.sh
        echo "image_id=''" >> home/ec2-user/script.sh
        echo "if [[ \$release_type == 'staging' ]]; then" >> home/ec2-user/script.sh
        echo "  image_id=111111111111.dkr.ecr.us-east-1.amazonaws.com/staging-repo:latest" >> home/ec2-user/script.sh
        echo "elif [[ \$release_type == 'production' ]]; then" >> home/ec2-user/script.sh
        echo "  image_id=111111111111.dkr.ecr.us-east-1.amazonaws.com/prod-repo:latest" >> home/ec2-user/script.sh
        echo "fi" >> home/ec2-user/script.sh
        echo "" >> home/ec2-user/script.sh
        echo "sudo docker run -d -p 80:80 \$image_id" >> home/ec2-user/script.sh
        chmod +x /home/ec2-user/script.sh
    Tags:
      - Key: Name
        Value: !Join ["-", [ec2-instance-1, !Ref EnvironmentType]]

EC2Instance2:
  Type: AWS::EC2::Instance
  Properties:
    ImageId: !Ref AmiID
    InstanceType:
      !FindInMap [
        EnvironmentToInstanceType,
        !Ref EnvironmentType,
        InstanceType,
      ]
    SubnetId: !Ref PublicSubnet2
    KeyName: !Ref KeyPairName
    SecurityGroupIds:
      - !Ref EC2SecurityGroup
    UserData:
        # Para o código não ficar muito grande foi excluido essa parte
    Tags:
      - Key: Name
        Value: !Join ["-", [ec2-instance-2, !Ref EnvironmentType]]
```

As instâncias EC2 são configuradas para iniciar com a AMI especificada, tipo de instância, par de chaves para SSH, e segurança apropriada. O `UserData` executa um script na inicialização para instalar o Docker, configurar credenciais AWS e iniciar o contêiner Docker.

#### ELB Security Group

```yaml
ELBSecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupName: !Join ["-", [elb-security-group, !Ref EnvironmentType]]
    GroupDescription: "ELB Security Group"
    VpcId: !Ref VPC
    SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: 8080
        ToPort: 8080
        CidrIp: 0.0.0.0/0
```

Este grupo de segurança permite o tráfego HTTP e HTTP alternativo (porta 8080) de qualquer endereço IP. Ele será usado pelo Elastic Load Balancer (ELB) para gerenciar o tráfego de entrada.

#### EC2 Security Group

```yaml
EC2SecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupName: !Join ["-", [ec2-security-group, !Ref EnvironmentType]]
    GroupDescription: "EC2 Security Group"
    VpcId: !Ref VPC
    SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        SourceSecurityGroupId: 
          Fn::GetAtt:
          - ELBSecurityGroup
          - GroupId
      - IpProtocol: tcp
        FromPort: 22
        ToPort: 22
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: 8080
        ToPort: 8080
        CidrIp: 0.0.0.0/0
```

Este grupo de segurança permite o tráfego HTTP e HTTP alternativo (porta 8080) do ELB e permite SSH (porta 22) de qualquer endereço IP. Ele será associado às instâncias EC2.

#### Target Group Configuration

```yaml
EC2TargetGroup:
  Type: AWS::ElasticLoadBalancingV2::TargetGroup
  Properties:
    HealthCheckIntervalSeconds: 30
    HealthCheckProtocol: HTTP
    HealthCheckTimeoutSeconds: 15
    HealthyThresholdCount: 5
    Matcher:
      HttpCode: '200'
    Name: EC2TargetGroup
    Port: 80
    Protocol: HTTP
    TargetGroupAttributes:
      - Key: deregistration_delay.timeout_seconds
        Value: '20'
    Targets:
      - Id: !Ref EC2Instance1
        Port: 80
      - Id: !Ref EC2Instance2
        Port: 80
      - Id: !Ref EC2Instance1
        Port: 8080
      - Id: !Ref EC2Instance2
        Port: 8080
    UnhealthyThresholdCount: 3
    VpcId: !Ref VPC
```

Este Target Group define a configuração do grupo de destinos para o balanceador de carga. Ele inclui parâmetros de verificação de saúde e atribui as instâncias EC2 às portas 80 e 8080.

#### Load Balancer Deployment

```yaml
ElasticLoadBalancer:
  Type: AWS::ElasticLoadBalancing::LoadBalancer
  Properties:
    Scheme: internet-facing
    Subnets:
      - !Ref PublicSubnet1
      - !Ref PublicSubnet2
    Listeners:
      - LoadBalancerPort: '80'
        InstancePort: '80'
        Protocol: HTTP
    SecurityGroups:
      - !GetAtt ELBSecurityGroup.GroupId
    Tags:
      - Key: Name
        Value: !Join ["-", [elb, !Ref EnvironmentType]]
```

Este trecho cria um Load Balancer (ELB) voltado para a internet, configurado para ouvir na porta 80 e encaminhar o tráfego para as instâncias EC2.

#### RDS Security Group

```yaml
RDSSecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupName: !Join ["-", [db-security-group, !Ref EnvironmentType]]
    GroupDescription: "RDS traffic"
    VpcId: !Ref VPC
    SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 5432
        ToPort: 5432
        SourceSecurityGroupId: 
          Fn::GetAtt:
          - ELBSecurityGroup
          - GroupId
    Tags:
      - Key: Name
        Value: !Join ["-", [db-security-group, !Ref EnvironmentType]]
```

Este grupo de segurança permite o tráfego na porta 5432 (PostgreSQL) a partir do grupo de segurança do ELB, garantindo que apenas o ELB possa acessar o banco de dados.

#### RDS Subnet Group

```yaml
RDSSubnetGroup:
  Type: AWS::RDS::DBSubnetGroup
  Properties:
    DBSubnetGroupName: subnetgroup
    DBSubnetGroupDescription: Subnet Group
    SubnetIds:
      - !Ref PublicSubnet1
      - !Ref PublicSubnet2
```

Este grupo de subnets define as subnets que serão usadas pela instância RDS.

#### RDS Instance

```yaml
RDSInstance:
  Type: AWS::RDS::DBInstance
  Properties:
    DBSubnetGroupName: !Ref RDSSubnetGroup
    VPCSecurityGroups:
      - !GetAtt RDSSecurityGroup.GroupId
    DBInstanceIdentifier: !Ref DBInstanceIdentifier
    AllocatedStorage: "5"
    DBInstanceClass: db.t3.micro
    Engine: postgres
    MasterUsername: !Ref DBUsername
    MasterUserPassword: !Ref DBPassword
    Tags:
      - Key: Name
        Value: !Join ["-", [rds, !Ref EnvironmentType]]
  DeletionPolicy: Snapshot
  UpdateReplacePolicy: Snapshot
```

Esta definição cria a instância RDS PostgreSQL, associando-a ao grupo de subnets e ao grupo de segurança previamente definidos. Ele também especifica que a instância deve ser excluída com um snapshot para preservação de dados.

#### Saídas

```yaml
Outputs:
  InstanceId:
    Description: "Instance ID of the EC2 instance 1"
    Value: !Ref EC2Instance1
```

As saídas fornecem informações úteis sobre os recursos criados, neste caso, o ID da primeira instância EC2.

### Conclusão do template

Este CloudFormation template fornece uma infraestrutura robusta para uma aplicação web, incluindo instâncias EC2, balanceador de carga, banco de dados RDS e configurações de segurança. Usando este template, é feito a implantação total da infraestrutura e gerenciamento na AWS de forma eficiente e consistente.

## **4. Como a integração de GitHub Actions com AWS CloudFormation e Amazon EC2 pode ser aplicada em projetos reais.**

A integração de GitHub Actions com AWS CloudFormation e Amazon EC2 é uma ótima abordagem para automatizar as pipelines de CI/CD, esse processo facilita a entrega contínua e promove a consistência, repetibilidade e segurança nas implantações das aplicações. Para demonstrar como isso pode ser feito em aplicações reais, podemos usar como exemplo uma aplicação web que é baseada em mais de um micro serviços, o problema atual é que cada mudança no código precisa ser testada, integrada e implantada em ambiente de produção de forma automática, tudo para garantir a qualidade do software. Com o Github Actions, AWS CloudFormation e Amazon EC2 podemos resolver isso da seguinte forma:

- **1. GitHub Actions como Orquestrador de Pipelines**: GitHub Actions para definição dos workflows que serão executados em resposta a eventos de pull requests. Esses workflows incluem as etapas para compilar o código, executar testes e, finalmente, implantar a aplicação.
- **2. AWS CloudFormation para Gerenciamento de Infraestrutura**: CloudFormation para a definição da infraestrutura como código (IaC), o que significa que a configuração da infraestrutura será versionada e reutilizada. Esse template CloudFormation especifica os recursos necessários como as instâncias EC2, balanceadores de carga, grupos de segurança, etc.
- **3. Amazon EC2 como Plataforma de Hospedagem**: As instâncias EC2 para hospedar as aplicações. O uso de Auto Scaling Groups pode garantir que a aplicação se ajuste automaticamente às mudanças de demanda, é um dos benefícios do EC2.

Com essa integração pode surgir desafios comuns, eles serão apresentados aqui juntamente com uma proposta para solucioná-los:

- 1. Gerenciar credenciais de acesso de forma segura é uma das principais preocupações ao realizar a integração dos serviços de CI/CD com a AWS. Mas pode ser utilizado o AWS Secrets Manager para armazenar secrets e parâmetros da configuração de forma segura. E ainda a configuração dos GitHub Secrets para armazenar as credenciais necessárias para acessar a AWS para que as permissões IAM sejam limitadas ao mínimo necessário para reduzir o risco de exposição.

- 2. A execução de testes extensivos e a implantação em múltiplas instâncias EC2 pode ser demorada, afetando a produtividade. Pode realizar a otimização dos testes dividindo-os em testes unitários, de integração e end-to-end, e executar eles em paralelo, utilizando instâncias spot do EC2 para reduzir custos durante esses testes intensivos de performance e implementar o uso de containers para acelerar a criação e destruição de ambientes de teste.

- 3. Manter os templates do CloudFormation atualizados e gerenciáveis à medida que a aplicação cresce e escala pode ser desafiador. Podemos adotar uma abordagem modular para os templates, dividindo a infraestrutura em componentes menores e reutilizáveis que possam ser combinados conforme necessário.
