# **Anotações - Aula sobre ligar e desligar instâncias EC2**

O intuito dessa aula é criar um script que ligue e desligue as instâncias EC2.

## **Parte 1 - Script de parar instâncias**

### **Criar a função Lambda**

- Acesse o serviço `Lambda`;
- Clique em `Create a function`;
  - Mantenha o tipo da lambda como `Author from scratch`;
  - Defina um nome, no meu caso, defini como `Stop_Instances`;
  - Defina um `runtime`, defini como `Python 3.9`;
  - Em `Change default execution role`, clique em `Create a new role with basic Lambda permissions`;
  - Clique em `Create function`.

#### **Corrigindo as permissões**

- Na função lambda, acesse a aba `Configuration`;
- No menu lateral esquerdo, selecione `Permissions`;
- Na seção `Execution role`, clique no link da role;
- Na seção `Permission policies`, clique no símbolo `+`, e então clique em `Edit`;
- Selecione a aba `JSON`;
- Cole o seguinte json:
  
  ```json
  {
  "Version": "2012-10-17",
  "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ],
        "Resource": "arn:aws:logs:*:*:*"
      },
      {
        "Effect": "Allow",
        "Action": [
          "ec2:DescribeInstances",
          "ec2:DescribeRegions",
          "ec2:StartInstances",
          "ec2:StopInstances"
        ],
        "Resource": "*"
      }
    ]
  }
  ```

- Clique em `Review policy` e em seguida em `Save changes`.

#### **Alterando o script**

- Na aba `Code`, você verá o código da lambda;
- Em `Code source`, selecione o arquivo `lambda_function.py`;
- Apague todo o conteúdo do arquivo;
- Coloque o seguinte script:
  
  ```python
  import boto3

  def lambda_handler(event, context):

    # Listar as regioes
    ec2_client = boto3.client('ec2')
    regions = [region['RegionName']
                for region in ec2_client.describe_regions()['Regions']]

    # Buscar em todas as regioes
    for region in regions:
      ec2 = boto3.resource('ec2', region_name=region)

      print("Region:", region)

      # Filtrar somente as instancias ligadas
      instances = ec2.instances.filter(
        Filters=[{'Name': 'instance-state-name',
                  'Values': ['running']}])

      # Parar as Instancias
      for instance in instances:
        instance.stop()
        print('Stopped instance: ', instance.id)
  ```

- Clique em `Deploy`.

### **Configurando o timeout da lambda**

- Na lambda criada, acesse a aba `Configuration`;
- No menu lateral esquerdo, selecione `General configuration`;
- Clique em `Edit`;
- Aumente o timeout para `2 minutos`;
- Clique em `Save`.

### **Configurando o agendamento de execução da lambda no CloudWatch**

- Acesse o serviço `CloudWatch`;
- No menu lateral esquerdo, acesse `Events` e clique em `Rules`;
- Clique no botão `Go to Amazon EventBridge`;
- Clique em `Create rule`;
- Defina um nome para essa regra, no meu caso dei o nome `Stop_Rule`;
- Defina uma descrição, no meu caso coloquei `Faz com que as instâncias EC2 parem às 20:00 de segunda-feira à terça-feira.`;
- Em `Rule type`, selecione a opção `Schedule`;
- Clique em `Next`;
- Em `Schedule`, selecione a opção `A fine-grained schedule that runs at a specific time, such as 8:00 a.m. PST on the first Monday of every month.`;
- Em `Cron expression`, defina o valor para `0 23 ? * MON-FRI *` e selecione `Local time zone`;
- Clique em `Next`;
- Selecione o target como `Lambda function`;
- Selecione a lambda criada, no meu caso, `Stop_Instances`;
- Clique em `Next`;
- Clique em `Next` novamente;
- Revise as configurações e clique em `Create rule`.

### **Concluido - Parte 1 de 2**

Agora, apenas para teste, volte nesta regra criada e edite para que ao invés de rodar às 20:00, rode a cada 1 minuto, assim fica mais fácil para ver se já está funcionando.

## **Parte 2 - Script de iniciar as instâncias**

### **Criar a função Lambda**

- Acesse o serviço `Lambda`;
- Clique em `Create a function`;
  - Mantenha o tipo da lambda como `Author from scratch`;
  - Defina um nome, no meu caso, defini como `Start_Instances`;
  - Defina um `runtime`, defini como `Python 3.9`;
  - Em `Change default execution role`, clique em `Create a new role with basic Lambda permissions`;
  - Clique em `Create function`.

#### **Corrigindo as permissões**

- Na função lambda, acesse a aba `Configuration`;
- No menu lateral esquerdo, selecione `Permissions`;
- Na seção `Execution role`, clique no link da role;
- Na seção `Permission policies`, clique no símbolo `+`, e então clique em `Edit`;
- Selecione a aba `JSON`;
- Cole o seguinte json:
  
  ```json
  {
  "Version": "2012-10-17",
  "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ],
        "Resource": "arn:aws:logs:*:*:*"
      },
      {
        "Effect": "Allow",
        "Action": [
          "ec2:DescribeInstances",
          "ec2:DescribeRegions",
          "ec2:StartInstances",
          "ec2:StopInstances"
        ],
        "Resource": "*"
      }
    ]
  }
  ```

- Clique em `Review policy` e em seguida em `Save changes`.

#### **Alterando o script**

- Na aba `Code`, você verá o código da lambda;
- Em `Code source`, selecione o arquivo `lambda_function.py`;
- Apague todo o conteúdo do arquivo;
- Coloque o seguinte script:
  
  ```python
  import boto3

  def lambda_handler(event, context):

    # Listar as regioes
    ec2_client = boto3.client('ec2')
    regions = [region['RegionName']
                for region in ec2_client.describe_regions()['Regions']]

    # Buscar em todas as regioes
    for region in regions:
      ec2 = boto3.resource('ec2', region_name=region)

      print("Region:", region)

      # Filtrar somente as instancias Desligadas
      instances = ec2.instances.filter(
          Filters=[{'Name': 'instance-state-name',
                    'Values': ['stopped']}])

      # Iniciar as Instancias
      for instance in instances:
        instance.start()
        print('Started instance: ', instance.id)
  ```

- Clique em `Deploy`.

### **Configurando o timeout da lambda**

- Na lambda criada, acesse a aba `Configuration`;
- No menu lateral esquerdo, selecione `General configuration`;
- Clique em `Edit`;
- Aumente o timeout para `2 minutos`;
- Clique em `Save`.

### **Configurando o agendamento de execução da lambda no CloudWatch**

- Acesse o serviço `CloudWatch`;
- No menu lateral esquerdo, acesse `Events` e clique em `Rules`;
- Clique no botão `Go to Amazon EventBridge`;
- Clique em `Create rule`;
- Defina um nome para essa regra, no meu caso dei o nome `Start_Rule`;
- Defina uma descrição, no meu caso coloquei `Faz com que as instâncias EC2 iniciem às 06:00 de segunda-feira à terça-feira.`;
- Em `Rule type`, selecione a opção `Schedule`;
- Clique em `Next`;
- Em `Schedule`, selecione a opção `A fine-grained schedule that runs at a specific time, such as 8:00 a.m. PST on the first Monday of every month.`;
- Em `Cron expression`, defina o valor para `0 9 ? * MON-FRI *` e selecione `Local time zone`;
- Clique em `Next`;
- Selecione o target como `Lambda function`;
- Selecione a lambda criada, no meu caso, `Start_Instances`;
- Clique em `Next`;
- Clique em `Next` novamente;
- Revise as configurações e clique em `Create rule`.

### **Concluido - Parte 2 de 2**

Agora, apenas para teste, volte nesta regra criada e edite para que ao invés de rodar às 06:00, rode a cada 1 minuto, assim fica mais fácil para ver se já está funcionando.
