# **Anotações - Aula sobre EC2 e Lambda**

O intuito dessa aula é criar uma forma de criação de instância EC2 de maneira fácil, com apenas um clique.

## **Criando uma Key Pair**

- Acesse o serviço EC2 no Console da AWS, no menu lateral esquerdo, clique em "Key Pairs" na seção "Network & Security";
- Após isso, clique no botão "Create key pair" para abrir o menu de criação da chave de pareamento;
- Dê um nome a sua chave, no meu caso, criei como `ec2-script`, e selecione o tipo de `key pair` como `RSA` e o formato de arquivo como `.ppk`;
- Clique em `Create key pair` e salve em um local seguro em seu computador, essa chave será usado posteriormente.

## **Criando uma Função Lambda**

- Acesse o serviço de Lambda no Console da AWS e clique em `Create a function`;
- Selecione `Author from scratch`;
- Defina um nome para a função, no meu caso, criei como `Create-EC2-t2micro-linux`;
- Selecione a linguagem como `Python`;
- Selecione a arquitetura como `x86_64`;
- Em permissões, selecione a opção `Create a new role with basic Lambda permissions`;
- Clique em `Create function`;

### **Definindo Permissões da Lambda**

- Na lambda criada, clique em `Configuration` e em seguida clique em `Permissions`;
- Na seção `Execution role`, clique no nome da `role` que foi gerado (O nome da role criada é basicamente o nome da função seguido de uma hash única);
- Na seção `Permissions policies`, abra a policy criada (clicando no símbolo de +) e clique em `Edit`;
- Clique na aba `JSON`;
- Cole o seguinte JSON:
  
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
        "Action": [
          "ec2:RunInstances"
        ],
        "Effect": "Allow",
        "Resource": "*"
      }
    ]
  }
  ```

- Em seguida, clique em `Review policy` e em seguida `Save changes`;
- Agora, acessando a tela de permissões da lambda, você deverá conseguir ver em `Resource summary` os dois serviços (CloudWatch e EC2);

#### **Adicionando o código Python**

Na aba `Code`, você deverá encontrar o código por trás da sua lambda, no arquivo `lambda_function.py` apague tudo e adicione o seguinte código:

```python
import os
import boto3

AMI = os.environ['AMI']
INSTANCE_TYPE = os.environ['INSTANCE_TYPE']
KEY_NAME = os.environ['KEY_NAME']
SUBNET_ID = os.environ['SUBNET_ID']

ec2 = boto3.resource('ec2')


def lambda_handler(event, context):

  instance = ec2.create_instances(
    ImageId=AMI,
    InstanceType=INSTANCE_TYPE,
    KeyName=KEY_NAME,
    SubnetId=SUBNET_ID,
    MaxCount=1,
    MinCount=1
  )

  print("New instance created:", instance[0].id)
```

Observe que no código é armazenado em variáveis, valores que são resgatados de variáveis de ambiente, essas variáveis de ambiente podem ser configuradas acessando o menu `Configuration`, e então acesse a opção do menu com o nome `Environment variables`.

Clique em `Edit` e então em `Add environment variable` para começar a adicionar as variáveis de ambiente.

Adicione as seguintes variáveis:

- AMI: O ID da AMI que deseja usar para sua instância EC2, no exemplo foi usado a `ami-0b7101e993ea27f3a`;
- INSTANCE_TYPE: O tipo da instância que deseja usar, no exemplo será usado a `t2.micro`;
- KEY_NAME: Nome da `key pair` criada, no exemplo foi `ec2-script`;
- SUBNET_ID: O ID da subnet que pode ser encontrada ao acessar o serviço `VPC` e em `subnets`, clicando em qualquer uma você deverá encontrar a `Subnet ID`.

Clique em `Save`.

#### **Deploy da Lambda**

Na aba `Code`, clique no botão `Deploy`, você então deverá ver uma mensagem de sucesso dizendo algo como o seguinte: *Successfully updated the function Create-EC2-t2micro-linux.*

### **Testando**

Acesse primeiramente o serviço EC2 da AWS e caso não tenha nada configurado por lá, deverá ver uma lista vazia de instâncias, caso não, apenas verifique como ela está nesse momento, pois isso irá mudar dentro de instantes.

#### **Configurando Novo Evento de Teste**

- Na aba `Test`, selecione a opção `Create new event`;
- Defina um nome para esse teste, no meu caso dei o nome `teste_criar_ec2`;
- Em `Event sharing settings`, deixe selecionado `Private`;
- Em `Template`, deixe selecionado `hello-world`;
- No `Event JSON`, pode deixar vazio como `{}`;
- Clique em `Save` na parte superior do teste que está criando;

#### **Rodando o Teste**

Para rodar o teste criado, você pode fazer pela própria aba `Test`, deixando o teste criado selecionado em `Event name` e então clicando no botão `Test`, ou então na aba `Code`, na seta ao lado do botão `Test`, selecione o teste criado e então clique no botão `Test`.

### **Sucesso**

Após rodar o teste, será criado uma instância, você pode conferir indo no serviço `EC2`.

Para conferir os logs, na lambda, clique em `Monitor` e em seguida clique em `Logs`, clique no botão `View logs in CloudWatch`, você será direcionado para o grupo de logs da lambda, em seguida clique no último log executado e ali então você verá o log da sua última execução da lambda.
