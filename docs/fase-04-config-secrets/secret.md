# Secret

Secret armazena dados sensíveis — senhas, tokens, certificados. Funciona de forma similar ao ConfigMap, mas com diferenças importantes de segurança.

---

## O que é base64 (e o que não é)

Os valores em um Secret são codificados em **base64**, não criptografados. Isso significa:

=== "Linux/macOS"

    ```bash
    echo -n "minha-senha" | base64
    # bWluaGEtc2VuaGE=

    echo -n "bWluaGEtc2VuaGE=" | base64 -d
    # minha-senha
    ```

=== "Windows (PowerShell)"

    ```powershell
    [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("minha-senha"))
    # bWluaGEtc2VuaGE=

    [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("bWluaGEtc2VuaGE="))
    # minha-senha
    ```

Qualquer um com acesso ao Secret consegue decodificar o valor. A proteção real vem do **RBAC** — controlar quem pode ler Secrets no cluster.

!!! warning "Secrets em etcd são armazenados em plaintext por padrão"
    Em produção, habilite **Encryption at Rest** para o etcd. Sem isso, Secrets são lidos em texto claro de dentro do etcd.

---

## YAML de referência

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque                    # genérico; outros tipos abaixo
data:
  password: bWluaGEtc2VuaGE=   # base64 de "minha-senha"
  username: YWRtaW4=            # base64 de "admin"
```

Para gerar os valores:

=== "Linux/macOS"

    ```bash
    echo -n "minha-senha" | base64    # -n evita incluir newline
    ```

=== "Windows (PowerShell)"

    ```powershell
    [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("minha-senha"))
    ```

---

## Tipos de Secret

| Tipo | Uso |
|---|---|
| `Opaque` | Genérico — use para senhas, tokens, dados arbitrários |
| `kubernetes.io/tls` | Certificado TLS (`tls.crt` e `tls.key`) |
| `kubernetes.io/dockerconfigjson` | Credenciais de registry de container |
| `kubernetes.io/service-account-token` | Token de ServiceAccount (gerenciado automaticamente) |

---

## Criando Secrets

=== "Linux / macOS"
    ```bash
    # A partir de valores literais (o kubectl faz o base64 automaticamente)
    kubectl create secret generic db-secret \
      --from-literal=password=minha-senha \
      --from-literal=username=admin
    ```

=== "Windows (PowerShell)"
    ```powershell
    # A partir de valores literais (o kubectl faz o base64 automaticamente)
    kubectl create secret generic db-secret `
      --from-literal=password=minha-senha `
      --from-literal=username=admin
    ```

=== "Linux / macOS"
    ```bash
    # A partir de um arquivo
    kubectl create secret generic tls-cert \
      --from-file=tls.crt=./cert.pem \
      --from-file=tls.key=./key.pem
    ```

=== "Windows (PowerShell)"
    ```powershell
    # A partir de um arquivo
    kubectl create secret generic tls-cert `
      --from-file=tls.crt=./cert.pem `
      --from-file=tls.key=./key.pem
    ```

```bash
# Aplicar YAML (valores já em base64)
kubectl apply -f secret.yaml
```

---

## Inspecionando Secrets

```bash
kubectl get secrets
kubectl describe secret db-secret    # mostra as chaves mas NÃO os valores
```

=== "Linux / macOS"
    ```bash
    # Ver o valor decodificado de uma chave específica
    kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 -d
    ```

=== "Windows (PowerShell)"
    ```powershell
    # Ver o valor decodificado de uma chave específica
    kubectl get secret db-secret -o jsonpath='{.data.password}' | ForEach-Object { [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($_)) }
    ```

---

## Próximo

➡️ [Injeção em Pods](injection.md) — como usar ConfigMap e Secret dentro de um container.
