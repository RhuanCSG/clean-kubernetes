# Deployment

O Deployment é o controller mais usado no dia a dia. Ele não gerencia Pods diretamente — ele gerencia **ReplicaSets**, que por sua vez gerenciam os Pods.

---

## Hierarquia de controle

```
Deployment
  └── ReplicaSet (versão atual)
        ├── Pod
        ├── Pod
        └── Pod
  └── ReplicaSet (versão anterior, 0 réplicas — para rollback)
```

Essa separação em duas camadas é o que torna o rolling update e o rollback possíveis: o Deployment apenas ajusta quantas réplicas cada ReplicaSet deve ter.

---

## YAML de referência

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: minha-app            # DEVE ser idêntico a spec.template.metadata.labels
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1         # Pods indisponíveis durante o update (absoluto ou %)
      maxSurge: 1               # Pods extras criados durante o update (absoluto ou %)
  template:
    metadata:
      labels:
        app: minha-app          # DEVE bater com spec.selector.matchLabels
    spec:
      containers:
        - name: app
          image: nginx:1.24
          resources:
            requests:
              cpu: "100m"
              memory: "64Mi"
            limits:
              cpu: "200m"
              memory: "128Mi"
```

!!! warning "selector e labels devem bater exatamente"
    `spec.selector.matchLabels` e `spec.template.metadata.labels` devem ter exatamente os mesmos valores. Se não baterem, o Deployment não consegue gerenciar seus próprios Pods e a criação falha.

---

## Rolling Update

O rolling update substitui Pods da versão antiga pela nova gradualmente, respeitando `maxUnavailable` e `maxSurge`:

```bash
# Disparar update (muda a imagem)
kubectl set image deployment/app-deployment app=nginx:1.25

# Ou editar o YAML e aplicar
kubectl apply -f deployment.yaml

# Acompanhar em tempo real
kubectl rollout status deployment/app-deployment
# Waiting for deployment "app-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
# deployment "app-deployment" successfully rolled out

# Ver os ReplicaSets durante o update
kubectl get replicasets -w
```

### O que acontece por baixo

Com `maxUnavailable: 1` e `maxSurge: 1` para 3 réplicas:

```
Estado inicial: RS-v1 com 3 Pods (nginx:1.24)

1. Deployment cria RS-v2 com 1 Pod (nginx:1.25) → 4 Pods total (surge=1)
2. Após RS-v2 Pod ficar Ready, diminui RS-v1 para 2 → 3 Pods total
3. Cria mais 1 Pod em RS-v2 → 4 Pods
4. Após Ready, diminui RS-v1 para 1 → 3 Pods
5. Cria mais 1 Pod em RS-v2 → 4 Pods
6. Após Ready, diminui RS-v1 para 0 → 3 Pods todos na nova versão

Estado final: RS-v2 com 3 Pods (nginx:1.25), RS-v1 com 0 Pods (preservado para rollback)
```

---

## Rollback

```bash
# Ver histórico de revisões
kubectl rollout history deployment/app-deployment

# Rollback para a revisão anterior
kubectl rollout undo deployment/app-deployment

# Rollback para uma revisão específica
kubectl rollout undo deployment/app-deployment --to-revision=2

# Verificar qual versão está rodando após o rollback
kubectl get pods -l app=minha-app -o jsonpath='{.items[0].spec.containers[0].image}'
```

O rollback simplesmente inverte o processo: aumenta as réplicas do RS antigo e diminui as do atual.

---

## Estratégias de update

### RollingUpdate (padrão)

Substitui Pods gradualmente. Zero downtime se configurado corretamente.

### Recreate

```yaml
strategy:
  type: Recreate    # mata todos os Pods antes de criar os novos
```

Causa downtime. Útil quando a nova versão é incompatível com a antiga e não podem rodar simultaneamente.

---

## Comandos essenciais

```bash
kubectl get deployments
kubectl describe deployment <nome>
kubectl rollout status deployment/<nome>
kubectl rollout history deployment/<nome>
kubectl rollout undo deployment/<nome>
kubectl scale deployment/<nome> --replicas=5
kubectl get replicasets                          # ver todos os RSes (incluindo antigos)
```

---

## Próximo

➡️ [StatefulSet](statefulset.md) — quando a identidade do Pod importa.
