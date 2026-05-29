# Namespaces

Namespaces são partições lógicas dentro de um cluster Kubernetes. Eles não isolam tráfego de rede por padrão — apenas organizam recursos e permitem aplicar quotas e políticas de acesso por grupo.

---

## O que um Namespace faz

- **Organiza recursos:** Pods, Services, ConfigMaps de projetos ou times diferentes ficam em namespaces separados
- **Escopa nomes:** dois Pods chamados `app` podem existir em namespaces diferentes sem conflito
- **Âncora para RBAC:** permissões (`Role`, `RoleBinding`) são aplicadas por namespace
- **Âncora para quotas:** `ResourceQuota` e `LimitRange` são configurados por namespace

!!! note "Namespaces não isolam rede"
    Um Pod no namespace `dev` pode, por padrão, falar com um Pod no namespace `prod`. Para isolamento de rede real, use [NetworkPolicy](../fase-03-networking/network-policy.md).

---

## Namespaces padrão do cluster

```bash
kubectl get namespaces
```

```
NAME              STATUS   AGE
default           Active   1d    ← onde seus recursos vão se você não especificar
kube-system       Active   1d    ← componentes internos do k8s (apiserver, etcd, etc.)
kube-public       Active   1d    ← informações públicas do cluster
kube-node-lease   Active   1d    ← heartbeats dos nós (não use)
```

---

## YAML de referência

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: staging
  labels:
    environment: staging    # label útil para NetworkPolicy e seleção
```

---

## Usando namespaces nos comandos

```bash
# Criar namespace
kubectl create namespace staging
# ou
kubectl apply -f namespace.yaml

# Listar recursos em um namespace específico
kubectl get pods -n staging
kubectl get all -n staging

# Listar em todos os namespaces
kubectl get pods --all-namespaces
kubectl get pods -A                  # atalho

# Definir namespace padrão da sessão
kubectl config set-context --current --namespace=staging
```

=== "Linux / macOS"
    ```bash
    kubectl config view --minify | grep namespace    # confirmar
    ```

=== "Windows (PowerShell)"
    ```powershell
    kubectl config view --minify | Select-String "namespace"    # confirmar
    ```

---

## Especificando namespace em um YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod
  namespace: staging        # sem isso, vai para "default"
spec:
  containers:
    - name: app
      image: nginx:1.25
```

!!! tip "Nunca omita o namespace em YAMLs de produção"
    Em ambientes com múltiplos namespaces, declarar o namespace explicitamente evita aplicar recursos no lugar errado.

---

## Recursos com escopo de namespace vs. cluster

| Escopo de namespace | Escopo de cluster |
|---|---|
| Pod, Deployment, Service | Node |
| ConfigMap, Secret | PersistentVolume |
| Role, RoleBinding | ClusterRole, ClusterRoleBinding |
| ResourceQuota, LimitRange | StorageClass, Namespace |

Recursos com escopo de cluster não pertencem a nenhum namespace — `kubectl get nodes -n default` retorna erro.

---

## Próximo

➡️ [InitContainers](init-containers.md) — como executar pré-condições antes do container principal.
