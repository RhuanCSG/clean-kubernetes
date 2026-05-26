# Fase 07 — Scheduling & Recursos

**Camada:** scheduler | **Estimativa:** ~3 semanas | **Ambiente:** kind multi-nó

O scheduler é o componente que decide onde cada Pod vai rodar. Esta fase explica as regras que o scheduler segue e como influenciar essas decisões.

!!! info "Ambiente necessário"
    Taints, tolerations e affinity só fazem sentido com múltiplos nós. Configure o cluster kind antes de começar: [Setup kind multi-nó](../setup/kind.md).

---

## O que você vai aprender

- Como o scheduler filtra e ranqueia nós para cada Pod
- A diferença entre requests (reserva) e limits (teto) e seus efeitos práticos
- Como taints e tolerations controlam quais Pods podem ir a quais nós
- Como affinity expressa preferências de co-localização
- Como o HPA escala automaticamente baseado em métricas

---

## O algoritmo do scheduler (simplificado)

```
Para cada Pod em Pending:
  1. Filtering (eliminação):
     - Remove nós sem recursos suficientes (requests)
     - Remove nós com taints sem toleration no Pod
     - Remove nós que não satisfazem nodeSelector ou affinity obrigatória
  
  2. Scoring (ranqueamento):
     - Pontua nós restantes (recursos disponíveis, affinity preferencial, etc.)
  
  3. Binding:
     - Atribui o Pod ao nó com maior pontuação
```

---

## Tópicos desta fase

1. **[Requests e Limits](resources.md)** — reserva de recursos e teto de consumo
2. **[LimitRange e Quota](limitrange-quota.md)** — defaults e tetos por namespace
3. **[Taints e Tolerations](taints.md)** — repelir e tolerar nós específicos
4. **[Affinity](affinity.md)** — preferências de co-localização
5. **[HPA](hpa.md)** — escalonamento automático horizontal

---

## Lab da fase

Aplicar LimitRange, adicionar taint em um nó, confirmar Pod em Pending e resolver com toleration.

Ver: `phases/07-scheduling/labs/lab.md` no repositório.

---

## Pronto quando

- [ ] Aplicar LimitRange e verificar defaults injetados em Pods sem resources
- [ ] Adicionar taint e confirmar Pod em Pending; resolver com toleration
- [ ] Resolver o cenário `01-oomkilled` sem ajuda
- [ ] Resolver o cenário `02-pending-taint` sem ajuda
- [ ] Diagnosticar um Pod em Pending e determinar se o problema é de recursos, taint ou affinity

---

## Próxima fase

➡️ [Fase 08 — Internals do Control Plane](../fase-08-control-plane/index.md)
