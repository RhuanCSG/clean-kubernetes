# Debugging — Fase 07

## Metodologia para problemas de scheduling

Quando um Pod fica `Pending`, sempre comece aqui:

```bash
kubectl describe pod <nome>
# Seção "Events" mostrará por que o scheduler não conseguiu alocar:
# "0/3 nodes are available: ..."
# A mensagem explica exatamente o que está faltando
```

---

## Cenário 01 — OOMKilled

### O que você vê

```bash
kubectl get pod oom-pod
# NAME      READY   STATUS             RESTARTS   AGE
# oom-pod   0/1     CrashLoopBackOff   3          2m
```

```bash
kubectl describe pod oom-pod
# Last State:     Terminated
#   Reason:       OOMKilled
#   Exit Code:    137
```

O container usou mais memória do que o `limits.memory` permitia. O kernel Linux matou o processo com sinal SIGKILL (exit code 137).

### Como investigar

```bash
kubectl describe pod oom-pod | grep -A5 "Last State"
# Windows (PowerShell): kubectl describe pod oom-pod | Select-String -Context 0,5 "Last State"
# Reason: OOMKilled    ← confirmação

kubectl describe pod oom-pod | grep -A4 "Limits"
# Windows (PowerShell): kubectl describe pod oom-pod | Select-String -Context 0,4 "Limits"
# Limits:
#   memory: 5Mi        ← muito baixo para o nginx

kubectl top pod oom-pod --containers    # uso real (se ainda estiver rodando)
```

### A correção

Aumentar o `limits.memory` para um valor que a aplicação consiga usar:

```yaml
resources:
  requests:
    memory: "64Mi"
  limits:
    memory: "128Mi"    # suficiente para o nginx
```

### Cenário de prática

```bash
kubectl apply -f phases/07-scheduling/debugging/01-oomkilled/broken.yaml
kubectl get pod oom-pod -w
kubectl describe pod oom-pod | grep -A5 "Last State"
# Windows (PowerShell): kubectl describe pod oom-pod | Select-String -Context 0,5 "Last State"
```

---

## Cenário 02 — Pod Pending por taint sem toleration

### O que você vê

```bash
kubectl get pod pending-pod
# NAME          READY   STATUS    RESTARTS   AGE
# pending-pod   0/1     Pending   0          5m
```

```bash
kubectl describe pod pending-pod
# Events:
#   Warning  FailedScheduling  scheduler  
#   0/2 nodes are available:
#   1 node(s) had untolerated taint {env: prod}: (effect NoSchedule),
#   1 node(s) were unschedulable.
```

O único nó worker tem um taint `env=prod:NoSchedule` e o Pod não tem a toleration correspondente.

### Como investigar

```bash
# Ver taints em todos os nós
kubectl describe nodes | grep Taints
# Windows (PowerShell): kubectl describe nodes | Select-String "Taints"
# Taints: env=prod:NoSchedule

# Confirmar que o Pod não tem toleration
kubectl get pod pending-pod -o yaml | grep -A5 tolerations
# Windows (PowerShell): kubectl get pod pending-pod -o yaml | Select-String -Context 0,5 "tolerations"
# (vazio ou ausente)
```

### A correção

Adicionar a toleration ao Pod:

```yaml
spec:
  tolerations:
    - key: "env"
      operator: "Equal"
      value: "prod"
      effect: "NoSchedule"
```

### Cenário de prática

```bash
# Pré-requisito: adicionar taint no nó worker
kubectl taint nodes k8s-study-worker env=prod:NoSchedule

kubectl apply -f phases/07-scheduling/debugging/02-pending-taint/broken.yaml
kubectl get pod pending-pod
kubectl describe pod pending-pod
```

---

## Cenário 03 — Pod Pending por recursos insuficientes

### O que você vê

```bash
kubectl describe pod grande-pod
# Events:
#   0/2 nodes are available:
#   2 Insufficient cpu.
```

### Como investigar

```bash
# Ver capacidade e uso atual dos nós
kubectl describe nodes | grep -A8 "Allocated resources"
# Windows (PowerShell): kubectl describe nodes | Select-String -Context 0,8 "Allocated resources"
# Resource           Requests    Limits
# cpu                1800m/2     2200m/2    ← quase no limite
# memory             1500Mi/2Gi  2Gi/2Gi

# Ver requests do Pod que não consegue alocar
kubectl get pod grande-pod -o yaml | grep -A4 requests
# Windows (PowerShell): kubectl get pod grande-pod -o yaml | Select-String -Context 0,4 "requests"
# cpu: 500m    ← não cabe no nó com 200m livre
```

### A correção

Reduzir os requests ou adicionar um nó ao cluster.

---

## Referência rápida

```bash
# Mensagem completa do scheduler
kubectl describe pod <nome> | grep -A5 "Events"
# Windows (PowerShell): kubectl describe pod <nome> | Select-String -Context 0,5 "Events"

# Recursos alocados por nó
kubectl describe nodes | grep -A10 "Allocated resources"
# Windows (PowerShell): kubectl describe nodes | Select-String -Context 0,10 "Allocated resources"

# Taints em todos os nós
kubectl describe nodes | grep Taints
# Windows (PowerShell): kubectl describe nodes | Select-String "Taints"

# Uso real de recursos
kubectl top nodes
kubectl top pods
```

---

## Próxima fase

➡️ [Fase 08 — Internals do Control Plane](../fase-08-control-plane/index.md)
