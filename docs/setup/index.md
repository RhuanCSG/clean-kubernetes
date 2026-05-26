# Setup — Ambiente de Prática

Este repositório usa dois ambientes diferentes dependendo da fase. Ambos rodam localmente usando Docker.

---

## Qual ambiente usar?

| Ferramenta | Fases | Por quê |
|---|---|---|
| **minikube** | 1 a 6 | Cluster single-node, fácil de instalar e resetar. Suficiente para aprender pods, controllers, rede, config, storage e RBAC. |
| **kind** (multi-nó) | 7 e 8 | Permite criar clusters com múltiplos nós localmente. Necessário para testar taints, affinity e comportamentos do scheduler. |

---

## Pré-requisito comum

Ambas as ferramentas precisam de **Docker** instalado e rodando.

```bash
docker --version
# Docker version 24.x ou superior
```

---

## Guias de configuração

1. **[minikube](minikube.md)** — Para as fases 1 a 6
2. **[kind multi-nó](kind.md)** — Para as fases 7 e 8

---

## Verificando kubectl

Ambos os ambientes precisam do `kubectl` instalado:

```bash
kubectl version --client
# Client Version: v1.28 ou superior
```

!!! tip "kubectl com Docker Desktop"
    Se você tem Docker Desktop instalado, o `kubectl` geralmente já vem incluído. Verifique com `kubectl version --client`.
