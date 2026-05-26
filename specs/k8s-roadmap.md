# Kubernetes Roadmap — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Criar um repositório de estudo Kubernetes bottom-up com READMEs teóricos, YAMLs anotados, labs práticos e cenários de debugging para cada camada do cluster.

**Architecture:** Cada fase vive em `phases/NN-nome/` com quatro subdiretórios fixos: `yaml/` (referências anotadas), `labs/` (lab prático), `debugging/` (cenários quebrados + soluções) e um `README.md` de teoria. A estrutura é idêntica entre fases para facilitar navegação.

**Tech Stack:** Kubernetes (kubectl), minikube (fases 1-6), kind multi-nó (fases 7-8), YAML puro.

---

## Estrutura de arquivos

```
README.md
setup/
  minikube.md
  kind.md
  kind-config.yaml
phases/
  01-pod-runtime/
    README.md
    yaml/pod-reference.yaml
    labs/lab.md
    debugging/README.md
    debugging/01-crashloop/broken.yaml
    debugging/01-crashloop/solution.yaml
    debugging/02-imagepull/broken.yaml
    debugging/02-imagepull/solution.yaml
  02-workload-controllers/
    README.md
    yaml/deployment.yaml
    yaml/statefulset.yaml
    yaml/daemonset.yaml
    yaml/job.yaml
    yaml/cronjob.yaml
    labs/lab.md
    debugging/README.md
    debugging/01-rolling-stuck/broken.yaml
    debugging/01-rolling-stuck/solution.yaml
    debugging/02-statefulset-pvc/broken.yaml
    debugging/02-statefulset-pvc/solution.yaml
  03-networking/
    README.md
    yaml/service-clusterip.yaml
    yaml/service-nodeport.yaml
    yaml/ingress.yaml
    yaml/network-policy.yaml
    labs/lab.md
    debugging/README.md
    debugging/01-empty-endpoints/broken.yaml
    debugging/01-empty-endpoints/solution.yaml
  04-config-secrets/
    README.md
    yaml/configmap.yaml
    yaml/secret.yaml
    yaml/pod-with-config.yaml
    labs/lab.md
    debugging/README.md
    debugging/01-wrong-ref/broken.yaml
    debugging/01-wrong-ref/solution.yaml
  05-storage/
    README.md
    yaml/pvc.yaml
    yaml/pod-with-pvc.yaml
    labs/lab.md
    debugging/README.md
    debugging/01-pvc-pending/broken.yaml
    debugging/01-pvc-pending/solution.yaml
  06-rbac/
    README.md
    yaml/serviceaccount.yaml
    yaml/role-rolebinding.yaml
    labs/lab.md
    debugging/README.md
    debugging/01-403-forbidden/broken.yaml
    debugging/01-403-forbidden/solution.yaml
  07-scheduling/
    README.md
    yaml/limitrange.yaml
    yaml/resourcequota.yaml
    yaml/taint-toleration.yaml
    yaml/affinity.yaml
    yaml/hpa.yaml
    labs/lab.md
    debugging/README.md
    debugging/01-oomkilled/broken.yaml
    debugging/01-oomkilled/solution.yaml
    debugging/02-pending-taint/broken.yaml
    debugging/02-pending-taint/solution.yaml
  08-control-plane/
    README.md
    labs/lab.md
    debugging/README.md
```

---

## Task 0: Setup do Repositório

**Files:**
- Create: `README.md`
- Create: `setup/minikube.md`
- Create: `setup/kind.md`
- Create: `setup/kind-config.yaml`

- [ ] **Step 1: Criar README.md raiz**

```markdown
# Kubernetes Study — Camadas do Cluster

Repositório de estudo prático de Kubernetes, organizado bottom-up: cada pasta em
`phases/` representa uma camada de abstração do cluster.

## Como usar

1. Configure o ambiente: veja `setup/minikube.md` (fases 1-6) ou `setup/kind.md` (fases 7-8)
2. Leia o `README.md` da fase antes de abrir qualquer YAML
3. Aplique os YAMLs de referência, explore com `kubectl describe` e `kubectl logs`
4. Execute o lab conforme `labs/lab.md`
5. Só avance para a próxima fase quando conseguir resolver o cenário de debugging sem ajuda

## Fases

| # | Nome | Camada | Tempo |
|---|---|---|---|
| 01 | Pod & Container Runtime | runtime | ~2 sem |
| 02 | Workload Controllers | controllers | ~3 sem |
| 03 | Networking | rede | ~3 sem |
| 04 | Configuração & Segredos | config | ~2 sem |
| 05 | Storage | armazenamento | ~2 sem |
| 06 | Controle de Acesso (RBAC) | segurança | ~2 sem |
| 07 | Scheduling & Recursos | scheduler | ~3 sem |
| 08 | Internals do Control Plane | control plane | ~3 sem |

## Critério de conclusão do roadmap

- Dado um cluster com problema desconhecido, saber por onde começar a investigar
- Ler um YAML de qualquer recurso k8s puro e entender o que cada campo faz
- Explicar o que acontece entre um `kubectl apply` e o Pod ficar `Running`
- Debugar os erros mais comuns sem consultar Stack Overflow como primeira ação
```

- [ ] **Step 2: Criar setup/minikube.md**

```markdown
# Setup — minikube (Fases 1 a 6)

## Pré-requisitos

- Docker Desktop instalado e rodando
- kubectl instalado (`kubectl version --client`)

## Instalação

```bash
# macOS
brew install minikube

# Windows (PowerShell como admin)
winget install Kubernetes.minikube

# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## Iniciando o cluster

```bash
minikube start --driver=docker --cpus=2 --memory=4g
```

Esperado:
```
✅  minikube v1.34 on ...
✅  Done! kubectl is now configured to use "minikube" cluster
```

## Comandos úteis

```bash
minikube status          # verifica estado do cluster
minikube stop            # para sem destruir
minikube delete          # destroi tudo (bom para reset)
minikube dashboard       # abre UI no navegador
minikube addons enable ingress   # necessário na Fase 3
```

## Verificando o cluster

```bash
kubectl get nodes
# NAME       STATUS   ROLES           AGE
# minikube   Ready    control-plane   1m
```
```

- [ ] **Step 3: Criar setup/kind.md**

```markdown
# Setup — kind multi-nó (Fases 7 e 8)

kind (Kubernetes in Docker) permite criar clusters com múltiplos nós localmente,
necessário para praticar taints, affinity e comportamentos do scheduler.

## Instalação

```bash
# macOS
brew install kind

# Windows (PowerShell como admin)
winget install Kubernetes.kind

# Linux
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
chmod +x kind && sudo mv kind /usr/local/bin/kind
```

## Criando o cluster multi-nó

```bash
kind create cluster --config setup/kind-config.yaml --name k8s-study
kubectl cluster-info --context kind-k8s-study
```

Esperado:
```
Kubernetes control plane is running at https://127.0.0.1:XXXXX
```

## Verificando nós

```bash
kubectl get nodes
# NAME                      STATUS   ROLES           AGE
# k8s-study-control-plane   Ready    control-plane   1m
# k8s-study-worker          Ready    <none>          1m
# k8s-study-worker2         Ready    <none>          1m
```

## Removendo o cluster

```bash
kind delete cluster --name k8s-study
```
```

- [ ] **Step 4: Criar setup/kind-config.yaml**

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker       # dois workers permitem testar affinity e taints
```

- [ ] **Step 5: Commit**

```bash
git init
git add README.md setup/
git commit -m "chore: setup inicial do repositório de estudo k8s"
```

---

## Task 1: Fase 01 — Pod & Container Runtime

**Files:**
- Create: `phases/01-pod-runtime/README.md`
- Create: `phases/01-pod-runtime/yaml/pod-reference.yaml`
- Create: `phases/01-pod-runtime/labs/lab.md`
- Create: `phases/01-pod-runtime/debugging/README.md`
- Create: `phases/01-pod-runtime/debugging/01-crashloop/broken.yaml`
- Create: `phases/01-pod-runtime/debugging/01-crashloop/solution.yaml`
- Create: `phases/01-pod-runtime/debugging/02-imagepull/broken.yaml`
- Create: `phases/01-pod-runtime/debugging/02-imagepull/solution.yaml`

- [ ] **Step 1: Criar phases/01-pod-runtime/README.md**

```markdown
# Fase 01 — Pod & Container Runtime

**Camada:** runtime | **Estimativa:** ~2 semanas

## O que é cada recurso

| Recurso | O que é |
|---|---|
| `Pod` | Menor unidade do k8s; encapsula um ou mais containers que compartilham rede e storage |
| `Namespace` | Partição lógica do cluster; isola grupos de recursos |
| `Container` | Processo executado dentro de um Pod, definido por imagem e entrypoint |
| `InitContainer` | Container que roda antes dos containers principais; usado para setup e pré-condições |

## Como o Pod funciona

O kubelet (agente no nó) é responsável por criar e manter os containers de cada Pod.
Quando você aplica um Pod, o fluxo é:

1. `kubectl apply` envia o objeto ao `kube-apiserver`
2. `kube-apiserver` persiste no `etcd` e notifica o scheduler
3. O scheduler atribui o Pod a um nó
4. O `kubelet` do nó recebe o Pod e instrui o container runtime (containerd) a criar os containers
5. O kubelet monitora os containers e reporta status ao apiserver

## Campos essenciais

- `spec.containers[].image` — imagem do container
- `spec.containers[].resources.requests` — mínimo garantido pelo scheduler
- `spec.containers[].resources.limits` — teto; ultrapassar memory = OOMKilled
- `spec.containers[].livenessProbe` — reinicia o container se falhar
- `spec.containers[].readinessProbe` — remove o Pod dos Endpoints do Service se falhar
- `spec.initContainers` — executam antes dos containers principais, em ordem

## Comandos de inspeção essenciais

```bash
kubectl get pods                          # lista Pods
kubectl get pods -o wide                  # mostra nó e IP
kubectl describe pod <nome>               # eventos, condições, estado
kubectl logs <nome>                       # stdout do container
kubectl logs <nome> -c <container>        # container específico
kubectl logs <nome> --previous            # logs do container que acabou de morrer
kubectl exec -it <nome> -- sh             # shell dentro do container
kubectl get events --sort-by=.lastTimestamp  # eventos do cluster
```

## Pronto quando

Você consegue:
- [ ] Aplicar o YAML de referência e inspecionar o Pod com `describe`
- [ ] Ver os logs do initContainer e do container principal separadamente
- [ ] Resolver o cenário de debugging `01-crashloop` sem consultar material externo
- [ ] Resolver o cenário de debugging `02-imagepull` sem consultar material externo
- [ ] Explicar a diferença entre `livenessProbe` e `readinessProbe` com suas próprias palavras
```

- [ ] **Step 2: Criar phases/01-pod-runtime/yaml/pod-reference.yaml**

```yaml
# YAML de Referência — Pod
# Aplique com: kubectl apply -f pod-reference.yaml
# Inspecione com: kubectl describe pod pod-referencia
# Remova com: kubectl delete pod pod-referencia

apiVersion: v1
kind: Pod
metadata:
  name: pod-referencia
  namespace: default          # partição lógica; omitir usa "default"
  labels:
    app: exemplo              # seletor usado por Services e Controllers
    fase: "01"
spec:
  initContainers:             # executam antes dos containers principais, em ordem
    - name: init-verificacao
      image: busybox:1.36
      command: ["sh", "-c", "echo 'init concluído' && sleep 2"]
  containers:
    - name: app
      image: nginx:1.25
      ports:
        - containerPort: 80   # apenas documentação; não abre porta no host
      env:
        - name: ENV_VAR
          value: "valor-de-exemplo"
      resources:
        requests:             # mínimo reservado pelo scheduler para este container
          cpu: "100m"         # 100 millicores = 0.1 CPU
          memory: "128Mi"
        limits:               # teto absoluto; ultrapassar memory = OOMKilled
          cpu: "500m"
          memory: "256Mi"
      livenessProbe:          # se falhar -> container é reiniciado
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5   # aguarda 5s antes da primeira verificação
        periodSeconds: 10        # verifica a cada 10s
        failureThreshold: 3      # falha 3 vezes antes de reiniciar
      readinessProbe:         # se falhar -> Pod sai dos Endpoints do Service
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 3
        periodSeconds: 5
```

- [ ] **Step 3: Aplicar o YAML e verificar**

```bash
kubectl apply -f phases/01-pod-runtime/yaml/pod-reference.yaml
kubectl get pod pod-referencia -w
```

Esperado (aguardar ~15s):
```
NAME             READY   STATUS     RESTARTS   AGE
pod-referencia   0/1     Init:0/1   0          2s
pod-referencia   0/1     PodInitializing   0   5s
pod-referencia   1/1     Running    0          8s
```

```bash
kubectl describe pod pod-referencia
kubectl logs pod-referencia -c init-verificacao
kubectl logs pod-referencia -c app
kubectl delete pod pod-referencia
```

- [ ] **Step 4: Criar phases/01-pod-runtime/labs/lab.md**

```markdown
# Lab 01 — Inspecionando um Pod Vivo

## Objetivo

Praticar os comandos de inspeção de Pod sem recorrer a documentação.

## Passos

1. Aplique o Pod de referência:
   ```bash
   kubectl apply -f ../yaml/pod-reference.yaml
   ```

2. Aguarde o Pod ficar `Running`:
   ```bash
   kubectl get pod pod-referencia -w
   ```

3. Responda sem olhar o YAML:
   - Qual é o nome do initContainer?
   - Qual image o container principal usa?
   - O Pod tem liveness probe? Em qual path?

   Verifique com:
   ```bash
   kubectl describe pod pod-referencia
   ```

4. Acesse o container com shell:
   ```bash
   kubectl exec -it pod-referencia -- sh
   # dentro do container:
   curl localhost
   env | grep ENV_VAR
   exit
   ```

5. Veja os logs do initContainer (já terminou, mas o log persiste):
   ```bash
   kubectl logs pod-referencia -c init-verificacao
   ```

6. Remova o Pod:
   ```bash
   kubectl delete pod pod-referencia
   ```

## Desafio extra

Crie um segundo Pod (sem usar o YAML de referência, escreva do zero) com:
- imagem `busybox:1.36`
- comando `["sh", "-c", "while true; do echo hello; sleep 5; done"]`
- sem probes
- label `app: meu-pod`

Verifique com `kubectl get pod -l app=meu-pod`.
```

- [ ] **Step 5: Criar phases/01-pod-runtime/debugging/README.md**

```markdown
# Debugging — Fase 01

## Como usar estes cenários

1. Aplique o `broken.yaml` do cenário
2. Observe o estado com `kubectl get pod -w` e `kubectl describe pod <nome>`
3. Identifique a causa sem olhar o `solution.yaml`
4. Corrija e aplique a correção
5. Confirme que o Pod fica `Running`
6. Compare com o `solution.yaml`

## Cenários

### 01-crashloop — CrashLoopBackOff

O Pod entra em loop de reinicializações.

Sintomas esperados:
- `kubectl get pod` mostra `CrashLoopBackOff`
- `RESTARTS` sobe a cada tentativa

Dica de investigação:
```bash
kubectl describe pod crash-pod   # olhe a seção "Events"
kubectl logs crash-pod           # logs da tentativa atual
kubectl logs crash-pod --previous  # logs da tentativa anterior (após 1 restart)
```

### 02-imagepull — ImagePullBackOff

O Pod não consegue baixar a imagem do container.

Sintomas esperados:
- `kubectl get pod` mostra `ImagePullBackOff` ou `ErrImagePull`

Dica de investigação:
```bash
kubectl describe pod pull-fail-pod   # olhe o campo "Image" e a seção "Events"
```
```

- [ ] **Step 6: Criar debugging/01-crashloop/broken.yaml**

```yaml
# Cenário: CrashLoopBackOff
# Problema: liveness probe aponta para path inexistente no nginx
# nginx não tem /healthz; a probe falha imediatamente e reinicia o container
#
# Aplique: kubectl apply -f broken.yaml
# Observe: kubectl get pod crash-pod -w
# Investigue: kubectl describe pod crash-pod
#             kubectl logs crash-pod --previous

apiVersion: v1
kind: Pod
metadata:
  name: crash-pod
  labels:
    debug: crashloop
spec:
  containers:
    - name: app
      image: nginx:1.25
      livenessProbe:
        httpGet:
          path: /healthz          # nginx não serve /healthz por padrão
          port: 80
        initialDelaySeconds: 2
        periodSeconds: 3
        failureThreshold: 1       # reinicia após 1 única falha
```

- [ ] **Step 7: Criar debugging/01-crashloop/solution.yaml**

```yaml
# Solução: CrashLoopBackOff
# Fix: liveness probe aponta para / que o nginx serve corretamente
# failureThreshold aumentado para 3 (valor razoável para produção)
#
# Aplique: kubectl apply -f solution.yaml
# Confirme: kubectl get pod crash-pod-fixed

apiVersion: v1
kind: Pod
metadata:
  name: crash-pod-fixed
  labels:
    debug: crashloop
spec:
  containers:
    - name: app
      image: nginx:1.25
      livenessProbe:
        httpGet:
          path: /                 # nginx responde 200 em /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 10
        failureThreshold: 3
```

- [ ] **Step 8: Criar debugging/02-imagepull/broken.yaml**

```yaml
# Cenário: ImagePullBackOff
# Problema: tag da imagem não existe no Docker Hub
# O kubelet tenta baixar e falha com "not found"
#
# Aplique: kubectl apply -f broken.yaml
# Observe: kubectl get pod pull-fail-pod -w
# Investigue: kubectl describe pod pull-fail-pod

apiVersion: v1
kind: Pod
metadata:
  name: pull-fail-pod
  labels:
    debug: imagepull
spec:
  containers:
    - name: app
      image: nginx:esta-tag-nao-existe   # tag inválida
```

- [ ] **Step 9: Criar debugging/02-imagepull/solution.yaml**

```yaml
# Solução: ImagePullBackOff
# Fix: usar tag válida e existente no Docker Hub
#
# Aplique: kubectl apply -f solution.yaml
# Confirme: kubectl get pod pull-ok-pod

apiVersion: v1
kind: Pod
metadata:
  name: pull-ok-pod
  labels:
    debug: imagepull
spec:
  containers:
    - name: app
      image: nginx:1.25   # tag válida
```

- [ ] **Step 10: Verificar cenários de debugging**

```bash
# Teste 01-crashloop
kubectl apply -f phases/01-pod-runtime/debugging/01-crashloop/broken.yaml
kubectl get pod crash-pod -w
# aguardar aparecer CrashLoopBackOff (30-60s)
kubectl describe pod crash-pod
kubectl delete pod crash-pod

kubectl apply -f phases/01-pod-runtime/debugging/01-crashloop/solution.yaml
kubectl get pod crash-pod-fixed -w
# deve ficar Running
kubectl delete pod crash-pod-fixed

# Teste 02-imagepull
kubectl apply -f phases/01-pod-runtime/debugging/02-imagepull/broken.yaml
kubectl get pod pull-fail-pod -w
# deve mostrar ImagePullBackOff ou ErrImagePull
kubectl delete pod pull-fail-pod

kubectl apply -f phases/01-pod-runtime/debugging/02-imagepull/solution.yaml
kubectl get pod pull-ok-pod -w
# deve ficar Running
kubectl delete pod pull-ok-pod
```

- [ ] **Step 11: Commit**

```bash
git add phases/01-pod-runtime/
git commit -m "feat: fase 01 - pod e container runtime"
```

---

## Task 2: Fase 02 — Workload Controllers

**Files:**
- Create: `phases/02-workload-controllers/README.md`
- Create: `phases/02-workload-controllers/yaml/deployment.yaml`
- Create: `phases/02-workload-controllers/yaml/statefulset.yaml`
- Create: `phases/02-workload-controllers/yaml/daemonset.yaml`
- Create: `phases/02-workload-controllers/yaml/job.yaml`
- Create: `phases/02-workload-controllers/yaml/cronjob.yaml`
- Create: `phases/02-workload-controllers/labs/lab.md`
- Create: `phases/02-workload-controllers/debugging/README.md`
- Create: `phases/02-workload-controllers/debugging/01-rolling-stuck/broken.yaml`
- Create: `phases/02-workload-controllers/debugging/01-rolling-stuck/solution.yaml`
- Create: `phases/02-workload-controllers/debugging/02-statefulset-pvc/broken.yaml`
- Create: `phases/02-workload-controllers/debugging/02-statefulset-pvc/solution.yaml`

- [ ] **Step 1: Criar phases/02-workload-controllers/README.md**

```markdown
# Fase 02 — Workload Controllers

**Camada:** controllers | **Estimativa:** ~3 semanas

## O que é cada recurso

| Recurso | O que é |
|---|---|
| `Deployment` | Gerencia ReplicaSets; garante N réplicas e permite rolling update/rollback |
| `ReplicaSet` | Mantém N Pods rodando; criado e gerenciado pelo Deployment, raramente criado diretamente |
| `DaemonSet` | Garante que um Pod rode em cada nó do cluster (agente de log, monitoramento) |
| `StatefulSet` | Pods com identidade estável (nome fixo, PVC próprio, ordem de criação garantida) |
| `Job` | Executa um Pod até conclusão com sucesso; não reinicia após completar |
| `CronJob` | Agenda criação de Jobs em horários definidos com sintaxe cron |

## Como o Deployment funciona

O Deployment não gerencia Pods diretamente — ele gerencia ReplicaSets.

```
Deployment → cria/gerencia → ReplicaSet → cria/gerencia → Pods
```

Em um rolling update:
1. O Deployment cria um novo ReplicaSet com a nova versão
2. Incrementa o novo e decrementa o antigo respeitando `maxUnavailable` e `maxSurge`
3. O ReplicaSet antigo permanece (com 0 réplicas) para permitir rollback

## Comandos essenciais de controllers

```bash
kubectl get deployments
kubectl rollout status deployment/<nome>   # acompanha update em tempo real
kubectl rollout history deployment/<nome>  # histórico de revisões
kubectl rollout undo deployment/<nome>     # rollback para revisão anterior
kubectl scale deployment/<nome> --replicas=5
kubectl get replicasets                    # veja os RSes antigos
kubectl get statefulsets
kubectl get daemonsets
kubectl get jobs
kubectl get cronjobs
```

## Pronto quando

- [ ] Executar rolling update e rollback de um Deployment
- [ ] Explicar o que o ReplicaSet antigo (com 0 réplicas) está fazendo após o update
- [ ] Resolver o cenário 01-rolling-stuck sem ajuda
- [ ] Resolver o cenário 02-statefulset-pvc sem ajuda
```

- [ ] **Step 2: Criar yaml/deployment.yaml**

```yaml
# YAML de Referência — Deployment
# Aplique: kubectl apply -f deployment.yaml
# Acompanhe: kubectl rollout status deployment/app-deployment

apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: minha-app            # DEVE ser idêntico a spec.template.metadata.labels
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1         # máximo de Pods que podem ficar indisponíveis durante update
      maxSurge: 1               # máximo de Pods extras criados durante update
  template:
    metadata:
      labels:
        app: minha-app          # DEVE bater com spec.selector.matchLabels
    spec:
      containers:
        - name: app
          image: nginx:1.24     # mude para 1.25 para disparar rolling update
          resources:
            requests:
              cpu: "100m"
              memory: "64Mi"
            limits:
              cpu: "200m"
              memory: "128Mi"
```

- [ ] **Step 3: Criar yaml/statefulset.yaml**

```yaml
# YAML de Referência — StatefulSet
# Diferencial: cada Pod tem nome estável (db-0, db-1, db-2) e PVC próprio
# Requer um Headless Service (clusterIP: None)
#
# Aplique: kubectl apply -f statefulset.yaml
# Observe nomes estáveis: kubectl get pods -l app=db

apiVersion: v1
kind: Service
metadata:
  name: db-headless
spec:
  clusterIP: None               # headless: sem IP de Service, DNS aponta para Pods individuais
  selector:
    app: db
  ports:
    - port: 5432

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db
spec:
  serviceName: "db-headless"    # referencia o headless service acima
  replicas: 2
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
          env:
            - name: POSTGRES_PASSWORD
              value: "senha-local"
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:         # cria um PVC por Pod: data-db-0, data-db-1
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

- [ ] **Step 4: Criar yaml/daemonset.yaml**

```yaml
# YAML de Referência — DaemonSet
# Um Pod por nó do cluster; útil para agentes de log, monitoramento, etc.
# Quando um nó é adicionado ao cluster, o DaemonSet cria um Pod nele automaticamente.
#
# Aplique: kubectl apply -f daemonset.yaml
# Verifique (um Pod por nó): kubectl get pods -l app=log-agent -o wide

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
spec:
  selector:
    matchLabels:
      app: log-agent
  template:
    metadata:
      labels:
        app: log-agent
    spec:
      containers:
        - name: agent
          image: busybox:1.36
          command: ["sh", "-c", "while true; do echo 'coletando logs'; sleep 30; done"]
          resources:
            requests:
              cpu: "50m"
              memory: "32Mi"
            limits:
              cpu: "100m"
              memory: "64Mi"
```

- [ ] **Step 5: Criar yaml/job.yaml**

```yaml
# YAML de Referência — Job
# Executa o Pod até completar com sucesso; após completar, o Pod fica em status Completed
#
# Aplique: kubectl apply -f job.yaml
# Acompanhe: kubectl get job processamento -w
# Veja resultado: kubectl logs -l job-name=processamento

apiVersion: batch/v1
kind: Job
metadata:
  name: processamento
spec:
  completions: 1                # número de completions bem-sucedidos necessários
  parallelism: 1                # Pods rodando em paralelo
  backoffLimit: 3               # tentativas antes de marcar como Failed
  template:
    spec:
      restartPolicy: OnFailure  # OnFailure ou Never (nunca use Always em Jobs)
      containers:
        - name: worker
          image: busybox:1.36
          command: ["sh", "-c", "echo 'processamento concluído'; exit 0"]
```

- [ ] **Step 6: Criar yaml/cronjob.yaml**

```yaml
# YAML de Referência — CronJob
# Agenda criação de Jobs em horários definidos
# Sintaxe cron: minuto hora dia-do-mes mes dia-da-semana
#
# Aplique: kubectl apply -f cronjob.yaml
# Liste: kubectl get cronjobs
# Disparo manual: kubectl create job --from=cronjob/backup backup-manual

apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup
spec:
  schedule: "*/2 * * * *"      # a cada 2 minutos (para testar rapidamente)
  concurrencyPolicy: Forbid    # não cria novo Job se o anterior ainda está rodando
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup
              image: busybox:1.36
              command: ["sh", "-c", "echo 'backup executado em' $(date)"]
```

- [ ] **Step 7: Criar labs/lab.md**

```markdown
# Lab 02 — Rolling Update e Rollback

## Objetivo

Executar um rolling update de um Deployment, acompanhar o processo e fazer rollback.

## Passos

### Parte 1: Deploy inicial

```bash
kubectl apply -f ../yaml/deployment.yaml
kubectl get pods -l app=minha-app
kubectl rollout status deployment/app-deployment
```

Esperado: 3 pods Running com nginx:1.24

### Parte 2: Rolling Update

Edite `../yaml/deployment.yaml` e mude `image: nginx:1.24` para `image: nginx:1.25`.

```bash
kubectl apply -f ../yaml/deployment.yaml
kubectl rollout status deployment/app-deployment
```

Em outro terminal, observe o que acontece com os ReplicaSets:
```bash
kubectl get replicasets -w
```

Você deve ver dois ReplicaSets: o antigo descendo para 0 e o novo subindo para 3.

### Parte 3: Inspecionar histórico

```bash
kubectl rollout history deployment/app-deployment
```

### Parte 4: Rollback

```bash
kubectl rollout undo deployment/app-deployment
kubectl rollout status deployment/app-deployment
kubectl get pods -l app=minha-app -o jsonpath='{.items[0].spec.containers[0].image}'
```

Esperado: nginx:1.24 (voltou para a versão anterior)

### Parte 5: Limpeza

```bash
kubectl delete deployment app-deployment
```

## Pergunta para reflexão

O ReplicaSet antigo (com 0 réplicas) continua existindo após o update. Por quê?
```

- [ ] **Step 8: Criar debugging/README.md**

```markdown
# Debugging — Fase 02

## Cenários

### 01-rolling-stuck — Rolling Update travado

Um Deployment ficou travado durante o update e não progride.

Dica de investigação:
```bash
kubectl rollout status deployment/stuck-deploy
kubectl describe deployment stuck-deploy   # olhe strategy.rollingUpdate
kubectl get replicasets
```

### 02-statefulset-pvc — StatefulSet com Pod em Pending

Um StatefulSet não consegue criar seus Pods.

Dica de investigação:
```bash
kubectl get pods -l app=db-debug
kubectl describe pod db-debug-0           # olhe a seção Events
kubectl get pvc                           # verifique o status do PVC
```
```

- [ ] **Step 9: Criar debugging/01-rolling-stuck/broken.yaml**

```yaml
# Cenário: Rolling Update Travado
# Problema: maxUnavailable=0 e maxSurge=0 torna o update impossível
# O Deployment não pode tirar nenhum Pod do ar nem criar extras
#
# Aplique: kubectl apply -f broken.yaml
# Dispare o update: kubectl set image deployment/stuck-deploy app=nginx:1.25
# Observe travado: kubectl rollout status deployment/stuck-deploy

apiVersion: apps/v1
kind: Deployment
metadata:
  name: stuck-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: stuck
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0         # não pode tirar nenhum Pod do ar
      maxSurge: 0               # não pode criar nenhum Pod extra
                                # resultado: matematicamente impossível fazer update
  template:
    metadata:
      labels:
        app: stuck
    spec:
      containers:
        - name: app
          image: nginx:1.24
```

- [ ] **Step 10: Criar debugging/01-rolling-stuck/solution.yaml**

```yaml
# Solução: Rolling Update Travado
# Fix: maxSurge=1 permite criar 1 Pod extra durante o update
#
# Aplique: kubectl apply -f solution.yaml
# Dispare: kubectl set image deployment/stuck-deploy-fixed app=nginx:1.25
# Observe progredindo: kubectl rollout status deployment/stuck-deploy-fixed

apiVersion: apps/v1
kind: Deployment
metadata:
  name: stuck-deploy-fixed
spec:
  replicas: 2
  selector:
    matchLabels:
      app: stuck-fixed
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1               # cria 1 Pod novo antes de remover o antigo
  template:
    metadata:
      labels:
        app: stuck-fixed
    spec:
      containers:
        - name: app
          image: nginx:1.24
```

- [ ] **Step 11: Criar debugging/02-statefulset-pvc/broken.yaml**

```yaml
# Cenário: StatefulSet com Pod em Pending
# Problema: storageClassName referencia uma classe que não existe no cluster
# O PVC fica Pending e o Pod não consegue ser criado
#
# Aplique: kubectl apply -f broken.yaml
# Observe: kubectl get pods -l app=db-debug
# Investigue: kubectl describe pod db-debug-0
#             kubectl get pvc

apiVersion: v1
kind: Service
metadata:
  name: db-debug-headless
spec:
  clusterIP: None
  selector:
    app: db-debug
  ports:
    - port: 5432

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db-debug
spec:
  serviceName: "db-debug-headless"
  replicas: 1
  selector:
    matchLabels:
      app: db-debug
  template:
    metadata:
      labels:
        app: db-debug
    spec:
      containers:
        - name: db
          image: postgres:15
          env:
            - name: POSTGRES_PASSWORD
              value: "senha"
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: "nao-existe"   # StorageClass inexistente no cluster
        resources:
          requests:
            storage: 1Gi
```

- [ ] **Step 12: Criar debugging/02-statefulset-pvc/solution.yaml**

```yaml
# Solução: StatefulSet com Pod em Pending
# Fix: usar "standard" que é a StorageClass padrão do minikube
# Verifique as classes disponíveis com: kubectl get storageclasses
#
# Aplique: kubectl apply -f solution.yaml
# Confirme: kubectl get pods -l app=db-fixed

apiVersion: v1
kind: Service
metadata:
  name: db-fixed-headless
spec:
  clusterIP: None
  selector:
    app: db-fixed
  ports:
    - port: 5432

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db-fixed
spec:
  serviceName: "db-fixed-headless"
  replicas: 1
  selector:
    matchLabels:
      app: db-fixed
  template:
    metadata:
      labels:
        app: db-fixed
    spec:
      containers:
        - name: db
          image: postgres:15
          env:
            - name: POSTGRES_PASSWORD
              value: "senha"
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: "standard"   # StorageClass padrão do minikube
        resources:
          requests:
            storage: 1Gi
```

- [ ] **Step 13: Verificar arquivos da fase 02**

```bash
# Verificar que os arquivos YAML são válidos (dry-run)
kubectl apply --dry-run=client -f phases/02-workload-controllers/yaml/deployment.yaml
kubectl apply --dry-run=client -f phases/02-workload-controllers/yaml/statefulset.yaml
kubectl apply --dry-run=client -f phases/02-workload-controllers/yaml/daemonset.yaml
kubectl apply --dry-run=client -f phases/02-workload-controllers/yaml/job.yaml
kubectl apply --dry-run=client -f phases/02-workload-controllers/yaml/cronjob.yaml
kubectl apply --dry-run=client -f phases/02-workload-controllers/debugging/01-rolling-stuck/broken.yaml
kubectl apply --dry-run=client -f phases/02-workload-controllers/debugging/01-rolling-stuck/solution.yaml
```

Esperado: cada comando retorna `... configured (dry run)` sem erros.

- [ ] **Step 14: Commit**

```bash
git add phases/02-workload-controllers/
git commit -m "feat: fase 02 - workload controllers"
```

---

## Task 3: Fase 03 — Networking

**Files:**
- Create: `phases/03-networking/README.md`
- Create: `phases/03-networking/yaml/service-clusterip.yaml`
- Create: `phases/03-networking/yaml/service-nodeport.yaml`
- Create: `phases/03-networking/yaml/ingress.yaml`
- Create: `phases/03-networking/yaml/network-policy.yaml`
- Create: `phases/03-networking/labs/lab.md`
- Create: `phases/03-networking/debugging/README.md`
- Create: `phases/03-networking/debugging/01-empty-endpoints/broken.yaml`
- Create: `phases/03-networking/debugging/01-empty-endpoints/solution.yaml`

- [ ] **Step 1: Criar phases/03-networking/README.md**

```markdown
# Fase 03 — Networking

**Camada:** rede | **Estimativa:** ~3 semanas

## O que é cada recurso

| Recurso | O que é |
|---|---|
| `Service (ClusterIP)` | IP estável interno ao cluster; balanceia entre Pods via seletor de label |
| `Service (NodePort)` | Expõe o Service em uma porta (30000-32767) de cada nó |
| `Service (LoadBalancer)` | Provisiona LB externo na cloud; no minikube requer `minikube tunnel` |
| `Endpoints` | Lista de IP:porta dos Pods selecionados pelo Service; criado automaticamente |
| `Ingress` | Roteamento HTTP/HTTPS por host/path; requer IngressController instalado |
| `NetworkPolicy` | Firewall declarativo entre Pods; sem NetworkPolicy, todos se comunicam livremente |

## Como o Service funciona

```
Pod A  →  Service (ClusterIP: 10.96.x.x)  →  kube-proxy (iptables/IPVS)  →  Pod B
```

O Service não é um processo — é uma regra de iptables mantida pelo `kube-proxy` em cada nó.
O DNS interno (CoreDNS) resolve `<service>.<namespace>.svc.cluster.local` para o IP do Service.

## DNS interno do cluster

Dentro de qualquer Pod, você pode resolver outros Services por nome:

```bash
# Dentro de um Pod:
curl http://meu-service                              # mesmo namespace
curl http://meu-service.outro-namespace             # namespace diferente
curl http://meu-service.outro-namespace.svc.cluster.local  # FQDN completo
```

## Diagnóstico de Service

```bash
kubectl get service <nome>
kubectl get endpoints <nome>      # se vazio: seletor de label não bate com nenhum Pod
kubectl describe service <nome>   # mostra selector e endpoints
kubectl get pods -l app=<label>   # verifica se os Pods existem com o label esperado
```

## Pronto quando

- [ ] Acessar um Pod de dentro de outro Pod usando o nome DNS do Service
- [ ] Resolver o cenário 01-empty-endpoints sem ajuda
- [ ] Explicar por que Endpoints vazio significa problema de seletor de label
```

- [ ] **Step 2: Criar yaml/service-clusterip.yaml**

```yaml
# YAML de Referência — Service ClusterIP
# ClusterIP é o tipo padrão: IP acessível apenas dentro do cluster
#
# Pré-requisito: ter um Deployment com label app=minha-app rodando
# Aplique: kubectl apply -f service-clusterip.yaml
# Verifique endpoints: kubectl get endpoints minha-app-svc

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
    app: backend                # seleciona Pods com este label
  ports:
    - port: 80                  # porta do Service (usada por outros Pods para acessar)
      targetPort: 80            # porta do container
  type: ClusterIP               # padrão; acessível apenas dentro do cluster
```

- [ ] **Step 3: Criar yaml/service-nodeport.yaml**

```yaml
# YAML de Referência — Service NodePort
# Expõe o Service em uma porta de cada nó do cluster
# Útil para acesso externo em labs locais sem IngressController
#
# Aplique: kubectl apply -f service-nodeport.yaml
# Acesse (minikube): minikube service frontend-svc --url

apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    app: backend                # reutiliza o Deployment do arquivo anterior
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080           # porta no nó (30000-32767); omitir = aleatória
  type: NodePort
```

- [ ] **Step 4: Criar yaml/ingress.yaml**

```yaml
# YAML de Referência — Ingress
# Roteamento HTTP por host/path; requer IngressController
#
# Pré-requisito (minikube): minikube addons enable ingress
# Aplique: kubectl apply -f ingress.yaml
# Adicione ao /etc/hosts: 127.0.0.1 app.local
# Acesse: curl http://app.local (após minikube tunnel)

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: app.local           # deve resolver para o IP do IngressController
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: backend-svc   # Service que deve existir no mesmo namespace
                port:
                  number: 80
```

- [ ] **Step 5: Criar yaml/network-policy.yaml**

```yaml
# YAML de Referência — NetworkPolicy
# Sem NetworkPolicy: todos os Pods se comunicam livremente
# Com NetworkPolicy: apenas tráfego explicitamente permitido passa
#
# Este exemplo: Pods de "backend" só aceitam tráfego de Pods de "frontend"
#
# Pré-requisito: CNI com suporte a NetworkPolicy (minikube usa Calico ou Cilium)
# Aplique: kubectl apply -f network-policy.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-allow-frontend
spec:
  podSelector:
    matchLabels:
      app: backend              # aplica esta regra aos Pods de backend
  policyTypes:
    - Ingress                   # controla tráfego de entrada
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend     # só Pods com label app=frontend podem acessar
      ports:
        - protocol: TCP
          port: 80
```

- [ ] **Step 6: Criar labs/lab.md**

```markdown
# Lab 03 — Service Discovery e DNS Interno

## Objetivo

Acessar um Pod de dentro de outro Pod usando o nome DNS do Service.

## Passos

### Parte 1: Criar o backend com Service

```bash
kubectl apply -f ../yaml/service-clusterip.yaml
kubectl get service backend-svc
kubectl get endpoints backend-svc   # deve mostrar IPs dos Pods
```

### Parte 2: Criar um Pod cliente

```bash
kubectl run cliente --image=busybox:1.36 --restart=Never -- sleep 3600
kubectl get pod cliente
```

### Parte 3: Testar DNS interno

```bash
kubectl exec -it cliente -- sh
# dentro do Pod:
nslookup backend-svc
curl http://backend-svc
curl http://backend-svc.default.svc.cluster.local   # FQDN completo
exit
```

Esperado: `curl` retorna a página padrão do nginx.

### Parte 4: Inspecionar o Service

```bash
# Qual IP o DNS resolve?
kubectl get service backend-svc -o jsonpath='{.spec.clusterIP}'

# Quais Pods estão no Endpoints?
kubectl get endpoints backend-svc
```

### Parte 5: Limpeza

```bash
kubectl delete deployment backend
kubectl delete service backend-svc
kubectl delete pod cliente
```

## Pergunta para reflexão

Se você mudar o label de um Pod de `app: backend` para `app: outro`, o que acontece
com os Endpoints do Service? Por quê?
```

- [ ] **Step 7: Criar debugging/README.md**

```markdown
# Debugging — Fase 03

## Cenários

### 01-empty-endpoints — Service não roteia tráfego

O Service existe mas não encaminha tráfego para nenhum Pod.

Dica de investigação:
```bash
kubectl get endpoints svc-quebrado
# se mostrar "<none>" → seletor de label não bate com nenhum Pod
kubectl describe service svc-quebrado   # olhe o campo "Selector"
kubectl get pods --show-labels           # compare os labels dos Pods
```
```

- [ ] **Step 8: Criar debugging/01-empty-endpoints/broken.yaml**

```yaml
# Cenário: Service com Endpoints vazio
# Problema: Service.spec.selector usa label "app: backend-v2"
# mas o Deployment cria Pods com label "app: backend"
# Resultado: kubectl get endpoints svc-quebrado mostra <none>
#
# Aplique: kubectl apply -f broken.yaml
# Investigue: kubectl get endpoints svc-quebrado
#             kubectl get pods --show-labels

apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-broken
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend              # label que os Pods recebem
  template:
    metadata:
      labels:
        app: backend            # Pods têm este label
    spec:
      containers:
        - name: app
          image: nginx:1.25

---
apiVersion: v1
kind: Service
metadata:
  name: svc-quebrado
spec:
  selector:
    app: backend-v2             # ERRADO: procura label que não existe nos Pods
  ports:
    - port: 80
      targetPort: 80
```

- [ ] **Step 9: Criar debugging/01-empty-endpoints/solution.yaml**

```yaml
# Solução: Service com Endpoints vazio
# Fix: selector do Service agora bate com o label dos Pods (app: backend)
#
# Aplique: kubectl apply -f solution.yaml
# Confirme: kubectl get endpoints svc-corrigido
# Deve mostrar os IPs dos Pods, não <none>

apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-fixed
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend-fixed
  template:
    metadata:
      labels:
        app: backend-fixed
    spec:
      containers:
        - name: app
          image: nginx:1.25

---
apiVersion: v1
kind: Service
metadata:
  name: svc-corrigido
spec:
  selector:
    app: backend-fixed          # CORRETO: bate com o label dos Pods
  ports:
    - port: 80
      targetPort: 80
```

- [ ] **Step 10: Verificar YAMLs com dry-run**

```bash
kubectl apply --dry-run=client -f phases/03-networking/yaml/service-clusterip.yaml
kubectl apply --dry-run=client -f phases/03-networking/yaml/service-nodeport.yaml
kubectl apply --dry-run=client -f phases/03-networking/yaml/ingress.yaml
kubectl apply --dry-run=client -f phases/03-networking/yaml/network-policy.yaml
kubectl apply --dry-run=client -f phases/03-networking/debugging/01-empty-endpoints/broken.yaml
kubectl apply --dry-run=client -f phases/03-networking/debugging/01-empty-endpoints/solution.yaml
```

Esperado: nenhum erro de validação.

- [ ] **Step 11: Commit**

```bash
git add phases/03-networking/
git commit -m "feat: fase 03 - networking"
```

---

## Task 4: Fase 04 — Configuração & Segredos

**Files:**
- Create: `phases/04-config-secrets/README.md`
- Create: `phases/04-config-secrets/yaml/configmap.yaml`
- Create: `phases/04-config-secrets/yaml/secret.yaml`
- Create: `phases/04-config-secrets/yaml/pod-with-config.yaml`
- Create: `phases/04-config-secrets/labs/lab.md`
- Create: `phases/04-config-secrets/debugging/README.md`
- Create: `phases/04-config-secrets/debugging/01-wrong-ref/broken.yaml`
- Create: `phases/04-config-secrets/debugging/01-wrong-ref/solution.yaml`

- [ ] **Step 1: Criar phases/04-config-secrets/README.md**

```markdown
# Fase 04 — Configuração & Segredos

**Camada:** config | **Estimativa:** ~2 semanas

## O que é cada recurso

| Recurso | O que é |
|---|---|
| `ConfigMap` | Armazena configuração não-sensível como strings ou arquivos inteiros |
| `Secret` | Armazena dados sensíveis codificados em base64 (não criptografados por padrão) |

## Formas de consumir ConfigMap e Secret em um Pod

| Método | Quando usar |
|---|---|
| `env.valueFrom.configMapKeyRef` | Injetar uma chave específica como variável de ambiente |
| `envFrom.configMapRef` | Injetar todas as chaves do ConfigMap como variáveis de ambiente |
| `volume + volumeMount` | Expor chaves como arquivos dentro do container |

### Propagação automática

Quando um ConfigMap é atualizado, os volumes que o montam recebem os novos valores
automaticamente (em ~1 minuto). Variáveis de ambiente (`env`/`envFrom`) NÃO são
atualizadas automaticamente — o Pod precisa ser recriado.

## Comandos essenciais

```bash
kubectl get configmap
kubectl describe configmap <nome>
kubectl get secret
kubectl get secret <nome> -o jsonpath='{.data.password}' | base64 -d  # decodifica o valor
```

## Pronto quando

- [ ] Injetar config via `envFrom` e confirmar com `kubectl exec -- env`
- [ ] Montar ConfigMap como volume e ler o arquivo dentro do container
- [ ] Resolver o cenário 01-wrong-ref sem ajuda
```

- [ ] **Step 2: Criar yaml/configmap.yaml**

```yaml
# YAML de Referência — ConfigMap
# Aplique: kubectl apply -f configmap.yaml
# Inspecione: kubectl describe configmap app-config

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "producao"           # chave simples: vira variável de ambiente
  LOG_LEVEL: "info"
  config.yaml: |               # valor multi-linha: vira arquivo quando montado como volume
    timeout: 30
    retries: 3
    database:
      host: db-svc
      port: 5432
```

- [ ] **Step 3: Criar yaml/secret.yaml**

```yaml
# YAML de Referência — Secret
# Os valores em data: são codificados em base64, não criptografados
# Para criar: kubectl create secret generic db-secret --from-literal=password=minha-senha
# Ou aplique este YAML (valor já em base64):
#   echo -n "minha-senha" | base64  →  bWluaGEtc2VuaGE=
#
# Aplique: kubectl apply -f secret.yaml
# Decodifique: kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 -d

apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque                    # tipo genérico; outros tipos: kubernetes.io/tls, kubernetes.io/dockerconfigjson
data:
  password: bWluaGEtc2VuaGE=   # base64 de "minha-senha"
  username: YWRtaW4=            # base64 de "admin"
```

- [ ] **Step 4: Criar yaml/pod-with-config.yaml**

```yaml
# YAML de Referência — Pod consumindo ConfigMap e Secret
# Pré-requisito: aplicar configmap.yaml e secret.yaml antes
# Aplique: kubectl apply -f pod-with-config.yaml

apiVersion: v1
kind: Pod
metadata:
  name: app-configurado
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "env && cat /etc/config/config.yaml && sleep 3600"]
      envFrom:
        - configMapRef:
            name: app-config    # injeta APP_ENV e LOG_LEVEL como variáveis de ambiente
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password     # injeta apenas a chave "password" do Secret
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config   # config.yaml aparece como /etc/config/config.yaml
  volumes:
    - name: config-vol
      configMap:
        name: app-config
        items:
          - key: config.yaml    # monta apenas esta chave, não todas
            path: config.yaml
```

- [ ] **Step 5: Criar labs/lab.md**

```markdown
# Lab 04 — Injetar Configuração via Env e Volume

## Objetivo

Confirmar que ConfigMap e Secret chegam corretamente dentro do container.

## Passos

### Parte 1: Criar os recursos de configuração

```bash
kubectl apply -f ../yaml/configmap.yaml
kubectl apply -f ../yaml/secret.yaml
kubectl apply -f ../yaml/pod-with-config.yaml
kubectl get pod app-configurado -w
```

### Parte 2: Verificar variáveis de ambiente

```bash
kubectl exec app-configurado -- env | grep -E "APP_ENV|LOG_LEVEL|DB_PASSWORD"
```

Esperado:
```
APP_ENV=producao
LOG_LEVEL=info
DB_PASSWORD=minha-senha
```

### Parte 3: Verificar arquivo montado

```bash
kubectl exec app-configurado -- cat /etc/config/config.yaml
```

Esperado: conteúdo do campo `config.yaml` do ConfigMap.

### Parte 4: Testar propagação de ConfigMap

```bash
# Edite o ConfigMap e mude LOG_LEVEL para "debug"
kubectl edit configmap app-config
# aguarde ~60 segundos
kubectl exec app-configurado -- cat /etc/config/config.yaml
# O arquivo deve ter sido atualizado automaticamente
# Mas: env | grep LOG_LEVEL ainda mostra "info" (não atualiza sem reiniciar o Pod)
```

### Parte 5: Limpeza

```bash
kubectl delete pod app-configurado
kubectl delete configmap app-config
kubectl delete secret db-secret
```
```

- [ ] **Step 6: Criar debugging/README.md**

```markdown
# Debugging — Fase 04

## Cenários

### 01-wrong-ref — Variável de ambiente ausente

O Pod não consegue iniciar porque referencia um ConfigMap que não existe.

Dica de investigação:
```bash
kubectl describe pod config-fail-pod   # olhe a seção Events
# Procure por: "configmap not found" ou "secret not found"
kubectl get configmaps                  # verifique quais ConfigMaps existem
```
```

- [ ] **Step 7: Criar debugging/01-wrong-ref/broken.yaml**

```yaml
# Cenário: Pod com referência a ConfigMap inexistente
# Problema: envFrom referencia "app-settings" que não existe no cluster
# O Pod fica em estado CreateContainerConfigError
#
# Aplique: kubectl apply -f broken.yaml
# Observe: kubectl get pod config-fail-pod
# Investigue: kubectl describe pod config-fail-pod

apiVersion: v1
kind: Pod
metadata:
  name: config-fail-pod
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "env && sleep 3600"]
      envFrom:
        - configMapRef:
            name: app-settings   # ConfigMap "app-settings" não existe no cluster
```

- [ ] **Step 8: Criar debugging/01-wrong-ref/solution.yaml**

```yaml
# Solução: Pod com referência correta ao ConfigMap
# Fix 1: criar o ConfigMap "app-settings" ou
# Fix 2: corrigir o nome para um ConfigMap que existe
# Este arquivo cria o ConfigMap e depois o Pod que o referencia corretamente
#
# Aplique: kubectl apply -f solution.yaml
# Confirme: kubectl get pod config-ok-pod
# Verifique env: kubectl exec config-ok-pod -- env | grep APP

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-settings            # agora o ConfigMap existe
data:
  APP_ENV: "dev"
  APP_VERSION: "1.0"

---
apiVersion: v1
kind: Pod
metadata:
  name: config-ok-pod
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "env && sleep 3600"]
      envFrom:
        - configMapRef:
            name: app-settings  # correto: ConfigMap existe
```

- [ ] **Step 9: Verificar com dry-run e commit**

```bash
kubectl apply --dry-run=client -f phases/04-config-secrets/yaml/configmap.yaml
kubectl apply --dry-run=client -f phases/04-config-secrets/yaml/secret.yaml
kubectl apply --dry-run=client -f phases/04-config-secrets/yaml/pod-with-config.yaml
kubectl apply --dry-run=client -f phases/04-config-secrets/debugging/01-wrong-ref/broken.yaml
kubectl apply --dry-run=client -f phases/04-config-secrets/debugging/01-wrong-ref/solution.yaml
git add phases/04-config-secrets/
git commit -m "feat: fase 04 - configuracao e segredos"
```

---

## Task 5: Fase 05 — Storage

**Files:**
- Create: `phases/05-storage/README.md`
- Create: `phases/05-storage/yaml/pvc.yaml`
- Create: `phases/05-storage/yaml/pod-with-pvc.yaml`
- Create: `phases/05-storage/labs/lab.md`
- Create: `phases/05-storage/debugging/README.md`
- Create: `phases/05-storage/debugging/01-pvc-pending/broken.yaml`
- Create: `phases/05-storage/debugging/01-pvc-pending/solution.yaml`

- [ ] **Step 1: Criar phases/05-storage/README.md**

```markdown
# Fase 05 — Storage

**Camada:** armazenamento | **Estimativa:** ~2 semanas

## O que é cada recurso

| Recurso | O que é |
|---|---|
| `PersistentVolume (PV)` | Disco provisionado pelo admin ou dinamicamente; existe independente de Pods |
| `PersistentVolumeClaim (PVC)` | Requisição de storage feita pelo usuário; vinculada a um PV |
| `StorageClass` | Define como PVs são provisionados dinamicamente (qual provisioner usar) |
| `emptyDir` | Volume temporário que existe enquanto o Pod existir; compartilhado entre containers |
| `hostPath` | Monta diretório do nó no Pod; funciona em minikube, perigoso em produção |

## Ciclo de vida de um PVC

```
PVC criado → StorageClass detecta → provisioner cria PV → PVC vincula ao PV → Pod usa o PVC
```

Se a StorageClass não existir ou não tiver provisioner, o PVC fica em `Pending` para sempre.

## Modos de acesso

| AccessMode | Significado |
|---|---|
| `ReadWriteOnce` (RWO) | Montado em um nó por vez (leitura e escrita) |
| `ReadOnlyMany` (ROX) | Montado em múltiplos nós (somente leitura) |
| `ReadWriteMany` (RWX) | Montado em múltiplos nós (leitura e escrita) — requer NFS/CSI |

## Comandos essenciais

```bash
kubectl get pvc                              # lista PVCs e seu status
kubectl get pv                               # lista PersistentVolumes
kubectl describe pvc <nome>                  # mostra o PV vinculado e eventos
kubectl get storageclass                     # lista StorageClasses disponíveis
```

## Pronto quando

- [ ] Criar PVC, vinculá-lo a um Pod e confirmar persistência de dados após delete do Pod
- [ ] Resolver o cenário 01-pvc-pending sem ajuda
- [ ] Explicar a diferença entre provisionamento estático e dinâmico
```

- [ ] **Step 2: Criar yaml/pvc.yaml**

```yaml
# YAML de Referência — PersistentVolumeClaim
# Aplique: kubectl apply -f pvc.yaml
# Verifique: kubectl get pvc dados-pvc
# Status Bound = vinculado a um PV; Pending = aguardando provisioner

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dados-pvc
spec:
  accessModes:
    - ReadWriteOnce             # montado em um nó por vez
  storageClassName: standard    # StorageClass padrão do minikube
  resources:
    requests:
      storage: 1Gi
```

- [ ] **Step 3: Criar yaml/pod-with-pvc.yaml**

```yaml
# YAML de Referência — Pod com PVC
# Pré-requisito: aplicar pvc.yaml
# Aplique: kubectl apply -f pod-with-pvc.yaml
# Teste persistência: escreva em /dados, delete o Pod, recrie e leia de volta

apiVersion: v1
kind: Pod
metadata:
  name: app-com-storage
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "echo 'dados persistidos' > /dados/arquivo.txt && sleep 3600"]
      volumeMounts:
        - name: storage
          mountPath: /dados
    - name: sidecar              # segundo container no mesmo Pod compartilha o volume
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: storage
          mountPath: /dados-readonly
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: dados-pvc    # referencia o PVC criado acima
```

- [ ] **Step 4: Criar labs/lab.md**

```markdown
# Lab 05 — Persistência de Dados com PVC

## Objetivo

Confirmar que dados sobrevivem à destruição e recriação de um Pod.

## Passos

### Parte 1: Criar PVC e Pod

```bash
kubectl apply -f ../yaml/pvc.yaml
kubectl get pvc dados-pvc   # aguardar status Bound
kubectl apply -f ../yaml/pod-with-pvc.yaml
kubectl get pod app-com-storage -w
```

### Parte 2: Escrever dados

```bash
kubectl exec app-com-storage -c app -- sh -c "echo 'dado-$(date)' >> /dados/log.txt"
kubectl exec app-com-storage -c app -- cat /dados/log.txt
```

### Parte 3: Destruir e recriar o Pod

```bash
kubectl delete pod app-com-storage
# O PVC continua existindo
kubectl get pvc dados-pvc
kubectl apply -f ../yaml/pod-with-pvc.yaml
kubectl get pod app-com-storage -w
```

### Parte 4: Confirmar que dados persistiram

```bash
kubectl exec app-com-storage -c app -- cat /dados/log.txt
```

O arquivo deve conter os dados escritos antes do delete.

### Parte 5: Limpeza

```bash
kubectl delete pod app-com-storage
kubectl delete pvc dados-pvc
```

## Pergunta para reflexão

O que acontece com o PV quando o PVC é deletado? Depende do `reclaimPolicy` da StorageClass.
Verifique com: `kubectl get storageclass standard -o yaml | grep reclaimPolicy`
```

- [ ] **Step 5: Criar debugging/README.md**

```markdown
# Debugging — Fase 05

## Cenários

### 01-pvc-pending — PVC que não vincula

O PVC fica em estado Pending e o Pod não consegue ser criado.

Dica de investigação:
```bash
kubectl get pvc pvc-quebrado
kubectl describe pvc pvc-quebrado   # olhe os eventos: "no matching PVs"
kubectl get storageclass             # verifique quais StorageClasses existem no cluster
```
```

- [ ] **Step 6: Criar debugging/01-pvc-pending/broken.yaml**

```yaml
# Cenário: PVC em estado Pending
# Problema: storageClassName aponta para "premium-ssd" que não existe no cluster
# O provisioner não existe e o PVC nunca vira Bound
#
# Aplique: kubectl apply -f broken.yaml
# Observe: kubectl get pvc pvc-quebrado
# Investigue: kubectl describe pvc pvc-quebrado
#             kubectl get storageclass

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-quebrado
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: premium-ssd   # não existe no minikube
  resources:
    requests:
      storage: 1Gi
```

- [ ] **Step 7: Criar debugging/01-pvc-pending/solution.yaml**

```yaml
# Solução: PVC em estado Pending
# Fix: usar "standard" que é a StorageClass provisionada pelo minikube
#
# Verifique StorageClasses disponíveis: kubectl get storageclass
# Aplique: kubectl apply -f solution.yaml
# Confirme: kubectl get pvc pvc-corrigido  →  status deve ser Bound

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-corrigido
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard      # StorageClass que existe no cluster
  resources:
    requests:
      storage: 1Gi
```

- [ ] **Step 8: Verificar e commit**

```bash
kubectl apply --dry-run=client -f phases/05-storage/yaml/pvc.yaml
kubectl apply --dry-run=client -f phases/05-storage/yaml/pod-with-pvc.yaml
kubectl apply --dry-run=client -f phases/05-storage/debugging/01-pvc-pending/broken.yaml
kubectl apply --dry-run=client -f phases/05-storage/debugging/01-pvc-pending/solution.yaml
git add phases/05-storage/
git commit -m "feat: fase 05 - storage"
```

---

## Task 6: Fase 06 — Controle de Acesso (RBAC)

**Files:**
- Create: `phases/06-rbac/README.md`
- Create: `phases/06-rbac/yaml/serviceaccount.yaml`
- Create: `phases/06-rbac/yaml/role-rolebinding.yaml`
- Create: `phases/06-rbac/labs/lab.md`
- Create: `phases/06-rbac/debugging/README.md`
- Create: `phases/06-rbac/debugging/01-403-forbidden/broken.yaml`
- Create: `phases/06-rbac/debugging/01-403-forbidden/solution.yaml`

- [ ] **Step 1: Criar phases/06-rbac/README.md**

```markdown
# Fase 06 — Controle de Acesso (RBAC)

**Camada:** segurança | **Estimativa:** ~2 semanas

## O que é cada recurso

| Recurso | O que é |
|---|---|
| `ServiceAccount` | Identidade de um Pod dentro do cluster; como o Pod se autentica na kube-apiserver |
| `Role` | Conjunto de permissões (verbs + resources) dentro de um Namespace específico |
| `ClusterRole` | Conjunto de permissões em todo o cluster (todos os namespaces) |
| `RoleBinding` | Vincula um Role a um Subject (SA, User, Group) em um Namespace |
| `ClusterRoleBinding` | Vincula um ClusterRole a um Subject globalmente |

## Como o RBAC funciona

```
Pod → usa → ServiceAccount → vinculada por → RoleBinding → ao → Role (permissões)
```

Todo Pod usa uma ServiceAccount. Se não especificada, usa a `default` do namespace,
que por padrão não tem permissões para acessar a API do cluster.

## Verbs disponíveis

`get`, `list`, `watch`, `create`, `update`, `patch`, `delete`, `deletecollection`

## Como investigar permissões

```bash
kubectl auth can-i get pods --as=system:serviceaccount:default:minha-sa
kubectl auth can-i create deployments --as=system:serviceaccount:default:minha-sa
kubectl get rolebindings -n <namespace>
kubectl describe rolebinding <nome>
```

## Pronto quando

- [ ] Criar SA com permissão mínima e verificar que não pode criar recursos
- [ ] Resolver o cenário 01-403-forbidden sem ajuda
- [ ] Usar `kubectl auth can-i` para investigar permissões de uma SA
```

- [ ] **Step 2: Criar yaml/serviceaccount.yaml**

```yaml
# YAML de Referência — ServiceAccount
# Aplique: kubectl apply -f serviceaccount.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: leitor-sa
  namespace: default
automountServiceAccountToken: true   # monta token JWT automaticamente em /var/run/secrets/kubernetes.io/serviceaccount/
```

- [ ] **Step 3: Criar yaml/role-rolebinding.yaml**

```yaml
# YAML de Referência — Role + RoleBinding
# Role: permissões dentro de um namespace
# RoleBinding: vincula o Role à ServiceAccount "leitor-sa"
#
# Aplique: kubectl apply -f role-rolebinding.yaml
# Teste: kubectl auth can-i list pods --as=system:serviceaccount:default:leitor-sa

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: leitor-pods
  namespace: default
rules:
  - apiGroups: [""]             # "" = core API group (Pod, Service, ConfigMap, etc.)
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]   # leitura apenas; sem create/delete/patch

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-leitor-sa
  namespace: default
subjects:
  - kind: ServiceAccount
    name: leitor-sa             # deve existir no namespace abaixo
    namespace: default
roleRef:
  kind: Role
  name: leitor-pods             # Role criada acima
  apiGroup: rbac.authorization.k8s.io
```

- [ ] **Step 4: Criar labs/lab.md**

```markdown
# Lab 06 — ServiceAccount com Permissão Mínima

## Objetivo

Criar uma SA com permissão de leitura apenas e confirmar que não consegue criar recursos.

## Passos

### Parte 1: Criar SA, Role e RoleBinding

```bash
kubectl apply -f ../yaml/serviceaccount.yaml
kubectl apply -f ../yaml/role-rolebinding.yaml
```

### Parte 2: Verificar permissões

```bash
# Pode listar pods?
kubectl auth can-i list pods --as=system:serviceaccount:default:leitor-sa
# Esperado: yes

# Pode criar pods?
kubectl auth can-i create pods --as=system:serviceaccount:default:leitor-sa
# Esperado: no

# Pode deletar deployments?
kubectl auth can-i delete deployments --as=system:serviceaccount:default:leitor-sa
# Esperado: no
```

### Parte 3: Rodar Pod com a SA

```bash
kubectl run pod-leitor \
  --image=bitnami/kubectl:latest \
  --serviceaccount=leitor-sa \
  --restart=Never \
  -- sleep 3600

kubectl exec -it pod-leitor -- sh

# Dentro do Pod — ações permitidas:
kubectl get pods          # funciona (tem permissão list)

# Dentro do Pod — ações proibidas:
kubectl create deployment teste --image=nginx   # deve retornar 403 Forbidden
exit
```

### Parte 4: Limpeza

```bash
kubectl delete pod pod-leitor
kubectl delete rolebinding bind-leitor-sa
kubectl delete role leitor-pods
kubectl delete serviceaccount leitor-sa
```
```

- [ ] **Step 5: Criar debugging/README.md**

```markdown
# Debugging — Fase 06

## Cenários

### 01-403-forbidden — Pod recebe 403 ao tentar listar recursos

Um Pod que precisa listar ConfigMaps recebe 403 Forbidden.

Dica de investigação:
```bash
kubectl describe pod forbidden-pod            # veja qual SA está sendo usada
kubectl get rolebindings                       # existe algum RoleBinding para essa SA?
kubectl auth can-i list configmaps \
  --as=system:serviceaccount:default:app-sa   # teste direto
```
```

- [ ] **Step 6: Criar debugging/01-403-forbidden/broken.yaml**

```yaml
# Cenário: Pod recebe 403 ao tentar listar ConfigMaps
# Problema: o Role "app-role" não inclui ConfigMaps nas permissões
# O Pod tenta listar configmaps mas recebe 403 Forbidden
#
# Aplique: kubectl apply -f broken.yaml
# Veja o erro: kubectl logs forbidden-pod

apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: default

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]         # apenas pods; configmaps está faltando
    verbs: ["get", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: default
subjects:
  - kind: ServiceAccount
    name: app-sa
    namespace: default
roleRef:
  kind: Role
  name: app-role
  apiGroup: rbac.authorization.k8s.io

---
apiVersion: v1
kind: Pod
metadata:
  name: forbidden-pod
spec:
  serviceAccountName: app-sa
  containers:
    - name: app
      image: bitnami/kubectl:latest
      command: ["sh", "-c", "kubectl get configmaps && sleep 3600"]
```

- [ ] **Step 7: Criar debugging/01-403-forbidden/solution.yaml**

```yaml
# Solução: 403 Forbidden ao listar ConfigMaps
# Fix: adicionar "configmaps" à lista de resources no Role
#
# Aplique: kubectl apply -f solution.yaml
# Confirme: kubectl logs allowed-pod  →  deve listar os ConfigMaps do namespace

apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa-fixed
  namespace: default

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role-fixed
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods", "configmaps"]   # agora inclui configmaps
    verbs: ["get", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding-fixed
  namespace: default
subjects:
  - kind: ServiceAccount
    name: app-sa-fixed
    namespace: default
roleRef:
  kind: Role
  name: app-role-fixed
  apiGroup: rbac.authorization.k8s.io

---
apiVersion: v1
kind: Pod
metadata:
  name: allowed-pod
spec:
  serviceAccountName: app-sa-fixed
  containers:
    - name: app
      image: bitnami/kubectl:latest
      command: ["sh", "-c", "kubectl get configmaps && sleep 3600"]
```

- [ ] **Step 8: Verificar e commit**

```bash
kubectl apply --dry-run=client -f phases/06-rbac/yaml/serviceaccount.yaml
kubectl apply --dry-run=client -f phases/06-rbac/yaml/role-rolebinding.yaml
kubectl apply --dry-run=client -f phases/06-rbac/debugging/01-403-forbidden/broken.yaml
kubectl apply --dry-run=client -f phases/06-rbac/debugging/01-403-forbidden/solution.yaml
git add phases/06-rbac/
git commit -m "feat: fase 06 - rbac"
```

---

## Task 7: Fase 07 — Scheduling & Recursos

**Files:**
- Create: `phases/07-scheduling/README.md`
- Create: `phases/07-scheduling/yaml/limitrange.yaml`
- Create: `phases/07-scheduling/yaml/resourcequota.yaml`
- Create: `phases/07-scheduling/yaml/taint-toleration.yaml`
- Create: `phases/07-scheduling/yaml/affinity.yaml`
- Create: `phases/07-scheduling/yaml/hpa.yaml`
- Create: `phases/07-scheduling/labs/lab.md`
- Create: `phases/07-scheduling/debugging/README.md`
- Create: `phases/07-scheduling/debugging/01-oomkilled/broken.yaml`
- Create: `phases/07-scheduling/debugging/01-oomkilled/solution.yaml`
- Create: `phases/07-scheduling/debugging/02-pending-taint/broken.yaml`
- Create: `phases/07-scheduling/debugging/02-pending-taint/solution.yaml`

- [ ] **Step 1: Criar phases/07-scheduling/README.md**

```markdown
# Fase 07 — Scheduling & Recursos

**Camada:** scheduler | **Estimativa:** ~3 semanas
**Ambiente recomendado:** kind multi-nó (`setup/kind.md`) para testar taints e affinity

## O que é cada recurso

| Recurso | O que é |
|---|---|
| `requests/limits` | `requests`: mínimo que o scheduler reserva; `limits`: teto que o container pode usar |
| `LimitRange` | Define requests/limits padrão para containers sem valores definidos num Namespace |
| `ResourceQuota` | Teto total de recursos (CPU, memória, número de objetos) em um Namespace |
| `taint/toleration` | Taint repele Pods de um nó; toleration é a "chave" que permite ao Pod ignorar o taint |
| `nodeSelector` | Agenda o Pod apenas em nós com determinado label |
| `affinity` | Regras mais expressivas que nodeSelector: preferência ou obrigação de co-localização |
| `HPA` | Escala Deployment automaticamente baseado em métricas (requer metrics-server) |

## Como o scheduler decide onde colocar um Pod

1. **Filtering:** elimina nós que não satisfazem os requisitos (resources, taints, nodeSelector)
2. **Scoring:** ranqueia os nós restantes por critérios como recursos disponíveis e affinity
3. O nó com maior score recebe o Pod

## OOMKilled vs throttling

- **OOMKilled:** container usou mais memória que o `limit` → kernel mata o processo → Pod reinicia
- **CPU throttling:** container quer mais CPU que o `limit` → kernel reduz (não mata); o processo fica lento

## Comandos essenciais

```bash
kubectl describe node <nome>           # recursos alocados vs disponíveis no nó
kubectl top nodes                      # uso real de CPU/memória (requer metrics-server)
kubectl top pods                       # uso real por Pod
kubectl taint nodes <nome> key=val:NoSchedule   # adiciona taint no nó
kubectl taint nodes <nome> key=val:NoSchedule-  # remove taint (note o -)
```

## Pronto quando

- [ ] Aplicar LimitRange e verificar que defaults são injetados em Pods sem requests/limits
- [ ] Adicionar taint em um nó e confirmar Pod em Pending; adicionar toleration e confirmar agendamento
- [ ] Resolver cenário 01-oomkilled sem ajuda
- [ ] Resolver cenário 02-pending-taint sem ajuda
```

- [ ] **Step 2: Criar yaml/limitrange.yaml**

```yaml
# YAML de Referência — LimitRange
# Aplique em um namespace: kubectl apply -f limitrange.yaml -n dev
# Teste: crie um Pod sem resources e veja os defaults injetados
#        kubectl describe pod <nome> | grep -A5 Limits

apiVersion: v1
kind: LimitRange
metadata:
  name: limites-padrao
  namespace: default
spec:
  limits:
    - type: Container
      default:                  # limits padrão (se container não definir limits)
        cpu: "500m"
        memory: "256Mi"
      defaultRequest:           # requests padrão (se container não definir requests)
        cpu: "100m"
        memory: "128Mi"
      max:                      # teto máximo permitido no namespace
        cpu: "2"
        memory: "1Gi"
      min:                      # mínimo obrigatório no namespace
        cpu: "50m"
        memory: "64Mi"
```

- [ ] **Step 3: Criar yaml/resourcequota.yaml**

```yaml
# YAML de Referência — ResourceQuota
# Define o teto total de recursos que podem ser usados em um Namespace
# Aplique: kubectl apply -f resourcequota.yaml
# Inspecione: kubectl describe resourcequota quota-dev

apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-dev
  namespace: default
spec:
  hard:
    requests.cpu: "2"           # total de CPU requests no namespace
    requests.memory: "2Gi"      # total de memória requests
    limits.cpu: "4"             # total de CPU limits
    limits.memory: "4Gi"
    pods: "20"                  # número máximo de Pods
    configmaps: "10"
    persistentvolumeclaims: "5"
```

- [ ] **Step 4: Criar yaml/taint-toleration.yaml**

```yaml
# YAML de Referência — Taint e Toleration
#
# Para adicionar o taint ao nó antes de aplicar este YAML:
#   kubectl taint nodes <nome-do-no> dedicated=gpu:NoSchedule
#
# Para ver taints dos nós:
#   kubectl describe nodes | grep Taints
#
# Este Pod tem a toleration correta e pode ser agendado no nó com taint

apiVersion: v1
kind: Pod
metadata:
  name: pod-com-toleration
spec:
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "gpu"
      effect: "NoSchedule"      # NoSchedule: Pods sem toleration não são agendados no nó
                                # PreferNoSchedule: preferência, mas não obrigatório
                                # NoExecute: expulsa Pods já rodando sem toleration
  nodeSelector:
    dedicated: gpu              # vai apenas para nós com este label (opcional)
  containers:
    - name: app
      image: nginx:1.25
```

- [ ] **Step 5: Criar yaml/affinity.yaml**

```yaml
# YAML de Referência — Node Affinity e Pod Anti-Affinity
# Node affinity: como nodeSelector mas com operadores (In, NotIn, Exists, etc.)
# Pod anti-affinity: evita que réplicas fiquem no mesmo nó

apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-com-affinity
spec:
  replicas: 2
  selector:
    matchLabels:
      app: affinidade
  template:
    metadata:
      labels:
        app: affinidade
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:  # obrigatório
            nodeSelectorTerms:
              - matchExpressions:
                  - key: kubernetes.io/os
                    operator: In
                    values: ["linux"]
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:  # preferência (não obrigatório)
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app: affinidade
                topologyKey: kubernetes.io/hostname    # prefere nós diferentes
      containers:
        - name: app
          image: nginx:1.25
```

- [ ] **Step 6: Criar yaml/hpa.yaml**

```yaml
# YAML de Referência — HorizontalPodAutoscaler
# Pré-requisito: metrics-server instalado
#   minikube addons enable metrics-server
#
# Aplique: kubectl apply -f hpa.yaml
# Monitore: kubectl get hpa app-hpa -w

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment        # Deployment da Fase 02
  minReplicas: 1
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50  # escala quando CPU média ultrapassa 50%
```

- [ ] **Step 7: Criar labs/lab.md**

```markdown
# Lab 07 — LimitRange e Taint/Toleration

## Objetivo

Verificar injeção de defaults pelo LimitRange e testar o comportamento de taints.

## Parte 1: LimitRange

```bash
kubectl apply -f ../yaml/limitrange.yaml

# Criar Pod sem resources
kubectl run sem-limits --image=nginx:1.25 --restart=Never

# Ver os defaults injetados pelo LimitRange
kubectl describe pod sem-limits | grep -A6 "Limits:"
# Deve mostrar: cpu: 500m, memory: 256Mi (os defaults do LimitRange)

kubectl delete pod sem-limits
```

## Parte 2: Taint e Toleration (requer kind multi-nó)

```bash
# Listar nós
kubectl get nodes

# Adicionar taint no worker
kubectl taint nodes k8s-study-worker dedicated=special:NoSchedule

# Criar Pod sem toleration (deve ficar Pending)
kubectl run sem-toleration --image=nginx:1.25 --restart=Never
kubectl get pod sem-toleration -w
# Status: Pending (nenhum nó aceita sem toleration)

# Remover o Pod
kubectl delete pod sem-toleration

# Criar Pod com toleration
kubectl apply -f ../yaml/taint-toleration.yaml
kubectl get pod pod-com-toleration -w
# Deve ser agendado no nó com taint

# Limpar
kubectl delete pod pod-com-toleration
kubectl taint nodes k8s-study-worker dedicated=special:NoSchedule-
```
```

- [ ] **Step 8: Criar debugging/README.md**

```markdown
# Debugging — Fase 07

## Cenários

### 01-oomkilled — Container morto por OOM

O container é reiniciado com status OOMKilled.

Dica de investigação:
```bash
kubectl get pod oom-pod
kubectl describe pod oom-pod   # olhe "Last State" e "Reason: OOMKilled"
```

### 02-pending-taint — Pod Pending por taint sem toleration

O Pod fica em Pending e não consegue ser agendado.

Dica de investigação:
```bash
kubectl describe pod pending-pod   # olhe os Events: "0/1 nodes are available: 1 node(s) had untolerated taint"
kubectl describe nodes | grep Taints
```
```

- [ ] **Step 9: Criar debugging/01-oomkilled/broken.yaml**

```yaml
# Cenário: OOMKilled
# Problema: memory limit de 5Mi é insuficiente para o nginx iniciar
# O container é morto pelo kernel e reinicia em loop (CrashLoopBackOff)
#
# Aplique: kubectl apply -f broken.yaml
# Observe: kubectl get pod oom-pod -w
# Investigue: kubectl describe pod oom-pod  (procure "OOMKilled" em Last State)

apiVersion: v1
kind: Pod
metadata:
  name: oom-pod
spec:
  containers:
    - name: app
      image: nginx:1.25
      resources:
        requests:
          memory: "5Mi"
        limits:
          memory: "5Mi"         # nginx precisa de mais memória para iniciar
```

- [ ] **Step 10: Criar debugging/01-oomkilled/solution.yaml**

```yaml
# Solução: OOMKilled
# Fix: aumentar memory limit para um valor que o nginx consiga usar
#
# Aplique: kubectl apply -f solution.yaml
# Confirme: kubectl get pod oom-pod-fixed

apiVersion: v1
kind: Pod
metadata:
  name: oom-pod-fixed
spec:
  containers:
    - name: app
      image: nginx:1.25
      resources:
        requests:
          memory: "64Mi"
        limits:
          memory: "128Mi"       # suficiente para nginx
```

- [ ] **Step 11: Criar debugging/02-pending-taint/broken.yaml**

```yaml
# Cenário: Pod Pending por taint sem toleration
# Pré-requisito: adicionar taint no nó worker
#   kubectl taint nodes k8s-study-worker env=prod:NoSchedule
#
# Problema: Pod não tem toleration para o taint "env=prod:NoSchedule"
# Resultado: Pod fica em Pending indefinidamente
#
# Aplique: kubectl apply -f broken.yaml
# Observe: kubectl get pod pending-pod -w
# Investigue: kubectl describe pod pending-pod

apiVersion: v1
kind: Pod
metadata:
  name: pending-pod
spec:
  containers:
    - name: app
      image: nginx:1.25
  # sem tolerations: o scheduler rejeita o nó com taint
```

- [ ] **Step 12: Criar debugging/02-pending-taint/solution.yaml**

```yaml
# Solução: Pod Pending por taint sem toleration
# Fix: adicionar toleration correspondente ao taint do nó
#
# Aplique: kubectl apply -f solution.yaml
# Confirme: kubectl get pod pending-pod-fixed

apiVersion: v1
kind: Pod
metadata:
  name: pending-pod-fixed
spec:
  tolerations:
    - key: "env"
      operator: "Equal"
      value: "prod"
      effect: "NoSchedule"     # toleration corresponde ao taint adicionado no nó
  containers:
    - name: app
      image: nginx:1.25
```

- [ ] **Step 13: Verificar e commit**

```bash
kubectl apply --dry-run=client -f phases/07-scheduling/yaml/limitrange.yaml
kubectl apply --dry-run=client -f phases/07-scheduling/yaml/resourcequota.yaml
kubectl apply --dry-run=client -f phases/07-scheduling/yaml/taint-toleration.yaml
kubectl apply --dry-run=client -f phases/07-scheduling/yaml/affinity.yaml
kubectl apply --dry-run=client -f phases/07-scheduling/yaml/hpa.yaml
kubectl apply --dry-run=client -f phases/07-scheduling/debugging/01-oomkilled/broken.yaml
kubectl apply --dry-run=client -f phases/07-scheduling/debugging/01-oomkilled/solution.yaml
kubectl apply --dry-run=client -f phases/07-scheduling/debugging/02-pending-taint/broken.yaml
kubectl apply --dry-run=client -f phases/07-scheduling/debugging/02-pending-taint/solution.yaml
git add phases/07-scheduling/
git commit -m "feat: fase 07 - scheduling e recursos"
```

---

## Task 8: Fase 08 — Internals do Control Plane

**Files:**
- Create: `phases/08-control-plane/README.md`
- Create: `phases/08-control-plane/labs/lab.md`
- Create: `phases/08-control-plane/debugging/README.md`

- [ ] **Step 1: Criar phases/08-control-plane/README.md**

```markdown
# Fase 08 — Internals do Control Plane

**Camada:** control plane | **Estimativa:** ~3 semanas
**Ambiente recomendado:** kind multi-nó (`setup/kind.md`)

## O que faz cada componente

| Componente | O que faz |
|---|---|
| `kube-apiserver` | Porta de entrada de toda comunicação; valida objetos e persiste no etcd; expõe a REST API |
| `etcd` | Banco de dados chave-valor distribuído; única fonte de verdade do estado do cluster |
| `kube-scheduler` | Observa Pods sem nó atribuído (`nodeName: ""`); decide onde colocá-los |
| `kube-controller-manager` | Executa todos os controllers em loop: Deployment, ReplicaSet, Node, Job, etc. |
| `kubelet` | Agente em cada nó; recebe Pods atribuídos e instrui o container runtime a criá-los |
| `kube-proxy` | Mantém regras de iptables/IPVS em cada nó para rotear tráfego dos Services |

## Fluxo completo: do kubectl apply ao Pod Running

```
1. kubectl apply -f pod.yaml
   └─ envia HTTP POST para kube-apiserver

2. kube-apiserver
   ├─ autentica (certificado TLS / ServiceAccount token)
   ├─ autoriza (RBAC)
   ├─ valida o objeto (admission controllers)
   └─ persiste no etcd

3. kube-scheduler
   ├─ watch etcd via apiserver: detecta Pod com nodeName=""
   ├─ filtra nós (resources, taints, affinity)
   ├─ pontua nós restantes
   └─ escreve nodeName no Pod via apiserver → etcd

4. kubelet (no nó escolhido)
   ├─ watch apiserver: detecta Pod atribuído ao seu nó
   ├─ instrui container runtime (containerd) a baixar a imagem
   ├─ cria o container
   ├─ executa livenessProbe/readinessProbe
   └─ atualiza status do Pod no apiserver → etcd

5. kube-proxy (em todos os nós)
   └─ watch apiserver: atualiza iptables quando Endpoints de Services mudam
```

## Onde ficam os componentes no minikube/kind

No minikube e no kind, os componentes do control plane rodam como **static Pods**
gerenciados diretamente pelo kubelet do nó control-plane:

```bash
# Ver os Pods do control plane:
kubectl get pods -n kube-system
# kube-apiserver-minikube
# kube-controller-manager-minikube
# kube-scheduler-minikube
# etcd-minikube

# Ver os manifestos estáticos (no nó, não no cluster):
# /etc/kubernetes/manifests/kube-apiserver.yaml
# /etc/kubernetes/manifests/etcd.yaml
```

## Comandos de diagnóstico do control plane

```bash
# Saúde dos componentes
kubectl get componentstatuses

# Logs dos componentes (como static Pods)
kubectl logs -n kube-system kube-apiserver-minikube
kubectl logs -n kube-system kube-scheduler-minikube
kubectl logs -n kube-system kube-controller-manager-minikube
kubectl logs -n kube-system etcd-minikube

# Estado dos nós
kubectl get nodes
kubectl describe node <nome>     # veja "Conditions" e "Events"

# kubelet (no nó — acesse via docker exec em kind)
docker exec -it k8s-study-worker journalctl -u kubelet -f

# Inspecionar etcd (via container em kind)
docker exec -it k8s-study-control-plane etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health
```

## Pronto quando

- [ ] Listar todos os componentes do control plane e seus status
- [ ] Ver os logs de cada componente sem consultar documentação
- [ ] Simular um nó NotReady (parar o kubelet) e diagnosticar a causa
- [ ] Explicar o fluxo completo de criação de um Pod com suas próprias palavras
```

- [ ] **Step 2: Criar labs/lab.md**

```markdown
# Lab 08 — Inspecionando o Control Plane

## Objetivo

Localizar, inspecionar e entender cada componente do control plane.

## Parte 1: Listar componentes

```bash
kubectl get pods -n kube-system
kubectl get componentstatuses
```

Identifique: qual Pod é o apiserver? O scheduler? O controller-manager? O etcd?

## Parte 2: Logs dos componentes

```bash
# Substitua "minikube" pelo nome do nó se usar kind (k8s-study-control-plane)
kubectl logs -n kube-system kube-apiserver-minikube --tail=20
kubectl logs -n kube-system kube-scheduler-minikube --tail=20
kubectl logs -n kube-system kube-controller-manager-minikube --tail=20
```

## Parte 3: Acompanhar o fluxo de criação de um Pod

Em um terminal, inicie o watch dos eventos:
```bash
kubectl get events --sort-by=.lastTimestamp -w
```

Em outro terminal, crie um Pod:
```bash
kubectl run teste-fluxo --image=nginx:1.25 --restart=Never
```

Observe os eventos em ordem:
1. `Scheduled` — scheduler atribuiu o Pod ao nó
2. `Pulling` — kubelet puxando a imagem
3. `Pulled` — imagem pronta
4. `Created` — container criado
5. `Started` — container iniciado

## Parte 4: Simular nó NotReady (kind multi-nó)

```bash
# Pausar o container do worker (simula falha do kubelet)
docker pause k8s-study-worker

# Em outro terminal, observar o nó mudar para NotReady
kubectl get nodes -w
# aguardar ~40s: k8s-study-worker   NotReady

# Retomar o nó
docker unpause k8s-study-worker
kubectl get nodes -w
# nó volta para Ready
```

## Parte 5: Localizar manifestos estáticos (minikube)

```bash
minikube ssh
ls /etc/kubernetes/manifests/
# kube-apiserver.yaml
# kube-controller-manager.yaml
# kube-scheduler.yaml
# etcd.yaml
cat /etc/kubernetes/manifests/kube-apiserver.yaml | head -50
exit
```
```

- [ ] **Step 3: Criar debugging/README.md**

```markdown
# Debugging — Fase 08

Esta fase não tem YAMLs quebrados para aplicar — os cenários de debugging
são simulados no próprio cluster.

## Cenários

### Cenário 1: Nó NotReady

**Simulação (kind):**
```bash
docker pause k8s-study-worker
kubectl get nodes -w
```

**Investigação:**
```bash
kubectl describe node k8s-study-worker
# Olhe a seção "Conditions":
# - Ready: False (kubelet parou de reportar)
# - MemoryPressure: Unknown
# - DiskPressure: Unknown

# O que acontece com os Pods no nó?
kubectl get pods -o wide   # Pods ficam em Unknown ou são rescheduled após ~5min
```

**Resolução:**
```bash
docker unpause k8s-study-worker
kubectl get nodes -w   # deve voltar para Ready
```

### Cenário 2: Pod travado em ContainerCreating

O Pod foi agendado mas o container não inicia.

**Causas comuns:**
- Container runtime parado no nó
- Imagem não pode ser baixada (sem acesso ao registry)
- PVC referenciado não existe

**Investigação:**
```bash
kubectl describe pod <nome>   # olhe Events: qual erro específico?
kubectl get events --field-selector involvedObject.name=<nome>
# Se for PVC: kubectl get pvc
# Se for imagem: docker pull <imagem> manualmente no nó
```

### Cenário 3: apiserver sem resposta

**Sintomas:**
```
The connection to the server localhost:8443 was refused
```

**Investigação (minikube):**
```bash
minikube status        # verifica estado geral
minikube logs          # logs do próprio minikube
# Se etcd está down: apiserver não consegue ler/escrever estado
```

## Critério de conclusão da fase 08 e do roadmap completo

- [ ] Você consegue listar e inspecionar todos os componentes do control plane
- [ ] Você sabe onde olhar quando um nó vai para NotReady
- [ ] Você consegue explicar o fluxo completo do `kubectl apply` ao Pod Running
- [ ] Você sabe a diferença entre o papel do scheduler, do kubelet e do controller-manager
- [ ] Você consegue diagnosticar os cenários das fases 1-7 sem consultar material externo

**Parabéns — você concluiu o roadmap de Kubernetes puro.**
```

- [ ] **Step 4: Verificar estrutura completa**

```bash
find phases/ -type f | sort
```

Esperado — pelo menos 50 arquivos organizados nas 8 fases.

- [ ] **Step 5: Commit final**

```bash
git add phases/08-control-plane/
git commit -m "feat: fase 08 - internals do control plane"
```

---

## Auto-revisão do plano

**Cobertura da spec:**

| Requisito da spec | Task que implementa |
|---|---|
| README com "O que é" por recurso | Step 1 de cada Task |
| YAML anotado por recurso | Steps 2-6 de cada Task (yaml/) |
| Lab prático por fase | Step de criação do labs/lab.md |
| Cenário debugging quebrado | Steps broken.yaml de cada Task |
| Cenário debugging solução | Steps solution.yaml de cada Task |
| Critério "pronto quando" | README.md de cada fase |
| Setup minikube e kind | Task 0 |
| Critérios globais de conclusão | Fase 08 debugging/README.md |

**Verificação de tipos e nomes:** todos os `claimName`, `configMapRef`, `secretKeyRef` e `serviceAccountName` referenciam objetos criados no mesmo arquivo ou na mesma fase. Nenhuma referência pendente identificada.

**Placeholders:** nenhum TBD, TODO ou "fill in" encontrado.
