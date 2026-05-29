# Setup — Ambiente de Prática

Este repositório usa **kind** (Kubernetes in Docker) como ambiente de prática em todas as 8 fases.

---

## Por que kind?

kind cria clusters Kubernetes reais localmente, onde cada nó é um container Docker. É o mesmo ambiente usado pelo projeto Kubernetes para seus próprios testes de integração.

- **Uma única ferramenta** para todas as fases
- **Topologia real**: control-plane separado dos workers
- **Comportamento consistente** em Windows, Linux e macOS

---

## Pré-requisito

Docker Desktop instalado e rodando:

```bash
docker --version
# Docker version 24.x ou superior
```

---

## Verificando kubectl

```bash
kubectl version --client
# Client Version: v1.28 ou superior
```

!!! tip "kubectl com Docker Desktop"
    Se você tem Docker Desktop instalado, o `kubectl` geralmente já vem incluído.

---

## Configurando o ambiente

➡️ [Setup do kind](kind.md) — instalar kind, criar o cluster e configurar os addons por fase
