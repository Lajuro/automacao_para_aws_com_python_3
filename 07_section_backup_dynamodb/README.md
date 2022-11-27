# **Anotações - Aula sobre backup de tabelas do DynamoDB**

O intuito dessa aula é criar um script que realize o backup de tabelas do DynamoDB e que é executado com agendamento.

## **Criar o banco de dados do DynamoDB**

- Acesse o serviço `DynamoDB`;
- Clique em `Create table`;
	- Em `Table name`, dê o nome para a sua tabela, no meu exemplo foi criado com o nome `carros`;
	- Em `Partition key`, dê o nome para campo que será a chave primária da tabela, no meu exemplo foi criado com o nome `name`;
	- Clique em `Create table`.
- Após a criação, vamos criar alguns itens, que seria uma linha da tabela, para isso, no menu lateral esquerdo procure por `Explore items`;
  - Selecione a tabela criada e então clique em `Create item`;
  - Defina um valor para o campo, no meu caso, para `name`, dei o nome `X1`;
  - Criei mais 2 outros carros na tabela, com os nomes `X3` e `X5`.

## **Criar a função Lambda**

- Acesse o serviço `Lambda`;
- Clique em `Create a function`;
  - Mantenha o tipo da lambda como `Author from scratch`;
  - Defina um nome, no meu caso, defini como `Backup_DynamoDB_carros`;
  - Defina um `runtime`, defini como `Python 3.9`;
  - Em `Change default execution role`, clique em `Create a new role with basic Lambda permissions`;
  - Clique em `Create function`.

### **Corrigindo as permissões**

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
        "Action": [
          "dynamodb:CreateBackup",
          "dynamodb:DeleteBackup",
          "dynamodb:ListBackups"
        ],
        "Effect": "Allow",
        "Resource": "*"
      }
    ]
  }
  ```

- Clique em `Review policy` e em seguida em `Save changes`.

### **Alterando o script**

- Na aba `Code`, você verá o código da lambda;
- Em `Code source`, selecione o arquivo `lambda_function.py`;
- Apague todo o conteúdo do arquivo;
- Coloque o seguinte script:
  
  ```python
  import boto3
  from datetime import datetime

  dynamo = boto3.client('dynamodb')

  # Verificar as tabelas
  def lambda_handler(event, context):
      if 'TableName' not in event:
          raise Exception("No table name specified.")
      table_name = event['TableName']

      create_backup(table_name)

  # Criar o backup da tabela
  def create_backup(table_name):
      print("Backing up table:", table_name)
      backup_name = table_name + '-' + datetime.now().strftime('%Y%m%d%H%M%S')

      response = dynamo.create_backup(
          TableName=table_name, BackupName=backup_name)

      print(f"Created backup {response['BackupDetails']['BackupName']}")
  ```

- Clique em `Deploy`.

### **Configurando o timeout da lambda**

- Na lambda criada, acesse a aba `Configuration`;
- No menu lateral esquerdo, selecione `General configuration`;
- Clique em `Edit`;
- Aumente o timeout para `2 minutos`;
- Clique em `Save`.

## **Configurando o agendamento de execução da lambda no CloudWatch**

- Acesse o serviço `CloudWatch`;
- No menu lateral esquerdo, acesse `Events` e clique em `Rules`;
- Clique no botão `Go to Amazon EventBridge`;
- Clique em `Create rule`;
- Defina um nome para essa regra, no meu caso dei o nome `Rule_Backup_DynamoDB_carros`;
- Defina uma descrição, no meu caso coloquei `Faz com que as instâncias EC2 parem às 20:00 de segunda-feira à terça-feira.`;
- Em `Rule type`, selecione a opção `Schedule`;
- Clique em `Next`;
- Em `Schedule`, selecione a opção `A schedule that runs at a regular rate, such as every 10 minutes`;
- Em `Rate expression`, defina o valor para `24` e selecione `Hours`;
- Clique em `Next`;
- Selecione o target como `Lambda function`;
- Selecione a lambda criada, no meu caso, `Backup_DynamoDB_carros`;
- Abra a seção `Additional settings`, em `Configure target input` selecione `Constant (JSON text)`;
- Em `Specify the constant in JSON`, coloque o seguinte:

  ``` json
  {
    "TableName": "carros"
  }
  ```

- Clique em `Next`;
- Clique em `Next` novamente;
- Revise as configurações e clique em `Create rule`.

## **Concluido**

Agora, apenas para teste, volte nesta regra criada e edite para que ao invés de rodar a cada 24 horas, rode a cada 1 minuto, assim fica mais fácil para ver se já está funcionando.

> Não esqueça de desativar após o teste, para que não haja cobranças indesejadas na AWS devido a quantidade de backups.
