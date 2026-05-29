# Volumes Temporários

Nem todo volume precisa persistir. O Kubernetes oferece volumes temporários para casos onde os dados existem apenas durante a vida do Pod ou precisam ser compartilhados entre containers.

---

## emptyDir

Criado vazio quando o Pod é iniciado. Existe enquanto o Pod existir — se o Pod for deletado, os dados somem. Se o container reinicia (sem deletar o Pod), os dados persistem.

### Casos de uso

- Compartilhar dados entre containers do mesmo Pod (sidecar pattern)
- Cache temporário que não precisa sobreviver ao Pod
- Espaço de trabalho para processamento de dados

### YAML de referência

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-compartilhado
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "echo 'gerado pelo app' > /dados/output.txt && sleep 3600"]
      volumeMounts:
        - name: temp
          mountPath: /dados

    - name: sidecar
      image: busybox:1.36
      command: ["sh", "-c", "while true; do cat /resultado/output.txt; sleep 5; done"]
      volumeMounts:
        - name: temp
          mountPath: /resultado       # mesmo volume, caminho diferente

  volumes:
    - name: temp
      emptyDir: {}                    # {} = padrão; pode especificar medium: Memory para tmpfs
```

### emptyDir em memória (tmpfs)

```yaml
volumes:
  - name: cache
    emptyDir:
      medium: Memory                  # armazenado em RAM — muito rápido mas consome memória
      sizeLimit: 256Mi
```

---

## hostPath

Monta um diretório ou arquivo do **nó** onde o Pod está rodando. Funciona bem em clusters locais (kind), mas é problemático em multi-node.

### Casos de uso (legítimos)

- Acessar logs do sistema do nó (`/var/log`)
- Acesso ao socket do container runtime (`/var/run/docker.sock`)
- DaemonSets que precisam de acesso ao filesystem do nó

### YAML de referência

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-hostpath
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "ls /host-logs && sleep 3600"]
      volumeMounts:
        - name: logs
          mountPath: /host-logs
          readOnly: true
  volumes:
    - name: logs
      hostPath:
        path: /var/log              # diretório no nó
        type: Directory             # Directory, File, DirectoryOrCreate, FileOrCreate
```

!!! warning "hostPath em multi-node é perigoso"
    Em clusters com múltiplos nós, o Pod pode ser agendado em qualquer nó. Se o diretório `/var/log/minha-app` só existe em um nó específico, o Pod vai falhar nos outros. Use `nodeSelector` ou evite hostPath em produção.

---

## Comparação dos volumes temporários

| | emptyDir | hostPath |
|---|---|---|
| Dados sobrevivem ao Pod | Não | Sim (no nó) |
| Compartilhado entre containers | Sim | Sim |
| Multi-node seguro | Sim | Não |
| Caso de uso | Compartilhamento intra-Pod | Acesso ao sistema do nó |

---

## Próximo

➡️ [Debugging](debugging.md) — PVC em Pending e problemas com volumes.
