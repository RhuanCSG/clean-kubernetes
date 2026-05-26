# StatefulSet

StatefulSet gerencia Pods que precisam de **identidade estável**: nome fixo, storage dedicado e ordem de criação garantida. É o controller certo para bancos de dados, filas e qualquer aplicação com estado.

---

## Por que não usar Deployment para banco de dados?

Com um Deployment, os Pods são intercambiáveis:

- Nomes aleatórios (`app-7d9f8b6c4-xkj2m`)
- Volumes compartilhados ou sem persistência garantida por Pod
- Podem subir em qualquer ordem

Um banco de dados precisa de:

- Nome previsível (`db-0`, `db-1`) — para que o primário saiba quais são suas réplicas
- Storage exclusivo por Pod — cada réplica tem seus próprios dados
- Ordem de inicialização — o primário deve subir antes das réplicas

---

## YAML de referência

```yaml
# Headless Service — obrigatório para StatefulSet
apiVersion: v1
kind: Service
metadata:
  name: db-headless
spec:
  clusterIP: None               # headless: DNS aponta direto para cada Pod
  selector:
    app: db
  ports:
    - port: 5432

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db
spec:
  serviceName: "db-headless"    # referencia o headless service
  replicas: 2
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
        - name: db
          image: postgres:15
          env:
            - name: POSTGRES_PASSWORD
              value: "senha-local"
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:         # cria um PVC por Pod: data-db-0, data-db-1
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

---

## Identidade estável

Cada Pod tem nome previsível: `<statefulset-name>-<ordinal>`:

```bash
kubectl get pods -l app=db
# NAME   READY   STATUS    RESTARTS   AGE
# db-0   1/1     Running   0          1m
# db-1   1/1     Running   0          45s
```

E o DNS interno de cada Pod é alcançável por:
```
db-0.db-headless.default.svc.cluster.local
db-1.db-headless.default.svc.cluster.local
```

Isso permite que `db-0` (primário) seja configurado fixamente como líder.

---

## volumeClaimTemplates

Diferente do Deployment, que compartilha volumes entre Pods, o StatefulSet cria um PVC exclusivo por Pod usando o `volumeClaimTemplates`. Os PVCs **não são deletados** quando o Pod ou o StatefulSet é deletado — os dados persistem.

```bash
kubectl get pvc
# NAME        STATUS   VOLUME    CAPACITY   ACCESS MODES
# data-db-0   Bound    pv-xxx    1Gi        RWO
# data-db-1   Bound    pv-yyy    1Gi        RWO
```

---

## Ordem de criação e deleção

- **Criação:** Pods sobem em ordem crescente (`db-0` → `db-1`). O próximo só sobe após o anterior estar `Ready`.
- **Deleção:** Pods são removidos em ordem decrescente (`db-1` → `db-0`).
- **Update:** por padrão (`RollingUpdate`), atualiza de `db-N` até `db-0`.

---

## Comandos essenciais

```bash
kubectl get statefulsets
kubectl describe statefulset <nome>
kubectl get pvc                          # PVCs criados pelo volumeClaimTemplates
kubectl rollout status statefulset/<nome>
kubectl scale statefulset/<nome> --replicas=3
```

---

## Próximo

➡️ [DaemonSet](daemonset.md) — garantir um Pod em cada nó do cluster.
