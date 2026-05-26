# Fase 03 — Networking

**Camada:** rede | **Estimativa:** ~3 semanas | **Ambiente:** minikube

A rede do Kubernetes abstrai a complexidade de conectar Pods que podem morrer e renascer em qualquer nó. Esta fase explica como o tráfego flui — e por que às vezes não flui.

---

## O que você vai aprender

- Como o Service fornece um IP estável para um conjunto de Pods efêmeros
- Por que `kubectl get endpoints` é o primeiro comando de debug de rede
- Como o DNS interno do cluster resolve nomes de Service
- Como o Ingress roteia tráfego HTTP externo
- Como NetworkPolicy implementa isolamento de rede

---

## O modelo mental de rede

No Kubernetes, todo Pod tem um IP único no cluster — mas esse IP muda quando o Pod morre e renasce. O **Service** resolve isso: é um IP virtual estável que sempre aponta para os Pods selecionados.

```
Cliente (Pod)
    ↓
Service (ClusterIP: 10.96.x.x)    ← IP que não muda
    ↓ kube-proxy (iptables/IPVS)
Pod A (10.244.x.1)  ou  Pod B (10.244.x.2)  ou  Pod C (10.244.x.3)
```

---

## Tópicos desta fase

1. **[Services](services.md)** — ClusterIP, NodePort, LoadBalancer e Endpoints
2. **[Ingress](ingress.md)** — roteamento HTTP/HTTPS por host e path
3. **[NetworkPolicy](network-policy.md)** — firewall declarativo entre Pods
4. **[DNS Interno](dns.md)** — como o CoreDNS resolve nomes dentro do cluster

---

## Lab da fase

Criar Service ClusterIP, fazer `kubectl exec` em um Pod cliente e acessar o backend via nome DNS do Service.

Ver: `phases/03-networking/labs/lab.md` no repositório.

---

## Pronto quando

- [ ] Acessar um Pod de dentro de outro Pod usando o nome DNS do Service
- [ ] Resolver o cenário `01-empty-endpoints` sem ajuda
- [ ] Explicar por que Endpoints vazio significa problema de seletor de label

---

## Próxima fase

➡️ [Fase 04 — Configuração & Segredos](../fase-04-config-secrets/index.md)
