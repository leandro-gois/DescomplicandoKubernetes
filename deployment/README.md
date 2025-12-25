# O que é um Deployment?

No Kubernetes um Deployment é um objeto que representa uma aplicação. Ele é responsável por gerenciar os Pods que compõem uma aplicação. Um Deployment é uma abstração que nos permite atualizar os Pods e também fazer o rollback para uma versão anterior caso algo dê errado.

### Como criar um Deployment?

Para criar um Deployment nós precisamos de um arquivo YAML. Vamos criar um arquivo chamado deployment.yaml e vamos adicionar o seguinte conteúdo:

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
      - image: nginx
        name: nginx
        resources:
          limits:
            cpu: 0.5
            memory: 250Mi
          requests:
            cpu: 0.25
            memory: 128Mi

```

### Como aplicar o Deployment?

```
kubectl apply -f deployment.yaml
```

### Como verificar se o Deployment foi criado?

```
kubectl get deployments -l app=nginx-deployment
```

O resultado será o seguinte:

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           84s
```

### Como verificar os Pods que o Deployment está gerenciando?

```
kubectl get pods -l app=nginx-deployment
```

O resultado será o seguinte:

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-646cc9b75c-7dmpj   1/1     Running   0          19m
nginx-deployment-646cc9b75c-cn68h   1/1     Running   0          19m
nginx-deployment-646cc9b75c-z7jpm   1/1     Running   0          19m
```

### Como verificar o ReplicaSet que o Deployment está gerenciando?

Caso eu queria listar os ReplicaSets que o Deployment está gerenciando eu posso executar o seguinte comando:

```
kubectl get replicasets -l app=nginx-deployment
```

O resultado será o seguinte:

```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-646cc9b75c   3         3         3       20m
```

### Como verificar os detalhes do Deployment?

Para verificar os detalhes do Deployment nós precisamos executar o seguinte comando:

```
kubectl describe deployment nginx-deployment
```

O resultado será o seguinte:

```
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Sat, 20 Dec 2025 18:19:11 -0300
Labels:                 app=nginx-deployment
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx-deployment
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx-deployment
  Containers:
   nginx:
    Image:      nginx
    Port:       <none>
    Host Port:  <none>
    Limits:
      cpu:     500m
      memory:  250Mi
    Requests:
      cpu:         250m
      memory:      128Mi
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-646cc9b75c (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  32m   deployment-controller  Scaled up replica set nginx-deployment-646cc9b75c from 0 to 3
```

### Como atualizar o Deployment?

Vamos imaginar que agora precisamos passar uma versão especifica da imagem do Nginx para o Deployment, para isso nós precisamos alterar o arquivo deployment.yaml e alterar a versão da imagem para nginx:1.16.0, por exemplo.

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
      - image: nginx:1.16.0
        name: nginx
        resources:
          limits:
            cpu: 0.5
            memory: 250Mi
          requests:
            cpu: 0.25
            memory: 128Mi
```

Agora que já alteramos a versão da imagem do Nginx, nós precisamos aplicar as alterações no Deployment, para isso nós precisamos executar o seguinte comando:

```
kubectl apply -f deployment.yaml
```

O resultado será o seguinte:

```
deployment.apps/nginx-deployment configured
```

Vamos ver os detalhes do Deployment para verificar se a versão da imagem foi alterada:

```
kubectl describe deployment nginx-deployment
```

Na saída do comando, podemos ver a linha onde está a versão da imagem do Nginx:

```


Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Sat, 20 Dec 2025 18:19:11 -0300
Labels:                 app=nginx-deployment
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=nginx-deployment
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx-deployment
  Containers:
   nginx:
    Image:      nginx:1.16.0
    Port:       <none>
    Host Port:  <none>
    Limits:
      cpu:     500m
      memory:  250Mi
    Requests:
      cpu:         250m
      memory:      128Mi
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  nginx-deployment-646cc9b75c (0/0 replicas created)
NewReplicaSet:   nginx-deployment-b54894bb9 (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  44m    deployment-controller  Scaled up replica set nginx-deployment-646cc9b75c from 0 to 3
  Normal  ScalingReplicaSet  3m19s  deployment-controller  Scaled up replica set nginx-deployment-b54894bb9 from 0 to 1
  Normal  ScalingReplicaSet  2m35s  deployment-controller  Scaled down replica set nginx-deployment-646cc9b75c from 3 to 2
  Normal  ScalingReplicaSet  2m35s  deployment-controller  Scaled up replica set nginx-deployment-b54894bb9 from 1 to 2
  Normal  ScalingReplicaSet  115s   deployment-controller  Scaled down replica set nginx-deployment-646cc9b75c from 2 to 1
  Normal  ScalingReplicaSet  115s   deployment-controller  Scaled up replica set nginx-deployment-b54894bb9 from 2 to 3
  Normal  ScalingReplicaSet  112s   deployment-controller  Scaled down replica set nginx-deployment-646cc9b75c from 1 to 0
  ```

### As estratégias de atualização do Deployment

O Kubernetes possui 2 estratégias de atualização para os Deployments:

RollingUpdate
Recreate

### Estratégia RollingUpdate

A estratégia RollingUpdate é a estratégia de atualização padrão do Kubernetes, ela é utilizada para atualizar os Pods de um Deployment de forma gradual, ou seja, ela atualiza um Pod por vez, ou um grupo de Pods por vez.

Nós podemos definir como será a atualização dos Pods, por exemplo, podemos definir a quantidade máxima de Pods que podem ficar indisponíveis durante a atualização, ou podemos definir a quantidade máxima de Pods que podem ser criados durante a atualização.

Vamos também aumentar a quantidade de réplicas do Deployment para 10, para que possamos ter um pouco mais de Pods para atualizar.

E para que possamos testar a estratégia RollingUpdate, vamos alterar a versão da imagem do Nginx para 1.15.0.

Para que possamos definir essas configurações, nós precisamos alterar o arquivo deployment.yaml e adicionar as seguintes configurações:

```
apiVersion: apps/v1
kind: Deployment
metadata: 
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.16.0
        name: nginx
        resources:
          limits:
            cpu: 0.5
            memory: 250Mi
          requests:
            cpu: 0.25
            memory: 128Mi
```

O que nós fizemos foi adicionar as seguintes configurações:

```
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 2
```

Onde:

maxSurge: define a quantidade máxima de Pods que podem ser criados a mais durante a atualização, ou seja, durante o processo de atualização, nós podemos ter 1 Pod a mais do que o número de Pods definidos no Deployment. Isso é útil pois agiliza o processo de atualização, pois o Kubernetes não precisa esperar que um Pod seja atualizado para criar um novo Pod.

maxUnavailable: define a quantidade máxima de Pods que podem ficar indisponíveis durante a atualização, ou seja, durante o processo de atualização, nós podemos ter 1 Pod indisponível por vez. Isso é útil pois garante que o serviço não fique indisponível durante a atualização.

type: define o tipo de estratégia de atualização que será utilizada, no nosso caso, nós estamos utilizando a estratégia RollingUpdate.

Agora que já alteramos o arquivo deployment.yaml, nós precisamos aplicar as alterações no Deployment, para isso nós precisamos executar o seguinte comando:

```
kubectl apply -f deployment.yaml
```

O resultado será o seguinte:

```
deployment.apps/nginx-deployment configured
```

Vamos verificar se as alterações foram aplicadas no Deployment:

```
kubectl describe deployment nginx-deployment
```

Na saída do comando, podemos ver que as linhas onde estão as configurações de atualização do Deployment foram alteradas:

```
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  2 max unavailable, 1 max surge
```

Com essa configuração estamos falando para o Kubernetes que ele pode criar até 1 Pod a mais durante a atualização, e que ele pode ter até 2 Pods indisponíveis durante a atualização, ou seja, ele vai atualizar de 2 em 2 Pods.

Um comando muito útil para acompanhar o processo de atualização dos Pods é o comando:

```
kubectl rollout status deployment nginx-l deployment
```

O comando rollout status é utilizado para acompanhar o processo de atualização de um Deployment, ReplicaSet, DaemonSet, StatefulSet, Job e CronJob. O comando rollout status é muito útil pois ele nos informa se o processo de atualização está em andamento, se ele foi concluído com sucesso ou se ele falhou.

Vamos verificar se os Pods foram atualizados:

```
kubectl get pods -l app=nginx-deployment -o yaml
```

Na saída do comando, podemos ver que os Pods foram atualizados:

```
...
  - image: nginx:1.15.0
...

```

Podemos também verificar a versão da imagem do Nginx no Pod:

```
kubectl exec -it nginx-deployment-b5cf44d84-2td6h -- nginx -v
```

O resultado será o seguinte:

```
nginx version: nginx/1.15.0
```

### Estratégia Recreate

A estratégia Recreate é uma estratégia de atualização que irá remover todos os Pods do Deployment e criar novos Pods com a nova versão da imagem. A parte boa é que o deploy acontecerá rapidamente, porém o ponto negativo é que o serviço ficará indisponível durante o processo de atualização.

Vamos alterar o arquivo deployment.yaml para utilizar a estratégia Recreate:

```
apiVersion: apps/v1
kind: Deployment
metadata: 
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.15.0
        name: nginx
        resources:
          limits:
            cpu: 0.5
            memory: 250Mi
          requests:
            cpu: 0.25
            memory: 128Mi
```

Perceba que agora somente temos a configuração type: Recreate. O Recreate não possui configurações de atualização, ou seja, não é possível definir o número máximo de Pods indisponíveis durante a atualização, afinal o Recreate irá remover todos os Pods do Deployment e criar novos Pods.

Agora que já alteramos o arquivo deployment.yaml, nós precisamos aplicar as alterações no Deployment, para isso nós precisamos executar o seguinte comando:

```
kubectl apply -f deployment.yaml
```

O resultado será o seguinte:

```
deployment.apps/nginx-deployment configured
```

Vamos verificar se as alterações foram aplicadas no Deployment:

```
kubectl describe deployment nginx-deployment
```

Na saída do comando, podemos ver que as linhas onde estão as configurações de atualização do Deployment foram alteradas:

```
StrategyType:           Recreate
```

Vamos novamente alterar a versão da imagem do Nginx para 1.16.0 no arquivo deployment.yaml:

```
image: nginx:1.16.0
```

Agora que já alteramos o arquivo deployment.yaml, nós precisamos aplicar as alterações no Deployment, para isso nós precisamos executar o seguinte comando:

```
kubectl apply -f deployment.yaml
```

O resultado será o seguinte:

```
deployment.apps/nginx-deployment configured
```

Vamos verificar os Pods do Deployment:

```
kubectl get pods -l app=nginx-deployment
```

O resultado será o seguinte:

```
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-b54894bb9-6hz9x   1/1     Running   0          4m23s
nginx-deployment-b54894bb9-74np5   1/1     Running   0          4m23s
nginx-deployment-b54894bb9-bhqht   1/1     Running   0          4m23s
nginx-deployment-b54894bb9-bx5jf   1/1     Running   0          4m23s
nginx-deployment-b54894bb9-h8g4t   1/1     Running   0          4m23s
nginx-deployment-b54894bb9-r48gd   1/1     Running   0          4m23s
nginx-deployment-b54894bb9-rmjks   1/1     Running   0          4m23s
nginx-deployment-b54894bb9-sdgp8   1/1     Running   0          4m23s
nginx-deployment-b54894bb9-sjkwv   1/1     Running   0          4m23s
nginx-deployment-b54894bb9-tpdfh   1/1     Running   0          4m23s
```

Vamos verificar a versão da imagem do Nginx no Pod:

```
kubectl exec -it nginx-deployment-b54894bb9-6hz9x -- nginx -v
```

O resultado será o seguinte:

```
nginx version: nginx/1.16.0
```

Pronto, agora nós temos a versão 1.16.0 do Nginx rodando no nosso cluster e já entendemos como funciona a estratégia Recreate.

### Fazendo o rollback de uma atualização

para isso nós precisamos executar o seguinte comando:

```
kubectl rollout undo deployment nginx-deployment
```

O resultado será o seguinte:

```
deployment.apps/nginx-deployment rolled back
```

O que estamos fazendo nesse momento é falar para o Kubernetes que queremos fazer o rollback para a versão anterior do Deployment.

Vamos verificar os Pods do Deployment:

```
kubectl get pods -l app=nginx-deployment
```

Vamos verificar a versão da imagem do Nginx no Pod:

```
kubectl exec -it nginx-deployment-b5cf44d84-282pl -- nginx -v
```

O resultado será o seguinte:

```
nginx version: nginx/1.15.0
```

Pronto, agora nós temos a versão 1.16.0 do Nginx rodando no nosso cluster e já entendemos como fazer o rollback de uma atualização. Mas como nós visualizamos o histórico de atualizações do Deployment?

Essa é fácil, nós precisamos executar o seguinte comando:

```
kubectl rollout history deployment nginx-deployment
```

Com isso ele vai nos mostrar o histórico de atualizações do Deployment:

```
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
4         <none>
5         <none>
```

Vamos verificar o histórico de atualizações da revisão 4:

```
kubectl rollout history deployment nginx-deployment --revision=4
```

O resultado será o seguinte:

```
deployment.apps/nginx-deployment with revision #4
Pod Template:
  Labels:	app=nginx-deployment
	pod-template-hash=b54894bb9
  Containers:
   nginx:
    Image:	nginx:1.16.0
    Port:	<none>
    Host Port:	<none>
    Limits:
      cpu:	500m
      memory:	250Mi
    Requests:
      cpu:	250m
      memory:	128Mi
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
  Node-Selectors:	<none>
  Tolerations:	<none>
```

Vamos verificar o histórico de atualizações da revisão 5:

```
kubectl rollout history deployment nginx-deployment --revision=5
```

O resultado será o seguinte:

deployment.apps/nginx-deployment with revision #5
Pod Template:
  Labels:	app=nginx-deployment
	pod-template-hash=b5cf44d84
  Containers:
   nginx:
    Image:	nginx:1.15.0
    Port:	<none>
    Host Port:	<none>
    Limits:
      cpu:	500m
      memory:	250Mi
    Requests:
      cpu:	250m
      memory:	128Mi
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
  Node-Selectors:	<none>
  Tolerations:	<none>

Ou seja, como podemos notar, a revisão 4 é a versão 1.16.0 do Nginx e a revisão 5 é a versão 1.15.0 do Nginx.

Se você quiser fazer o rollback para a revisão 4, basta executar o seguinte comando:

```
kubectl rollout undo deployment nginx-deployment --to-revision=4
```

O resultado será o seguinte:

```
deployment.apps/nginx-deployment rolled back
```

### Removendo um Deployment

Para remover um Deployment nós precisamos executar o seguinte comando:

```
kubectl delete deployment nginx-deployment
```

O resultado será o seguinte:

```
deployment.apps "nginx-deployment" deleted
```

Caso queira remover o Deployment utilizando o manifesto, basta executar o seguinte comando:

```
kubectl delete -f deployment.yaml
```

O resultado será o seguinte:

```
deployment.apps "nginx-deployment" deleted
```





