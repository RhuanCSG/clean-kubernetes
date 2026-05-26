# Affinity

Affinity é uma forma mais expressiva de controlar onde Pods são agendados. Enquanto `nodeSelector` só permite igualdade exata, affinity suporta operadores lógicos, condições preferenciais e relações entre Pods.

---

## nodeSelector vs. nodeAffinity

```yaml
# nodeSelector — simples, apenas igualdade
nodeSelector:
  tipo: gpu

# nodeAffinity — expressivo, suporta operadores
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: tipo
              operator: In
              values: ["gpu", "tpu"]    # "In" não existe em nodeSelector
```

---

## Tipos de affinity

### nodeAffinity — afinidade com nós

```yaml
affinity:
  nodeAffinity:
    # Obrigatório: Pod não é agendado se não satisfeito
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/os
              operator: In
              values: ["linux"]

    # Preferencial: scheduler tenta satisfazer, mas não é obrigatório
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80               # peso 0-100; maior = mais preferido
        preference:
          matchExpressions:
            - key: zona
              operator: In
              values: ["us-east-1a"]
      - weight: 20
        preference:
          matchExpressions:
            - key: tipo
              operator: In
              values: ["high-memory"]
```

### podAffinity — co-localizar com outros Pods

```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: cache              # ir para o mesmo nó que o Pod de cache
        topologyKey: kubernetes.io/hostname   # "mesmo nó"
```

### podAntiAffinity — separar Pods de outros Pods

```yaml
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              app: minha-app        # evitar o mesmo nó que outras réplicas
          topologyKey: kubernetes.io/hostname
```

---

## YAML de referência completo

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-com-affinity
spec:
  replicas: 3
  selector:
    matchLabels:
      app: minha-app
  template:
    metadata:
      labels:
        app: minha-app
    spec:
      affinity:
        # Nó deve rodar Linux
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: kubernetes.io/os
                    operator: In
                    values: ["linux"]
        # Preferir distribuir réplicas em nós diferentes
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app: minha-app
                topologyKey: kubernetes.io/hostname
      containers:
        - name: app
          image: nginx:1.25
```

---

## Operators disponíveis para matchExpressions

| Operator | Descrição |
|---|---|
| `In` | O valor do label está na lista |
| `NotIn` | O valor do label não está na lista |
| `Exists` | O label existe (qualquer valor) |
| `DoesNotExist` | O label não existe |
| `Gt` | Valor do label é maior que o especificado (numérico) |
| `Lt` | Valor do label é menor que o especificado (numérico) |

---

## Próximo

➡️ [HPA](hpa.md) — escalonamento automático baseado em métricas.
