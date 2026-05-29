# StorageClass

StorageClass define como PersistentVolumes são provisionados dinamicamente. Em vez de um administrador criar PVs manualmente, o Kubernetes cria o PV automaticamente quando um PVC é criado — desde que uma StorageClass compatível exista.

---

## Provisionamento estático vs. dinâmico

### Estático (sem StorageClass)

1. Administrador cria PersistentVolumes manualmente
2. Usuário cria PVC
3. Kubernetes vincula PVC ao PV compatível manualmente criado

Trabalhoso e difícil de escalar.

### Dinâmico (com StorageClass)

1. Administrador configura StorageClass com um provisioner
2. Usuário cria PVC referenciando a StorageClass
3. Kubernetes chama o provisioner → cria o PV automaticamente → vincula

O que a maioria dos clusters de produção usa.

---

## YAML de referência

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"   # torna padrão
provisioner: kubernetes.io/gce-pd                          # provisioner da cloud
parameters:
  type: pd-ssd                                             # parâmetros do disco
  replication-type: regional-pd
reclaimPolicy: Delete                                      # o que fazer quando o PVC é deletado
volumeBindingMode: WaitForFirstConsumer                    # aguarda o Pod ser agendado antes de criar o PV
allowVolumeExpansion: true                                 # permite aumentar o tamanho depois
```

---

## StorageClass no kind

O kind provisiona PVs usando `local-path` (diretórios no nó via hostPath por baixo):

```bash
kubectl get storageclass
# NAME                 PROVISIONER                    RECLAIM POLICY   VOLUME BINDING MODE
# standard (default)   rancher.io/local-path          Delete           WaitForFirstConsumer
```

Qualquer PVC que referenciar `standard` (ou não especificar StorageClass) será provisionado automaticamente.

---

## StorageClass padrão

Se um PVC não especifica `storageClassName`, a StorageClass marcada como padrão é usada:

Ver qual é o padrão:

=== "Linux / macOS"
    ```bash
    kubectl get storageclass | grep default
    ```

=== "Windows (PowerShell)"
    ```powershell
    kubectl get storageclass | Select-String "default"
    ```

PVCs sem `storageClassName` usam o padrão automaticamente.

```yaml
spec:
  resources:
    requests:
      storage: 1Gi
  # sem storageClassName → usa a padrão
```

---

## `volumeBindingMode`

| Modo | Comportamento |
|---|---|
| `Immediate` | PV criado assim que o PVC é criado |
| `WaitForFirstConsumer` | PV criado somente quando um Pod usa o PVC (considera a zona do Pod) |

`WaitForFirstConsumer` é recomendado em clusters multi-zona para evitar que o PV seja criado em uma zona diferente do Pod.

---

## Próximo

➡️ [Volumes Temporários](temp-volumes.md) — emptyDir e hostPath para dados não persistentes.
