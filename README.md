# 🚀 Laboratório DevOps - Projeto 1: Containerização com Docker e Deploy Manual na AWS

## 📋 Índice
1. [Visão Geral](#visão-geral)
2. [Pré-requisitos](#pré-requisitos)
3. [Arquitetura do Projeto](#arquitetura-do-projeto)
4. [Fase 1: Preparação do Ambiente Local](#fase-1-preparação-do-ambiente-local)
5. [Fase 2: Containerização com Docker](#fase-2-containerização-com-docker)
6. [Fase 3: Teste Local do Container](#fase-3-teste-local-do-container)
7. [Fase 4: Configuração do Amazon ECR](#fase-4-configuração-do-amazon-ecr)
8. [Fase 5: Push da Imagem para o ECR](#fase-5-push-da-imagem-para-o-ecr)
9. [Fase 6: Provisionamento da Instância EC2](#fase-6-provisionamento-da-instância-ec2)
10. [Fase 7: Deploy na EC2](#fase-7-deploy-na-ec2)
11. [Verificação e Testes](#verificação-e-testes)
12. [Limpeza de Recursos](#limpeza-de-recursos)

---

## 🎯 Visão Geral

### O que vamos construir?
Neste projeto, compartilho o passo a passo de como realizei a containerização de um website estático (HTML, CSS e JS) utilizando Docker e como fiz o deploy manual em uma instância EC2 na AWS, gerenciando minhas imagens através do Amazon ECR.

### Por que isso é importante?
- **Portabilidade**: Seu site funcionará da mesma forma em qualquer ambiente
- **Isolamento**: Elimina problemas de "funciona na minha máquina"
- **Escalabilidade**: Base para futuras implementações mais complexas
- **Padrão da Indústria**: Docker é amplamente utilizado no mercado

### Tempo estimado: 2-3 horas

---

## 🔧 Pré-requisitos

### Ferramentas Necessárias

#### 1. **Docker Desktop**
#### 2. **AWS CLI**
#### 3. **Conta AWS**
#### 4. **Editor de Código**

### Estrutura do Projeto
```
meu-projeto/
├── website/
│   ├── index.html
│   ├── styles.css
│   ├── script.js
│   └── assets/
│       └── (imagens, fontes, etc.)
└── Dockerfile (vamos criar)
```

---

## 🏗️ Arquitetura do Projeto

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Código Local   │────▶│   Docker Image  │────▶│    Amazon ECR   │
│  (HTML/CSS/JS)  │     │   (Container)   │     │   (Registry)    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                          │
                                                          ▼
                        ┌─────────────────┐     ┌─────────────────┐
                        │    Browser      │◀────│    Amazon EC2   │
                        │  (User Access)  │     │   (Container)   │
                        └─────────────────┘     └─────────────────┘
```

---

## 📦 Fase 1: Preparação do Ambiente Local

### Passo 1.1: Verificar estrutura do projeto

Navegue até o diretório do seu projeto:
```bash
cd caminho/para/seu/projeto
ls -la
```

Você deve ver a pasta `website/` com seus arquivos:
```bash
ls -la website/
```


## 🐳 Fase 2: Containerização com Docker

### Passo 2.1: Criar o Dockerfile

Na raiz do projeto (mesmo nível da pasta `website/`), crie um arquivo chamado `Dockerfile`:


### Passo 2.2: Escrever o Dockerfile

Abra o Dockerfile no seu editor e adicione:

```dockerfile
# Imagem base - Nginx Alpine (leve e eficiente)
FROM nginx:alpine

# Copia os arquivos do website para o diretório do Nginx
COPY website/ /usr/share/nginx/html/

# Expõe a porta 80 (documentação - não abre a porta realmente)
EXPOSE 80

# Comando padrão quando o container iniciar
CMD ["nginx", "-g", "daemon off;"]
```


### Passo 2.3: Construir a imagem Docker

No terminal, na raiz do projeto, execute:

```bash
docker build -t meu-website:v1.0 .
```
### Passo 2.4: Verificar a imagem criada

```bash
docker images
```

Você deve ver sua imagem listada:
```
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
meu-website    v1.0      abc123def456   30 seconds ago   23.5MB
```

## 🧪 Fase 3: Teste Local do Container

### Passo 3.1: Executar o container localmente

```bash
docker run -d -p 8080:80 --name meu-website-container meu-website:v1.0
```


### Passo 3.2: Verificar se o container está rodando

```bash
docker ps
```

Você verá algo como:
```
CONTAINER ID   IMAGE              COMMAND                  CREATED         STATUS         PORTS                  NAMES
xyz789abc123   meu-website:v1.0   "nginx -g 'daemon..."   10 seconds ago  Up 9 seconds   0.0.0.0:8080->80/tcp   meu-website-container
```

### Passo 3.3: Testar no navegador

Abra seu navegador e acesse:
```
http://localhost:8080
```

## ☁️ Fase 4: Configuração do Amazon ECR

### Passo 4.1: Acessar o Console AWS

1. Acesse [console.aws.amazon.com](https://console.aws.amazon.com)
2. Faça login com suas credenciais


### Passo 4.2: Navegar para o ECR

1. Na barra de busca superior, digite "ECR"
2. Clique em "Elastic Container Registry"

### Passo 4.3: Criar um repositório

1. Clique em "Create repository"
2. Configure:
   - **Visibility settings**: Private
   - **Repository name**: `meu-website`
   - **Tag immutability**: Disabled (padrão)
   - **Scan on push**: Enabled (recomendado para segurança)
3. Clique em "Create repository"


### Passo 4.4: Anotar a URI do repositório

Após criar, você verá algo como:
```
123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website
```

⚠️ **Importante**: Copie e guarde esta URI, você precisará dela!


## 📤 Fase 5: Push da Imagem para o ECR

### Passo 5.1: Configurar AWS CLI

Se ainda não configurou, execute:
```bash
aws configure
```

Você precisará fornecer:
- **AWS Access Key ID**: Obtida no IAM
- **AWS Secret Access Key**: Obtida no IAM
- **Default region**: ex: us-east-1
- **Default output format**: json

### Passo 5.2: Autenticar Docker com ECR

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
```

⚠️ **Substitua**: 
- `us-east-1` pela sua região
- `123456789012` pelo seu Account ID

Você deve ver:
```
Login Succeeded
```


### Passo 5.3: Tagar a imagem para o ECR

```bash
docker tag meu-website:v1.0 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
```

### Passo 5.4: Push da imagem

```bash
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
```

Você verá o progresso do upload:
```
The push refers to repository [123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website]
abc123: Pushed
def456: Pushed
v1.0: digest: sha256:xyz789... size: 1234
```

### Passo 5.5: Verificar no Console AWS

1. Volte ao ECR no console AWS
2. Clique no seu repositório
3. Você deve ver a imagem com a tag v1.0


---

## 🖥️ Fase 6: Provisionamento da Instância EC2

### Passo 6.1: Navegar para EC2

1. No console AWS, busque por "EC2"
2. Clique em "EC2"

### Passo 6.2: Lançar instância

1. Clique em "Launch Instance"
2. Configure:

#### Nome e tags
- **Name**: `meu-website-server`

#### Imagem de aplicação e sistema operacional
- **AMI**: Amazon Linux 2023 (Free tier)

#### Tipo de instância
- **Instance type**: t3.micro (Free tier)


#### Par de chaves
- Clique em "Create new key pair"
- **Key pair name**: `meu-website-key`
- **Key pair type**: RSA
- **Private key file format**: .pem (Linux/Mac) ou .ppk (Windows/PuTTY)
- Clique em "Create key pair" e salve o arquivo

⚠️ **IMPORTANTE**: Guarde este arquivo com segurança! Você precisará dele para acessar a EC2.


#### Configurações de rede
- **VPC**: Default
- **Subnet**: No preference
- **Auto-assign public IP**: Enable
- **Firewall (security groups)**: Create security group
  - **Security group name**: `meu-website-sg`
  - **Description**: Security group for website

#### Regras do Security Group
Adicione as seguintes regras:

| Type | Protocol | Port Range | Source |
|------|----------|------------|--------|
| SSH  | TCP      | 22         | My IP  |
| HTTP | TCP      | 80         | 0.0.0.0/0 |


#### Configurar armazenamento
- **Volume**: 8 GiB gp3 (padrão)

### Passo 6.3: Configurar IAM Role (Permissões para ECR)

#### Criar IAM Role
1. Em "Advanced details", encontre "IAM instance profile"
2. Clique em "Create new IAM profile"
3. Ou vá para IAM Console e:
   - Clique em "Roles" → "Create role"
   - **Trusted entity**: AWS service
   - **Use case**: EC2
   - **Permissions**: Adicione `AmazonEC2ContainerRegistryReadOnly`
   - **Role name**: `EC2-ECR-Role`


4. Volte para a configuração da EC2 e selecione o role criado

### Passo 6.4: Revisar e lançar

1. Revise todas as configurações
2. Clique em "Launch instance"
3. Aguarde a instância inicializar (status: running)


### Passo 6.5: Anotar informações importantes

Anote:
- **Public IP**: Ex: 54.123.45.67
- **Instance ID**: Ex: i-0abc123def456789

---

## 🚀 Fase 7: Deploy na EC2

### Passo 7.1: Conectar à instância EC2

#### No Linux

# Conectar via SSH
ssh -i meu-website-key.pem ec2-user@54.123.45.67
```


Você verá:
```
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
[ec2-user@ip-172-31-xx-xx ~]$
```

*[Espaço para print: Conexão SSH estabelecida]*

### Passo 7.2: Instalar Docker na EC2

```bash
# Atualizar pacotes
sudo yum update -y

# Instalar Docker
sudo yum install docker -y

# Iniciar serviço Docker
sudo systemctl start docker

# Habilitar Docker no boot
sudo systemctl enable docker

# Adicionar ec2-user ao grupo docker
sudo usermod -a -G docker ec2-user

# Verificar instalação
docker --version
```

### Passo 7.3: Fazer logout e login novamente

```bash
# Sair
exit

# Conectar novamente
ssh -i meu-website-key.pem ec2-user@54.123.45.67
```

### Passo 7.4: Autenticar Docker com ECR na EC2

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
```


### Passo 7.5: Pull da imagem do ECR

```bash
docker pull 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
```

Você verá:
```
v1.0: Pulling from meu-website
Status: Downloaded newer image for 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
```


### Passo 7.6: Executar o container

```bash
docker run -d -p 80:80 --name meu-website-prod --restart always 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
```

### Passo 7.7: Verificar se está rodando

```bash
# Verificar container
docker ps

# Verificar logs
docker logs meu-website-prod
```

---

## ✅ Verificação e Testes

### Teste 1: Acessar pelo navegador

1. Abra seu navegador
2. Digite o IP público da EC2 + Porta: `http://54.123.45.67:80`
3. Seu website deve aparecer! 🎉


### Teste 2: Verificar logs na EC2

```bash
# Logs do container
docker logs -f meu-website-prod

# Status do container
docker stats meu-website-prod
```

### Teste 3: Testar reinicialização

```bash
# Parar o container
docker stop meu-website-prod

# Verificar se parou
docker ps

# Iniciar novamente
docker start meu-website-prod

# Verificar se voltou
docker ps
```

---

## 🔧 Troubleshooting

### Problema 1: "Cannot connect to the Docker daemon"

**Solução**:
```bash
sudo systemctl start docker
sudo usermod -a -G docker $USER
# Fazer logout e login novamente
```

### Problema 2: Site não abre no navegador

**Verificações**:
1. Security Group tem porta 80 aberta?
2. Container está rodando? (`docker ps`)
3. IP público está correto?
4. Teste com curl na EC2: `curl localhost`

### Problema 3: "No basic auth credentials" no pull do ECR

**Solução**:
```bash
# Re-autenticar
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin [ECR_URI]
```

### Problema 4: Permissão negada no Docker

**Solução**:
```bash
# Adicionar usuário ao grupo docker
sudo usermod -a -G docker ec2-user
# Logout e login
exit
ssh -i key.pem ec2-user@IP
```

---

## 🧹 Limpeza de Recursos

⚠️ **IMPORTANTE**: Para evitar custos, limpe os recursos após o laboratório!

### Passo 1: Parar e remover container na EC2

```bash
docker stop meu-website-prod
docker rm meu-website-prod
docker rmi 123456789012.dkr.ecr.us-east-1.amazonaws.com/meu-website:v1.0
```

### Passo 2: Terminar instância EC2

1. Console AWS → EC2
2. Selecione sua instância
3. Actions → Instance State → Terminate

*[Espaço para print: Confirmação de terminate]*

### Passo 3: Deletar imagem do ECR

1. Console AWS → ECR
2. Selecione o repositório
3. Selecione a imagem
4. Delete

### Passo 4: Deletar Security Group

1. EC2 → Security Groups
2. Selecione `meu-website-sg`
3. Actions → Delete

### Passo 5: Deletar IAM Role

1. IAM → Roles
2. Selecione `EC2-ECR-Role`
3. Delete

---

## 🎓 Conceitos Aprendidos

✅ **Containerização**: Empacotamento de aplicações com suas dependências

✅ **Docker**: Plataforma para criar e executar containers

✅ **Dockerfile**: Arquivo de configuração para construir imagens

✅ **ECR**: Registro privado de imagens Docker na AWS

✅ **EC2**: Máquinas virtuais na nuvem AWS

✅ **Security Groups**: Firewall virtual para EC2

✅ **IAM Roles**: Gerenciamento de permissões na AWS

---

---

## 📚 Recursos Adicionais

- [Documentação Docker](https://docs.docker.com/)
- [AWS ECR Documentation](https://docs.aws.amazon.com/ecr/)
- [AWS EC2 User Guide](https://docs.aws.amazon.com/ec2/)
- [Best Practices for Dockerfile](https://docs.docker.com/develop/dev-best-practices/)

---


Desenvolvido com ❤️ para a jornada DevOps