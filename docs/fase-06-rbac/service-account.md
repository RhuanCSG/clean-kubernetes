# ServiceAccount

ServiceAccount é a identidade de um Pod dentro do cluster Kubernetes. É como o Pod se autentica na kube-apiserver para fazer chamadas à API.

---

## Por que ServiceAccounts existem

Aplicações às vezes precisam interagir com a API do Kubernetes:

- Um operador que lista Pods para monitoramento
- Um Job que cria recursos dinamicamente
- Uma aplicação que lê Secrets ou ConfigMaps via API

Para isso, o Pod precisa de uma identidade que o apiserver reconheça e possa autorizar via RBAC.

---

## O token automático

Todo Pod recebe automaticamente o token JWT da sua ServiceAccount montado em:

```
/var/run/secrets/kubernetes.io/serviceaccount/token
```

```bash
kubectl exec <pod> -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
# eyJhbGciOiJSUzI1NiIsImtpZCI...  ← JWT com a identidade da SA
```

A SA padrão de cada namespace é `default`, e ela geralmente não tem permissões para nada útil.

---

## YAML de referência

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: minha-sa
  namespace: default
automountServiceAccountToken: true    # true é o padrão
                                       # false: não monta o token (mais seguro se não precisar da API)
```

---

## Usando uma ServiceAccount em um Pod

```yaml
spec:
  serviceAccountName: minha-sa          # Pod usa esta SA (padrão é "default")
  automountServiceAccountToken: false   # desativa o token se o Pod não precisa da API
  containers:
    - name: app
      image: nginx:1.25
```

---

## Comandos essenciais

```bash
kubectl get serviceaccounts
kubectl get sa    # abreviação

# Ver tokens e secrets associados
kubectl describe sa minha-sa

# Criar SA rapidamente
kubectl create serviceaccount minha-sa
```

---

## SA padrão vs. SA dedicada

| | SA `default` | SA dedicada |
|---|---|---|
| Criada automaticamente | Sim, em cada namespace | Não — você cria |
| Permissões | Nenhuma por padrão | Somente o que você define no Role |
| Recomendado para | Aplicações sem acesso à API | Aplicações que precisam da API |

!!! tip "Princípio do menor privilégio"
    Crie uma SA dedicada para cada aplicação que precisa da API e dê apenas as permissões necessárias. Não adicione permissões à SA `default`.

---

## Próximo

➡️ [Role e ClusterRole](roles.md) — definindo o que uma identidade pode fazer.
