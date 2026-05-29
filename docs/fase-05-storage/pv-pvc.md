# PersistentVolume e PersistentVolumeClaim

PersistentVolume (PV) representa um disco provisionado. PersistentVolumeClaim (PVC) é o pedido de um usuário por armazenamento. O Kubernetes vincula PVCs a PVs automaticamente.

---

## Ciclo de vida

```
PVC criado (usuário)
  ↓
Kubernetes procura PV compatível (ou StorageClass cria um novo)
  ↓
PVC vincula ao PV → status: Bound
  ↓
Pod referencia o PVC → dados persistem enquanto o PVC existir
  ↓
PVC deletado → PV entra em Released
  ↓
reclaimPolicy determina o que acontece ao PV (Delete, Retain, Recycle)
```

---

## YAML de referência — PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dados-pvc
spec:
  accessModes:
    - ReadWriteOnce             # RWO: montado em leitura+escrita em um nó por vez
    # - ReadOnlyMany            # ROX: múltiplos nós, somente leitura
    # - ReadWriteMany           # RWX: múltiplos nós, leitura+escrita (requer NFS/CSI)
  storageClassName: standard    # StorageClass que fará o provisionamento
  resources:
    requests:
      storage: 2Gi
```

---

## Usando o PVC em um Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-com-storage
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "echo 'dado' > /dados/arquivo.txt && sleep 3600"]
      volumeMounts:
        - name: storage
          mountPath: /dados           # caminho dentro do container
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: dados-pvc          # referencia o PVC criado acima
```

---

## Access Modes

| AccessMode | Abreviação | Descrição |
|---|---|---|
| `ReadWriteOnce` | RWO | Um nó pode montar com leitura e escrita |
| `ReadOnlyMany` | ROX | Múltiplos nós podem montar, somente leitura |
| `ReadWriteMany` | RWX | Múltiplos nós podem montar com leitura e escrita |

!!! note "RWX requer suporte do sistema de arquivos"
    A maioria dos sistemas de armazenamento locais só suporta RWO. Para RWX, você precisa de NFS, GlusterFS ou um CSI driver específico.

---

## Verificando PVCs e PVs

```bash
kubectl get pvc
# NAME        STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS
# dados-pvc   Bound    pvc-xxx-yyy   2Gi        RWO            standard

kubectl get pv
# NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM
# pvc-xxx-yyy   2Gi        RWO            Delete           Bound    default/dados-pvc

kubectl describe pvc dados-pvc    # ver o PV vinculado e eventos
```

---

## reclaimPolicy

Define o que acontece com o PV quando o PVC é deletado:

| Política | Comportamento |
|---|---|
| `Delete` | PV e os dados são deletados automaticamente |
| `Retain` | PV fica em Released, dados preservados, requer intervenção manual |
| `Recycle` | Deprecated — apagava os dados e disponibilizava o PV novamente |

Para ver a política da StorageClass:

=== "Linux / macOS"
    ```bash
    kubectl get storageclass standard -o yaml | grep reclaimPolicy
    ```

=== "Windows (PowerShell)"
    ```powershell
    kubectl get storageclass standard -o yaml | Select-String "reclaimPolicy"
    ```

---

## Próximo

➡️ [StorageClass](storage-class.md) — como o provisionamento dinâmico funciona.
