# LimitRange e ResourceQuota

LimitRange e ResourceQuota permitem que administradores controlem o uso de recursos por namespace — definindo defaults para containers e tetos totais para o namespace inteiro.

---

## LimitRange — defaults e limites por container

LimitRange define valores padrão de requests e limits para containers que não especificam os seus próprios. Também pode definir valores mínimos e máximos permitidos.

### YAML de referência

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limites-padrao
  namespace: dev
spec:
  limits:
    - type: Container
      default:                # limits padrão (se container não definir limits)
        cpu: "500m"
        memory: "256Mi"
      defaultRequest:         # requests padrão (se container não definir requests)
        cpu: "100m"
        memory: "128Mi"
      max:                    # container não pode definir limits maiores que isso
        cpu: "2"
        memory: "1Gi"
      min:                    # container não pode definir requests menores que isso
        cpu: "50m"
        memory: "64Mi"
```

### Verificando defaults injetados

```bash
kubectl apply -f limitrange.yaml
kubectl run sem-limits --image=nginx:1.25 --restart=Never
kubectl describe pod sem-limits | grep -A6 "Limits:"
# Windows (PowerShell): kubectl describe pod sem-limits | Select-String -Context 0,6 "Limits:"
# Limits:
#   cpu:     500m         ← injetado pelo LimitRange
#   memory:  256Mi
# Requests:
#   cpu:     100m         ← injetado pelo LimitRange
#   memory:  128Mi
```

---

## ResourceQuota — teto total por namespace

ResourceQuota define limites para o consumo total de recursos e o número de objetos em um namespace.

### YAML de referência

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-dev
  namespace: dev
spec:
  hard:
    # Limites de recursos totais
    requests.cpu: "4"            # soma de todos os requests.cpu do namespace
    requests.memory: "4Gi"
    limits.cpu: "8"
    limits.memory: "8Gi"
    # Limites de número de objetos
    pods: "20"
    services: "10"
    configmaps: "20"
    secrets: "20"
    persistentvolumeclaims: "10"
```

### Verificando o uso atual

```bash
kubectl describe resourcequota quota-dev -n dev
# Name:               quota-dev
# Namespace:          dev
# Resource            Used    Hard
# --------            ----    ----
# limits.cpu          400m    8
# limits.memory       512Mi   8Gi
# pods                2       20
# requests.cpu        200m    4
# requests.memory     256Mi   4Gi
```

### Comportamento ao exceder a quota

Se um Pod for criado e ultrapassar a quota:

```bash
kubectl run novo-pod --image=nginx:1.25
# Error from server (Forbidden): pods "novo-pod" is forbidden:
# exceeded quota: quota-dev, requested: pods=1, used: pods=20, limited: pods=20
```

---

## LimitRange + ResourceQuota juntos

Uma configuração típica de namespace seguro:

```
LimitRange: define defaults para que nenhum Pod fique sem resources (necessário para que ResourceQuota funcione com pods sem resources explícitos)

ResourceQuota: garante que o namespace não consume mais do que seu share do cluster
```

!!! tip "LimitRange é prerequisito para ResourceQuota funcionar bem"
    Se um namespace tem ResourceQuota mas não tem LimitRange, Pods sem `requests` explícito são rejeitados pela quota (quota não sabe quanto reservar para eles).

---

## Próximo

➡️ [Taints e Tolerations](taints.md) — controlar quais Pods vão a quais nós.
