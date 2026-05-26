# Setup — kind multi-nó (Fases 7 e 8)

kind (Kubernetes in Docker) cria clusters com múltiplos nós localmente, onde cada nó é um container Docker. É necessário para as Fases 7 e 8, onde comportamentos como taints, affinity e componentes do control plane só fazem sentido com mais de um nó.

---

## Por que kind para as fases avançadas?

| Recurso | minikube (single-node) | kind (multi-node) |
|---|---|---|
| Taints por nó | Não faz sentido | ✅ Essencial |
| Pod affinity entre nós | ✅ Funciona mas não é útil | ✅ Comportamento real |
| Simular nó NotReady | Limitado | ✅ `docker pause <nó>` |
| Control plane separado dos workers | Não | ✅ Igual produção |

---

## Instalação

=== "macOS"

    ```bash
    brew install kind
    ```

=== "Windows (PowerShell admin)"

    ```powershell
    winget install Kubernetes.kind
    ```

=== "Linux"

    ```bash
    curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
    chmod +x kind && sudo mv kind /usr/local/bin/kind
    ```

Verificar:

```bash
kind version
# kind v0.23 ou superior
```

---

## Criando o cluster multi-nó

O arquivo `setup/kind-config.yaml` (na raiz do repositório) define 1 control-plane e 2 workers:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

Criar o cluster:

```bash
kind create cluster --config setup/kind-config.yaml --name k8s-study
```

Configurar o kubectl para usar este cluster:

```bash
kubectl cluster-info --context kind-k8s-study
```

---

## Verificando os nós

```bash
kubectl get nodes
```

```
NAME                      STATUS   ROLES           AGE
k8s-study-control-plane   Ready    control-plane   1m
k8s-study-worker          Ready    <none>          1m
k8s-study-worker2         Ready    <none>          1m
```

---

## Containers do cluster

Como cada nó é um container Docker, você pode inspecioná-los:

```bash
docker ps --filter name=k8s-study
# k8s-study-control-plane
# k8s-study-worker
# k8s-study-worker2
```

Para simular um nó com problema (usado na Fase 08):

```bash
docker pause k8s-study-worker      # pausa o container (simula falha do kubelet)
docker unpause k8s-study-worker    # retoma
```

---

## Removendo o cluster

```bash
kind delete cluster --name k8s-study
```

---

## Contexto kubectl

kind configura um contexto no kubeconfig automaticamente. Para alternar entre minikube e kind:

```bash
kubectl config get-contexts
kubectl config use-context kind-k8s-study
kubectl config use-context minikube
```

!!! warning "Atenção ao contexto"
    Sempre verifique em qual cluster você está antes de aplicar YAMLs. `kubectl config current-context` mostra o contexto ativo.

---

## Próximo

➡️ [Fase 07 — Scheduling & Recursos](../fase-07-scheduling/index.md)
