# Probes — Verificação de Saúde

Probes são verificações periódicas que o kubelet faz nos containers para determinar seu estado. Existem três tipos, com propósitos distintos.

---

## Os três tipos de probe

### `livenessProbe` — o container está vivo?

Se falhar, o kubelet **reinicia o container**. Usado para detectar deadlocks ou estados corrompidos de onde o processo não consegue se recuperar sozinho.

### `readinessProbe` — o container está pronto para receber tráfego?

Se falhar, o Pod é **removido dos Endpoints do Service** — para de receber requisições. O container não é reiniciado. Usado durante inicialização lenta ou sobrecarga temporária.

### `startupProbe` — o container terminou de inicializar?

Bloqueia as outras probes até completar. Usado quando a aplicação tem tempo de startup variável (ex: JVM aquecendo cache). Evita que a `livenessProbe` mate um container que ainda está inicializando.

---

## Diferença crítica: liveness vs. readiness

| Situação | liveness | readiness |
|---|---|---|
| App em deadlock, não responde | Reinicia o container | Remove do Service |
| App inicializando (demora 30s) | Matar antes de pronto (bug!) | Remove do Service durante init |
| App sobrecarregada, responde devagar | Reinicia (pode piorar) | Remove do Service (correto) |

!!! warning "liveness probe muito agressiva é perigosa"
    Uma liveness probe com `initialDelaySeconds` pequeno em uma aplicação com startup lento vai matar o container repetidamente, causando `CrashLoopBackOff`. Use `startupProbe` para esse caso.

---

## Mecanismos de verificação

### HTTP GET

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
    httpHeaders:
      - name: Custom-Header
        value: Awesome
```

O kubelet faz uma requisição HTTP GET. Status 2xx ou 3xx = sucesso. Qualquer outro = falha.

### TCP Socket

```yaml
livenessProbe:
  tcpSocket:
    port: 5432
```

Verifica se a porta está aberta. Útil para bancos de dados e serviços que não têm endpoint HTTP.

### Exec (comando)

```yaml
livenessProbe:
  exec:
    command:
      - sh
      - -c
      - "redis-cli ping | grep PONG"
```

Executa um comando dentro do container. Exit 0 = sucesso, qualquer outro = falha.

---

## YAML de referência completo

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-com-probes
spec:
  containers:
    - name: app
      image: nginx:1.25
      startupProbe:
        httpGet:
          path: /
          port: 80
        failureThreshold: 30      # tenta por até 30 * 10s = 5 minutos
        periodSeconds: 10         # antes de considerar falha definitiva
      livenessProbe:
        httpGet:
          path: /healthz
          port: 80
        initialDelaySeconds: 0    # após startupProbe ok, começa imediatamente
        periodSeconds: 10
        failureThreshold: 3       # 3 falhas consecutivas → reinicia
        timeoutSeconds: 5         # aguarda até 5s por resposta
      readinessProbe:
        httpGet:
          path: /ready
          port: 80
        initialDelaySeconds: 0
        periodSeconds: 5
        failureThreshold: 3       # 3 falhas → remove dos Endpoints
        successThreshold: 1       # 1 sucesso → retorna aos Endpoints
```

---

## Parâmetros comuns

| Parâmetro | Padrão | Descrição |
|---|---|---|
| `initialDelaySeconds` | 0 | Aguarda N segundos antes da primeira verificação |
| `periodSeconds` | 10 | Intervalo entre verificações |
| `timeoutSeconds` | 1 | Tempo máximo para a verificação responder |
| `failureThreshold` | 3 | Falhas consecutivas para marcar como falhou |
| `successThreshold` | 1 | Sucessos consecutivos para marcar como ok (readiness) |

---

## Inspecionando o estado das probes

```bash
kubectl describe pod <nome>
# Procure a seção "Containers" → "Liveness" e "Readiness"
# Procure em "Events" por "Liveness probe failed"
```

---

## Próximo

➡️ [Debugging](debugging.md) — CrashLoopBackOff, ImagePullBackOff e Pod em Pending.
