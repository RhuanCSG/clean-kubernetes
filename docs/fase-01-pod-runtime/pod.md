# Pod e Containers

O Pod é a menor unidade deployável do Kubernetes. Entender sua estrutura e ciclo de vida é o fundamento de tudo.

---

## O que é um Pod

Um Pod encapsula um ou mais containers que sempre rodam juntos no mesmo nó e compartilham:

- **Rede:** mesmo IP e namespace de rede — containers se comunicam via `localhost`
- **Storage:** volumes definidos no Pod são acessíveis a todos os seus containers
- **Ciclo de vida:** se o Pod é removido, todos os containers são removidos juntos

Na prática, a maioria dos Pods tem **um único container**. Múltiplos containers no mesmo Pod (padrão sidecar) são a exceção, não a regra.

---

## Como o Pod chega ao estado Running

```
kubectl apply -f pod.yaml
    ↓
kube-apiserver valida e persiste no etcd
    ↓
kube-scheduler detecta Pod sem nó atribuído
kube-scheduler escolhe um nó e atualiza o Pod no etcd
    ↓
kubelet no nó detecta o Pod atribuído a ele
kubelet instrui o container runtime (containerd) a criar os containers
    ↓
container runtime baixa a imagem e inicia o container
    ↓
kubelet monitora e reporta status ao apiserver
```

---

## YAML de referência

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-referencia
  namespace: default          # partição lógica; omitir usa "default"
  labels:
    app: exemplo              # seletor usado por Services e Controllers
spec:
  containers:
    - name: app
      image: nginx:1.25
      ports:
        - containerPort: 80   # apenas documentação; não abre porta no host
      env:
        - name: ENV_VAR
          value: "valor"
      resources:
        requests:             # mínimo reservado pelo scheduler para este container
          cpu: "100m"         # 100 millicores = 0.1 CPU
          memory: "128Mi"
        limits:               # teto absoluto; ultrapassar memory = OOMKilled
          cpu: "500m"
          memory: "256Mi"
```

---

## Campos essenciais

### `metadata.labels`

Labels são pares chave-valor usados para selecionar recursos. **Services e Controllers dependem de labels para encontrar Pods.** Um Pod sem labels não é alcançado por nenhum Service.

### `spec.containers[].resources`

| Campo | Efeito |
|---|---|
| `requests.cpu` / `requests.memory` | Mínimo garantido. O scheduler só agenda o Pod em nós com essa capacidade disponível. |
| `limits.cpu` | Teto de CPU. Ultrapassar causa throttling (processo fica lento, mas não morre). |
| `limits.memory` | Teto de memória. Ultrapassar causa **OOMKilled** — o container é reiniciado. |

!!! warning "Sempre defina requests e limits"
    Pods sem `requests` podem ser agendados em nós sobrecarregados. Pods sem `limits` podem consumir toda a memória do nó e afetar outros workloads.

### `spec.containers[].env`

Injeta variáveis de ambiente direto no YAML. Para configuração gerenciada, use [ConfigMap](../fase-04-config-secrets/configmap.md) e [Secret](../fase-04-config-secrets/secret.md).

---

## Comandos de inspeção essenciais

```bash
kubectl get pods                          # lista Pods no namespace atual
kubectl get pods -o wide                  # mostra nó e IP de cada Pod
kubectl describe pod <nome>               # eventos, condições, estado detalhado
kubectl logs <nome>                       # stdout do container
kubectl logs <nome> -c <container>        # container específico (multi-container)
kubectl logs <nome> --previous            # logs do container que acabou de morrer
kubectl exec -it <nome> -- sh             # shell interativo dentro do container
kubectl get events --sort-by=.lastTimestamp  # todos os eventos do cluster, em ordem
```

!!! tip "`kubectl describe` é seu melhor amigo"
    A seção **Events** no final do `kubectl describe pod` mostra exatamente o que aconteceu: qual imagem foi puxada, quando o container iniciou, por que reiniciou. Sempre leia os eventos antes de qualquer outra coisa.

---

## Estados comuns do Pod

| Status | Significa |
|---|---|
| `Pending` | Pod aceito pelo cluster, mas ainda não agendado ou aguardando imagem |
| `Running` | Pelo menos um container está rodando |
| `Succeeded` | Todos os containers terminaram com sucesso (Jobs) |
| `Failed` | Pelo menos um container terminou com falha |
| `Unknown` | Estado do Pod não pode ser determinado (problema de comunicação com o nó) |
| `CrashLoopBackOff` | Container reiniciando repetidamente; k8s aplica backoff exponencial |
| `ImagePullBackOff` | Falha ao baixar a imagem; k8s aumenta o intervalo entre tentativas |

---

## Próximo

➡️ [Namespaces](namespace.md) — como o Kubernetes isola recursos dentro do cluster.
