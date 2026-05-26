# Job & CronJob

Jobs executam Pods até completar com sucesso — ao contrário de Deployments, que mantêm Pods rodando indefinidamente. CronJobs agendam a criação de Jobs em horários definidos.

---

## Job

### Quando usar

- Processamento de dados em lote (ETL, relatórios)
- Migrações de banco de dados
- Tarefas únicas de setup ou limpeza
- Qualquer operação que deve ter início e fim

### YAML de referência

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: processamento
spec:
  completions: 1                # número de completions bem-sucedidos necessários
  parallelism: 1                # Pods executando em paralelo ao mesmo tempo
  backoffLimit: 3               # tentativas antes de marcar o Job como Failed
  activeDeadlineSeconds: 300    # mata o Job se não completar em 5 minutos
  template:
    spec:
      restartPolicy: OnFailure  # OnFailure: reinicia o container no mesmo Pod
                                # Never: cria um novo Pod a cada falha
      containers:
        - name: worker
          image: busybox:1.36
          command: ["sh", "-c", "echo 'processamento concluído'; exit 0"]
```

!!! warning "`restartPolicy: Always` não é válido em Jobs"
    Jobs só aceitam `OnFailure` ou `Never`. `Always` (padrão dos Pods) causaria reinicialização infinita mesmo após sucesso.

### Verificando Jobs

```bash
kubectl get jobs
# NAME             COMPLETIONS   DURATION   AGE
# processamento    1/1           5s         1m

kubectl logs -l job-name=processamento    # logs de todos os Pods do Job
kubectl describe job processamento        # detalhes e eventos
```

### Job paralelo

Para processar múltiplos itens em paralelo:

```yaml
spec:
  completions: 10     # 10 completions necessários
  parallelism: 3      # 3 Pods rodando ao mesmo tempo
```

---

## CronJob

CronJob cria Jobs automaticamente em horários definidos usando a sintaxe cron.

### YAML de referência

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup
spec:
  schedule: "0 2 * * *"          # às 02:00 todo dia
  concurrencyPolicy: Forbid      # Forbid: não cria novo Job se o anterior ainda roda
                                  # Allow: permite execução paralela
                                  # Replace: cancela o anterior e cria novo
  successfulJobsHistoryLimit: 3   # mantém últimos 3 Jobs com sucesso
  failedJobsHistoryLimit: 1       # mantém último Job com falha
  startingDeadlineSeconds: 60     # se passou 60s do horário, pula esta execução
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup
              image: busybox:1.36
              command: ["sh", "-c", "echo 'backup executado em' $(date)"]
```

### Sintaxe cron

```
┌──────────── minuto (0-59)
│ ┌────────── hora (0-23)
│ │ ┌──────── dia do mês (1-31)
│ │ │ ┌────── mês (1-12)
│ │ │ │ ┌──── dia da semana (0-7, 0=domingo)
│ │ │ │ │
* * * * *

"*/2 * * * *"  → a cada 2 minutos
"0 * * * *"    → a cada hora (no minuto 0)
"0 2 * * *"    → às 02:00 todo dia
"0 2 * * 1"    → às 02:00 toda segunda-feira
"0 0 1 * *"    → meia-noite do dia 1 de cada mês
```

### Comandos essenciais

```bash
kubectl get cronjobs
kubectl describe cronjob backup
kubectl get jobs                              # Jobs criados pelo CronJob

# Disparar manualmente (sem aguardar o horário)
kubectl create job --from=cronjob/backup backup-manual
```

---

## Próximo

➡️ [Debugging](debugging.md) — rolling update travado e StatefulSet com Pod em Pending.
