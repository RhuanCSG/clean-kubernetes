# Setup — minikube (Fases 1 a 6)

minikube é a forma mais simples de rodar Kubernetes localmente. Cria um cluster single-node dentro de um container Docker ou VM.

---

## Pré-requisitos

- Docker Desktop instalado e rodando
- kubectl instalado (`kubectl version --client`)

---

## Instalação

=== "macOS"

    ```bash
    brew install minikube
    ```

=== "Windows (PowerShell admin)"

    ```powershell
    winget install Kubernetes.minikube
    ```

=== "Linux"

    ```bash
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    ```

Verificar:

```bash
minikube version
# minikube version: v1.34 ou superior
```

---

## Iniciando o cluster

```bash
minikube start --driver=docker --cpus=2 --memory=4g
```

Saída esperada:

```
✅  minikube v1.34 on ...
✅  Using the docker driver
✅  Starting control plane node minikube in cluster minikube
✅  Done! kubectl is now configured to use "minikube" cluster
```

---

## Verificando o cluster

```bash
kubectl get nodes
```

```
NAME       STATUS   ROLES           AGE
minikube   Ready    control-plane   1m
```

```bash
kubectl cluster-info
# Kubernetes control plane is running at https://192.168.x.x:8443
```

---

## Comandos essenciais

```bash
minikube status          # estado geral do cluster
minikube stop            # para sem destruir (dados preservados)
minikube delete          # destrói tudo (bom para reset completo)
minikube dashboard       # abre a UI no navegador
minikube ssh             # acessa o nó via SSH
```

---

## Addons necessários por fase

Alguns recursos precisam de addons habilitados:

```bash
# Fase 03 — Networking: IngressController
minikube addons enable ingress

# Fase 07 — Scheduling: HPA (metrics-server)
minikube addons enable metrics-server

# Ver todos os addons disponíveis
minikube addons list
```

---

## Resetando o ambiente

Se o cluster ficar em estado inconsistente:

```bash
minikube delete
minikube start --driver=docker --cpus=2 --memory=4g
```

!!! tip "Dica de uso"
    Use `minikube stop` ao final de cada sessão de estudo. É mais rápido do que deletar e recriar quando você retoma o estudo.

---

## Próximo

➡️ [Fase 01 — Pod & Runtime](../fase-01-pod-runtime/index.md)
