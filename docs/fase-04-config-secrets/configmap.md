# ConfigMap

ConfigMap armazena dados de configuração não-sensíveis como pares chave-valor. Pode armazenar strings simples, arquivos de configuração inteiros ou qualquer dado que não seja senha ou token.

---

## YAML de referência

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # Chaves simples → viram variáveis de ambiente
  APP_ENV: "producao"
  LOG_LEVEL: "info"
  MAX_CONNECTIONS: "100"

  # Chave com valor multi-linha → vira arquivo quando montado como volume
  config.yaml: |
    timeout: 30
    retries: 3
    database:
      host: db-svc
      port: 5432
```

---

## Criando ConfigMaps

```bash
# A partir de um arquivo YAML
kubectl apply -f configmap.yaml

# A partir de um arquivo existente
kubectl create configmap nginx-config --from-file=nginx.conf

# A partir de valores literais
kubectl create configmap app-config \
  --from-literal=APP_ENV=producao \
  --from-literal=LOG_LEVEL=info

# A partir de um diretório inteiro
kubectl create configmap configs --from-file=./configs/
```

---

## Inspecionando ConfigMaps

```bash
kubectl get configmaps
kubectl describe configmap app-config    # mostra as chaves e valores
kubectl get configmap app-config -o yaml # YAML completo
```

---

## Atualizando ConfigMaps

```bash
# Editar diretamente
kubectl edit configmap app-config

# Substituir completamente
kubectl create configmap app-config --from-literal=LOG_LEVEL=debug \
  --dry-run=client -o yaml | kubectl apply -f -
```

### Propagação automática

Quando um ConfigMap é atualizado:

| Forma de consumo | Propagação |
|---|---|
| Volume (`volumeMount`) | Automática em ~1 minuto |
| `env` / `envFrom` | **NÃO** — Pod precisa ser recriado |

Isso é uma das razões pelas quais volumes são preferidos para configuração que muda.

---

## Próximo

➡️ [Secret](secret.md) — dados sensíveis com camada extra de proteção.
