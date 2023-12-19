# Modelo-Deployer-continuos-gcp
Repositorio oferece o wiki guia do modelo deployer GCP run para execução remota continua

#### Cabeçalho da env definida - altere no codigo e na env em settings

```
environment: twes
```

#### Faça as seguintes SETs em secrets ENV github

=====================================================

ENVs secrets

GCP_PROJECT_ID = nome id do projeto

GCP_CREDENTIALS = chave de autentivação IAM
=====================================================

#### Faça as seguintes SETs direto ao codigo

NAME_PROJECT = nome do projeto setado pelo dev
=====================================================

=====================================================

#### Não altere

IMAGE_NAME = nomeSET padrão não altere
=====================================================

#### Configurações set da maquina cloud
```
--region us-central1 --memory 128Mi --min-instances 0 --max-instances 1 --platform managed --port 80 --allow-unauthenticated
```

```
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
      # Git checkout
      - name: Checkout
        uses: actions/checkout@v2

      # Login to GCP
      - uses: google-github-actions/setup-gcloud@v0.2.0
        with:
          service_account_key: ${{ secrets.GCP_CREDENTIALS }}
          project_id: ${{ secrets.GCP_PROJECT_ID }}

      # gcloud configure docker
      - name: Configure Docker
        run: gcloud auth configure-docker --quiet

      # build image
      - name: Build Docker image
        run: docker build -t $IMAGE_NAME$NAME_PROJECT .

      # push image to registry
      - name: Push Docker image
        run: docker push $IMAGE_NAME$NAME_PROJECT

      # deploy image
      - name: Deploy Docker image
        run: gcloud run deploy $NAME_PROJECT --image $IMAGE_NAME$NAME_PROJECT --region us-central1 --memory 128Mi --min-instances 0 --max-instances 1 --platform managed --port 80 --allow-unauthenticated
``` 
