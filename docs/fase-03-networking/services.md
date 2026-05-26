# Services

Um Service é um IP virtual estável que distribui tráfego para um conjunto de Pods selecionados por label. É a peça central do networking interno do Kubernetes.

---

## Por que Services existem

Pods são efêmeros: quando um Pod morre, o novo recebe um IP diferente. Qualquer cliente que guardasse o IP direto do Pod perderia a conexão. O Service resolve isso com um IP que permanece estável independente de quantos Pods sobem e descem.

---

## O papel dos Endpoints

O Kubernetes cria automaticamente um objeto `Endpoints` para cada Service. Ele contém a lista de IPs e portas dos Pods que o Service deve encaminhar tráfego.

```bash
kubectl get endpoints meu-service
# NAME          ENDPOINTS                     AGE
# meu-service   10.244.0.5:8080,10.244.0.6:8080   1m
```

**Se Endpoints está vazio (`<none>`), o Service não encaminha tráfego para nenhum Pod.** Isso quase sempre significa que o seletor de label do Service não bate com nenhum Pod.

---

## Tipos de Service

### ClusterIP (padrão)

IP acessível apenas dentro do cluster. Ideal para comunicação entre serviços internos.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  selector:
    app: backend                # seleciona Pods com este label
  ports:
    - port: 80                  # porta do Service (usada pelos clientes)
      targetPort: 8080          # porta do container (onde a app escuta)
  type: ClusterIP               # padrão; pode omitir
```

### NodePort

Expõe o Service em uma porta de cada nó do cluster. Acessível de fora do cluster via `<IP do nó>:<nodePort>`.

```yaml
spec:
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080           # porta no nó (30000-32767); omitir = aleatória
  type: NodePort
```

No minikube:
```bash
minikube service frontend-svc --url   # abre o túnel e retorna a URL de acesso
```

### LoadBalancer

Provisiona um load balancer externo (cloud). No minikube, precisa de `minikube tunnel`:

```yaml
spec:
  type: LoadBalancer
```

```bash
minikube tunnel    # mantém o túnel aberto em outro terminal
kubectl get service minha-app-svc   # aguardar EXTERNAL-IP aparecer
```

---

## YAML completo com Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend            # deve bater com Service.spec.selector
    spec:
      containers:
        - name: app
          image: nginx:1.25

---
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  selector:
    app: backend                # deve bater com Deployment.spec.template.metadata.labels
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

---

## Comandos de diagnóstico

```bash
kubectl get service <nome>
kubectl get endpoints <nome>                    # ← sempre verifique isso primeiro
kubectl describe service <nome>                 # mostra selector e endpoints
kubectl get pods --show-labels                  # compare com o selector do Service
```

!!! tip "Diagnóstico rápido de Service"
    ```bash
    # Verificar se o Service está selecionando os Pods corretos
    kubectl get endpoints <nome>
    
    # Se vazio, comparar selector do Service com labels dos Pods
    kubectl describe service <nome> | grep Selector
    # Windows (PowerShell): kubectl describe service <nome> | Select-String "Selector"
    kubectl get pods --show-labels | grep <label>
    # Windows (PowerShell): kubectl get pods --show-labels | Select-String "<label>"
    ```

---

## Próximo

➡️ [Ingress](ingress.md) — roteamento HTTP externo por host e path.
