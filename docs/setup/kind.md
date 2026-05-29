# Setup — kind (Todas as Fases)

kind (Kubernetes in Docker) cria clusters Kubernetes localmente onde cada nó é um container Docker. É o ambiente de prática para todas as 8 fases deste repositório.

---

## Pré-requisitos

- Docker Desktop instalado e rodando
- kubectl instalado

---

## Instalação do kind

=== "Linux"

    ```bash
    curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
    chmod +x kind && sudo mv kind /usr/local/bin/kind
    ```

=== "macOS"

    ```bash
    brew install kind
    ```

=== "Windows (PowerShell admin)"

    ```powershell
    winget install Kubernetes.kind
    ```

Verificar:

```bash
kind version
# kind v0.23 ou superior
```

---

## Criando o cluster

O arquivo `setup/kind-config.yaml` define 1 control-plane e 2 workers:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true    # necessário para instalar Calico (suporte a NetworkPolicy)
nodes:
  - role: control-plane
    kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "ingress-ready=true"
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP
  - role: worker
  - role: worker
```

Criar o cluster:

```bash
kind create cluster --config setup/kind-config.yaml --name k8s-study
```

---

## Instalando o CNI (Calico)

O cluster foi criado sem CNI padrão. Instalar Calico antes de criar qualquer Pod:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml

# Aguardar os pods do Calico ficarem Ready (~2 min)
kubectl get pods -n calico-system -w
```

Calico é necessário para a Fase 03 (NetworkPolicy). Sem ele, as regras de NetworkPolicy são ignoradas.

---

## Verificando os nós

```bash
kubectl get nodes
```

Saída esperada (após Calico Ready):

```
NAME                      STATUS   ROLES           AGE
k8s-study-control-plane   Ready    control-plane   2m
k8s-study-worker          Ready    <none>          2m
k8s-study-worker2         Ready    <none>          2m
```

---

## Addons por fase

Alguns recursos precisam ser instalados antes da fase correspondente.

### Fase 03 — Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

# Aguardar o controller ficar Ready
kubectl get pods -n ingress-nginx -w
```

Após instalar, Ingresses são acessíveis via `http://localhost`.

### Fase 07 — metrics-server (HPA)

=== "Linux / macOS"

    ```bash
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
    kubectl patch deployment metrics-server -n kube-system --type='json' \
      -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
    ```

=== "Windows (PowerShell)"

    ```powershell
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
    kubectl patch deployment metrics-server -n kube-system --type='json' `
      -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
    ```

A flag `--kubelet-insecure-tls` é necessária no kind porque os certificados kubelet são autoassinados.

---

## Contexto kubectl

kind configura o contexto automaticamente. Para verificar:

```bash
kubectl config current-context
# kind-k8s-study

kubectl config get-contexts
```

---

## Comandos essenciais

```bash
kubectl get nodes                          # estado dos nós
docker ps --filter name=k8s-study         # containers do cluster
kind get clusters                          # clusters kind ativos
```

Acessar o shell de um nó:

```bash
docker exec -it k8s-study-control-plane bash   # control-plane
docker exec -it k8s-study-worker bash          # worker
```

---

## Resetando o ambiente

Se o cluster ficar em estado inconsistente:

```bash
kind delete cluster --name k8s-study
kind create cluster --config setup/kind-config.yaml --name k8s-study
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml
```

!!! tip "Dica de uso"
    O cluster kind para automaticamente quando o Docker é encerrado. Para retomar, reinicie o Docker — o cluster volta ao estado anterior sem precisar recriar.

---

## Por que kind para todas as fases?

| Recurso | kind |
|---|---|
| Topologia multi-nó | ✅ 1 control-plane + 2 workers |
| NetworkPolicy (Calico) | ✅ Instalado no setup |
| Ingress | ✅ nginx-ingress via kubectl apply |
| Simular nó NotReady | ✅ `docker pause k8s-study-worker` |
| Comportamento idêntico no Windows/Linux | ✅ Só Docker containers |

---

## Próximo

➡️ [Fase 01 — Pod & Runtime](../fase-01-pod-runtime/index.md)
