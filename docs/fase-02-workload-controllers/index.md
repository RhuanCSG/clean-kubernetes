# Fase 02 — Workload Controllers

**Camada:** controllers | **Estimativa:** ~3 semanas | **Ambiente:** kind

Controllers são loops de reconciliação que garantem que o estado real do cluster corresponde ao estado desejado. Esta fase cobre os principais tipos de workload que você gerencia no dia a dia.

---

## O que você vai aprender

- Por que o Kubernetes usa controllers em vez de gerenciar Pods diretamente
- Como o Deployment orquestra ReplicaSets para rolling updates e rollbacks
- Quando usar StatefulSet, DaemonSet, Job e CronJob
- O modelo mental do reconciliation loop

---

## O modelo mental dos controllers

Todo controller segue o mesmo padrão:

```
loop infinito:
  estado_atual = observar o cluster
  estado_desejado = ler a spec
  se estado_atual != estado_desejado:
    executar ações para convergir
```

O `kube-controller-manager` roda todos esses loops simultaneamente. Você nunca instrui um controller a "fazer X" — você declara o estado que deseja, e o controller faz o necessário para chegar lá.

---

## Tópicos desta fase

1. **[Deployment](deployment.md)** — o controller mais usado; rolling update e rollback
2. **[StatefulSet](statefulset.md)** — Pods com identidade estável e storage dedicado
3. **[DaemonSet](daemonset.md)** — garantir um Pod por nó do cluster
4. **[Job & CronJob](job-cronjob.md)** — execuções únicas e agendadas

---

## Lab da fase

Executar rolling update de um Deployment, acompanhar com `kubectl rollout status` e fazer rollback com `kubectl rollout undo`.

Ver: `fases/02-workload-controllers/labs/lab.md` no repositório.

---

## Pronto quando

- [ ] Executar rolling update e rollback de um Deployment
- [ ] Explicar o que o ReplicaSet antigo (com 0 réplicas) está fazendo após o update
- [ ] Resolver o cenário `01-rolling-stuck` sem ajuda
- [ ] Resolver o cenário `02-statefulset-pvc` sem ajuda

---

## Próxima fase

➡️ [Fase 03 — Networking](../fase-03-networking/index.md)
