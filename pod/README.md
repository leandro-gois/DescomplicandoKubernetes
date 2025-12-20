# O que é um Pod?

Primeira coisa, o Pod é a menor unidade dentro de um cluster Kubernetes.

Quando estamos falando sobre Pod, precisamos pensar que o Pod é uma caixinha que contém um ou mais containers. E esses containers compartilham os mesmos recursos do Pod, como por exemplo, o IP, o namespace, o volume, etc.

Então, quando falamos de Pod, estamos falando de um ou mais containers que compartilham os mesmos recursos, ponto.

### Criando um Pod

```
kubectl run meu-primeiro-pod --image=nginx --port=80
```

O comando acima irá criar um Pod chamado giropops, com uma imagem do nginx e com a porta 80 exposta.

### Visualizando detalhes sobre os Pods

```
kubectl get pods
```

O comando acima irá listar todos os Pods que estão em execução no cluster, na namespace default.

Para ver os Pods em execução em todas as namespaces, podemos usar o comando:

```
kubectl get pods --all-namespaces
```

Ou ainda, podemos usar o comando:

```
kubectl get pods -A
```

Agora, se você quiser ver todos os Pods de uma namespace específica, você pode usar o comando:

```
kubectl get pods -n kube-system
```

O comando acima irá listar todos os Pods que estão em execução na namespace kube-system, que é a namespace onde o Kubernetes irá criar todos os objetos relacionados ao cluster, como por exemplo, os Pods do CoreDNS, do Kube-Proxy, do Kube-Controller-Manager, do Kube-Scheduler, etc.

Caso você queira ver ainda mais detalhes sobre o Pod, você pode pedir para o Kubernetes mostrar os detalhes do Pod em formato YAML, usando o comando:

```
kubectl get pods <nome-do-pod> -o yaml
```

Por exemplo:

```
kubectl get pods meu-primeiro-pod -o yaml
```

Ahh, a saída wide é interessante, pois ela mostra mais detalhes sobre o Pod, como por exemplo, o IP do Pod e o Node onde o Pod está sendo executado.

```
kubectl get pods meu-primeiro-pod -o wide
```

Agora, se você quiser ver os detalhes do Pod, mas sem precisar usar o comando get, você pode usar o comando:

```
kubectl describe pods meu-primeiro-pod
```

### Removendo um Pod

```
kubectl delete pods meu-primeiro-pod
```

### Criando um Pod através de um arquivo YAML

Vamos criar um arquivo YAML chamado pod.yaml com o seguinte conteúdo:

```
apiVersion: v1
kind: Pod
metadata:
  name: meu-nginx
  labels: 
    run: meu-nginx
spec: 
  containers:
  - name: meu-nginx
    image: nginx
    ports:
    - containerPort: 80
```

Agora, vamos criar o Pod usando o arquivo YAML que acabamos de criar.

```
kubectl apply -f pod.yaml
```

O comando acima irá criar o Pod usando o arquivo YAML que criamos.

Para ver o Pod criado, podemos usar o comando:

```
kubectl get pods
```

Agora, vamos ver os detalhes do Pod que acabamos de criar.

```
kubectl describe pods meu-nginx
```

#### Visualizando os logs do Pod


```
kubectl logs meu-nginx
```

Se você quiser ver os logs do container em tempo real, você pode usar o comando:

```
kubectl logs -f meu-nginx
```

Simples né? Agora, vamos remover o Pod que criamos, usando o comando:

```
kubectl delete pods meu-nginx
```

#### Criando um Pod com mais de um container

Vamos criar um arquivo YAML chamado pod-multi-container.yaml com o seguinte conteúdo:

```
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod
  labels: 
    run: meu-pod
spec: 
  containers:
  - name: meu-nginx
    image: nginx
    ports:
    - containerPort: 80 
  - name: meu-alpine
    image: alpine
    args:
    - sleep
    - "1800"
```

Agora, vamos criar o Pod usando o arquivo YAML que acabamos de criar.

```
kubectl apply -f pod-multi-container.yaml
```

Para ver o Pod criado, podemos usar o comando:

```
kubectl get pods
```

Agora, vamos ver os detalhes do Pod que acabamos de criar.

```
kubectl describe pods meu-pod
```

### Criando um container com limites de memória e CPU

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-limitado
  labels: 
    run: pod-limitado
spec: 
  containers:
  - name: pod-limitado
    image: nginx
    ports:
    - containerPort: 80
    resources:
      limits:
        memory: "128Mi"
        cpu: "0.5"
      requests:
        memory: "64Mi"
        cpu: "0.3"  
```

Agora vamos criar o Pod com os limites de memória e CPU.

```
kubectl create -f pod-limitado.yaml
```

Agora vamos verificar se o Pod foi criado.

```
kubectl get pods
```

Vamos verificar os detalhes do Pod.

```
kubectl describe pod pod-limitado
```

Veja que o Pod foi criado com sucesso, e que os limites de memória e CPU foram definidos conforme o arquivo YAML.

Veja abaixo a parte da saída do comando describe que mostra os limites de memória e CPU.


```
Name:             pod-limitado
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-multinodes-worker/172.18.0.5
Start Time:       Fri, 19 Dec 2025 23:38:44 -0300
Labels:           run=pod-limitado
Annotations:      <none>
Status:           Running
IP:               10.244.1.3
IPs:
  IP:  10.244.1.3
Containers:
  pod-limitado:
    Container ID:   containerd://9a48b2a2a941dd43cbbab4d931cef429379b5bd8ecd292f61f0d469b313d80f6
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:fb01117203ff38c2f9af91db1a7409459182a37c87cced5cb442d1d8fcc66d19
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 19 Dec 2025 23:38:47 -0300
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  128Mi
    Requests:
      cpu:        300m
      memory:     64Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-w4jbp (ro)
```