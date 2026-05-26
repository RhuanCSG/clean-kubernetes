# Kubernetes — Camadas do Cluster

Repositório de estudo prático de Kubernetes, organizado **bottom-up**: cada fase explora uma camada de abstração do cluster, de dentro para fora.

## Documentação

A documentação completa está disponível em: **https://rhuancsg.github.io/clean-kubernetes/**

## Para quem é este repositório

Para quem **já operou clusters Kubernetes** (EKS, GKE, AKS) na prática, mas nunca entendeu os internals — sabe *o que fazer*, mas não *por que funcionou* nem *onde olhar quando quebra*.

## As 8 fases

| # | Fase | Camada | Ambiente | Estimativa |
|---|---|---|---|---|
| 01 | Pod & Container Runtime | runtime | minikube | ~2 sem |
| 02 | Workload Controllers | controllers | minikube | ~3 sem |
| 03 | Networking | rede | minikube | ~3 sem |
| 04 | Configuração & Segredos | config | minikube | ~2 sem |
| 05 | Storage | armazenamento | minikube | ~2 sem |
| 06 | Controle de Acesso (RBAC) | segurança | minikube | ~2 sem |
| 07 | Scheduling & Recursos | scheduler | kind multi-nó | ~3 sem |
| 08 | Internals do Control Plane | control plane | kind multi-nó | ~3 sem |

## Como usar

1. Configure o ambiente: veja `setup/minikube.md` (fases 1-6) ou `setup/kind.md` (fases 7-8)
2. Leia o `README.md` de cada fase antes de abrir qualquer YAML
3. Aplique os YAMLs de referência em `yaml/` e explore com `kubectl describe` e `kubectl logs`
4. Execute o lab em `labs/lab.md`
5. Só avance quando conseguir resolver o cenário de debugging sem ajuda

## Estrutura do repositório

```
docs/                         # documentação GitHub Pages (MkDocs)
phases/
  01-pod-runtime/
    README.md                 # teoria e critério de conclusão
    yaml/                     # YAMLs anotados com comentários
    labs/lab.md               # exercício prático com saída esperada
    debugging/                # cenários quebrados + soluções comentadas
  02-workload-controllers/
  03-networking/
  04-config-secrets/
  05-storage/
  06-rbac/
  07-scheduling/
  08-control-plane/
setup/
  minikube.md                 # setup do ambiente minikube
  kind.md                     # setup do ambiente kind
  kind-config.yaml            # config do cluster kind (1 control-plane + 2 workers)
```

## Critério de conclusão do roadmap

- Dado um cluster com problema desconhecido, você sabe por onde começar
- Você lê qualquer YAML de recurso k8s e entende o que cada campo faz
- Você explica o que acontece entre `kubectl apply` e o Pod ficar `Running`
- Você debuga os erros mais comuns sem abrir o StackOverflow como primeira ação
