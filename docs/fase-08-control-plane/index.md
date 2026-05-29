# Fase 08 — Internals do Control Plane

**Camada:** control plane | **Estimativa:** ~3 semanas | **Ambiente:** kind multi-nó

Esta é a fase final. Aqui você vai abrir o capô do Kubernetes e entender o que acontece por dentro quando você executa qualquer comando.

!!! info "Ambiente necessário"
    Use o cluster kind multi-nó para poder simular falhas de nó e inspecionar os componentes de forma mais próxima ao ambiente real.

---

## O que você vai aprender

- O papel de cada componente do control plane (apiserver, etcd, scheduler, controller-manager, kubelet, kube-proxy)
- O fluxo completo de um `kubectl apply` até o Pod ficar `Running`
- Como inspecionar logs de cada componente
- O que acontece quando um nó vai para `NotReady`

---

## Por que entender o control plane

Depois de completar as fases 1-7, você sabe **o que** os recursos do Kubernetes fazem. Esta fase responde **como** eles funcionam — o que acontece entre um `kubectl apply` e o Pod aparecer como `Running`.

Esse conhecimento é o que separa quem "sabe usar" de quem "sabe debugar incidentes em produção".

---

## Tópicos desta fase

1. **[Componentes](components.md)** — o papel de cada processo do control plane
2. **[Fluxo de Criação](pod-flow.md)** — do `kubectl apply` ao Pod `Running`
3. **[Debugging](debugging.md)** — nó NotReady, apiserver sem resposta, ContainerCreating

---

## Lab da fase

Inspecionar componentes, acompanhar a criação de um Pod via eventos, e simular um nó NotReady.

Ver: `fases/08-control-plane/labs/lab.md` no repositório.

---

## Pronto quando — e do roadmap completo

- [ ] Listar todos os componentes do control plane e seus status
- [ ] Ver logs de cada componente sem consultar documentação
- [ ] Simular nó NotReady e diagnosticar a causa
- [ ] Explicar o fluxo completo do `kubectl apply` ao Pod `Running`
- [ ] Conseguir diagnosticar os cenários das fases 1-7 sem ajuda externa

**Parabéns — você concluiu o roadmap de Kubernetes puro.**
