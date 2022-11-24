# **Anotações - Aula sobre Backup EC2**

O intuito dessa aula é criar um script que faça o backup de uma EC2.

## **Criando uma EC2 com a tag de backup**

- Caso não tenha uma instância criada:
  - Acesse o serviço de EC2;
  - Crie uma instância;
  - Na tela de tags, adicione uma tag com o nome `backup` e o valor como `true`.
- Caso tenha uma instância já criada
  - Abra a instância clicando nela;
  - Acesse a aba `Tags`;
  - Clique em `Manage tags`;
  - Clique em `Add new tag`;
  - Adicione uma tag com o nome `backup` e o valor como `true`;
  - Clique em `Save`.

## **Criando uma Lambda**

- Acesse o serviço `Lambda`;
- Clique em `Create a function`;
  - Mantenha o tipo da lambda como `Author from scratch`;
  - Defina um nome, no meu caso, defini como `EC2-Snapshot`;
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
    "Statement": [{
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
          "ec2:CreateSnapshot",
          "ec2:CreateTags",
          "ec2:DeleteSnapshot",
          "ec2:Describe*",
          "ec2:ModifySnapshotAttribute",
          "ec2:ResetSnapshotAttribute"
        ],
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
  from datetime import datetime
  import boto3

  def lambda_handler(event, context):
    # Listando regioes
    ec2_client = boto3.client('ec2')
    regions = [region['RegionName'] for region in ec2_client.describe_regions()['Regions']]

    for region in regions:

      print('Instances in EC2 Region {0}:'.format(region))
      ec2 = boto3.resource('ec2', region_name=region)

      instances = ec2.instances.filter(
        Filters=[
          {'Name': 'tag:backup', 'Values': ['true']}
        ]
      )

      # Adicionando timestamp
      timestamp = datetime.utcnow().replace(microsecond=0).isoformat()

      for i in instances.all():
        for v in i.volumes.all():

          desc = 'Backup of {0}, volume {1}, created {2}'.format(
            i.id, v.id, timestamp)
          print(desc)

          snapshot = v.create_snapshot(Description=desc)

          print("Created snapshot:", snapshot.id)
  ```

- Clique em `Deploy`.

## **Configurando o agendamento de execução da lambda no CloudWatch**

- Acesse o serviço `CloudWatch`;
- No menu lateral esquerdo, acesse `Events` e cliquem em `Rules`;
- Clique no botão `Go to Amazon EventBridge`;
- Clique em `Create rule`;
- Defina um nome para essa regra, no meu caso dei o nome `EC2-Snapshot-1dia`;
- Defina uma descrição, no meu caso coloquei `Realiza o backup das instâncias EC2 a cada 24 horas.`;
- Clique em `Next`;
- Em `Schedule`, selecione a opção `A schedule that runs at a regular rate, such as every 10 minutes`;
- Em `Rate expression`, defina o valor para `2` e selecione `Minutes`;
- Clique em `Next`;
- Selecione o target como `Lambda function`;
- Selecione a lambda criada, no meu caso, `EC2-Snapshot`;
- Clique em `Next`;
- Clique em `Next` novamente;
- Revise as configurações e clique em `Create rule`.

## **Configurando o timeout da lambda**

- Na lambda criada, acesse a aba `Configuration`;
- No menu lateral esquerdo, selecione `General configuration`;
- Clique em `Edit`;
- Aumente o timeout para `1 minuto`;
- Clique em `Save`.

## **Concluido**

Acessando as snapshots de EC2 você verá que está criando com sucesso, e inclusive você deverá conseguir ver os logs no CloudWatch.

## **Importante**

Como esse script foi feito para testes, é importante que seja excluido a lambda e o evento que foi criado no EventBridge, para que não seja cobrado pelas snapshots.
