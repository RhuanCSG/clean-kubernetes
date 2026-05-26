# Debugging — Fase 02

## Metodologia para controllers

Controllers adicionam uma camada de indireção — o problema pode estar no controller, no ReplicaSet ou no Pod.

```
1. kubectl get <controller>          → estado geral (READY, UP-TO-DATE)
2. kubectl describe <controller>     → events do controller
3. kubectl get replicasets / pods    → estado dos objetos criados
4. kubectl describe pod <nome>       → o problema real geralmente está aqui
```

---

## Cenário 01 — Rolling Update Travado

### O que você vê

```bash
kubectl rollout status deployment/stuck-deploy
# Waiting for deployment "stuck-deploy" rollout to finish: 0 out of 2 new replicas have been updated...
# (nunca progride)
```

### Como investigar

```bash
kubectl describe deployment stuck-deploy
# Procure em "Strategy":
#   RollingUpdateStrategy:  0 max unavailable, 0 max surge
```

Com `maxUnavailable: 0` e `maxSurge: 0`, o Deployment não pode:

- Criar Pods novos (surge = 0)
- Remover Pods antigos (unavailable = 0)

É matematicamente impossível fazer o update. O rollout trava indefinidamente.

### A correção

```yaml
strategy:
  rollingUpdate:
    maxUnavailable: 0   # preservar disponibilidade
    maxSurge: 1         # criar 1 Pod extra durante o update
```

### Cenário de prática

```bash
kubectl apply -f phases/02-workload-controllers/debugging/01-rolling-stuck/broken.yaml
kubectl set image deployment/stuck-deploy app=nginx:1.25
kubectl rollout status deployment/stuck-deploy   # observe travando

# Após identificar o problema:
kubectl apply -f phases/02-workload-controllers/debugging/01-rolling-stuck/solution.yaml
```

---

## Cenário 02 — StatefulSet com Pod em Pending

### O que você vê

```bash
kubectl get pods -l app=db-debug
# NAME       READY   STATUS    RESTARTS   AGE
# db-debug-0 0/1     Pending   0          5m
```

### Como investigar

```bash
# 1. Ver eventos do Pod
kubectl describe pod db-debug-0
# Events:
#   Warning  FailedScheduling  scheduler  0/1 nodes are available:
#            pod has unbound immediate PersistentVolumeClaims

# 2. Verificar o PVC
kubectl get pvc
# NAME            STATUS    VOLUME   CAPACITY   STORAGECLASS
# data-db-debug-0 Pending             1Gi        nao-existe

# 3. O PVC está Pending — verificar StorageClasses disponíveis
kubectl get storageclass
# NAME                 PROVISIONER
# standard (default)   k8s.io/minikube-hostpath
```

A StorageClass `nao-existe` não existe no cluster. O PVC não consegue ser provisionado, então o Pod não pode ser criado.

### A correção

Mudar `storageClassName` no `volumeClaimTemplates` para uma StorageClass que existe:

```yaml
volumeClaimTemplates:
  - spec:
      storageClassName: "standard"    # StorageClass que existe no minikube
```

### Cenário de prática

```bash
kubectl apply -f phases/02-workload-controllers/debugging/02-statefulset-pvc/broken.yaml
kubectl get pods -l app=db-debug
kubectl describe pod db-debug-0
kubectl get pvc
kubectl get storageclass
```

---

## Outros problemas comuns em controllers

### Deployment não atualiza os Pods

```bash
kubectl get pods -l app=<nome> -o jsonpath='{range .items[*]}{.spec.containers[0].image}{"\n"}{end}'
```

Se todos os Pods ainda têm a imagem antiga mas o Deployment foi atualizado, verifique se o seletor de label mudou (Deployment imutável) ou se o update foi aplicado no contexto (namespace) errado.

### Job em estado Failed

```bash
kubectl describe job <nome>
# Procure: "BackoffLimitExceeded" — atingiu o limite de tentativas
kubectl logs -l job-name=<nome> --previous    # ver o erro que causou a falha
```

---

## Próxima fase

➡️ [Fase 03 — Networking](../fase-03-networking/index.md)
