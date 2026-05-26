# RoleBinding e ClusterRoleBinding

Bindings são a "cola" do RBAC: vinculam um Role ou ClusterRole a um Subject (ServiceAccount, User ou Group).

---

## RoleBinding — escopo de namespace

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-leitor-sa
  namespace: producao              # o binding e o Role aplicam neste namespace
subjects:
  - kind: ServiceAccount
    name: minha-sa                 # a SA que receberá as permissões
    namespace: producao            # namespace da SA (obrigatório para SA)
  # Pode ter múltiplos subjects:
  # - kind: User
  #   name: joao@empresa.com
  # - kind: Group
  #   name: developers
roleRef:
  kind: Role
  name: leitor-pods                # Role a ser vinculado
  apiGroup: rbac.authorization.k8s.io
```

!!! note "roleRef é imutável"
    Após criar um RoleBinding, você não pode alterar o `roleRef`. Se precisar mudar o Role, delete e recrie o binding.

---

## ClusterRoleBinding — escopo de cluster

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-global
subjects:
  - kind: ServiceAccount
    name: admin-sa
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin              # ClusterRole built-in com acesso total
  apiGroup: rbac.authorization.k8s.io
```

---

## Combinações possíveis

| Tipo de binding | Tipo de role | Resultado |
|---|---|---|
| `RoleBinding` | `Role` | Permissões em um namespace específico |
| `RoleBinding` | `ClusterRole` | Permissões de um ClusterRole, mas restrito a um namespace |
| `ClusterRoleBinding` | `ClusterRole` | Permissões em todos os namespaces |
| `ClusterRoleBinding` | `Role` | **Inválido** — Role não tem escopo de cluster |

### Reutilizando ClusterRole com RoleBinding

Um ClusterRole pode ser vinculado com RoleBinding para economizar duplicação:

```yaml
# ClusterRole definido uma vez
kind: ClusterRole
metadata:
  name: leitor-pods
# ...

# RoleBinding em cada namespace que precisa
kind: RoleBinding
metadata:
  namespace: producao
roleRef:
  kind: ClusterRole
  name: leitor-pods
```

Isso é mais eficiente que criar um Role idêntico em cada namespace.

---

## YAML completo — SA + Role + RoleBinding

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: default

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["configmaps", "secrets"]
    verbs: ["get", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: default
subjects:
  - kind: ServiceAccount
    name: app-sa
    namespace: default
roleRef:
  kind: Role
  name: app-role
  apiGroup: rbac.authorization.k8s.io
```

---

## Inspecionando bindings

```bash
kubectl get rolebindings -n default
kubectl get clusterrolebindings
kubectl describe rolebinding <nome>    # ver subjects e roleRef
kubectl describe clusterrolebinding <nome>
```

---

## Próximo

➡️ [Debugging](debugging.md) — investigando erros 403 Forbidden.
