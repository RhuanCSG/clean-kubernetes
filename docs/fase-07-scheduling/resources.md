# Requests e Limits

Requests e limits são a linguagem que o scheduler usa para entender quanto recurso cada container precisa e quanto pode consumir.

---

## Requests — reserva para o scheduler

`requests` define o mínimo de recurso **garantido** ao container. O scheduler usa esse valor para decidir em qual nó o Pod cabe:

```
Pod cabe no nó se:
  (recursos alocados no nó + requests do Pod) ≤ capacidade do nó
```

O container pode usar mais do que o valor de requests — é um piso, não um teto.

---

## Limits — teto de consumo

`limits` define o máximo que o container pode consumir:

| Recurso | Ultrapassar o limit |
|---|---|
| CPU | Throttling — o processo fica lento, mas não morre |
| Memória | **OOMKilled** — o kernel mata o processo; o container reinicia |

---

## YAML de referência

```yaml
spec:
  containers:
    - name: app
      image: nginx:1.25
      resources:
        requests:
          cpu: "100m"       # 100 millicores = 0.1 CPU
          memory: "128Mi"   # 128 MiB
        limits:
          cpu: "500m"       # 0.5 CPU — throttling acima disso
          memory: "256Mi"   # OOMKilled acima disso
```

### Unidades de CPU

| Valor | Equivalente |
|---|---|
| `1` | 1 CPU completo |
| `500m` | 0.5 CPU (500 millicores) |
| `100m` | 0.1 CPU |

### Unidades de memória

| Valor | Bytes |
|---|---|
| `128Mi` | 134.217.728 bytes (mebibytes) |
| `1Gi` | 1.073.741.824 bytes |
| `128M` | 128.000.000 bytes (megabytes — diferente de Mi!) |

---

## Quality of Service (QoS)

O Kubernetes classifica Pods em classes de QoS baseado nos resources definidos:

| Classe | Condição | Comportamento sob pressão de memória |
|---|---|---|
| `Guaranteed` | `requests == limits` para todos os containers | Último a ser removido |
| `Burstable` | `requests < limits` (mais comum) | Removido após BestEffort |
| `BestEffort` | Sem `requests` nem `limits` | Primeiro a ser removido |

=== "Linux / macOS"
    ```bash
    kubectl describe pod <nome> | grep QoS
    # QoS Class: Burstable
    ```

=== "Windows (PowerShell)"
    ```powershell
    kubectl describe pod <nome> | Select-String "QoS"
    # QoS Class: Burstable
    ```

---

## Diagnóstico de recursos

=== "Linux / macOS"
    ```bash
    # Ver recursos alocados vs. disponíveis por nó
    kubectl describe nodes | grep -A6 "Allocated resources"
    ```

=== "Windows (PowerShell)"
    ```powershell
    # Ver recursos alocados vs. disponíveis por nó
    kubectl describe nodes | Select-String -Context 0,6 "Allocated resources"
    ```

```bash
# Ver uso real (requer metrics-server)
kubectl top nodes
kubectl top pods
kubectl top pods --sort-by=memory    # ordenar por memória
```

---

## Próximo

➡️ [LimitRange e Quota](limitrange-quota.md) — como definir defaults e tetos por namespace.
