# Debugging — Fase 08

Nesta fase, os cenários de debugging envolvem o próprio cluster, não apenas workloads. Os problemas são mais profundos e requerem investigação nos componentes do control plane.

---

## Cenário 01 — Nó NotReady

### O que você vê

```bash
kubectl get nodes
# NAME                    STATUS     ROLES           AGE
# k8s-study-control-plane Ready      control-plane   1d
# k8s-study-worker        NotReady   <none>          1d    ← problema
```

### Simulação (kind)

```bash
# Pausar o container do worker (simula falha do kubelet)
docker pause k8s-study-worker

# Aguardar ~40 segundos — o apiserver detecta que o kubelet parou de enviar heartbeats
kubectl get nodes -w
# k8s-study-worker   NotReady   <none>   40s
```

### Como investigar

```bash
kubectl describe node k8s-study-worker
# Conditions:
#   Type              Status
#   MemoryPressure    Unknown    ← kubelet parou de reportar
#   DiskPressure      Unknown
#   Ready             False

# Eventos do nó
kubectl get events --field-selector involvedObject.name=k8s-study-worker

# Logs do kubelet no nó (via docker exec no kind)
docker exec -it k8s-study-worker journalctl -u kubelet -n 50
```

### O que acontece com os Pods

```bash
kubectl get pods -o wide
# Pods no nó NotReady ficam em "Unknown" após ~1 minuto
# Após ~5 minutos, o Node controller recria os Pods em outros nós (se houver)
```

### Resolução (simulação)

```bash
docker unpause k8s-study-worker
kubectl get nodes -w
# k8s-study-worker   Ready   <none>   (volta após 40s)
```

---

## Cenário 02 — Pod travado em ContainerCreating

### O que você vê

```bash
kubectl get pod meu-pod
# NAME      READY   STATUS              RESTARTS   AGE
# meu-pod   0/1     ContainerCreating   0          5m
```

### Como investigar

```bash
kubectl describe pod meu-pod
# Events:
#   Warning  FailedMount  kubelet  Unable to attach or mount volumes:
#            persistentvolumeclaim "dados-pvc" not found
```

Causas comuns e diagnóstico:

| Causa | Mensagem nos Events |
|---|---|
| PVC não existe | `persistentvolumeclaim "nome" not found` |
| Falha ao baixar imagem | `Failed to pull image` |
| Container runtime parado | `failed to create containerd task` |
| Problemas de rede no nó | `network plugin not ready` |

---

## Cenário 03 — apiserver sem resposta

### O que você vê

```bash
kubectl get nodes
# The connection to the server localhost:8443 was refused
```

### Como investigar (minikube)

```bash
minikube status
# minikube: Running
# cluster: Stopped   ← cluster parado

minikube logs | tail -20   # ver o que aconteceu
# Windows (PowerShell): minikube logs | Select-Object -Last 20

# Tentar reiniciar
minikube start
```

### Causas comuns

| Causa | Como identificar |
|---|---|
| etcd parou | `kubectl logs -n kube-system etcd-minikube` |
| Certificado expirou | Erro TLS nos logs do apiserver |
| Memória insuficiente no host | apiserver OOMKilled pelo host OS |
| Docker parou | `docker ps` retorna erro |

---

## Referência de logs dos componentes

```bash
# Control plane (como static Pods)
kubectl logs -n kube-system kube-apiserver-minikube --tail=50
kubectl logs -n kube-system kube-scheduler-minikube --tail=50
kubectl logs -n kube-system kube-controller-manager-minikube --tail=50
kubectl logs -n kube-system etcd-minikube --tail=50

# kubelet (no nó — minikube)
minikube ssh
journalctl -u kubelet -n 100

# kubelet (no nó — kind)
docker exec -it k8s-study-worker journalctl -u kubelet -n 100

# kube-proxy (em todos os nós)
kubectl logs -n kube-system -l k8s-app=kube-proxy

# CoreDNS
kubectl logs -n kube-system -l k8s-app=kube-dns
```

---

## Checklist de diagnóstico de cluster

```bash
# 1. Estado dos nós
kubectl get nodes

# 2. Estado dos componentes do control plane
kubectl get pods -n kube-system
kubectl get componentstatuses

# 3. Eventos recentes do cluster
kubectl get events --all-namespaces --sort-by=.lastTimestamp | tail -20
# Windows (PowerShell): kubectl get events --all-namespaces --sort-by=.lastTimestamp | Select-Object -Last 20

# 4. Recursos no cluster
kubectl top nodes

# 5. Logs dos componentes críticos
kubectl logs -n kube-system kube-apiserver-<sufixo> --tail=30
kubectl logs -n kube-system etcd-<sufixo> --tail=30
```

---

## Conclusão do roadmap

Ao chegar aqui, você passou por todas as camadas do Kubernetes:

```
runtime → controllers → rede → config → storage → segurança → scheduling → control plane
```

Você não apenas sabe usar o Kubernetes — você sabe **como ele funciona** e **onde olhar quando algo quebra**.
