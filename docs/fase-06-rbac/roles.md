# Role e ClusterRole

Role e ClusterRole definem conjuntos de permissões. A diferença é o escopo: Role aplica em um namespace, ClusterRole aplica em todo o cluster.

---

## Anatomia de um Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: leitor-pods
  namespace: producao              # escopo: apenas este namespace
rules:
  - apiGroups: [""]                # "" = core API group (Pod, Service, ConfigMap, Secret...)
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]            # "apps" group: Deployment, ReplicaSet, StatefulSet...
    resources: ["deployments"]
    verbs: ["get", "list"]
```

---

## API Groups

| apiGroup | Recursos incluídos |
|---|---|
| `""` (vazio) | Pod, Service, ConfigMap, Secret, Namespace, Node, PV, PVC, ServiceAccount... |
| `"apps"` | Deployment, ReplicaSet, StatefulSet, DaemonSet |
| `"batch"` | Job, CronJob |
| `"networking.k8s.io"` | Ingress, NetworkPolicy |
| `"rbac.authorization.k8s.io"` | Role, ClusterRole, RoleBinding, ClusterRoleBinding |
| `"autoscaling"` | HorizontalPodAutoscaler |

Para descobrir o API group de um recurso:
```bash
kubectl api-resources | grep deployment
# Windows (PowerShell): kubectl api-resources | Select-String "deployment"
# deployments   apps   true   Deployment
```

---

## Verbs disponíveis

| Verb | O que permite |
|---|---|
| `get` | Ler um recurso específico por nome |
| `list` | Listar todos os recursos |
| `watch` | Receber atualizações em tempo real |
| `create` | Criar novos recursos |
| `update` | Substituir um recurso existente |
| `patch` | Atualizar campos específicos |
| `delete` | Deletar um recurso |
| `deletecollection` | Deletar múltiplos recursos |
| `*` | Todos os verbs |

---

## ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: leitor-global              # sem namespace — aplica em todo o cluster
rules:
  - apiGroups: [""]
    resources: ["nodes"]           # Nodes são recursos de escopo de cluster
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["namespaces"]
    verbs: ["get", "list"]
```

### Quando usar ClusterRole

- Recursos de escopo de cluster (Nodes, PersistentVolumes, Namespaces)
- Permissões que devem se aplicar a todos os namespaces
- Como base para RoleBinding em múltiplos namespaces (economiza duplicação)

---

## Verificando permissões

```bash
# Testar permissão de uma SA específica
kubectl auth can-i list pods \
  --as=system:serviceaccount:default:minha-sa

# Testar em namespace específico
kubectl auth can-i create deployments \
  --as=system:serviceaccount:producao:deploy-sa \
  -n producao

# Ver todas as permissões de uma SA
kubectl auth can-i --list \
  --as=system:serviceaccount:default:minha-sa
```

---

## Próximo

➡️ [Bindings](bindings.md) — vinculando permissões a identidades.
