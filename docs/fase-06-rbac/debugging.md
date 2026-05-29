# Debugging — Fase 06

## Metodologia para problemas de RBAC

```
1. kubectl logs <pod>                         → qual operação está falhando?
2. kubectl auth can-i <verb> <resource> --as=system:serviceaccount:<ns>:<sa>
3. kubectl get rolebindings / clusterrolebindings   → existe um binding para essa SA?
4. kubectl describe rolebinding <nome>         → qual Role está vinculado?
5. kubectl describe role <nome>                → o Role inclui o recurso/verb necessário?
```

---

## Cenário 01 — 403 Forbidden

### O que você vê

```bash
kubectl logs forbidden-pod
# Error from server (Forbidden): configmaps is forbidden:
# User "system:serviceaccount:default:app-sa" cannot list resource "configmaps"
# in API group "" in the namespace "default"
```

### Como investigar

```bash
# 1. Confirmar que o erro é RBAC
kubectl auth can-i list configmaps --as=system:serviceaccount:default:app-sa
# no
```

Ver os bindings existentes para essa SA:

=== "Linux / macOS"
    ```bash
    kubectl get rolebindings -n default -o yaml | grep -A5 app-sa
    ```

=== "Windows (PowerShell)"
    ```powershell
    kubectl get rolebindings -n default -o yaml | Select-String -Context 0,5 "app-sa"
    ```

```bash
# 3. Ver o Role vinculado
kubectl describe rolebinding app-rolebinding
# Role: app-role
kubectl describe role app-role
# Rules:
#   Resources: pods (← configmaps está faltando)
#   Verbs: get, list
```

### A correção

Adicionar `configmaps` à lista de resources no Role:

```yaml
rules:
  - apiGroups: [""]
    resources: ["pods", "configmaps"]    # configmaps adicionado
    verbs: ["get", "list"]
```

### Cenário de prática

```bash
kubectl apply -f fases/06-rbac/debugging/01-403-forbidden/broken.yaml
kubectl logs forbidden-pod    # ver o erro 403
kubectl auth can-i list configmaps --as=system:serviceaccount:default:app-sa
```

---

## Cenário 02 — SA não tem binding

### O que você vê

A SA existe, mas não há RoleBinding associado.

```bash
kubectl auth can-i get pods --as=system:serviceaccount:default:minha-sa
# no

kubectl get rolebindings -n default
# Nenhum binding que mencione minha-sa
```

A SA existe, mas sem um RoleBinding ela não tem permissões para nada. A ausência de binding é equivalente a ter permissão zero.

---

## Cenário 03 — Namespace errado no binding

```bash
kubectl describe rolebinding meu-binding
# Subjects:
#   Kind: ServiceAccount
#   Name: app-sa
#   Namespace: staging          ← SA está em default, não staging
```

O binding referencia a SA no namespace errado — o vínculo não se aplica.

```bash
# Verificar em qual namespace a SA existe
kubectl get serviceaccount app-sa --all-namespaces
```

---

## Referência rápida

```bash
# Testar permissão diretamente
kubectl auth can-i <verb> <resource> --as=system:serviceaccount:<namespace>:<sa-name>
kubectl auth can-i --list --as=system:serviceaccount:default:minha-sa

# Inspecionar bindings
```

=== "Linux / macOS"
    ```bash
    kubectl get rolebindings,clusterrolebindings --all-namespaces | grep <sa-name>
    ```

=== "Windows (PowerShell)"
    ```powershell
    kubectl get rolebindings,clusterrolebindings --all-namespaces | Select-String "<sa-name>"
    ```

```bash
kubectl describe rolebinding <nome>
kubectl describe role <nome>
```

---

## Próxima fase

➡️ [Fase 07 — Scheduling & Recursos](../fase-07-scheduling/index.md)
