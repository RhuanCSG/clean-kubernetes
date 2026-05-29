# DaemonSet

DaemonSet garante que **exatamente um Pod** rode em cada nó do cluster. Quando um novo nó é adicionado, o DaemonSet cria um Pod nele automaticamente. Quando um nó é removido, o Pod é coletado pelo garbage collector.

---

## Casos de uso

- **Coleta de logs:** Fluentd, Filebeat — precisam ler os arquivos de log de cada nó
- **Monitoramento:** Prometheus node-exporter, Datadog agent — métricas de cada nó
- **Rede:** CNI plugins (Calico, Cilium), kube-proxy — precisam rodar em todos os nós
- **Armazenamento:** agentes de CSI, provisionadores locais

---

## YAML de referência

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
spec:
  selector:
    matchLabels:
      app: log-agent
  template:
    metadata:
      labels:
        app: log-agent
    spec:
      containers:
        - name: agent
          image: busybox:1.36
          command: ["sh", "-c", "while true; do echo 'coletando logs'; sleep 30; done"]
          resources:
            requests:
              cpu: "50m"
              memory: "32Mi"
            limits:
              cpu: "100m"
              memory: "64Mi"
          volumeMounts:
            - name: varlog
              mountPath: /var/log
              readOnly: true
      volumes:
        - name: varlog
          hostPath:
            path: /var/log              # acessa logs do nó diretamente
```

---

## Verificando DaemonSets

```bash
kubectl get daemonsets
# NAME        DESIRED   CURRENT   READY   UP-TO-DATE   NODE SELECTOR
# log-agent   1         1         1       1            <none>

# Ver em quais nós o Pod está rodando
kubectl get pods -l app=log-agent -o wide
# NAME              READY   STATUS    NODE
# log-agent-xkj2m   1/1     Running   k8s-study-worker
```

`DESIRED` sempre igual ao número de nós elegíveis.

---

## DaemonSet vs. Deployment

| Característica | DaemonSet | Deployment |
|---|---|---|
| Número de réplicas | 1 por nó (automático) | Definido pelo usuário |
| Restrição de nó | Por padrão, todos os nós | Qualquer nó disponível |
| Caso de uso | Agentes de infraestrutura | Aplicações de negócio |

---

## Restringindo nós com nodeSelector ou affinity

Para rodar o DaemonSet apenas em nós específicos:

```yaml
spec:
  template:
    spec:
      nodeSelector:
        tipo: gpu               # apenas em nós com label tipo=gpu
```

---

## Próximo

➡️ [Job & CronJob](job-cronjob.md) — execuções que devem completar, não ficar rodando.
