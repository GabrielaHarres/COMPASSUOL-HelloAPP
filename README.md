# 🚀 Projeto 4 - CI/CD com GitHub Actions & ArgoCD

Este projeto implementa um pipeline completo de **CI/CD** para uma aplicação simples em **FastAPI**, utilizando **GitHub Actions**, **Docker Hub** e **ArgoCD** em um cluster Kubernetes local com **Rancher Desktop**.

O objetivo é automatizar desde o build e push da imagem até o deploy contínuo no Kubernetes, aplicando conceitos modernos de **DevSecOps**.

---

## 📌 Objetivo

Automatizar o ciclo completo de desenvolvimento, build, deploy e execução de uma aplicação **FastAPI**, usando:

- **GitHub Actions** → para CI/CD  
- **Docker Hub** → como registry de imagens  
- **ArgoCD** → para entrega contínua em Kubernetes  

---

## ⚙️ Pré-requisitos

Antes de iniciar, você precisa ter:

- Conta no **GitHub** (repositório público)  
- Conta no **Docker Hub** com **token de acesso**  
- **Rancher Desktop** com Kubernetes habilitado  
- **kubectl** configurado corretamente (`kubectl get nodes`)  
- **ArgoCD** instalado no cluster local  
- **Git** instalado  
- **Python 3** e **Docker** instalados  

---

## 🛠️ Etapas do Projeto

### 🔹 Etapa 1 – Criar a aplicação FastAPI

1. Criar um repositório Git para o projeto (ex: `hello-app`).  
2. Adicionar o arquivo `main.py`:  

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

Criar um Dockerfile para rodar a aplicação.

Criar um repositório separado para os manifestos do ArgoCD (ex: hello-manifests).

### 🔹 Etapa 2 – Configurar o GitHub Actions (CI/CD)

Criar um workflow no GitHub Actions para:

Buildar a imagem

Publicar no Docker Hub

Atualizar os manifests no repositório do ArgoCD

Criar segredos no GitHub:

DOCKER_USERNAME

DOCKER_PASSWORD

SSH_PRIVATE_KEY

Garantir acesso de gravação ao repositório de manifests.

### 🔹 Etapa 3 – Repositório de manifests do ArgoCD

Criar os manifestos Kubernetes de Deployment e Service para o Hello App.

Esse repositório será usado pelo ArgoCD para sincronizar o deploy.

### 🔹 Etapa 4 – Criar o App no ArgoCD

Na interface do ArgoCD:

Criar o vínculo com o repositório de manifests

Configurar o App

### 🔹 Etapa 5 – Testar a aplicação localmente

Usar kubectl port-forward para acessar via navegador:
👉 http://localhost:8080/

Alterar a mensagem em main.py e verificar se o CI/CD atualiza a imagem no Kubernetes automaticamente.

### IMAGENS QUE DEVEM APARECER

Se rodou certo:

<img width="414" height="144" alt="Captura de tela 2025-09-26 170830" src="https://github.com/user-attachments/assets/a48f88ca-f428-4f4d-b137-676a852cd45a" />



ArgoCD:


<img width="1635" height="537" alt="image" src="https://github.com/user-attachments/assets/8ba3d28d-300e-46b5-8882-9af6bcf29e8e" />


