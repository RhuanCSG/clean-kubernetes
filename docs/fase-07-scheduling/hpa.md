# HorizontalPodAutoscaler (HPA)

HPA escala automaticamente o número de réplicas de um Deployment (ou StatefulSet) baseado em métricas observadas — CPU, memória ou métricas customizadas.

---

## Pré-requisito: metrics-server

O HPA precisa do metrics-server para coletar métricas de uso:

```bash
# Habilitar no minikube
minikube addons enable metrics-server

# Verificar instalação
kubectl get pods -n kube-system | grep metrics-server
# Windows (PowerShell): kubectl get pods -n kube-system | Select-String "metrics-server"
kubectl top nodes     # deve retornar dados (não erro)
```

---

## Como o HPA funciona

```
Loop a cada 15 segundos:
  1. Coleta métricas dos Pods via metrics-server
  2. Calcula a utilização média atual
  3. Calcula número desejado de réplicas:
     réplicas = ceil(atual * (uso_atual / uso_alvo))
  4. Atualiza o número de réplicas do Deployment
```

---

## YAML de referência

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: minha-app                   # Deployment a ser escalado
  minReplicas: 2                      # nunca abaixo de 2
  maxReplicas: 10                     # nunca acima de 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70      # alvo: 70% de uso de CPU
    - type: Resource
      resource:
        name: memory
        target:
          type: AverageValue
          averageValue: 200Mi         # alvo: 200Mi por Pod
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # aguarda 5min antes de fazer scale down
    scaleUp:
      stabilizationWindowSeconds: 0    # scale up imediato
```

---

## Monitorando o HPA

```bash
kubectl get hpa
# NAME      REFERENCE            TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
# app-hpa   Deployment/minha-app 15%/70%   2         10        2          1m

kubectl describe hpa app-hpa
# Metrics:  ( current / target )
#   resource cpu on pods:  15% / 70%
# Events:
#   ScalingReplicaSet  Scaled up replica set minha-app-xxx to 4
```

---

## Gerando carga para testar

```bash
# Pod de teste que gera carga no serviço
kubectl run load-test --image=busybox:1.36 --restart=Never -- \
  sh -c "while true; do wget -q -O- http://minha-app-svc; done"
# Windows (PowerShell): use backtick (`) no lugar de \ para quebra de linha:
# kubectl run load-test --image=busybox:1.36 --restart=Never -- `
#   sh -c "while true; do wget -q -O- http://minha-app-svc; done"

# Em outro terminal, observar o HPA escalar
kubectl get hpa app-hpa -w
```

---

## Limitações importantes

!!! warning "HPA não funciona sem requests definidos"
    O HPA calcula a utilização como `uso_atual / requests.cpu`. Se o container não tem `requests` definido, o HPA não consegue calcular e fica sem métricas.

!!! warning "HPA e réplicas manuais não combinam"
    Se você tem um HPA ativo e usa `kubectl scale deployment/minha-app --replicas=5`, o HPA vai reverter o número de réplicas na próxima iteração.

---

## Próximo

➡️ [Debugging](debugging.md) — OOMKilled e Pod em Pending por taint.
