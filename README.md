# Arquitetura Kubernetes com Minikube

Este projeto apresenta a implementação prática de uma arquitetura básica em **Kubernetes** executada localmente via **Minikube** (com driver Docker em ambiente Windows 11 / WSL2). O objetivo é demonstrar o provisionamento de um `Pod` executando uma aplicação Python containerizada e sua exposição externa através de um `Service` do tipo `NodePort`.

---

## 📁 Estrutura do Repositório

```text
arquitetura-kubernetes-main/
├── kubernetes.drawio   # Diagrama visual da arquitetura do cluster (Control Plane e Worker Node)
├── pod.yml             # Manifesto declarativo do Pod (meu-app-pod)
├── service.yml         # Manifesto declarativo do Service NodePort (meu-service-app)
├── prints/             # Pasta opcional com as capturas de tela das evidências
└── README.md           # Documentação técnica, instruções de execução e evidências
```

---

## Visão Geral da Arquitetura

O diagrama (`kubernetes.drawio`) ilustra o fluxo de comunicação entre o usuário, os componentes de controle do cluster e a carga de trabalho:

1. **Control Plane (Master):**
   * **API Server:** Ponto central de comunicação que recebe os comandos do `kubectl` e valida os manifestos YAML.
   * **etcd:** Banco de dados chave-valor que armazena o estado atual e desejado do cluster.
   * **Scheduler:** Responsável por alocar o `meu-app-pod` no nó disponível (`minikube`).
   * **Controller Manager:** Garante que o estado do cluster permaneça conforme configurado.

2. **Worker Node (Minikube):**
   * **Kubelet & Container Runtime (Docker):** Gerenciam o ciclo de vida do container `app-container` baseado na imagem `marianamonteiro1998/meu_app_python:1.0`.
   * **Pod (`meu-app-pod`):** Executa a aplicação na porta interna `5000`.
   * **Service (`meu-service-app`):** Roteia o tráfego externo da porta `30080` (`NodePort`) para a porta `80` do serviço e redireciona para a `targetPort: 5000` do Pod através do seletor `app: meu-app`.

---

## Manifestos Kubernetes

### 1. Pod (`pod.yml`)

Define o Pod `meu-app-pod` utilizando a imagem publicada no Docker Hub (`marianamonteiro1998/meu_app_python:1.0`) e um servidor HTTP ativo na porta `5000`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-app-pod
  labels:
    app: meu-app
spec:
  containers:
  - name: app-container
    image: marianamonteiro1998/meu_app_python:1.0
    command: ["python3", "-m", "http.server", "5000"]
    ports:
    - containerPort: 5000
```

### 2. Service (`service.yml`)

Expõe o Pod externamente utilizando o tipo `NodePort` na porta `30080`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: meu-service-app
spec:
  type: NodePort
  selector:
    app: meu-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
      nodePort: 30080
```

---

## Passo a Passo de Execução

### Pré-requisitos

* **Docker Desktop** em execução (`Engine running`)
* **Minikube** e **kubectl** instalados e configurados no PATH

### Comandos Utilizados

1. **Iniciar o cluster Minikube:**
   ```powershell
   $env:DOCKER_API_VERSION="1.44"
   minikube start --driver=docker --cpus=1 --memory=1800mb --preload=false --force
   ```

2. **Aplicar os manifestos do Pod e do Service:**
   ```powershell
   kubectl apply -f pod.yml
   kubectl apply -f service.yml
   ```

3. **Verificar o status dos recursos criados:**
   ```powershell
   kubectl get pods
   kubectl get services
   kubectl describe pod meu-app-pod
   ```

4. **Expor e acessar a aplicação no navegador:**
   ```powershell
   minikube service meu-service-app
   ```

---

## 📸 Evidências de Funcionamento

* **Cluster e recursos ativos (`kubectl get pods` e `kubectl get services`):**
  * `meu-app-pod` com status `Running` (`1/1`) e `0` reinícios.
  * `meu-service-app` configurado como `NodePort` na porta `80:30080/TCP`.
* **Acesso via navegador (`minikube service meu-service-app`):**
  * Túnel iniciado com sucesso redirecionando o tráfego local para a aplicação em execução na porta `5000`.

*(Para exibir os prints diretamente no GitHub, salve as imagens na pasta `prints/` do repositório e descomente as linhas abaixo)*:

```markdown
![Status dos Pods e Services](prints/terminal-pods.png)
![Aplicação no Navegador](prints/navegador-app.png)
```
