# Fluxo de Criação de um Pod

Esta é a pergunta favorita de entrevistas de Kubernetes: "O que acontece quando você executa `kubectl apply -f pod.yaml`?" Agora você vai entender a resposta completa.

---

## O fluxo completo

```
1. kubectl apply -f pod.yaml
   └─ cliente monta HTTP POST para /api/v1/namespaces/default/pods

2. kube-apiserver recebe a requisição
   ├─ Autenticação: verifica certificado TLS ou token do kubeconfig
   ├─ Autorização: verifica RBAC — você tem permissão para criar Pods?
   ├─ Admission Controllers: validam e podem modificar o objeto
   │   (ex: LimitRanger injeta requests/limits, PodSecurityAdmission valida)
   └─ Persiste o Pod no etcd com nodeName="" e status.phase=Pending

3. kube-scheduler observa (via watch) novos Pods com nodeName=""
   ├─ Filtering: elimina nós sem recursos, com taints sem toleration, etc.
   ├─ Scoring: ranqueia nós restantes
   └─ Escreve nodeName no Pod via apiserver → etcd

4. kubelet no nó escolhido observa (via watch) Pods com seu nodeName
   ├─ Baixa a imagem do container (via container runtime)
   ├─ Cria o container (containerd → runc)
   ├─ Inicia os containers (initContainers primeiro, depois containers principais)
   ├─ Executa probes (startup → liveness → readiness)
   └─ Reporta status ao apiserver: status.phase=Running

5. kube-proxy em todos os nós observa (via watch) Services e Endpoints
   └─ Atualiza regras de iptables/IPVS quando Endpoints mudam
      (quando o Pod novo passa na readinessProbe e entra nos Endpoints)
```

---

## Acompanhando o fluxo em tempo real

Abra dois terminais:

**Terminal 1 — observar eventos:**
```bash
kubectl get events --sort-by=.lastTimestamp -w
```

**Terminal 2 — criar o Pod:**
```bash
kubectl run teste-fluxo --image=nginx:1.25 --restart=Never
```

No terminal 1 você vai ver, em sequência:

```
REASON       MESSAGE
Scheduled    Successfully assigned default/teste-fluxo to minikube
Pulling      Pulling image "nginx:1.25"
Pulled       Successfully pulled image "nginx:1.25"
Created      Created container teste-fluxo
Started      Started container teste-fluxo
```

Cada evento corresponde a uma etapa do fluxo.

---

## Watch no etcd

Para visualizar o fluxo de escrita no etcd (kind):

```bash
# Abrir watch no etcd
docker exec -it k8s-study-control-plane etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  watch /registry/pods/default/ --prefix
# Windows (PowerShell): substitua \ por ` (backtick) para quebra de linha

# Em outro terminal, criar um Pod
kubectl run observado --image=nginx:1.25 --restart=Never
# O etcdctl mostrará as escritas: criação (nodeName=""), update (nodeName="worker"), update (Running)
```

---

## O papel do watch

Todos os componentes usam o mesmo mecanismo: **watch** na API do Kubernetes. Ninguém faz polling — todos se registram para receber notificações quando objetos mudam.

| Componente | O que observa | Ação |
|---|---|---|
| scheduler | Pods com `nodeName: ""` | Atribui um nó |
| kubelet | Pods com `nodeName: <meu-nó>` | Cria containers |
| kube-proxy | Services e Endpoints | Atualiza iptables |
| Deployment controller | ReplicaSets com réplicas erradas | Cria/deleta Pods |

---

## Por que `kubectl get pod` mostra `ContainerCreating` por alguns segundos?

Porque o kubelet recebeu o Pod, mas o container runtime ainda está:
1. Verificando se a imagem existe localmente
2. Fazendo pull da imagem (se não existe)
3. Criando o container (namespaces de rede, cgroups)

Tudo isso leva alguns segundos mesmo com imagem em cache.

---

## Próximo

➡️ [Debugging](debugging.md) — nó NotReady, apiserver sem resposta, ContainerCreating travado.
