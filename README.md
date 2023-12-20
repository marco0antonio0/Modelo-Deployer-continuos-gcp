# Modelo de Deploy Contínuo no Google Cloud Platform (GCP)

Este repositório fornece um guia prático para configurar e executar um modelo de deploy contínuo no Google Cloud Platform (GCP) usando o Cloud Run. O objetivo é permitir a execução remota contínua de seus projetos, e este README serve como um guia detalhado para configurar e utilizar essa prática de desenvolvimento.

## Configuração do Ambiente

### Cabeçalho do Código

No cabeçalho do seu código, altere o nome do ambiente para refletir o ambiente desejado:

```yaml
environment: nome-do-main-secret-envs
```

### Configuração no GitHub

Configure as seguintes variáveis de ambiente secretas no GitHub qu devem ser guardadas com segurança nas configurações do seu repositório:

- GCP_PROJECT_ID: ID do projeto GCP.
- GCP_CREDENTIALS: Chave de autenticação IAM.

### Configuração Direta no Código GITHUB ACTIONS

Defina as seguintes variáveis diretamente no código:

- NAME_PROJECT: "nome-do-projeto" <<< defina o nome do projeto >>>
- DOC_REPOSITORY: "nome-do-repositorio/" <<< defina o nome do repositorio >>>
- LOCATIONS: "us-central1" <<< defina a região da maquina e arquivos >>>
- MIN_INSTANCE: 0 <<< defina a quantidade de instancias minimas>>>
- MAX_INSTANCE: 1 <<< defina a quantidade de instancias maximas>>>
- MEMORY_RAM: '128Mi' <<<< defina a quantidade de memoria ram da VM>>>>

**Não altere:**

- IMAGE_NAME: Nome padrão, não altere.

### Configurações da Máquina no Google Cloud

### Deploy no Container Registry

```yaml
# Variaveis secretas a serem definidas
#
# environment: nome-da-main-env
# GCP_PROJECT_ID: nome-do-id-projeto-gcp
# GCP_CREDENTIALS: chaves IAM credenciais
#
# Variaveis a serem definidas
#
# NAME_PROJECT: "nome-do-projeto" <<< defina o nome do projeto  >>>
# DOC_REPOSITORY: "nome-do-repositorio/" <<< defina o nome do repositorio  >>>
# LOCATIONS: "us-central1" <<< defina a região da maquina e arquivos  >>>
# MIN_INSTANCE: 0 <<< defina a quantidade de instancias minimas>>>
# MAX_INSTANCE: 1 <<< defina a quantidade de instancias maximas>>>
# MEMORY_RAM: '128Mi' <<<< defina a quantidade de memoria ram da VM>>>>
#
#
# não altere
# IMAGE_NAME: "${{ secrets.GCP_PROJECT_ID }}/"
#
# Nome da Action
name: GCP

# Acionamento da ação
on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Setup Gcloud Account
    runs-on: ubuntu-latest

    # defina a secret ENVs
    environment: twes
    # defina as ENVs
    env:
      # defina nome do projeto
      NAME_PROJECT: "testessss"
      # defina o nome do repositorio existente
      DOC_REPOSITORY: "nome-do-repositorio/"
      # defina a região
      LOCATIONS: "us-central1"
      # defina a instancia minima
      MIN_INSTANCE: 0
      # defina a instancia maxima
      MAX_INSTANCE: 1
      # defina a quantidaded e memoria ram para vm
      MEMORY_RAM: "128Mi"
      IMAGE_NAME: "${{ secrets.GCP_PROJECT_ID }}/"
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      # etapa autenticação
      - uses: google-github-actions/setup-gcloud@v0.2.0
        with:
          service_account_key: ${{ secrets.GCP_CREDENTIALS }}
          project_id: ${{ secrets.GCP_PROJECT_ID }}

      - name: Configure Docker
        run: gcloud auth configure-docker --quiet
      # criando a imagem
      - name: Build Docker image
        run: docker build -t $DOC_REPOSITORY$IMAGE_NAME$NAME_PROJECT .
      # levantando a imagem para server Container Registry
      - name: Push Docker image
        run: docker push $DOC_REPOSITORY$IMAGE_NAME$NAME_PROJECT
      # exectando o deploy da imagem no servidor
      - name: Deploy Docker image
        run: gcloud run deploy $NAME_PROJECT --image $DOC_REPOSITORY$IMAGE_NAME$NAME_PROJECT --region $LOCATIONS --memory $MEMORY_RAM  --min-instances $MIN_INSTANCE --max-instances $MAX_INSTANCE --platform managed --port 80 --allow-unauthenticated
```

### Deploy no Artifact Registry

Código Completo:

```yaml
# Variaveis secretas a serem definidas
#
# environment: nome-da-main-env
# GCP_PROJECT_ID: nome-do-id-projeto-gcp
# GCP_CREDENTIALS: chaves IAM credenciais
#
# Variaveis a serem definidas
#
# NAME_PROJECT: "nome-do-projeto" <<< defina o nome do projeto  >>>
# DOC_REPOSITORY: "nome-do-repositorio/" <<< defina o nome do repositorio  >>>
# LOCATIONS: "us-central1" <<< defina a região da maquina e arquivos  >>>
# MIN_INSTANCE: 0 <<< defina a quantidade de instancias minimas>>>
# MAX_INSTANCE: 1 <<< defina a quantidade de instancias maximas>>>
# MEMORY_RAM: '128Mi' <<<< defina a quantidade de memoria ram da VM>>>>
#
#
# não altere
# IMAGE_NAME: "-docker.pkg.dev/${{ secrets.GCP_PROJECT_ID }}/"
#
# Nome da Action
name: GCP

on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Setup Gcloud Account
    runs-on: ubuntu-latest
    # defina a secret ENVs
    environment: nextProjectEnvs
    # defina a ENVs
    env:
      # defina o nome do projeto
      NAME_PROJECT: "nome-do-projeto"
      # defina o nome do repositorio existente
      DOC_REPOSITORY: "nome-do-repositorio/"
      # defina a região
      LOCATIONS: "us-central1"
      # defina a instancia minima
      MIN_INSTANCE: 0
      # defina a instancia maxima
      MAX_INSTANCE: 1
      # defina a quantidaded e memoria ram para vm
      MEMORY_RAM: "128Mi"
      # não altere
      IMAGE_NAME: "-docker.pkg.dev/${{ secrets.GCP_PROJECT_ID }}/"
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      # etapa autenticação
      - name: Login to GCP
        uses: google-github-actions/setup-gcloud@v0.2.0
        with:
          service_account_key: ${{ secrets.GCP_CREDENTIALS }}
          project_id: ${{ secrets.GCP_PROJECT_ID }}

      - name: Configure Docker
        run: gcloud auth configure-docker us-central1-docker.pkg.dev --quiet

      # criando a imagem
      - name: Build Docker image
        run: docker build -t $LOCATIONS$IMAGE_NAME$DOC_REPOSITORY$NAME_PROJECT .

      # levantando a imagem para server Artifact Registry
      - name: Tag and Push Docker image to Artifact Registry
        run: docker push $LOCATIONS$IMAGE_NAME$DOC_REPOSITORY$NAME_PROJECT

      # exectando o deploy da imagem no servidor
      - name: Deploy Docker image to Cloud Run
        run: gcloud run deploy $NAME_PROJECT --image $LOCATIONS$IMAGE_NAME$DOC_REPOSITORY$NAME_PROJECT --region $LOCATIONS --memory $MEMORY_RAM --min-instances $MIN_INSTANCE --max-instances $MAX_INSTANCE --platform managed --port 80 --allow-unauthenticated
```
