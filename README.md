# HelloApp - CI/CD com ArgoCD, Kubernetes e Docker

Este projeto configura uma aplicação **FastAPI** simples com **CI/CD** utilizando **GitHub Actions**, **Docker Hub**, e **ArgoCD** para entrega contínua e gerenciamento de Kubernetes.

A aplicação consiste em um **"Hello World"** simples servido através do **FastAPI**, com a construção e publicação automatizada de imagens Docker via GitHub Actions e o uso do **ArgoCD** para orquestrar o deploy da aplicação no Kubernetes.

## Tecnologias Utilizadas

- **FastAPI**: Framework para construção da API.
- **GitHub Actions**: Para automação do CI/CD.
- **Docker**: Para empacotamento da aplicação em containers.
- **Docker Hub**: Para armazenamento das imagens Docker.
- **Kubernetes**: Para gerenciamento da aplicação em cluster.
- **ArgoCD**: Para integração contínua com Kubernetes e deploy automático.
- **Rancher Desktop**: Para provisionamento do Kubernetes localmente.

## Pré-requisitos

Antes de começar, você precisa ter o seguinte instalado:

1. **Docker** - Para construir e rodar containers.
2. **Kubernetes (via Rancher Desktop)** - Para orquestrar os containers.
3. **kubectl** - Ferramenta de linha de comando para interagir com o Kubernetes.
4. **ArgoCD** - Para gerenciar a entrega contínua no Kubernetes.
5. **Conta no Docker Hub** - Para armazenar a imagem Docker.
6. **Conta no GitHub** - Para integrar com GitHub Actions.

## Passos de Configuração

### 1. Criar a Aplicação FastAPI

A aplicação é composta por um simples servidor FastAPI com a rota básica de `GET /`.

```python
from fastapi import FastAPI
  
app = FastAPI()
  
@app.get("/")
async def root():
    return {"message": "Hello World"}
```

### 2. Dockerfile para a Aplicação

O Dockerfile utilizado para criar a imagem da aplicação é o seguinte:

```python
FROM python:3.11-slim 
 
WORKDIR /app 
COPY requirements.txt . 
RUN pip install --no-cache-dir -r requirements.txt 
 
COPY . . 
 
EXPOSE 80 
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "80"]
```

### 3. GitHub Actions para CI/CD

O workflow ci-cd.yml no GitHub Actions é responsável por construir, testar e empurrar a imagem Docker para o Docker Hub e atualizar os manifestos do Kubernetes:

```python

name: CI/CD - Build, Push, Update Manifests

on:
  push:
    branches: [ main ]

env:
  IMAGE_NAME: ${{ secrets.DOCKER_USERNAME }}/hello-app

jobs:
  build_and_push:
    runs-on: ubuntu-latest
    outputs:
      image_tag: ${{ steps.set_tag.outputs.IMAGE_TAG }}
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Set image tag
        id: set_tag
        run: echo "IMAGE_TAG=${GITHUB_SHA::8}" >> $GITHUB_OUTPUT

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push image
        uses: docker/build-push-action@v4
        with:
          push: true
          tags: ${{ env.IMAGE_NAME }}:${{ steps.set_tag.outputs.image_tag }}
          file: ./Dockerfile

  update_manifests:
    needs: build_and_push
    runs-on: ubuntu-latest
    steps:
      - name: Setup SSH for pushing to manifests repo
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts

      - name: Clone manifests repo
        run: |
          git clone git@github.com:GabrielaHarres/hello-manifests.git manifests
          cd manifests
          git checkout -b update-image-${{ needs.build_and_push.outputs.image_tag }}

      - name: Update deployment image tag
        run: |
          cd manifests
          TAG=${{ needs.build_and_push.outputs.image_tag }}
          IMAGE="${{ env.IMAGE_NAME }}:${TAG}"
          sed -i "s|image: .*|image: ${IMAGE}|g" k8s/deployment.yaml
          git add k8s/deployment.yaml
          git commit -m "ci: update image -> ${IMAGE}" || echo "no changes to commit"
          git push origin HEAD

```

### 4. Configuração do ArgoCD

O ArgoCD gerencia o deploy automático da aplicação no Kubernetes. Para configurar a aplicação no ArgoCD, você deve:

Criar a aplicação no painel do ArgoCD com a URL do repositório de manifestos no GitHub.

Sincronizar os manifestos com o Kubernetes.

Manifestos Kubernetes:

deployment.yaml:

```python
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
  labels:
    app: hello-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
        - name: hello-app
          image: gabrielaharres/hello-app:3d7b1cd5
          ports:
            - containerPort: 80
```

service.yaml:

```python
apiVersion: v1
kind: Service
metadata:
  name: hello-service
  labels:
    app: hello-app
spec:
  selector:
    app: hello-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

### 5. Acessar a Aplicação

Depois que o ArgoCD sincronizar a aplicação no Kubernetes, você pode usar o kubectl port-forward para acessar a aplicação localmente:

```python
kubectl port-forward svc/hello-service 8080:80
```

Acesse a aplicação em http://localhost:8080

### 6. Imagem que deve aparecer:

<img width="414" height="144" alt="Captura de tela 2025-09-26 170830" src="https://github.com/user-attachments/assets/1c77138c-f896-4fc5-ade2-8aef395ddafb" />



ArgoCD deve estar assim para que a aplicação funcione:



<img width="1639" height="540" alt="Captura de tela 2025-09-26 171145" src="https://github.com/user-attachments/assets/af88c3e0-a24d-4356-9d90-a363fd92018a" />

