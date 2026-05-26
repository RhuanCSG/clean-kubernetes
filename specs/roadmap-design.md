# Design: Roadmap de Estudo Kubernetes — Camadas do Cluster

**Data:** 2026-05-22  
**Status:** aprovado

---

## Contexto

### Perfil do estudante

- Experiência prévia: operou clusters Kubernetes (EKS/GKE/AKS) sem entender os internals
- Objetivo: operar e debugar clusters com confiança no dia a dia
- Ambiente de prática: local — minikube, kind ou k3s
- Dedicação: 1-3h por semana (ritmo leve)
- Escopo: Kubernetes puro (sem Helm, ArgoCD, Prometheus nesta fase)

### Problema a resolver

Quem opera k8s sem entender o porquê sabe *o que fazer* mas não *por que funcionou* nem *onde olhar quando quebra*. O roadmap precisa preencher esse gap de forma sistemática, sem re-ensinar o que o estudante já sabe na prática.

---

## Abordagem escolhida: Bottom-up (Camadas do Cluster)

Cada fase estuda uma camada de abstração do cluster, de dentro para fora:

```
runtime → controllers → rede → config → storage → segurança → scheduling → control plane
```

Essa ordem garante que cada novo conceito apoia-se em algo já compreendido. O debugging não é um exercício extra — é o critério de conclusão de cada fase.

---

## Estrutura de cada fase

Toda fase segue o mesmo formato:

1. **O que é** — explicação conceitual do recurso e seu papel no cluster
2. **Referência YAML** — estrutura anotada com os campos essenciais e seus efeitos
3. **Lab prático** — exercício hands-on no ambiente local
4. **Cenário de debugging** — cluster quebrado para diagnosticar e consertar
5. **Pronto quando...** — critério explícito de conclusão antes de avançar

---

## Fases do Roadmap

### Fase 1 — Pod & Container Runtime

**Estimativa:** ~2 semanas  
**Camada:** runtime

**O que é cada recurso:**

| Recurso | O que é |
|---|---|
| `Pod` | Menor unidade do k8s; encapsula um ou mais containers que compartilham rede e storage |
| `Namespace` | Partição lógica do cluster; isola grupos de recursos |
| `Container` | Processo executado dentro de um Pod, definido por imagem e entrypoint |
| `InitContainer` | Container que roda antes dos containers principais; usado para setup e pré-condições |

**Referência YAML — Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod
  namespace: default          # partição lógica do cluster
  labels:
    app: exemplo              # seletor usado por Services e Controllers
spec:
  initContainers:             # executam antes dos containers principais
    - name: init
      image: busybox
      command: ["sh", "-c", "echo init concluído"]
  containers:
    - name: app
      image: nginx:1.25
      ports:
        - containerPort: 80   # documentação; não abre porta no host
      env:
        - name: ENV_VAR
          value: "valor"
      resources:
        requests:             # mínimo garantido ao container
          cpu: "100m"
          memory: "128Mi"
        limits:               # teto; ultrapassar memory = OOMKilled
          cpu: "500m"
          memory: "256Mi"
      livenessProbe:          # reinicia container se falhar
        httpGet:
          path: /healthz
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 10
      readinessProbe:         # remove Pod do Service se falhar
        httpGet:
          path: /ready
          port: 80
```

**Lab:** iniciar um Pod com initContainer, inspecionar com `kubectl describe`, `kubectl logs`, `kubectl exec`.

**Debugging:**
- `CrashLoopBackOff` — container reiniciando em loop (verificar logs e liveness probe)
- `ImagePullBackOff` — imagem não encontrada ou sem credencial de registry
- Pod em `Pending` — sem node disponível ou recursos insuficientes

**Pronto quando:** conseguir identificar e corrigir um `CrashLoopBackOff` sem consultar material externo.

---

### Fase 2 — Workload Controllers

**Estimativa:** ~3 semanas  
**Camada:** controllers

**O que é cada recurso:**

| Recurso | O que é |
|---|---|
| `Deployment` | Gerencia ReplicaSets; garante N réplicas de um Pod e permite rolling update/rollback |
| `ReplicaSet` | Mantém N Pods rodando; geralmente não criado diretamente |
| `DaemonSet` | Garante que um Pod rode em cada nó do cluster (ex: agente de log) |
| `StatefulSet` | Pods com identidade estável e ordem de criação garantida (ex: bancos de dados) |
| `Job` | Executa um Pod até conclusão com sucesso |
| `CronJob` | Agenda Jobs em horários definidos (sintaxe cron) |

**Referência YAML — Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minha-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: minha-app          # deve bater com template.metadata.labels
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1       # Pods indisponíveis durante update
      maxSurge: 1             # Pods extras durante update
  template:
    metadata:
      labels:
        app: minha-app
    spec:
      containers:
        - name: app
          image: nginx:1.25
```

**Referência YAML — StatefulSet:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db
spec:
  serviceName: "db"           # headless service obrigatório
  replicas: 3
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
        - name: db
          image: postgres:15
  volumeClaimTemplates:       # PVC criado por Pod, com nome estável
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

**Referência YAML — CronJob:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup
spec:
  schedule: "0 2 * * *"      # sintaxe cron: às 02:00 todo dia
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup
              image: busybox
              command: ["sh", "-c", "echo fazendo backup"]
```

**Lab:** fazer rolling update de um Deployment, acompanhar com `kubectl rollout status`, executar rollback com `kubectl rollout undo`.

**Debugging:**
- Deployment travado em rolling update (maxUnavailable/maxSurge mal configurado)
- Pod não sobe em StatefulSet por PVC não provisionado
- Job completando com falha em loop

**Pronto quando:** conseguir executar rollback de um Deployment em produção (simulada) e explicar o que o ReplicaSet antigo estava fazendo.

---

### Fase 3 — Networking

**Estimativa:** ~3 semanas  
**Camada:** rede

**O que é cada recurso:**

| Recurso | O que é |
|---|---|
| `Service (ClusterIP)` | Endereço IP estável interno ao cluster; balanceia entre Pods via seletor |
| `Service (NodePort)` | Expõe o serviço em uma porta de cada nó do cluster |
| `Service (LoadBalancer)` | Provisiona load balancer externo (cloud); não funciona puro no minikube sem addon |
| `Endpoints` | Lista de IPs/portas que o Service aponta; criado automaticamente |
| `Ingress` | Roteamento HTTP/HTTPS por host/path; requer IngressController instalado |
| `NetworkPolicy` | Firewall declarativo: define quais Pods podem falar com quem |

**Referência YAML — Service ClusterIP:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: minha-app-svc
spec:
  selector:
    app: minha-app            # seleciona Pods com este label
  ports:
    - port: 80                # porta do Service (interna ao cluster)
      targetPort: 8080        # porta do container
  type: ClusterIP             # padrão; só acessível dentro do cluster
```

**Referência YAML — Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minha-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: app.exemplo.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: minha-app-svc
                port:
                  number: 80
```

**Referência YAML — NetworkPolicy:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      app: backend            # aplica-se aos Pods de backend
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend   # só frontend pode acessar backend
      ports:
        - port: 8080
```

**Lab:** criar Service ClusterIP, fazer `kubectl exec` em um Pod e usar `curl` para acessar outro Pod via nome DNS (`<service>.<namespace>.svc.cluster.local`).

**Debugging:**
- Service não roteia (seletor de label errado — Endpoints vazio)
- DNS não resolve dentro do cluster (CoreDNS com problema)
- Ingress retorna 404 (IngressController não instalado ou backend errado)

**Pronto quando:** conseguir diagnosticar um Service com Endpoints vazio e corrigir o seletor de label sem ajuda.

---

### Fase 4 — Configuração & Segredos

**Estimativa:** ~2 semanas  
**Camada:** config

**O que é cada recurso:**

| Recurso | O que é |
|---|---|
| `ConfigMap` | Armazena configuração não-sensível (strings, arquivos de config) |
| `Secret` | Armazena dados sensíveis em base64 (senhas, tokens, certificados) |

**Referência YAML — ConfigMap:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  config.yaml: |             # arquivo montado como volume
    timeout: 30
    retries: 3
```

**Referência YAML — uso no Pod (env e volume):**
```yaml
spec:
  containers:
    - name: app
      envFrom:
        - configMapRef:
            name: app-config  # injeta todas as chaves como variáveis de env
      env:
        - name: DB_PASS
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password   # injeta só a chave específica do Secret
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: app-config      # monta o ConfigMap como diretório
```

**Lab:** criar um ConfigMap com arquivo de configuração, montá-lo como volume em um Pod, e atualizar o ConfigMap observando a propagação automática.

**Debugging:**
- Variável de ambiente ausente (envFrom com nome errado de ConfigMap)
- Secret não decodificado corretamente (base64 com newline)
- Volume não monta (ConfigMap deletado que Pod referencia)

**Pronto quando:** conseguir injetar configuração via env e via arquivo de volume sem consultar documentação.

---

### Fase 5 — Storage

**Estimativa:** ~2 semanas  
**Camada:** armazenamento

**O que é cada recurso:**

| Recurso | O que é |
|---|---|
| `PersistentVolume (PV)` | Disco provisionado pelo admin; existe independente de Pods |
| `PersistentVolumeClaim (PVC)` | Requisição de storage pelo usuário; vinculada a um PV |
| `StorageClass` | Define como PVs são provisionados dinamicamente |
| `emptyDir` | Volume temporário; existe enquanto o Pod existir; compartilhado entre containers do Pod |
| `hostPath` | Monta diretório do nó no Pod; útil em labs locais, perigoso em produção |

**Referência YAML — PVC:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dados-pvc
spec:
  accessModes:
    - ReadWriteOnce           # montado em um nó por vez (RWO)
    # - ReadWriteMany         # montado em múltiplos nós (RWX) — requer NFS/CSI
  storageClassName: standard  # deve existir no cluster
  resources:
    requests:
      storage: 2Gi
```

**Referência YAML — uso no Pod:**
```yaml
spec:
  containers:
    - name: app
      volumeMounts:
        - name: dados
          mountPath: /var/data
  volumes:
    - name: dados
      persistentVolumeClaim:
        claimName: dados-pvc
    - name: temp
      emptyDir: {}            # temporário; zerado ao reiniciar Pod
```

**Lab:** criar um StatefulSet com PVC, escrever dados, deletar o Pod e confirmar que os dados persistem no Pod recriado.

**Debugging:**
- PVC em estado `Pending` (nenhum PV disponível ou StorageClass inexistente)
- Volume monta mas container sem permissão de escrita (fsGroup/securityContext)
- `hostPath` funciona em minikube mas falha em cluster multi-nó

**Pronto quando:** conseguir diagnosticar um PVC Pending e entender a diferença entre provisionamento estático e dinâmico.

---

### Fase 6 — Controle de Acesso (RBAC)

**Estimativa:** ~2 semanas  
**Camada:** segurança

**O que é cada recurso:**

| Recurso | O que é |
|---|---|
| `ServiceAccount` | Identidade de um Pod dentro do cluster; como o Pod se autentica na API |
| `Role` | Conjunto de permissões dentro de um Namespace específico |
| `ClusterRole` | Conjunto de permissões em todo o cluster |
| `RoleBinding` | Liga um Role a um usuário/grupo/ServiceAccount em um Namespace |
| `ClusterRoleBinding` | Liga um ClusterRole globalmente |

**Referência YAML — Role + RoleBinding:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: leitor-pods
  namespace: producao
rules:
  - apiGroups: [""]           # "" = core API group (pods, services, etc.)
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]  # sem create/delete/patch

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-leitor
  namespace: producao
subjects:
  - kind: ServiceAccount
    name: minha-sa
    namespace: producao
roleRef:
  kind: Role
  name: leitor-pods
  apiGroup: rbac.authorization.k8s.io
```

**Referência YAML — ServiceAccount no Pod:**
```yaml
spec:
  serviceAccountName: minha-sa   # Pod usa esta identidade na API
  automountServiceAccountToken: false  # desativa token automático quando não necessário
  containers:
    - name: app
      image: nginx
```

**Lab:** criar uma ServiceAccount com Role de leitura apenas, rodar um Pod usando essa SA e confirmar que não consegue criar recursos.

**Debugging:**
- `403 Forbidden` ao chamar a API (SA sem Role adequado)
- Pod não consegue listar seus próprios recursos (falta verb `list`)
- `kubectl auth can-i` para verificar permissões rapidamente

**Pronto quando:** conseguir investigar um `403 Forbidden` e descobrir qual verbo/recurso está faltando no Role.

---

### Fase 7 — Scheduling & Recursos

**Estimativa:** ~3 semanas  
**Camada:** scheduler

**O que é cada recurso:**

| Recurso | O que é |
|---|---|
| `requests/limits` | `requests`: mínimo garantido; `limits`: teto do container |
| `LimitRange` | Define defaults e limites por container em um Namespace |
| `ResourceQuota` | Define teto total de recursos consumidos por um Namespace |
| `taint/toleration` | Taint repele Pods de um nó; toleration permite que o Pod ignore o taint |
| `nodeSelector` | Agenda Pods apenas em nós com labels específicos |
| `affinity/anti-affinity` | Regras de preferência ou obrigação de co-localização de Pods |
| `HPA` | Escala Deployment automaticamente baseado em métricas (CPU/memória) |

**Referência YAML — LimitRange:**
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limites-padrao
  namespace: dev
spec:
  limits:
    - type: Container
      default:                # aplicado se container não definir limits
        cpu: "500m"
        memory: "256Mi"
      defaultRequest:         # aplicado se container não definir requests
        cpu: "100m"
        memory: "128Mi"
```

**Referência YAML — taint e toleration:**
```yaml
# No nó (via kubectl):
# kubectl taint nodes node1 dedicated=gpu:NoSchedule

# No Pod (toleration):
spec:
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "gpu"
      effect: "NoSchedule"   # NoSchedule | PreferNoSchedule | NoExecute
```

**Referência YAML — HPA:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: minha-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70   # escala quando CPU média > 70%
```

**Lab:** criar um Pod sem resource requests, aplicar LimitRange e verificar os defaults injetados. Adicionar taint em um nó e confirmar que o Pod fica Pending até adicionar a toleration.

**Debugging:**
- `OOMKilled` — container ultrapassou memory limit
- Pod em `Pending` por CPU insuficiente no cluster
- HPA não escala (metrics-server não instalado)

**Pronto quando:** conseguir diagnosticar um Pod em Pending e determinar se o problema é de recursos, taint ou affinity.

---

### Fase 8 — Internals do Control Plane

**Estimativa:** ~3 semanas  
**Camada:** control plane

**O que é cada componente:**

| Componente | O que faz |
|---|---|
| `kube-apiserver` | Porta de entrada de toda comunicação com o cluster; valida e persiste objetos no etcd |
| `etcd` | Banco de dados chave-valor distribuído; única fonte de verdade do estado do cluster |
| `kube-scheduler` | Observa Pods sem nó atribuído e decide onde colocá-los |
| `kube-controller-manager` | Executa os controllers (Deployment, ReplicaSet, Node, etc.) em loop |
| `kubelet` | Agente em cada nó; garante que os containers definidos nos Pods estão rodando |
| `kube-proxy` | Mantém regras de iptables/IPVS nos nós para rotear tráfego dos Services |

**Fluxo de criação de um Pod:**
```
kubectl apply → apiserver valida e persiste no etcd
    → scheduler detecta Pod sem nó → escolhe nó → atualiza no etcd
    → kubelet no nó detecta Pod atribuído → instrui container runtime
    → container runtime cria o container
    → kubelet reporta status de volta ao apiserver
```

**Lab:** usar `kubectl get componentstatuses` e `kubectl get nodes` para inspecionar saúde dos componentes. Em kind/minikube, ver os containers do control plane com `docker ps` ou `crictl ps`.

**Debugging:**
- Nó em `NotReady` — kubelet com problema de comunicação ou container runtime parado
- `apiserver` sem resposta — etcd inacessível ou TLS com problema
- Pod travado em `ContainerCreating` — kubelet consegue ver o Pod mas container runtime falhou

**Pronto quando:** conseguir listar os componentes do control plane, verificar seus logs e entender o papel de cada um num incidente simulado de nó NotReady.

---

## Critérios globais de conclusão do roadmap

Ao final das 8 fases, o estudante deve conseguir:

1. Dado um cluster com problema desconhecido, saber **por onde começar a investigar**
2. Ler um YAML de qualquer recurso k8s puro e entender o que cada campo faz
3. Explicar o que acontece no cluster entre um `kubectl apply` e o Pod ficar `Running`
4. Debugar os erros mais comuns sem consultar StackOverflow como primeira ação

---

## Ambiente de prática recomendado

| Ferramenta | Quando usar |
|---|---|
| `minikube` | Fases 1-6; fácil de instalar, reiniciar e resetar |
| `kind` (Kubernetes in Docker) | Fases 7-8; permite criar clusters multi-nó localmente |

```bash
# Cluster minikube básico
minikube start --driver=docker

# Cluster kind multi-nó (para Fase 7+)
kind create cluster --config kind-config.yaml
```

---

## O que este roadmap não cobre (fora de escopo)

- Helm / Kustomize (gestão de templates)
- ArgoCD / GitOps
- Prometheus / Grafana / Loki (observabilidade)
- Vault (gestão de segredos externa)
- Service Mesh (Istio, Linkerd)
- Multi-cluster

Esses tópicos compõem uma fase 2 natural após dominar o k8s puro.
