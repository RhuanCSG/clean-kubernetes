# Debugging — Fase 05

## Metodologia para problemas de storage

```
1. kubectl get pvc                   → qual é o status? (Bound ou Pending?)
2. kubectl describe pvc <nome>       → qual PV foi vinculado? qual o erro?
3. kubectl get storageclass          → a StorageClass referenciada existe?
4. kubectl describe pod <nome>       → o Pod consegue montar o volume?
```

---

## Cenário 01 — PVC em Pending

### O que você vê

```bash
kubectl get pvc
# NAME          STATUS    VOLUME   CAPACITY   STORAGECLASS
# pvc-quebrado  Pending             1Gi        premium-ssd
```

O PVC foi criado mas não encontrou um PV compatível para se vincular.

### Como investigar

```bash
kubectl describe pvc pvc-quebrado
# Events:
#   Warning  ProvisioningFailed  storageclass.storage.k8s.io "premium-ssd" not found

kubectl get storageclass
# NAME                 PROVISIONER                    AGE
# standard (default)   k8s.io/minikube-hostpath       1d
# (não existe "premium-ssd")
```

A StorageClass `premium-ssd` não existe no cluster. O provisioner nunca é chamado, o PV nunca é criado, o PVC fica `Pending` indefinidamente.

### A correção

```yaml
spec:
  storageClassName: standard    # StorageClass que existe no cluster
```

### Cenário de prática

```bash
kubectl apply -f phases/05-storage/debugging/01-pvc-pending/broken.yaml
kubectl get pvc pvc-quebrado   # deve mostrar Pending
kubectl describe pvc pvc-quebrado
kubectl get storageclass
```

---

## Cenário 02 — Pod não consegue montar o volume

### O que você vê

```bash
kubectl describe pod app-storage-pod
# Events:
#   Warning  FailedMount  kubelet  Unable to attach or mount volumes:
#            persistentvolumeclaim "dados-pvc" not found
```

### Como investigar

```bash
# O PVC existe?
kubectl get pvc
# Se não existe, criar o PVC antes de criar o Pod

# O PVC está Bound?
kubectl get pvc dados-pvc
# STATUS deve ser Bound, não Pending
```

Um Pod que referencia um PVC não-existente ou em Pending fica em estado `Pending` também.

---

## Cenário 03 — Container sem permissão de escrita

### O que você vê

A aplicação falha com `permission denied` ao tentar escrever no volume.

### Como investigar

```bash
kubectl exec <pod> -- ls -la /dados
# drwxr-xr-x  root root  ← o container não tem permissão de escrita

kubectl exec <pod> -- id
# uid=1000(app) gid=1000(app)   ← processo rodando como usuário não-root
```

### A correção

Usar `securityContext` para definir o grupo de propriedade do volume:

```yaml
spec:
  securityContext:
    fsGroup: 1000               # define o grupo dono dos arquivos montados
  containers:
    - name: app
      securityContext:
        runAsUser: 1000
        runAsGroup: 1000
```

---

## Referência rápida

```bash
kubectl get pvc
kubectl get pv
kubectl describe pvc <nome>
kubectl get storageclass
kubectl describe storageclass <nome>
kubectl get events --field-selector involvedObject.name=<pod-com-problema>
```

---

## Próxima fase

➡️ [Fase 06 — RBAC](../fase-06-rbac/index.md)
