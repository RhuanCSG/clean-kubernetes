# InitContainers

InitContainers são containers especiais que executam **antes** dos containers principais de um Pod, em sequência, e devem terminar com sucesso para que o Pod prossiga.

---

## Por que InitContainers existem

Casos de uso comuns:

- **Aguardar dependências:** esperar um banco de dados ou serviço ficar disponível antes de iniciar a aplicação
- **Preparar dados:** baixar arquivos, gerar configurações, popular um diretório compartilhado
- **Verificar permissões:** testar conectividade de rede ou credenciais antes de expor o container principal
- **Separar responsabilidades:** manter a imagem do container principal limpa, sem ferramentas de setup

---

## Como funcionam

```
Pod inicia
  ↓
initContainer[0] roda → deve terminar com exit 0
  ↓
initContainer[1] roda → deve terminar com exit 0
  ↓
(todos os initContainers concluídos)
  ↓
containers principais sobem em paralelo
```

Se qualquer initContainer falhar, o Pod é reiniciado (respeitando `restartPolicy`). Os containers principais **nunca sobem** enquanto os initContainers não concluírem.

---

## YAML de referência

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-com-init
spec:
  initContainers:
    - name: aguardar-db
      image: busybox:1.36
      command:
        - sh
        - -c
        - |
          until nc -z db-svc 5432; do
            echo "aguardando banco de dados..."
            sleep 2
          done
          echo "banco disponível, iniciando app"
    - name: preparar-config
      image: busybox:1.36
      command: ["sh", "-c", "cp /tmp/config.yaml /shared/config.yaml"]
      volumeMounts:
        - name: shared-data
          mountPath: /shared
  containers:
    - name: app
      image: minha-app:1.0
      volumeMounts:
        - name: shared-data
          mountPath: /etc/app               # recebe o arquivo preparado pelo init
  volumes:
    - name: shared-data
      emptyDir: {}                          # volume compartilhado entre init e container principal
```

---

## Inspecionando initContainers

```bash
# Ver status do initContainer
kubectl get pod pod-com-init
# NAME           READY   STATUS     RESTARTS   AGE
# pod-com-init   0/1     Init:0/2   0          5s   ← 0 de 2 inits concluídos

# Logs do initContainer (enquanto roda ou após concluir)
kubectl logs pod-com-init -c aguardar-db

# Detalhes de cada initContainer
kubectl describe pod pod-com-init
# Procure a seção "Init Containers" — mostra estado, image e events de cada um
```

---

## Diferenças entre InitContainer e container principal

| Característica | initContainer | Container principal |
|---|---|---|
| Roda em paralelo | Não — sequencial | Sim — todos sobem juntos |
| Deve terminar | Sim (`exit 0`) | Não (fica rodando) |
| Probes (liveness/readiness) | Não suportadas | Suportadas |
| Pode ter volumes compartilhados | Sim | Sim |

---

## Próximo

➡️ [Probes](probes.md) — como o Kubernetes verifica se seus containers estão saudáveis.
