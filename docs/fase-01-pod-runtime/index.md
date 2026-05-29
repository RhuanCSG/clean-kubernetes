# Fase 01 — Pod & Container Runtime

**Camada:** runtime | **Estimativa:** ~2 semanas | **Ambiente:** kind

Esta é a fase mais importante. Tudo que vem depois — controllers, networking, storage — é construído sobre o Pod. Não avance sem ter clareza total aqui.

---

## O que você vai aprender

- O que é um Pod e por que é a menor unidade do Kubernetes (não o container)
- Como o kubelet gerencia o ciclo de vida dos containers
- O papel dos Namespaces na organização do cluster
- Como InitContainers funcionam e quando usá-los
- A diferença entre liveness e readiness probes

---

## Por que o Pod e não o container?

O Docker gerencia containers individualmente. O Kubernetes gerencia **Pods** — grupos de um ou mais containers que compartilham:

- O mesmo endereço de rede (mesmo IP)
- O mesmo namespace de processo (podem se comunicar via localhost)
- Os mesmos volumes

Isso permite padrões como o **sidecar**: um container principal e um container auxiliar (proxy, coletor de logs) que trabalham juntos no mesmo Pod.

---

## Tópicos desta fase

1. **[Pod e Containers](pod.md)** — estrutura do Pod, campos essenciais, ciclo de vida
2. **[Namespaces](namespace.md)** — isolamento lógico de recursos no cluster
3. **[InitContainers](init-containers.md)** — pré-condições antes do container principal
4. **[Probes](probes.md)** — como o Kubernetes verifica a saúde dos containers

---

## Lab da fase

Criar um Pod com initContainer, inspecionar com os comandos essenciais e resolver cenários de debugging.

Ver: `fases/01-pod-runtime/labs/lab.md` no repositório.

---

## Pronto quando

- [ ] Aplicar o YAML de referência e inspecionar o Pod com `describe`
- [ ] Ver os logs do initContainer e do container principal separadamente
- [ ] Resolver o cenário `01-crashloop` sem consultar material externo
- [ ] Resolver o cenário `02-imagepull` sem consultar material externo
- [ ] Explicar a diferença entre `livenessProbe` e `readinessProbe` com suas próprias palavras

---

## Próxima fase

➡️ [Fase 02 — Workload Controllers](../fase-02-workload-controllers/index.md)
