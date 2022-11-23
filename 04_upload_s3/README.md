# **Anotações - Aula sobre Upload S3**

O intuito dessa aula é criar um script que faça o redimensionamento de uma imagem enviada com um tamanho pré definido.

## **Criando Buckets**

Acesse o serviço de S3 na AWS e crie 2 buckets, criei como `camargor-destino` e `camargor-origem`.

## **Criando uma Função Lambda**

Será criado basicamente da mesma forma que foi feito na seção sobre [EC2 e Lambda](../03_section_ec2_e_lambda/), porém com um nome diferente, o nome da lambda que será criado como exemplo é `thumbnail-website`, conforme feito na aula.

Selecione `Author from scratch`, a linguagem `Python` e mantenha a opção de criar uma role em `Permissions`, e então clique em `Create function`.

### **Definindo as permissões da lambda**

- Acesse a aba `Configuration`, acesse então o menu `Permissions` e clique no link na seção `Execution role`;
- Clique no símbolo de + na role da seção `Permission policies` e então clique em `Edit`;
- Clique em `JSON` então selecione tudo e cole o seguinte:
  
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
          "s3:GetObject"
        ],
        "Resource": "arn:aws:s3:::bucket/*"
      },
      {
        "Effect": "Allow",
        "Action": [
          "s3:PutObject"
        ],
        "Resource": "arn:aws:s3:::bucket/*"
      }
    ]
  }
  ```

- Altere o valor `bucket` da ação `s3:GetObject` pelo bucket que deu o nome semelhante a `camargor-origem`;
- Altere o valor `bucket` da ação `s3:PutObject` pelo bucket que deu o nome semelhante a `camargor-destino`;
- Clique em `Review policy`, e você deverá ver que esse JSON criado concede permissão ao CloudWatch e ao S3;
- Clique em `Save changes`, voltando a lambda em `Configuration > Permissions` na seção `Resource summary` já deverá ver o serviço S3 e o serviço CloudWatch.

### **Criando um Trigger**

O `trigger` é o que irá rodar a função lambda, no nosso caso queremos que ao subir algo no bucket de origem, seja acionado essa lambda.

- Na seção `Code`, clique no botão `Add trigger`;
- No menu `Select a source`, selecione a opção S3;
- Em `Bucket`, selecione a bucket de origem, no meu caso, `camargor-origem`;
- Em `Event type`, mantenha como `All object create events`;
- Clique no checkbox confirmando que tem ciência de que não está subindo algo na bucket de origem através da lambda que está usará esse `trigger`, pois isso faria ocorrer uma invocação recursiva, interminável;
- Clique em `Add`;

### **Adicionado a biblioteca Pillow na Lambda**

A lambda não vem com todas as bibliotecas existentes, a biblioteca pillow, por exemplo tem que ser adicionada na lambda, para isso você deve baixar a biblioteca e compactar em um arquivo .zip e então subir na AWS.

- Na seção `Code` da lambda, clique em `Upload from` e selecione `.zip file`;
- Suba o arquivo `Archive.zip` contida nesta pasta;
- Após isso, deverá ter o o projeto 100% configurado, apenas exclua a pasta `__MACOSX` clicando com o botão direito e então em `Delete`;
- Clique em `Deploy`;

### **Configurando as variáveis de ambiente**

- Acesse a opção do menu `Configuration`;
- No menu lateral esquerdo, acesso a opção `Environment variables`;
- Clique em `Edit`;
- Clique em `Add environment variable`;
- Adicione a variável `DEST_BUCKET` com o nome do bucket de destino, que no meu caso é `camargor-destino`;
- Clique em `Save`;

### **Testando**

Para testar, é legal já deixar aberto os buckets `camargor-origem` e `camargor-destino`, além disso é legal deixar aberto o CloudWatch para acompanhar a execução da lambda.

#### **Abrindo os logs**

- Acesse a opção do menu `Monitor`;
- Clique em `Logs`;
- Clique em `View logs in CloudWatch`;

#### **Testando subir uma imagem**

- Nesta pasta, tem uma imagem com o nome `iceland.jpeg`, acesse o bucket de origem e clique em `Upload` para subir esse arquivo;
- Após subir a imagem, você poderá acessar o `CloudWatch` e verá os logs da execução da lambda, além disso deverá conseguir ver o arquivo redimensionado na pasta destino.

> O código dessa aula está quebrado e apesar da abertura do questionamento de um aluno, o mesmo não foi respondido.
