# CLAUDE.md — clean-kubernetes

Repositório de estudo prático de Kubernetes, organizado por camadas de abstração (bottom-up).

## Objetivo do projeto

Preencher o gap de quem já operou clusters Kubernetes (EKS/GKE/AKS) na prática mas sem entender os internals. O foco é operar e debugar com confiança — não re-ensinar o básico.

## Estrutura do repositório

```
CLAUDE.md
README.md
setup/                        # guias de ambiente (kind)
docs/
fases/
  01-pod-runtime/
  02-workload-controllers/
  03-networking/
  04-config-secrets/
  05-storage/
  06-rbac/
  07-scheduling/
  08-control-plane/
```

### Estrutura interna de cada fase

```
fases/NN-nome/
  README.md          # teoria: "o que é cada recurso" + critério de conclusão
  yaml/              # YAMLs anotados com comentários explicando cada campo
  labs/lab.md        # exercício prático passo a passo com comandos e saída esperada
  debugging/
    README.md        # descrição dos cenários e dicas de investigação
    NN-cenario/
      broken.yaml    # manifesto com problema intencional
      solution.yaml  # manifesto corrigido com explicação do fix
```

## Convenções

- **YAMLs anotados:** todo campo não-óbvio deve ter comentário inline explicando o efeito
- **Debugging scenarios:** `broken.yaml` inclui cabeçalho comentado descrevendo o problema e como reproduzir; `solution.yaml` explica o fix
- **Labs:** incluem saída esperada dos comandos para o estudante verificar que está no caminho certo
- **Nomes de recursos:** sufixos `-fixed`, `-ok`, `-corrigido` para soluções de debugging; evita conflito com o recurso quebrado no mesmo cluster

## Ambiente de prática

| Ferramenta | Fases |
|---|---|
| kind | 1 a 8 — cluster com 1 control-plane e 2 workers |

Configuração do kind: `setup/kind-config.yaml` (1 control-plane + 2 workers, Calico CNI, extraPortMappings para Ingress)

## Conteúdo e versionamento

- **Referência oficial:** todo conteúdo do repositório (explicações, YAMLs, comandos, comportamentos) deve ter como base a [documentação oficial do Kubernetes](https://kubernetes.io/docs/). Em caso de conflito entre o que está documentado aqui e o que a documentação oficial diz, a documentação oficial prevalece.
- **Versão do Kubernetes:** sempre usar a versão estável mais recente disponível. Isso vale para imagens de exemplo (ex: `nginx:1.xx`), APIs (ex: `autoscaling/v2` em vez de `v1`), flags e comportamentos documentados. Ao criar ou revisar conteúdo, verificar se há versão mais recente antes de fixar qualquer versão específica.

## Escopo desta fase

Kubernetes puro apenas. Fora de escopo: Helm, ArgoCD, Prometheus, Vault, Service Mesh, multi-cluster.
