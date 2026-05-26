# Componentes do Control Plane

O control plane é o "cérebro" do cluster Kubernetes. É composto por um conjunto de processos que trabalham juntos para manter o estado desejado.

---

## Os seis componentes

### kube-apiserver

**O único ponto de entrada.** Toda comunicação com o cluster passa por ele — `kubectl`, controllers, kubelet, kube-proxy. Tudo.

Responsabilidades:
- Autenticar e autorizar requisições (TLS + RBAC)
- Validar objetos antes de persistir
- Expor a REST API do Kubernetes
- Persistir objetos no etcd

```bash
kubectl logs -n kube-system kube-apiserver-minikube
```

### etcd

**A única fonte de verdade.** Banco de dados chave-valor distribuído onde o estado de todos os objetos do cluster é armazenado. Se o etcd parar, o cluster para de funcionar — mas Pods existentes continuam rodando.

```bash
# Verificar saúde do etcd (via kind)
docker exec -it k8s-study-control-plane etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health
# Windows (PowerShell): substitua \ por ` (backtick) para quebra de linha
```

### kube-scheduler

**O alocador de Pods.** Observa Pods recém-criados sem nó atribuído (`nodeName: ""`), decide onde colocá-los (baseado em resources, taints, affinity) e escreve o `nodeName` no Pod.

```bash
kubectl logs -n kube-system kube-scheduler-minikube
```

### kube-controller-manager

**O controlador de controladores.** Roda todos os reconciliation loops em um único processo:

- **Deployment controller** — garante o número de réplicas
- **ReplicaSet controller** — cria e deleta Pods
- **Node controller** — monitora saúde dos nós
- **Job controller** — garante conclusão de Jobs
- E mais de 30 outros controllers

```bash
kubectl logs -n kube-system kube-controller-manager-minikube
```

### kubelet

**O agente em cada nó.** Único componente que roda fora do control plane. Roda em todos os nós (incluindo o control-plane em single-node).

Responsabilidades:
- Receber Pods atribuídos ao seu nó
- Instruir o container runtime (containerd) a criar os containers
- Executar probes (liveness, readiness, startup)
- Reportar status dos Pods ao apiserver

```bash
# Logs do kubelet (no nó — via kind)
docker exec -it k8s-study-worker journalctl -u kubelet -f
```

### kube-proxy

**O mantenedor das regras de rede.** Roda em cada nó e mantém as regras de iptables/IPVS que implementam os Services. Quando você cria um Service ClusterIP, o kube-proxy cria regras de iptables em todos os nós para encaminhar o tráfego.

```bash
kubectl get pods -n kube-system -l k8s-app=kube-proxy
kubectl logs -n kube-system -l k8s-app=kube-proxy
```

---

## Onde ficam os componentes

No minikube e kind, os componentes do control plane rodam como **static Pods** — manifestos gerenciados diretamente pelo kubelet, fora da API normal do Kubernetes:

```bash
# Ver os Pods do control plane
kubectl get pods -n kube-system
# kube-apiserver-minikube            Running
# kube-controller-manager-minikube   Running
# kube-scheduler-minikube            Running
# etcd-minikube                      Running
# kube-proxy-xxxxx                   Running  (um por nó)
# coredns-xxxxx                      Running  (DNS)

# Manifestos estáticos no nó (minikube)
minikube ssh
ls /etc/kubernetes/manifests/
# etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```

---

## Verificando saúde dos componentes

```bash
kubectl get componentstatuses
# Warning: v1 ComponentStatus is deprecated
# NAME                 STATUS    MESSAGE
# controller-manager   Healthy   ok
# scheduler            Healthy   ok
# etcd-0               Healthy   {"health":"true","reason":""}
```

---

## Próximo

➡️ [Fluxo de Criação](pod-flow.md) — o que acontece exatamente entre o `kubectl apply` e o Pod ficar `Running`.
