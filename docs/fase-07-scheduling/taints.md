# Taints e Tolerations

Taints permitem que nós **repelem** Pods. Tolerations permitem que Pods **tolerem** taints e sejam agendados em nós que de outra forma os recusariam.

---

## O mecanismo

```
Nó com taint: "Só aceito Pods com toleration para mim"
Pod sem toleration: "Não posso ir para esse nó" → Pending
Pod com toleration: "Posso ignorar esse taint" → agendado
```

---

## Gerenciando taints em nós

```bash
# Adicionar taint
kubectl taint nodes k8s-study-worker dedicated=gpu:NoSchedule
#                   ↑ nó            ↑ key=value  ↑ effect

# Ver taints de todos os nós
kubectl describe nodes | grep Taints
# Windows (PowerShell): kubectl describe nodes | Select-String "Taints"

# Remover taint (note o "-" no final)
kubectl taint nodes k8s-study-worker dedicated=gpu:NoSchedule-
```

---

## Effects disponíveis

| Effect | Comportamento |
|---|---|
| `NoSchedule` | Novos Pods sem toleration não são agendados; Pods existentes continuam |
| `PreferNoSchedule` | Scheduler evita o nó, mas agenda se não houver alternativa |
| `NoExecute` | Novos Pods sem toleration não são agendados **e** Pods existentes são expulsos |

---

## YAML de referência — Pod com toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-gpu
spec:
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "gpu"
      effect: "NoSchedule"
    # Toleration para NoExecute com tolerationSeconds:
    # - key: "node.kubernetes.io/not-ready"
    #   operator: "Exists"
    #   effect: "NoExecute"
    #   tolerationSeconds: 300    # aguarda 300s antes de ser expulso
  containers:
    - name: app
      image: nvidia/cuda:12.0-base
```

---

## Operators

| Operator | Quando usar |
|---|---|
| `Equal` | Comparação exata de key e value |
| `Exists` | Tolera qualquer taint com essa key, independente do value |

```yaml
# Tolera qualquer taint com key "dedicated", qualquer value e effect
tolerations:
  - key: "dedicated"
    operator: "Exists"
```

---

## Caso de uso: nós dedicados

```bash
# Marcar nós de GPU para uso exclusivo
kubectl taint nodes gpu-node-1 dedicated=gpu:NoSchedule
kubectl label nodes gpu-node-1 accelerator=gpu

# Pod que vai para o nó de GPU (e só ele)
spec:
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "gpu"
      effect: "NoSchedule"
  nodeSelector:
    accelerator: gpu              # garante que vai para o nó certo
```

---

## Taints automáticos do Kubernetes

O Kubernetes adiciona taints automaticamente em alguns casos:

| Taint | Quando | Effect |
|---|---|---|
| `node.kubernetes.io/not-ready` | Nó sem comunicação | NoExecute |
| `node.kubernetes.io/unreachable` | Nó inacessível | NoExecute |
| `node.kubernetes.io/memory-pressure` | Nó com pressão de memória | NoSchedule |
| `node.kubernetes.io/disk-pressure` | Nó com pressão de disco | NoSchedule |

---

## Próximo

➡️ [Affinity](affinity.md) — regras mais expressivas de co-localização de Pods.
