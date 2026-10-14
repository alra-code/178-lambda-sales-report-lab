<div align="center">

![header](https://capsule-render.vercel.app/api?type=waving&color=232F3E&height=200&section=header&text=Lambda%20Sales%20Report&fontSize=46&fontColor=FFFFFF&animation=fadeIn&desc=Camadas%2C%20fun%C3%A7%C3%B5es%20encadeadas%2C%20SNS%20e%20agendamento%20com%20EventBridge&descSize=17&descAlignY=72)

![Lambda](https://img.shields.io/badge/AWS-Lambda-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)
![SNS](https://img.shields.io/badge/Amazon-SNS-D6336C?style=for-the-badge&logo=amazonaws&logoColor=white)
![EventBridge](https://img.shields.io/badge/Amazon-EventBridge-E7157B?style=for-the-badge&logo=amazonaws&logoColor=white)
![IAM](https://img.shields.io/badge/AWS-IAM-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white)
![Systems Manager](https://img.shields.io/badge/Systems_Manager-Parameter_Store-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![AWS CLI](https://img.shields.io/badge/AWS_CLI-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/Status-Conclu%C3%ADdo-2ea44f?style=for-the-badge)

**Um relatório diário de vendas gerado por duas funções Lambda, enviado por e-mail e disparado por agenda** ☕📊📧

</div>

---

## 📌 Sobre o projeto

Neste laboratório do **AWS re/Start**, construí o fluxo de um **relatório de análise de vendas** de uma cafeteria, usando **duas funções AWS Lambda** que trabalham juntas:

| ⚡ Função | 📝 O que faz |
|---|---|
| **`salesAnalysisReportDataExtractor`** | Conecta ao banco **MySQL** (em uma instância EC2 dentro de uma VPC) e extrai os dados dos pedidos |
| **`salesAnalysisReport`** | Lê as credenciais no **Parameter Store**, **invoca a função extratora**, formata o relatório e o publica em um tópico do **SNS**, que envia o **e-mail** |

O relatório é disparado **automaticamente por agenda**, com uma regra do **Amazon EventBridge** (expressão cron).

O laboratório também envolveu **troubleshooting**: a função extratora deu *timeout* por causa de uma **porta bloqueada**, e a função do relatório falhou por um **ARN de tópico incorreto**.

---

## 🎯 Objetivos

- ✅ Entender os **perfis IAM** e as permissões de cada função Lambda
- ✅ Criar uma **camada (layer)** do Lambda para uma biblioteca externa (PyMySQL)
- ✅ Criar uma função Lambda **dentro de uma VPC** para acessar um banco de dados
- ✅ Diagnosticar e corrigir um **timeout** pelos logs e pelo grupo de segurança
- ✅ Criar um **tópico SNS** com assinatura de e-mail
- ✅ Criar uma função Lambda pela **AWS CLI**
- ✅ Configurar **variáveis de ambiente**
- ✅ Agendar a execução com um **gatilho do EventBridge**

---

## 🗺️ Arquitetura

```mermaid
flowchart LR
    EB["⏰ Amazon EventBridge<br/>regra agendada (cron)"] -->|"aciona"| R["⚡ Lambda<br/>salesAnalysisReport"]

    R -->|"lê credenciais"| SSM["🔑 Systems Manager<br/>Parameter Store<br/>/cafe/dbUrl · dbName · dbUser · dbPassword"]
    R -->|"invoca"| X["⚡ Lambda<br/>salesAnalysisReportDataExtractor<br/>+ camada pymysqlLibrary"]

    subgraph VPC["🔷 Cafe VPC · sub-rede pública"]
        X -->|"MySQL · porta 3306"| DB["🗄️ EC2 CafeInstance<br/>banco de dados MySQL"]
    end

    R -->|"publish"| SNS["📣 Amazon SNS<br/>salesAnalysisReportTopic"]
    SNS --> E["📧 E-mail<br/>Daily Sales Analysis Report"]

    style EB fill:#E7157B,color:#fff,stroke:#b01060
    style R fill:#FF9900,color:#000,stroke:#cc7a00
    style X fill:#FF9900,color:#000,stroke:#cc7a00
    style SSM fill:#8C4FFF,color:#fff,stroke:#6f3fd1
    style DB fill:#4479A1,color:#fff,stroke:#2f5877
    style SNS fill:#D6336C,color:#fff,stroke:#a8264f
    style E fill:#0d6efd,color:#fff,stroke:#0a58ca
```

---

## 🧰 Serviços utilizados

| 🎨 Categoria | 🔧 Serviço | 📝 Função no projeto |
|:---:|:---:|---|
| ⚡ Compute | **AWS Lambda** | Duas funções em Python, uma camada e o fluxo do relatório |
| 🔐 Segurança | **AWS IAM** | Perfis `salesAnalysisReportRole` e `salesAnalysisReportDERole` |
| 🔑 Configuração | **Systems Manager Parameter Store** | Guarda as credenciais do banco de dados |
| 🛡️ Rede | **VPC e Security Group** | Conexão da função com o banco; regra da porta 3306 |
| 🗄️ Banco | **MySQL em EC2** | Armazena os pedidos da cafeteria |
| 📣 Mensageria | **Amazon SNS** | Entrega o relatório por e-mail |
| ⏰ Agendamento | **Amazon EventBridge** | Aciona a função por expressão cron |
| ⌨️ Automação | **AWS CLI** | Criação da função `salesAnalysisReport` |
| 📊 Monitoramento | **CloudWatch Logs** | Registros das execuções |

---

## ⚙️ Configuração utilizada

| 🔹 Recurso | 🔸 Valor |
|---|---|
| **Região** | `us-west-2` (Oregon) |
| **Camada** | `pymysqlLibrary` · versão 1 · biblioteca PyMySQL |
| **Função extratora** | `salesAnalysisReportDataExtractor` · Python 3.12 · handler `salesAnalysisReportDataExtractor.lambda_handler` |
| **Perfil da extratora** | `salesAnalysisReportDERole` |
| **Rede da extratora** | Cafe VPC · Cafe Public Subnet 1 · `CafeSecurityGroup` |
| **Função do relatório** | `salesAnalysisReport` · criada pela AWS CLI · handler `salesAnalysisReport.lambda_handler` |
| **Perfil do relatório** | `salesAnalysisReportRole` |
| **Variável de ambiente** | `topicARN`: ARN do tópico SNS |
| **Tópico SNS** | `salesAnalysisReportTopic` · tipo **Standard** · nome de exibição `SARTopic` |
| **Assinatura** | Protocolo **Email** (confirmada) |
| **Gatilho** | EventBridge (regra `salesAnalysisReportDailyTrigger`) com expressão cron |
| **Cron de produção** | `cron(0 20 ? * MON-SAT *)`: segunda a sábado, às 20h UTC |

---

## 🪜 Passo a passo resumido

### 🔐 1. Perfis IAM
1. Revisar a **relação de confiança** e as **políticas** dos perfis `salesAnalysisReportRole` e `salesAnalysisReportDERole`

### ⚡ 2. Função extratora
2. Criar a **camada** `pymysqlLibrary` com o `.zip` da biblioteca
3. Criar a função `salesAnalysisReportDataExtractor` com o perfil `salesAnalysisReportDERole`
4. Adicionar a camada, definir o **handler** e importar o código
5. Configurar a **VPC**, a sub-rede e o `CafeSecurityGroup`

### 🧪 3. Teste e troubleshooting
6. Copiar os parâmetros do banco do **Parameter Store**
7. Testar e observar o erro de **timeout**
8. Liberar a **porta 3306** no grupo de segurança e testar de novo (resultado com `body` vazio)
9. Fazer **pedidos** no site da cafeteria e testar de novo (resultado com dados)

### 📣 4. Notificações
10. Criar o tópico **SNS** e confirmar a assinatura de e-mail

### 📊 5. Função do relatório
11. Configurar a **AWS CLI** no CLI Host e criar a função `salesAnalysisReport` com `create-function`
12. Definir a variável de ambiente `topicARN`
13. Testar a função, corrigir o erro de endpoint e receber o **e-mail**
14. Criar o **gatilho do EventBridge** com a expressão cron e receber o e-mail disparado pela agenda

---

## 📸 Evidências

## 🔐 Tarefa 1: Perfis IAM

### 1️⃣ Relação de confiança
O perfil `salesAnalysisReportRole` confia no serviço **`lambda.amazonaws.com`**, o que permite que o Lambda use esse perfil.

![Relação de confiança](./images/01-iam-role-sar-confianca.png)

### 2️⃣ Permissões do `salesAnalysisReportRole`
Quatro políticas: acesso ao **SNS**, leitura do **Systems Manager**, logs do **CloudWatch** e permissão para **invocar outra função Lambda**.

![Permissões do relatório](./images/02-iam-role-sar-permissoes.png)

### 3️⃣ Permissões do `salesAnalysisReportDERole`
Duas políticas: logs do **CloudWatch** e acesso de rede a uma **VPC** (`AWSLambdaVPCAccessRunRole`).

![Permissões da extratora](./images/03-iam-role-sarde-permissoes.png)

## ⚡ Tarefa 2: Camada e função extratora

### 4️⃣ Camada criada
A camada **`pymysqlLibrary`**, versão 1, com a mensagem de criação bem-sucedida e o runtime compatível.

![Camada criada](./images/04-lambda-camada-pymysql.png)

### 5️⃣ Camada adicionada à função
A função `salesAnalysisReportDataExtractor` com **Layers (1)** e a camada `pymysqlLibrary` listada.

![Função com camada](./images/05-lambda-extratora-camada.png)

![Camada na lista](./images/05-1-lambda-extratora-camada.png)

### 6️⃣ Handler e código
O **handler** `salesAnalysisReportDataExtractor.lambda_handler` e o código importado, que recebe `dbUrl`, `dbName`, `dbUser` e `dbPassword` pelo evento e se conecta ao MySQL com PyMySQL.

![Handler](./images/06-1-lambda-extratora-codigo.png)

![Código da função](./images/06-lambda-extratora-codigo.png)

### 7️⃣ Configuração de VPC
A função na **Cafe VPC**, na **Cafe Public Subnet 1** (`us-west-2a`) e com o `CafeSecurityGroup`, para alcançar o banco de dados.

![VPC da função](./images/07-lambda-extratora-vpc.png)

## 🧪 Tarefa 3: Teste e troubleshooting

### 8️⃣ Parâmetros do banco
O Parameter Store com os parâmetros `/cafe/dbUrl`, `/cafe/dbName`, `/cafe/dbUser` e `/cafe/dbPassword`. Os valores não são exibidos.

![Parameter Store](./images/08-ssm-parametros.png)

### 9️⃣ Primeiro teste: timeout
O teste falhou com **`Task timed out after 3.00 seconds`**. A função tentava se conectar ao MySQL, mas a conexão nunca se completava.

![Timeout](./images/09-lambda-teste-timeout.png)

### 🔟 Correção: regra da porta 3306
O `CafeSecurityGroup` ganhou uma regra de entrada **MYSQL/Aurora (TCP 3306)**, com o próprio grupo de segurança como origem, o que libera a conexão apenas entre os recursos que o usam.

![Regra 3306](./images/10-sg-regra-3306.png)

### 1️⃣1️⃣ Teste com sucesso, sem dados
Depois da correção, o teste passou, com `statusCode: 200`. O campo `body` veio **vazio** porque o banco ainda não tinha pedidos.

![Sucesso com body vazio](./images/11-lambda-teste-sucesso-vazio.png)

### 1️⃣2️⃣ Pedidos no site da cafeteria
Fiz pedidos pelo site para preencher o banco. A página de confirmação mostra o número do pedido, os itens e o valor total.

![Pedido no site](./images/12-site-cafe-pedidos.png)

### 1️⃣3️⃣ Teste com sucesso e dados
Agora o `body` traz os produtos e quantidades (por exemplo, **Pastries / Croissant / 2**), comprovando que a função lê o banco.

![Sucesso com dados](./images/13-lambda-teste-sucesso-dados.png)

## 📣 Tarefa 4: SNS

### 1️⃣4️⃣ Tópico criado
O tópico **`salesAnalysisReportTopic`**, tipo **Standard**, com o nome de exibição `SARTopic`.

![Tópico SNS](./images/14-sns-topico-sar.png)

### 1️⃣5️⃣ Assinatura confirmada
A assinatura por **e-mail** com o status **Confirmed**.

![Assinatura confirmada](./images/15-sns-assinatura-confirmada.png)

## 📊 Tarefa 5: Função do relatório

### 1️⃣6️⃣ AWS CLI configurada
A AWS CLI configurada no **CLI Host** (região `us-west-2`, formato `json`).

![CLI configurada](./images/16-cli-configure.png)

### 1️⃣7️⃣ Função criada pela CLI
O comando `aws lambda create-function` retornou os atributos da função `salesAnalysisReport`: handler, runtime, memória e estado **Pending** (em criação).

![create-function](./images/17-cli-create-function.png)

### 1️⃣8️⃣ Variável de ambiente
A variável **`topicARN`** com o ARN do tópico SNS.

![Variável de ambiente](./images/18-lambda-sar-variavel-ambiente.png)

### 1️⃣9️⃣ Erro de endpoint do SNS
O primeiro teste falhou com **`Invalid endpoint: https://sns..amazonaws.com`**: a região do endpoint estava **vazia**. A função lê a região do próprio ARN do tópico, e o valor da variável `topicARN` estava incorreto.

![Erro de endpoint](./images/19-1-lambda-sar-erro-invalid-endpoint.png)

### 2️⃣0️⃣ Teste com sucesso
Depois de corrigir o ARN na variável `topicARN`, o teste passou e retornou **`Sale Analysis Report sent.`**

![Teste com sucesso](./images/19-lambda-sar-teste-sucesso.png)

### 2️⃣1️⃣ E-mail com o relatório de vendas
O e-mail **Daily Sales Analysis Report**, enviado pelo tópico `SARTopic`, com os itens vendidos agrupados por categoria.

![E-mail do relatório](./images/20-email-relatorio-vendas.png)

### 2️⃣2️⃣ Gatilho do EventBridge
A função `salesAnalysisReport` com o **EventBridge** conectado como gatilho.

![Gatilho EventBridge](./images/21-lambda-sar-gatilho-eventbridge.png)

### 2️⃣3️⃣ E-mail disparado pela agenda
Um novo e-mail do relatório, agora disparado **automaticamente** pela regra do EventBridge, com os dados atualizados dos pedidos.

![E-mail agendado](./images/22-email-relatorio-agendado.png)

---

## 🐞 Dificuldades encontradas

| 🔴 Erro | 🔍 Causa | ✅ Solução |
|---|---|---|
| `Task timed out after 3.00 seconds` na função extratora | O grupo de segurança não permitia a porta **3306** do MySQL, então a conexão com o banco nunca se completava | Adicionar uma regra de entrada **MYSQL/Aurora (3306)**, com o próprio `CafeSecurityGroup` como origem |
| `Invalid endpoint: https://sns..amazonaws.com` na função do relatório | O ARN do tópico na variável `topicARN` estava incorreto, e a **região** (lida do ARN) ficou vazia | Copiar o ARN completo do tópico SNS e colá-lo na variável `topicARN` |
| `body` vazio no resultado da extratora | O banco de dados ainda não tinha pedidos | Fazer pedidos no site da cafeteria e testar de novo |

---

## 🧠 Aprendizados

- 🔐 Cada função Lambda precisa de um **perfil IAM** com a relação de confiança e as permissões certas
- 🧩 **Camadas** permitem reutilizar bibliotecas sem incluí-las no pacote de cada função
- 🛡️ Uma função Lambda em uma **VPC** depende das regras do **Security Group** para alcançar outros recursos
- ⏱️ Um *timeout* de conexão muitas vezes indica **bloqueio de rede**, e não erro de código
- 🧭 Ler a **pilha de erros** ajuda: o erro `sns..amazonaws.com` apontou para uma região vazia e levou direto à variável incorreta
- 🔑 O **Parameter Store** guarda configurações e credenciais fora do código
- 🔗 Uma função pode **invocar outra**, montando um fluxo em etapas
- ⏰ O **EventBridge** agenda execuções com **cron**, sempre em **UTC**
- 📣 O SNS só entrega e-mail depois que a assinatura é **confirmada**

---

## 🔐 Observação sobre os prints

> ⚠️ Os recursos que aparecem nas imagens pertencem a um **ambiente temporário de laboratório**, já encerrado. IDs de conta, ARNs, credenciais e endereços de e-mail foram cobertos. Em projetos reais, evite publicar IDs de conta, ARNs, endereços de e-mail e chaves de acesso.

---

<div align="center">

### 👩‍💻 Autora

**LUCIANA RB DE ASSIS**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/lucianarbassis/)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/alra-code)

⭐ Se este projeto foi útil, deixe uma estrela no repositório!

![footer](https://capsule-render.vercel.app/api?type=waving&color=232F3E&height=100&section=footer)

</div>
