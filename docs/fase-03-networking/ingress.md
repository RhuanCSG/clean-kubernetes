# Ingress

Ingress define regras de roteamento HTTP/HTTPS para acessar Services dentro do cluster a partir do exterior. É uma abstração sobre o LoadBalancer — um único ponto de entrada que roteia para múltiplos Services por host ou path.

---

## Ingress vs. Service LoadBalancer

| | Service LoadBalancer | Ingress |
|---|---|---|
| IP externo | Um por Service | Um para todos os Services |
| Roteamento | Somente por porta | Por host e/ou path |
| TLS termination | Não | Sim |
| Custo em cloud | Um LB por Service | Um LB para tudo |

---

## Pré-requisito: IngressController

O objeto `Ingress` sozinho não faz nada. Ele precisa de um **IngressController** — um Pod que lê as regras e implementa o roteamento. O mais comum é o NGINX Ingress Controller.

No minikube:
```bash
minikube addons enable ingress
kubectl get pods -n ingress-nginx   # aguardar o Pod ficar Running
```

---

## YAML de referência

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /    # reescreve o path antes de encaminhar
spec:
  ingressClassName: nginx                            # qual IngressController usar
  rules:
    - host: app.local                                # roteamento por host
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: backend-svc                    # Service que deve existir
                port:
                  number: 80
          - path: /api                               # path diferente → Service diferente
            pathType: Prefix
            backend:
              service:
                name: api-svc
                port:
                  number: 8080
```

---

## Testando localmente

=== "Linux/macOS"

    ```bash
    # Adicionar ao /etc/hosts (requer sudo)
    echo "$(minikube ip)  app.local" | sudo tee -a /etc/hosts

    # Testar
    curl http://app.local
    ```

=== "Windows (PowerShell admin)"

    ```powershell
    # Obter o IP do IngressController
    $minikubeIp = minikube ip

    # Adicionar ao hosts (requer PowerShell como Administrador)
    Add-Content -Path "C:\Windows\System32\drivers\etc\hosts" -Value "$minikubeIp  app.local"

    # Testar
    curl http://app.local
    ```

    !!! warning "Execute o PowerShell como Administrador"
        O arquivo `hosts` do Windows é protegido. Clique com o botão direito no PowerShell e selecione "Executar como administrador".

---

## TLS com Ingress

```yaml
spec:
  tls:
    - hosts:
        - app.exemplo.com
      secretName: app-tls-secret   # Secret do tipo kubernetes.io/tls
  rules:
    - host: app.exemplo.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: backend-svc
                port:
                  number: 80
```

---

## Diagnóstico

```bash
kubectl get ingress
kubectl describe ingress app-ingress     # mostra regras e o address do IngressController
kubectl get pods -n ingress-nginx        # verificar se o controller está Running
kubectl logs -n ingress-nginx <pod>      # logs do NGINX — mostra erros de configuração
```

### Problemas comuns

| Problema | Causa provável |
|---|---|
| `curl` retorna 404 | path errado ou backend Service não existe |
| `curl` retorna 502/503 | Pod do backend não está Running ou Endpoints vazio |
| `curl` retorna `Connection refused` | IngressController não está rodando |
| Host não resolve | /etc/hosts não configurado ou IP do minikube errado |

---

## Próximo

➡️ [NetworkPolicy](network-policy.md) — controle de tráfego entre Pods.
