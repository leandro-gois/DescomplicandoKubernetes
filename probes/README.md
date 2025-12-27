# As Probes do Kubernetes

### O que são as Probes?

As probes são uma forma de você monitorar o seu Pod e saber se ele está em um estado saudável ou não. Com elas é possível assegurar que seus Pods estão rodando e respondendo de maneira correta, e mais do que isso, que o Kubernetes está testando o que está sendo executado dentro do seu Pod.

Hoje nós temos disponíveis três tipos de probes, a livenessProbe, a readinessProbe e a startupProbe. Vamos ver no detalhe cada uma delas.

### Liveness Probe

A livenessProbe é a nossa probe de verificação de integridade, o que ela faz é verificar se o que está rodando dentro do Pod está saudável. O que fazemos é criar uma forma de testar se o que temos dentro do Pod está respondendo conforme esperado. Se por acaso o teste falhar, o Pod será reiniciado.

Para ficar mais claro, vamos mais uma vez utilizar o exemplo com o Nginx. Gosto de usar o Nginx como exemplo, pois sei que toda pessoa já o conhece, e assim, fica muito mais fácil de entender o que está acontecendo. Afinal, você está aqui para aprender Kubernetes, e se for com algo que você já conhece, fica muito mais fácil de entender.

Bem, vamos lá, hora de criar um novo Deployment com o Nginx, vamos utilizar o exemplo que já utilizamos quando aprendemos sobre o Deployment.

Para isso, crie um arquivo chamado nginx-liveness.yaml e cole o seguinte conteúdo.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.19.2
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
        livenessProbe: # Aqui é onde vamos adicionar a nossa livenessProbe
          tcpSocket: # Aqui vamos utilizar o tcpSocket, onde vamos se conectar ao container através do protocolo TCP
            port: 80 # Qual porta TCP vamos utilizar para se conectar ao container
          initialDelaySeconds: 10 # Quantos segundos vamos esperar para executar a primeira verificação
          periodSeconds: 10 # A cada quantos segundos vamos executar a verificação
          timeoutSeconds: 5 # Quantos segundos vamos esperar para considerar que a verificação falhou
          failureThreshold: 3 # Quantos falhas consecutivas vamos aceitar antes de reiniciar o container
```

O que declaramos com a regra acima é que queremos testar se o Pod está respondendo através do protocolo TCP, através da opção tcpSocket, na porta 80 que foi definida pela opção port. E também definimos que queremos esperar 10 segundos para executar a primeira verificação utilizando initialDelaySeconds e por conta da periodSecondsfalamos que queremos que a cada 10 segundos seja realizada a verificação. Caso a verificação falhe, vamos esperar 5 segundos, por conta da timeoutSeconds, para tentar novamente, e como utilizamos o failureThreshold, se falhar mais 3 vezes, vamos reiniciar o Pod.

Vamos imaginar que agora não queremos mais utilizar o tcpSocket, mas sim o httpGet para tentar acessar um endpoint dentro do nosso Pod.

Para isso, vamos alterar o nosso nginx-deployment.yaml para o seguinte.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.19.2
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
        livenessProbe: # Aqui é onde vamos adicionar a nossa livenessProbe
          httpGet: # Aqui vamos utilizar o httpGet, onde vamos se conectar ao container através do protocolo HTTP
            path: / # Qual o endpoint que vamos utilizar para se conectar ao container
            port: 80 # Qual porta TCP vamos utilizar para se conectar ao container
          initialDelaySeconds: 10 # Quantos segundos vamos esperar para executar a primeira verificação
          periodSeconds: 10 # A cada quantos segundos vamos executar a verificação
          timeoutSeconds: 5 # Quantos segundos vamos esperar para considerar que a verificação falhou
          failureThreshold: 3 # Quantos falhas consecutivas vamos aceitar antes de reiniciar o container
```

Perceba que agora somente mudamos algumas coisas, apesar de seguir com o mesmo objetivo, que é verificar se o Nginx está respondendo corretamente, mudamos como iremos testar isso. Agora estamos utilizando o httpGet para testar se o Nginx está respondendo corretamente através do protocolo HTTP, e para isso, estamos utilizando o endpoint / e a porta 80.

O que temos de novo aqui é a opção path, que é o endpoint que vamos utilizar para testar se o Nginx está respondendo corretamente, e claro, a httpGet é a forma como iremos realizar o nosso teste, através do protocolo HTTP.

 

Escolha qual dois dois exemplos você quer utilizar, e crie o seu Deployment através do comando abaixo.

```
kubectl apply -f nginx-deployment.yaml
```

Para verificar se o Deployment foi criado corretamente, execute o comando abaixo.

```
kubectl get deployments
```

Você deve ver algo parecido com isso.

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           103s
```

Para verificar os pods do deployment criado, use o comando:

```
kubectl get pods -l app=nginx-deployment
```

A saída deve ser parecida com essa.

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-66c9c9c54b-cd8vc   1/1     Running   0          15m
nginx-deployment-66c9c9c54b-qwmt4   1/1     Running   0          15m
nginx-deployment-66c9c9c54b-vrvdv   1/1     Running   0          15m
```

Para que você possa ver mais detalhes sobre o seu Pod e saber se a nossa probe está funcionando corretamente, vamos utilizar o comando abaixo.

```
kubectl describe pod nginx-deployment-66c9c9c54b-cd8vc
```

A saída deve ser parecida com essa.

```
Name:             nginx-deployment-66c9c9c54b-cd8vc
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-multinodes-worker2/172.18.0.2
Start Time:       Thu, 25 Dec 2025 21:49:39 -0300
Labels:           app=nginx-deployment
                  pod-template-hash=66c9c9c54b
Annotations:      <none>
Status:           Running
IP:               10.244.2.2
IPs:
  IP:           10.244.2.2
Controlled By:  ReplicaSet/nginx-deployment-66c9c9c54b
Containers:
  nginx:
    Container ID:   containerd://4554b31b69d5a7f7fd24745b52217d1edd3407460394c3472eb44f4175560b83
    Image:          nginx:1.19.2
    Image ID:       docker.io/library/nginx@sha256:c628b67d21744fce822d22fdcc0389f6bd763daac23a6b77147d0712ea7102d0
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 25 Dec 2025 21:49:40 -0300
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  256Mi
    Requests:
      cpu:        250m
      memory:     128Mi
    Liveness:     http-get http://:80/ delay=10s timeout=5s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-56jk4 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-56jk4:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  5m45s  default-scheduler  Successfully assigned default/nginx-deployment-66c9c9c54b-cd8vc to kind-multinodes-worker2
  Normal  Pulled     5m45s  kubelet            Container image "nginx:1.19.2" already present on machine
  Normal  Created    5m45s  kubelet            Created container: nginx
  Normal  Started    5m44s  kubelet            Started container nginx
  ```

Aqui temos a informação mais importante para nós nesse momento:

```
    Liveness:     http-get http://:80/ delay=10s timeout=5s period=10s #success=1 #failure=3
```

Agora vamos fazer o seguinte, vamos alterar o nosso Deployment, para que a nossa probe falhe. Para isso vamos alterar o endpoint que estamos utilizando. Vamos alterar o path para /giropops.

```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.19.2
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
        livenessProbe: # Aqui é onde vamos adicionar a nossa livenessProbe
          httpGet: # Aqui vamos utilizar o httpGet, onde vamos se conectar ao container através do protocolo HTTP
            path: /giropops # Qual o endpoint que vamos utilizar para se conectar ao container
            port: 80 # Qual porta TCP vamos utilizar para se conectar ao container
          initialDelaySeconds: 10 # Quantos segundos vamos esperar para executar a primeira verificação
          periodSeconds: 10 # A cada quantos segundos vamos executar a verificação
          timeoutSeconds: 5 # Quantos segundos vamos esperar para considerar que a verificação falhou
          failureThreshold: 3 # Quantos falhas consecutivas vamos aceitar antes de reiniciar o container
```

Vamos aplicar as alterações no nosso Deployment:

```
kubectl apply -f deployment.yaml
```

Depois de um tempo, você perceberá que o Kubernetes finalizou a atualização do nosso Deployment. Se você aguardar um pouco mais, você irá perceber que os Pods estã̀o sendo reiniciados com frequência.

Tudo isso porque a nossa livenessProbe está falhando, afinal o nosso endpoint está errado.

Podemos ver mais detalhes sobre o que está acontecendo na saída do comando kubectl describe pod:

```
kubectl describe pod nginx-deployment-698b68d97c-h2q7d
```

```
Name:             nginx-deployment-698b68d97c-h2q7d
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-multinodes-worker/172.18.0.4
Start Time:       Thu, 25 Dec 2025 22:13:58 -0300
Labels:           app=nginx-deployment
                  pod-template-hash=698b68d97c
Annotations:      <none>
Status:           Running
IP:               10.244.1.3
IPs:
  IP:           10.244.1.3
Controlled By:  ReplicaSet/nginx-deployment-698b68d97c
Containers:
  nginx:
    Container ID:   containerd://81610fae3b0eb80842f4d2602900db11c824606f0ef49cb753f2cccec3de6395
    Image:          nginx:1.19.2
    Image ID:       docker.io/library/nginx@sha256:c628b67d21744fce822d22fdcc0389f6bd763daac23a6b77147d0712ea7102d0
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 25 Dec 2025 22:34:36 -0300
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Thu, 25 Dec 2025 22:28:49 -0300
      Finished:     Thu, 25 Dec 2025 22:29:28 -0300
    Ready:          True
    Restart Count:  10
    Limits:
      cpu:     500m
      memory:  256Mi
    Requests:
      cpu:        250m
      memory:     128Mi
    Liveness:     http-get http://:80/giropos delay=10s timeout=5s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2n7rp (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-2n7rp:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  21m                   default-scheduler  Successfully assigned default/nginx-deployment-698b68d97c-h2q7d to kind-multinodes-worker
  Normal   Started    17m (x6 over 21m)     kubelet            Started container nginx
  Normal   Created    15m (x7 over 21m)     kubelet            Created container: nginx
  Warning  Unhealthy  5m35s (x30 over 20m)  kubelet            Liveness probe failed: HTTP probe failed with statuscode: 404
  Normal   Killing    5m35s (x10 over 20m)  kubelet            Container nginx failed liveness probe, will be restarted
  Warning  BackOff    42s (x70 over 17m)    kubelet            Back-off restarting failed container nginx in pod nginx-deployment-698b68d97c-h2q7d_default(e532158f-7b04-437a-acb1-3db9903c36f0)
  Normal   Pulled     27s (x11 over 21m)    kubelet            Container image "nginx:1.19.2" already present on machine
```

Na última parte da saída do comando kubectl describe pod, você pode ver que o Kubernetes está tentando executar a nossa livenessProbe e ela está falhando, inclusive ele mostra a quantidade de vezes que ele tentou executar a livenessProbe e falhou, e com isso, ele reiniciou o nosso Pod.


### Readiness Probe

A readinessProbe é uma forma de o Kubernetes verificar se o seu container está pronto para receber tráfego, se ele está pronto para receber requisições vindas de fora.

Essa é a nossa probe de leitura, ela fica verificando se o nosso container está pronto para receber requisições, e se estiver pronto, ele irá receber requisições, caso contrário, ele não irá receber requisições, pois será removido do endpoint do serviço, fazendo com que o tráfego não chegue até ele.

Para o nosso exemplo, vamos criar um arquivo chamado nginx-readiness.yaml e vamos colocar o seguinte conteúdo:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.19.2
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
        readinessProbe: # Onde definimos a nossa probe de leitura
          httpGet: # O tipo de teste que iremos executar, neste caso, iremos executar um teste HTTP
            path: / # O caminho que iremos testar
            port: 80 # A porta que iremos testar
          initialDelaySeconds: 10 # O tempo que iremos esperar para executar a primeira vez a probe
          periodSeconds: 10 # De quanto em quanto tempo iremos executar a probe
          timeoutSeconds: 5 # O tempo que iremos esperar para considerar que a probe falhou
          successThreshold: 2 # O número de vezes que a probe precisa passar para considerar que o container está pronto
          failureThreshold: 3 # O número de vezes que a probe precisa falhar para considerar que o container não está pronto
```

Vamos ver se os nossos Pods estão rodando:

```
kubectl get pods
```

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-84988d4cf9-8kszc   0/1     Running   0          8s
nginx-deployment-84988d4cf9-sp5xp   0/1     Running   0          8s
nginx-deployment-84988d4cf9-vp9n9   0/1     Running   0          8s
```

Podemos ver que agora os Pods demoram um pouco mais para ficarem prontos, pois estamos executando a nossa readinessProbe, e por esse motivo temos que aguardar os 10 segundos inicias que definimos para que seja executada a primeira vez a nossa probe, lembra?

Se você aguardar um pouco, você verá que os Pods irão ficar prontos, e você pode ver isso executando o comando:

```
kubectl get pods
```

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-84988d4cf9-8kszc   1/1     Running   0          3m32s
nginx-deployment-84988d4cf9-sp5xp   1/1     Running   0          3m32s
nginx-deployment-84988d4cf9-vp9n9   1/1     Running   0          3m32s
```

Vamos dar uma olhada no describe do nosso Pod:

```
kubectl describe pod nginx-deployment-84988d4cf9-8kszc
```

```
Name:             nginx-deployment-84988d4cf9-8kszc
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-multinodes-worker2/172.18.0.2
Start Time:       Thu, 25 Dec 2025 23:11:04 -0300
Labels:           app=nginx-deployment
                  pod-template-hash=84988d4cf9
Annotations:      <none>
Status:           Running
IP:               10.244.2.8
IPs:
  IP:           10.244.2.8
Controlled By:  ReplicaSet/nginx-deployment-84988d4cf9
Containers:
  nginx:
    Container ID:   containerd://0701c23a35296bd80112ee97af0656c28a7f3191ca984672bbd33342e2706a36
    Image:          nginx:1.19.2
    Image ID:       docker.io/library/nginx@sha256:c628b67d21744fce822d22fdcc0389f6bd763daac23a6b77147d0712ea7102d0
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 25 Dec 2025 23:11:05 -0300
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  256Mi
    Requests:
      cpu:        250m
      memory:     128Mi
    Readiness:    http-get http://:80/ delay=10s timeout=5s period=10s #success=2 #failure=3
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2mzp2 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-2mzp2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  13m   default-scheduler  Successfully assigned default/nginx-deployment-84988d4cf9-8kszc to kind-multinodes-worker2
  Normal  Pulled     13m   kubelet            Container image "nginx:1.19.2" already present on machine
  Normal  Created    13m   kubelet            Created container: nginx
  Normal  Started    13m   kubelet            Started container nginx
```

Pronto, a nossa probe está lá e funcionando, e com isso podemos garantir que os nossos Pods estão prontos para receber requisições.

Vamos mudar o nosso path para /giropops e ver o que acontece:

```
...
        readinessProbe: # Onde definimos a nossa probe de leitura
          httpGet: # O tipo de teste que iremos executar, neste caso, iremos executar um teste HTTP
            path: /giropops # O caminho que iremos testar
            port: 80 # A porta que iremos testar
          initialDelaySeconds: 10 # O tempo que iremos esperar para executar a primeira vez a probe
          periodSeconds: 10 # De quanto em quanto tempo iremos executar a probe
          timeoutSeconds: 5 # O tempo que iremos esperar para considerar que a probe falhou
          successThreshold: 2 # O número de vezes que a probe precisa passar para considerar que o container está pronto
          failureThreshold: 3 # O número de vezes que a probe precisa falhar para considerar que o container não está pronto
```

```
kubectl apply -f nginx-deployment.yaml
```

```
kubectl get pods
```

Nesse ponto você pode ver que o Kubernetes está tentando realizar a atualização do nosso Deployment, mas não está conseguindo, pois no primeiro Pod que ele tentou atualizar, a probe falhou.

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5c459b47db-7m792   0/1     Running   0          44s
nginx-deployment-84988d4cf9-8kszc   1/1     Running   0          18m
nginx-deployment-84988d4cf9-sp5xp   1/1     Running   0          18m
nginx-deployment-84988d4cf9-vp9n9   1/1     Running   0          18m
```

Vamos ver o nosso rollout:

```
kubectl rollout status deployment/nginx-deployment
```

```
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
```

Podemos ver os detalhes do Pod que está com problema:

```
kubectl describe pod nginx-deployment-5c459b47db-7m792
```

```
Name:             nginx-deployment-5c459b47db-7m792
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-multinodes-worker/172.18.0.4
Start Time:       Thu, 25 Dec 2025 23:29:13 -0300
Labels:           app=nginx-deployment
                  pod-template-hash=5c459b47db
Annotations:      <none>
Status:           Running
IP:               10.244.1.7
IPs:
  IP:           10.244.1.7
Controlled By:  ReplicaSet/nginx-deployment-5c459b47db
Containers:
  nginx:
    Container ID:   containerd://48719296f53ec22320067c5c293b7414b9430b2e8e9972727b3f550044565a6b
    Image:          nginx:1.19.2
    Image ID:       docker.io/library/nginx@sha256:c628b67d21744fce822d22fdcc0389f6bd763daac23a6b77147d0712ea7102d0
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 25 Dec 2025 23:29:13 -0300
    Ready:          False
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  256Mi
    Requests:
      cpu:        250m
      memory:     128Mi
    Readiness:    http-get http://:80/giropops delay=10s timeout=5s period=10s #success=2 #failure=3
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-925bx (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  kube-api-access-925bx:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  16m                 default-scheduler  Successfully assigned default/nginx-deployment-5c459b47db-7m792 to kind-multinodes-worker
  Normal   Pulled     16m                 kubelet            Container image "nginx:1.19.2" already present on machine
  Normal   Created    16m                 kubelet            Created container: nginx
  Normal   Started    16m                 kubelet            Started container nginx
  Warning  Unhealthy  57s (x97 over 16m)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 404
  ```

  Podemos ver que o nosso Pod não está saudável, e por isso o Kubernetes não está conseguindo atualizar o nosso Deployment.

### Startup Probe

Ela é a responsável por verificar se o nosso container foi inicializado corretamente, e se ele está pronto para receber requisições.

Ele é muito parecido com a readinessProbe, mas a diferença é que a startupProbe é executada apenas uma vez no começo da vida do nosso container, e a readinessProbe é executada de tempos em tempos.

Para entender melhor, vamos ver um exemplo criando um arquivo chamado nginx-startup.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.19.2
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
        startupProbe: # Onde definimos a nossa probe de inicialização
          httpGet: # O tipo de teste que iremos executar, neste caso, iremos executar um teste HTTP
            path: / # O caminho que iremos testar
            port: 80 # A porta que iremos testar
          initialDelaySeconds: 10 # O tempo que iremos esperar para executar a primeira vez a probe
          periodSeconds: 10 # De quanto em quanto tempo iremos executar a probe
          timeoutSeconds: 5 # O tempo que iremos esperar para considerar que a probe falhou
          successThreshold: 2 # O número de vezes que a probe precisa passar para considerar que o container está pronto
          failureThreshold: 3 # O número de vezes que a probe precisa falhar para considerar que o container não está pronto
```

Agora vamos aplicar a nossa configuração:

```
kubectl apply -f nginx-startup.yaml
```

Quando você tentar aplicar, receberá um erro, pois a successThreshold não pode ser maior que 1, pois a startupProbe é executada apenas uma vez, lembra?

Da mesma forma o failureThreshold não pode ser maior que 1, então vamos alterar o nosso arquivo para:

```
...
        startupProbe: # Onde definimos a nossa probe de inicialização
          httpGet: # O tipo de teste que iremos executar, neste caso, iremos executar um teste HTTP
            path: / # O caminho que iremos testar
            port: 80 # A porta que iremos testar
          initialDelaySeconds: 10 # O tempo que iremos esperar para executar a primeira vez a probe
          periodSeconds: 10 # De quanto em quanto tempo iremos executar a probe
          timeoutSeconds: 5 # O tempo que iremos esperar para considerar que a probe falhou
          successThreshold: 2 # O número de vezes que a probe precisa passar para considerar que o container está pronto
          failureThreshold: 1 # O número de vezes que a probe precisa falhar para considerar que o container não está pronto
```

Agora vamos aplicar novamente:

```
kubectl apply -f nginx-startup.yaml
```

Perceba que sua definição é super parecida com a readinessProbe, mas lembre-se, ela somente será executada uma vez, quando o container for inicializado. Portanto, se alguma coisa acontecer de errado depois disso, ele não irá te salvar, pois ele não irá executar novamente.

Por isso é super importante sempre ter uma combinação entre as probes, para que você tenha um container mais resiliente e que problemas possam ser detectados mais rapidamente.

Vamos ver se os nossos Pods estão saudáveis:

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6947bf5c68-gzqp7   1/1     Running   0          35s
nginx-deployment-6947bf5c68-hgrpz   1/1     Running   0          35s
nginx-deployment-6947bf5c68-m528d   1/1     Running   0          35s
```

Caso você queira conferir se a nossa probe está lá, basta usar o comando:

```
kubectl describe pod nginx-deployment-6947bf5c68-gzqp7
```

E você verá algo parecido com isso:

```
    Startup:      http-get http://:80/ delay=10s timeout=5s period=10s #success=1 #failure=1
```

### Exemplo com todas as probes

Vamos para o nosso exemplo final de hoje, vamos utilizar todas as probes que vimos até aqui, e vamos criar um arquivo chamado nginx-todas-probes.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.19.2
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
        livenessProbe: # Onde definimos a nossa probe de vida
          exec: # O tipo exec é utilizado quando queremos executar algo dentro do container.
            command: # Onde iremos definir qual comando iremos executar
              - curl
              - -f
              - http://localhost:80/
          initialDelaySeconds: 10 # O tempo que iremos esperar para executar a primeira vez a probe
          periodSeconds: 10 # De quanto em quanto tempo iremos executar a probe
          timeoutSeconds: 5 # O tempo que iremos esperar para considerar que a probe falhou
          successThreshold: 1 # O número de vezes que a probe precisa passar para considerar que o container está pronto
          failureThreshold: 3 # O número de vezes que a probe precisa falhar para considerar que o container não está pronto
        readinessProbe: # Onde definimos a nossa probe de prontidão
          httpGet: # O tipo de teste que iremos executar, neste caso, iremos executar um teste HTTP
            path: / # O caminho que iremos testar
            port: 80 # A porta que iremos testar
          initialDelaySeconds: 10 # O tempo que iremos esperar para executar a primeira vez a probe
          periodSeconds: 10 # De quanto em quanto tempo iremos executar a probe
          timeoutSeconds: 5 # O tempo que iremos esperar para considerar que a probe falhou
          successThreshold: 1 # O número de vezes que a probe precisa passar para considerar que o container está pronto
          failureThreshold: 3 # O número de vezes que a probe precisa falhar para considerar que o container não está pronto
        startupProbe: # Onde definimos a nossa probe de inicialização
          tcpSocket: # O tipo de teste que iremos executar, neste caso, iremos executar um teste TCP
            port: 80 # A porta que iremos testar
          initialDelaySeconds: 10 # O tempo que iremos esperar para executar a primeira vez a probe
          periodSeconds: 10 # De quanto em quanto tempo iremos executar a probe
          timeoutSeconds: 5 # O tempo que iremos esperar para considerar que a probe falhou
          successThreshold: 1 # O número de vezes que a probe precisa passar para considerar que o container está pronto
          failureThreshold: 3 # O número de vezes que a probe precisa falhar para considerar que o container não está pronto
```

Pronto, estamos utilizando as três probes, vamos aplicar:

```
kubectl apply -f nginx-todas-probes.yaml
```

E vamos ver se os nossos Pods estão saudáveis:

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7dc9cd667c-6phxt   1/1     Running   0          50s
nginx-deployment-7dc9cd667c-dwb8k   1/1     Running   0          50s
nginx-deployment-7dc9cd667c-rjxcz   1/1     Running   0          50s
```

Vamos ver na saída do describe pods se as nossa probes estão por lá.

```
Liveness:     exec [curl -f http://localhost:80/] delay=10s timeout=5s period=10s #success=1 #failure=3
Readiness:    http-get http://:80/ delay=10s timeout=5s period=10s #success=1 #failure=3
Startup:      tcp-socket :80 delay=10s timeout=5s period=10s #success=1 #failure=3
```

