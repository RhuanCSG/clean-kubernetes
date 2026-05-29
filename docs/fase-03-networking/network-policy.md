# NetworkPolicy

Por padrão, todos os Pods do cluster podem se comunicar livremente — qualquer Pod pode falar com qualquer outro, em qualquer namespace. NetworkPolicy implementa um firewall declarativo para restringir esse tráfego.

---

## Como NetworkPolicy funciona

Uma NetworkPolicy seleciona Pods e define quais conexões de entrada (ingress) e saída (egress) são permitidas. **Todo tráfego não explicitamente permitido é negado** assim que pelo menos uma NetworkPolicy se aplica ao Pod.

!!! tip "CNI no kind"
    O cluster kind deste repositório usa **Calico** como CNI (instalado no setup). NetworkPolicy funciona sem configuração adicional — basta aplicar o YAML e as regras são aplicadas.

---

## YAML de referência

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-isolado
  namespace: producao
spec:
  podSelector:
    matchLabels:
      app: backend              # aplica esta policy aos Pods de backend
  policyTypes:
    - Ingress                   # controla tráfego de entrada
    - Egress                    # controla tráfego de saída
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend     # só Pods com label app=frontend podem acessar
          namespaceSelector:
            matchLabels:
              environment: producao   # e somente no namespace producao
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: database
      ports:
        - protocol: TCP
          port: 5432
    - ports:
        - port: 53              # permite DNS (sempre necessário)
          protocol: UDP
        - port: 53
          protocol: TCP
```

---

## Padrões comuns

### Isolar todos os Pods de um namespace

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: producao
spec:
  podSelector: {}               # aplica a todos os Pods do namespace
  policyTypes:
    - Ingress
    - Egress
  # sem regras = nega tudo
```

Aplique isso e depois adicione NetworkPolicies específicas para permitir apenas o necessário.

### Permitir tráfego de outro namespace

```yaml
ingress:
  - from:
      - namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: monitoring   # namespace "monitoring"
```

### Permitir apenas tráfego interno do namespace

```yaml
ingress:
  - from:
      - podSelector: {}         # qualquer Pod do mesmo namespace
```

---

## Testando NetworkPolicy

```bash
# Criar dois Pods para testar
kubectl run sender --image=busybox:1.36 --labels="app=frontend" -- sleep 3600
kubectl run receiver --image=nginx:1.25 --labels="app=backend"

# Pegar o IP do receiver
kubectl get pod receiver -o jsonpath='{.status.podIP}'

# Testar sem NetworkPolicy (deve funcionar)
kubectl exec sender -- wget -qO- http://<IP-do-receiver>

# Aplicar NetworkPolicy que nega todo ingresso
kubectl apply -f deny-all-policy.yaml

# Testar com NetworkPolicy (deve falhar)
kubectl exec sender -- wget -qO- --timeout=3 http://<IP-do-receiver>
```

---

## Próximo

➡️ [DNS Interno](dns.md) — como o Kubernetes resolve nomes de Service sem configuração manual.
