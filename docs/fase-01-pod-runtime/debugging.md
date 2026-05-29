# Debugging — Fase 01

## Metodologia de investigação

Sempre nesta ordem:

```
1. kubectl get pod <nome>          → qual é o status?
2. kubectl describe pod <nome>     → o que aconteceu? (leia os Events)
3. kubectl logs <nome>             → o que o container diz?
4. kubectl logs <nome> --previous  → e o container anterior (se reiniciou)?
```

---

## Cenário 01 — CrashLoopBackOff

### O que você vê

```bash
kubectl get pod crash-pod
# NAME        READY   STATUS             RESTARTS   AGE
# crash-pod   0/1     CrashLoopBackOff   4          2m
```

O container está reiniciando em loop. O Kubernetes aplica backoff exponencial entre as tentativas (10s, 20s, 40s...), por isso o status fica alternando entre `Error` e `CrashLoopBackOff`.

### Como investigar

```bash
# 1. Ver os eventos
kubectl describe pod crash-pod
# Procure em "Events":
# Warning  BackOff  kubelet  Back-off restarting failed container

# 2. Logs da tentativa atual
kubectl logs crash-pod

# 3. Logs da tentativa anterior (após pelo menos 1 restart)
kubectl logs crash-pod --previous
```

### Causas comuns

| Causa | Como identificar |
|---|---|
| Liveness probe com path errado | Events: `Liveness probe failed: HTTP probe failed with statuscode: 404` |
| Container termina imediatamente (exit code != 0) | `kubectl logs --previous` mostra o erro |
| OOMKilled — memória insuficiente | `kubectl describe`: `Last State: Terminated Reason: OOMKilled` |
| Aplicação com bug na inicialização | Logs mostram exception/panic antes de terminar |

### Cenário de prática

Ver `fases/01-pod-runtime/debugging/01-crashloop/` no repositório:

```bash
# Aplicar o manifesto com problema
kubectl apply -f fases/01-pod-runtime/debugging/01-crashloop/broken.yaml

# Investigar
kubectl get pod crash-pod -w
kubectl describe pod crash-pod
kubectl logs crash-pod --previous

# Após encontrar o problema, comparar com a solução
# fases/01-pod-runtime/debugging/01-crashloop/solution.yaml
```

---

## Cenário 02 — ImagePullBackOff

### O que você vê

```bash
kubectl get pod pull-fail-pod
# NAME           READY   STATUS             RESTARTS   AGE
# pull-fail-pod  0/1     ImagePullBackOff   0          30s
```

O Kubernetes não conseguiu baixar a imagem do container.

### Como investigar

```bash
kubectl describe pod pull-fail-pod
# Procure em "Events":
# Warning  Failed  kubelet  Failed to pull image "nginx:tag-errada": rpc error: ... not found
```

### Causas comuns

| Causa | Como identificar |
|---|---|
| Tag da imagem não existe | Events: `manifest unknown` ou `not found` |
| Imagem privada sem credencial | Events: `unauthorized: authentication required` |
| Registry inacessível | Events: `connection refused` ou timeout |
| Nome da imagem com typo | Events: `not found` |

### Cenário de prática

```bash
kubectl apply -f fases/01-pod-runtime/debugging/02-imagepull/broken.yaml
kubectl describe pod pull-fail-pod
# Identifique qual tag está errada nos Events
```

---

## Cenário 03 — Pod em Pending

### O que você vê

```bash
kubectl get pod meu-pod
# NAME      READY   STATUS    RESTARTS   AGE
# meu-pod   0/1     Pending   0          5m
```

O Pod foi aceito pelo cluster mas ainda não foi agendado em nenhum nó.

### Como investigar

```bash
kubectl describe pod meu-pod
# Procure em "Events":
# Warning  FailedScheduling  scheduler  0/1 nodes are available: ...
```

### Causas comuns

| Causa | Mensagem do scheduler |
|---|---|
| CPU/memória insuficiente no cluster | `Insufficient cpu` ou `Insufficient memory` |
| Nó com taint sem toleration no Pod | `node(s) had untolerated taint` |
| nodeSelector sem nó correspondente | `node(s) didn't match Pod's node affinity` |
| PVC não provisionado | Pod aguarda PVC — veja `kubectl get pvc` |

!!! tip "Para ver recursos disponíveis nos nós"
    === "Linux / macOS"
        ```bash
        kubectl describe nodes | grep -A5 "Allocated resources"
        ```

    === "Windows (PowerShell)"
        ```powershell
        kubectl describe nodes | Select-String -Context 0,5 "Allocated resources"
        ```

---

## Comandos de referência rápida

```bash
# Ciclo completo de investigação
kubectl get pod <nome>
kubectl describe pod <nome>
kubectl logs <nome>
kubectl logs <nome> --previous
kubectl get events --sort-by=.lastTimestamp

# Filtrar eventos de um Pod específico
kubectl get events --field-selector involvedObject.name=<nome>
```

---

## Próxima fase

➡️ [Fase 02 — Workload Controllers](../fase-02-workload-controllers/index.md)
