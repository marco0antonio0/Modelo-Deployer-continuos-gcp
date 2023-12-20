# Modelo de Deploy Contínuo no Google Cloud Platform (GCP)

Este repositório fornece um guia prático para configurar e executar um modelo de deploy contínuo no Google Cloud Platform (GCP) usando o Cloud Run. O objetivo é permitir a execução remota contínua de seus projetos, e este README serve como um guia detalhado para configurar e utilizar essa prática de desenvolvimento.

## Configuração do Ambiente

### Cabeçalho do Código

No cabeçalho do seu código, altere o nome do ambiente para refletir o ambiente desejado:

```yaml
environment: twes
```

### Configuração no GitHub
Configure as seguintes variáveis de ambiente secretas no GitHub nas configurações do seu repositório:

- GCP_PROJECT_ID: ID do projeto GCP.
- GCP_CREDENTIALS: Chave de autenticação IAM.

### Configuração Direta no Código

Além disso, defina as seguintes variáveis diretamente no código:

- NAME_PROJECT: Nome do projeto configurado pelo desenvolvedor.

Não altere:

- IMAGE_NAME: Nome padrão, não altere.
  
### Configurações da Máquina no Google Cloud
```yaml
name: GCP

on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Setup Gcloud Account
    runs-on: ubuntu-latest
    environment: twes
    env:
      NAME_PROJECT: testessss
      IMAGE_NAME: gcr.io/${{ secrets.GCP_PROJECT_ID }}/
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - uses: google-github-actions/setup-gcloud@v0.2.0
        with:
          service_account_key: ${{ secrets.GCP_CREDENTIALS }}
          project_id: ${{ secrets.GCP_PROJECT_ID }}

      - name: Configure Docker
        run: gcloud auth configure-docker --quiet

      - name: Build Docker image
        run: docker build -t $IMAGE_NAME$NAME_PROJECT .

      - name: Push Docker image
        run: docker push $IMAGE_NAME$NAME_PROJECT

      - name: Deploy Docker image
        run: gcloud run deploy $NAME_PROJECT --image $IMAGE_NAME$NAME_PROJECT --region us-central1 --memory 128Mi --min-instances 0 --max-instances 1 --platform managed --port 80 --allow-unauthenticated
```

### Deploy no Registro de Artefatos

Código Completo:

```yaml
name: GCP

on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Setup Gcloud Account
    runs-on: ubuntu-latest
    environment: nextProjectEnvs
    env:
      NAME_PROJECT: 'nome-do-projeto'
      DOC_REPOSITORY: 'nome-do-repositorio/'
      LOCATIONS: 'us-central1'
      IMAGE_NAME: '-docker.pkg.dev/${{ secrets.GCP_PROJECT_ID }}/'
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Login to GCP
        uses: google-github-actions/setup-gcloud@v0.2.0
        with:
          service_account_key: ${{ secrets.GCP_CREDENTIALS }}
          project_id: ${{ secrets.GCP_PROJECT_ID }}

      - name: Configure Docker
        run: gcloud auth configure-docker us-central1-docker.pkg.dev --quiet

      - name: Build Docker image
        run: docker build -t $LOCATIONS$IMAGE_NAME$DOC_REPOSITORY$NAME_PROJECT .

      - name: Tag and Push Docker image to Artifact Registry
        run: docker push $LOCATIONS$IMAGE_NAME$DOC_REPOSITORY$NAME_PROJECT

      - name: Deploy Docker image to Cloud Run
        run: gcloud run deploy $NAME_PROJECT --image $LOCATIONS$IMAGE_NAME$DOC_REPOSITORY$NAME_PROJECT --region $LOCATIONS --memory 128Mi --min-instances 0 --max-instances 1 --platform managed --port 80 --allow-unauthenticated
```
