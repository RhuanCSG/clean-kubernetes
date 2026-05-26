# Debugging — Fase 03

## Metodologia para problemas de rede

```
1. kubectl get endpoints <service>    → o Service está selecionando Pods?
2. kubectl get pods --show-labels     → os Pods têm o label que o Service procura?
3. kubectl exec cliente -- curl <url> → o tráfego passa de dentro do cluster?
4. kubectl get pods -n kube-system    → o CoreDNS está saudável?
```

A maioria dos problemas de rede no Kubernetes se resume a: **seletor de label errado no Service**.

---

## Cenário 01 — Service com Endpoints Vazio

### O que você vê

```bash
kubectl get endpoints svc-quebrado
# NAME           ENDPOINTS   AGE
# svc-quebrado   <none>      1m
```

O Service existe, mas não está encaminhando tráfego para nenhum Pod.

### Como investigar

```bash
# 1. Ver o selector do Service
kubectl describe service svc-quebrado | grep Selector
# Windows (PowerShell): kubectl describe service svc-quebrado | Select-String "Selector"
# Selector:  app=backend-v2

# 2. Ver os labels dos Pods existentes
kubectl get pods --show-labels
# NAME              LABELS
# backend-xxx-yyy   app=backend   ← label é "backend", não "backend-v2"
```

O seletor `app=backend-v2` não encontra nenhum Pod com esse label. Os Endpoints ficam vazios.

### A correção

Alinhar o `spec.selector` do Service com os labels dos Pods:

```yaml
spec:
  selector:
    app: backend    # corrigido para bater com os labels dos Pods
```

### Cenário de prática

```bash
kubectl apply -f phases/03-networking/debugging/01-empty-endpoints/broken.yaml
kubectl get endpoints svc-quebrado
kubectl describe service svc-quebrado
kubectl get pods --show-labels
```

---

## Cenário 02 — DNS não resolve

### O que você vê

```bash
kubectl exec cliente -- nslookup backend-svc
# Server:   10.96.0.10
# Address:  10.96.0.10:53
# ** server can't find backend-svc: NXDOMAIN
```

### Como investigar

```bash
# 1. Verificar se o Service existe
kubectl get service backend-svc

# 2. Verificar se está no namespace correto
kubectl get service backend-svc -n <namespace>

# 3. Testar com FQDN completo
kubectl exec cliente -- nslookup backend-svc.default.svc.cluster.local

# 4. Verificar CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

### Causas comuns

| Causa | Diagnóstico |
|---|---|
| Service em namespace diferente | Usar `<svc>.<namespace>` no curl |
| Service não existe | `kubectl get service` retorna `not found` |
| CoreDNS com problema | Pods do CoreDNS em CrashLoopBackOff |
| Pod sem acesso ao DNS | `/etc/resolv.conf` do Pod incorreto |

---

## Cenário 03 — Ingress retorna 404 ou 502

### 404 Not Found

```bash
kubectl describe ingress app-ingress
# Verifique: o Service referenciado no Ingress existe?
kubectl get service backend-svc
# Verifique: o path está correto?
```

### 502 Bad Gateway

O IngressController alcança o Service, mas o Service não consegue chegar ao Pod:

```bash
kubectl get endpoints backend-svc    # Endpoints vazio?
kubectl get pods -l app=backend      # Pods estão Running?
```

### IngressController não instalado

```bash
kubectl get pods -n ingress-nginx
# No resources found in ingress-nginx namespace.
# → Instalar: minikube addons enable ingress
```

---

## Referência rápida

```bash
# Diagnóstico completo de networking
kubectl get service <nome>
kubectl get endpoints <nome>
kubectl describe service <nome>
kubectl get pods --show-labels
kubectl exec <cliente> -- curl http://<service>
kubectl exec <cliente> -- nslookup <service>
```

---

## Próxima fase

➡️ [Fase 04 — Configuração & Segredos](../fase-04-config-secrets/index.md)
