# Injeção em Pods

Há três formas de consumir ConfigMap e Secret dentro de um container. Cada uma tem casos de uso distintos.

---

## As três formas

| Método | Resultado | Propagação automática |
|---|---|---|
| `env.valueFrom` | Variável de ambiente de uma chave específica | Não |
| `envFrom` | Todas as chaves como variáveis de ambiente | Não |
| `volume + volumeMount` | Chaves como arquivos dentro do container | Sim (~1min) |

---

## YAML de referência completo

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-configurado
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "env && cat /etc/config/config.yaml && sleep 3600"]

      # Forma 1: todas as chaves do ConfigMap como variáveis de ambiente
      envFrom:
        - configMapRef:
            name: app-config        # chaves APP_ENV e LOG_LEVEL viram env vars

      # Forma 2: uma chave específica do Secret como variável de ambiente
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password         # apenas a chave "password"
        - name: APP_VERSION
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_VERSION      # uma chave específica do ConfigMap

      # Forma 3: ConfigMap montado como volume (propagação automática)
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config    # config.yaml aparece como /etc/config/config.yaml

  volumes:
    - name: config-volume
      configMap:
        name: app-config
        items:
          - key: config.yaml        # monta apenas esta chave (não todas)
            path: config.yaml       # nome do arquivo dentro do mountPath
```

---

## Verificando a injeção

```bash
# Aplicar e aguardar o Pod
kubectl apply -f app-config.yaml
kubectl apply -f secret.yaml
kubectl apply -f pod-with-config.yaml
kubectl get pod app-configurado -w

# Verificar variáveis de ambiente
kubectl exec app-configurado -- sh -c "env | grep -E 'APP_ENV|LOG_LEVEL|DB_PASSWORD'"
# APP_ENV=producao
# LOG_LEVEL=info
# DB_PASSWORD=minha-senha

# Verificar arquivo montado como volume
kubectl exec app-configurado -- cat /etc/config/config.yaml
# timeout: 30
# retries: 3
# ...
```

---

## Propagação automática de volumes

```bash
# Editar o ConfigMap
kubectl edit configmap app-config
# Mude LOG_LEVEL de "info" para "debug" e salve

# Aguardar ~60 segundos e verificar o arquivo
kubectl exec app-configurado -- cat /etc/config/config.yaml
# A chave config.yaml deve ter o novo conteúdo

# Mas as variáveis de ambiente NÃO são atualizadas
kubectl exec app-configurado -- sh -c "env | grep LOG_LEVEL"
# LOG_LEVEL=info  ← ainda o valor antigo
```

Para que variáveis de ambiente reflitam mudanças no ConfigMap, o Pod deve ser recriado.

---

## Quando usar cada método

| Use | Quando |
|---|---|
| `envFrom` | A aplicação espera variáveis de ambiente; você quer injetar todas as chaves |
| `env.valueFrom` | Você quer injetar apenas chaves específicas, possivelmente renomeadas |
| Volume | A aplicação lê arquivos de configuração; você precisa de propagação automática |

---

## Próximo

➡️ [Debugging](debugging.md) — Pod com referência a ConfigMap que não existe.
