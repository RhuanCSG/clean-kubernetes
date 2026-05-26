# DNS Interno do Cluster

O Kubernetes inclui um servidor DNS interno (CoreDNS) que permite que Pods se descubram pelo nome, sem precisar saber IPs. É o mecanismo que torna o service discovery funcionar.

---

## Como o DNS interno funciona

Quando você cria um Service, o CoreDNS automaticamente cria um registro DNS para ele:

```
<service-name>.<namespace>.svc.cluster.local → ClusterIP do Service
```

Qualquer Pod do cluster pode resolver esse nome e alcançar o Service.

---

## Formas de resolver um Service

De dentro de qualquer Pod:

```bash
# Nome curto (mesmo namespace)
curl http://backend-svc

# Nome com namespace
curl http://backend-svc.producao

# FQDN completo
curl http://backend-svc.producao.svc.cluster.local
```

O Kubernetes configura o arquivo `/etc/resolv.conf` de cada Pod com o domínio de busca correto, por isso o nome curto funciona. Em namespaces diferentes, use pelo menos `<service>.<namespace>`.

---

## Resolução de nomes na prática

```bash
# Criar um Pod cliente para testar
kubectl run cliente --image=busybox:1.36 --restart=Never -- sleep 3600

# Dentro do Pod
kubectl exec -it cliente -- sh

# Resolver o Service
nslookup backend-svc
# Server:    10.96.0.10       ← IP do CoreDNS
# Address:   10.96.0.10:53
#
# Name:  backend-svc.default.svc.cluster.local
# Address: 10.96.x.x          ← ClusterIP do Service

# Acessar o Service
wget -qO- http://backend-svc
```

---

## DNS para StatefulSets (headless)

Para Services headless (`clusterIP: None`), o DNS resolve direto para o IP de cada Pod:

```
db-0.db-headless.default.svc.cluster.local → IP do Pod db-0
db-1.db-headless.default.svc.cluster.local → IP do Pod db-1
```

Isso permite que um Pod saiba o endereço exato de cada réplica do StatefulSet.

---

## CoreDNS

O CoreDNS roda no namespace `kube-system` como um Deployment:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
# NAME                      READY   STATUS    RESTARTS
# coredns-xxxxxxxxxx-xxxx   1/1     Running   0
```

### Sintomas de problema no CoreDNS

- `nslookup` dentro de um Pod retorna `NXDOMAIN` ou timeout
- Services não são alcançados pelo nome, mas funcionam pelo IP
- `kubectl get pods -n kube-system` mostra CoreDNS em CrashLoopBackOff

```bash
# Verificar logs do CoreDNS
kubectl logs -n kube-system -l k8s-app=kube-dns
```

---

## Resolução de nomes externos

Pods também podem resolver nomes externos (internet) via CoreDNS, que encaminha para o DNS do nó:

```bash
# De dentro de um Pod
nslookup google.com    # deve resolver normalmente
```

---

## Próximo

➡️ [Debugging](debugging.md) — Service com Endpoints vazio e problemas de DNS.
