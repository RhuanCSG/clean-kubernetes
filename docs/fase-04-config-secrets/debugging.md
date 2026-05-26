# Debugging — Fase 04

## Metodologia para problemas de configuração

```
1. kubectl get pod <nome>              → qual é o status? (CreateContainerConfigError?)
2. kubectl describe pod <nome>         → qual referência está faltando?
3. kubectl get configmaps / secrets    → o objeto referenciado existe?
4. kubectl exec <nome> -- env          → as variáveis chegaram corretamente?
```

---

## Cenário 01 — CreateContainerConfigError

### O que você vê

```bash
kubectl get pod config-fail-pod
# NAME              READY   STATUS                       RESTARTS   AGE
# config-fail-pod   0/1     CreateContainerConfigError   0          10s
```

O container não conseguiu ser criado porque uma referência a ConfigMap ou Secret está inválida.

### Como investigar

```bash
kubectl describe pod config-fail-pod
# Events:
#   Warning  Failed  kubelet  Error: configmap "app-settings" not found
```

O Pod tenta referenciar `app-settings` que não existe no cluster.

### Causas comuns

| Causa | Mensagem nos Events |
|---|---|
| ConfigMap não existe | `configmap "nome" not found` |
| Secret não existe | `secret "nome" not found` |
| Chave não existe no ConfigMap | `key "CHAVE" not found in ConfigMap` |
| Typo no nome | Mesmo erro — o nome não bate exatamente |

### Diagnóstico

```bash
# Verificar ConfigMaps disponíveis
kubectl get configmaps
# NAME         DATA   AGE
# app-config   3      1m
# (não existe "app-settings")

# Verificar Secrets
kubectl get secrets
```

### A correção

Duas opções:

1. Criar o ConfigMap referenciado:

```bash
kubectl create configmap app-settings --from-literal=APP_ENV=dev
```

2. Ou corrigir o nome no YAML do Pod para referenciar um ConfigMap que existe:

```yaml
envFrom:
  - configMapRef:
      name: app-config    # nome correto do ConfigMap existente
```

### Cenário de prática

```bash
kubectl apply -f phases/04-config-secrets/debugging/01-wrong-ref/broken.yaml
kubectl get pod config-fail-pod
kubectl describe pod config-fail-pod
kubectl get configmaps
```

---

## Cenário 02 — Variável de ambiente com valor errado

### O que você vê

A aplicação se comporta incorretamente, mas o Pod está `Running`.

### Como investigar

```bash
# Ver todas as variáveis de ambiente dentro do container
kubectl exec app-configurado -- env

# Ou filtrar por prefixo
kubectl exec app-configurado -- sh -c "env | grep APP_"

# Verificar o conteúdo do ConfigMap
kubectl get configmap app-config -o yaml
```

Se o valor no container é diferente do esperado, verifique se há ConfigMaps com o mesmo nome em namespaces diferentes, ou se o Pod foi recriado após a mudança do ConfigMap (para variáveis de ambiente, é necessário recriar).

---

## Cenário 03 — Volume não monta

```bash
kubectl describe pod <nome>
# Events:
#   Warning  Failed  kubelet  MountVolume.SetUp failed:
#                    configmap "app-config" not found
```

O ConfigMap foi deletado após o Pod ser criado, ou foi referenciado com nome errado no volume.

```bash
kubectl get pvc   # verificar se é PVC (se for storage)
kubectl get configmap app-config   # verificar se o ConfigMap existe
```

---

## Próxima fase

➡️ [Fase 05 — Storage](../fase-05-storage/index.md)
