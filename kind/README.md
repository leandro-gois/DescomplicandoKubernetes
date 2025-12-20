# Kind

O Kind (Kubernetes in Docker) é outra alternativa para executar o Kubernetes num ambiente local para testes e aprendizado, mas não é recomendado para uso em produção.

## Instalação no GNU/Linux

```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64

chmod +x ./kind

sudo mv ./kind /usr/local/bin/kind
```

### Criando um cluster com o Kind

```
kind create cluster --name meu-cluster
```

```
Creating cluster "meu-cluster" ...
 • Ensuring node image (kindest/node:v1.34.0) 🖼  ...
 ✓ Ensuring node image (kindest/node:v1.34.0) 🖼
 • Preparing nodes 📦   ...
 ✓ Preparing nodes 📦 
 • Writing configuration 📜  ...
 ✓ Writing configuration 📜
 • Starting control-plane 🕹️  ...
 ✓ Starting control-plane 🕹️
 • Installing CNI 🔌  ...
 ✓ Installing CNI 🔌
 • Installing StorageClass 💾  ...
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-meu-cluster"
You can now use your cluster with:

kubectl cluster-info --context kind-meu-cluster

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
```


### Criando um cluster com múltiplos nós locais com o Kind

Execute o comando a seguir para selecionar e remover todos os clusters locais criados no Kind.

```
kind delete clusters $(kind get clusters)

Deleted clusters: ["meu-cluster"]
```

Crie um arquivo de configuração para definir quantos e quais os tipos de nós que você deseja criar no cluster. No exemplo a seguir, será criado o arquivo de configuração kind-3nodes.yaml para especificar um cluster com 1 nó control-plane (que executará o control plane) e 2 workers.

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

  Agora vamos criar um cluster chamado kind-multinodes utilizando as especificações definidas no arquivo kind-3nodes.yaml.

```
kind create cluster --name kind-multinodes --config ../files/kind-3nodes.yaml
```

```
Creating cluster "kind-multinodes" ...
 • Ensuring node image (kindest/node:v1.34.0) 🖼  ...
 ✓ Ensuring node image (kindest/node:v1.34.0) 🖼
 • Preparing nodes 📦 📦 📦   ...
 ✓ Preparing nodes 📦 📦 📦 
 • Writing configuration 📜  ...
 ✓ Writing configuration 📜
 • Starting control-plane 🕹️  ...
 ✓ Starting control-plane 🕹️
 • Installing CNI 🔌  ...
 ✓ Installing CNI 🔌
 • Installing StorageClass 💾  ...
 ✓ Installing StorageClass 💾
 • Joining worker nodes 🚜  ...
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-kind-multinodes"
You can now use your cluster with:

kubectl cluster-info --context kind-kind-multinodes

Have a nice day! 👋
```

  Valide a criação do cluster com o comando a seguir.

```
kubectl get nodes
```

### Primeiros passos no k8s

#### Verificando os namespaces e pods


O k8s organiza tudo dentro de namespaces. Por meio deles, podem ser realizadas limitações de segurança e de recursos dentro do cluster, tais como pods, replication controllers e diversos outros. Para visualizar os namespaces disponíveis no cluster, digite:



```
kubectl get namespaces
```

  Vamos listar os pods do namespace kube-system utilizando o comando a seguir.

```
kubectl get pod -n kube-system
```

  Será que há algum pod escondido em algum namespace? É possível listar todos os pods de todos os namespaces com o comando a seguir.

```
kubectl get pods -A
```

  Há a possibilidade ainda, de utilizar o comando com a opção -o wide, que disponibiliza maiores informações sobre o recurso, inclusive em qual nó o pod está sendo executado. Exemplo:

```
kubectl get pods -A -o wide
```

#### Executando nosso primeiro pod no k8s

Iremos iniciar o nosso primeiro pod no k8s. Para isso, executaremos o comando a seguir.

```
kubectl run nginx --image nginx

pod/nginx created
```

  Listando os pods com kubectl get pods, obteremos a seguinte saída.

```
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          66s
```

  Vamos agora remover o nosso pod com o seguinte comando.

```
kubectl delete pod nginx
```

  A saída deve ser algo como:

```
pod "nginx" deleted
```

#### Executando nosso primeiro pod no k8s utilizando um arquivo manifesto

Uma outra forma de criar um pod ou qualquer outro objeto no Kubernetes é através da utilizaçâo de uma arquivo manifesto, que é uma arquivo em formato YAML onde você passa todas as definições do seu objeto. Mas pra frente vamos falar mais sobre como construir arquivos manifesto, mas agora eu quero que você conheça a opção --dry-run do kubectl, com ela podemos simular a criação de um resource e ainda ter um manifesto criado automaticamente.

```
kubectl run meu-nginx --image nginx --dry-run=client -o yaml > pod-template.yaml
```

Com o arquivo gerado em mãos, agora você consegue criar um pod utilizando o manifesto que criamos da seguinte forma:


```
kubectl apply -f pod-template.yaml
```

#### Expondo o pod e criando um Service

Dispositivos fora do cluster, por padrão, não conseguem acessar os pods criados, como é comum em outros sistemas de contêineres. Para expor um pod, execute o comando a seguir.


```
kubectl expose pod nmeu-nginx
```

Será apresentada a seguinte mensagem de erro:

```
error: couldn't find port via --port flag or introspection
See 'kubectl expose -h' for help and examples
```

O erro ocorre devido ao fato do k8s não saber qual é a porta de destino do contêiner que deve ser exposta (no caso, a 80/TCP). Para configurá-la, vamos primeiramente remover o nosso pod antigo:


```
kubectl delete -f pod-template.yaml
```

Agora vamos executar novamente o comando para a criação do pod utilizando o parametro 'dry-run', adicionando o parametro '--port' para dizer qual a porta que o container está escutando, lembrando que estamos utilizando o nginx nesse exemplo, um webserver que escuta por padrão na porta 80.


```
kubectl run meu-nginx --image nginx --port 80 --dry-run=client -o yaml > pod-template.yaml
kubectl create -f pod-template.yaml
```

Liste os pods.

```
kubectl get pods

NAME    READY   STATUS    RESTARTS   AGE
meu-nginx   1/1     Running   0          32s
```

O comando a seguir cria um objeto do k8s chamado de Service, que é utilizado justamente para expor pods para acesso externo.


```
kubectl expose pod meu-nginx
```

Podemos listar todos os services com o comando a seguir.

```
kubectl get services

NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   8d
nginx        ClusterIP   10.105.41.192   <none>        80/TCP    2m30s
```

Como é possível observar, há dois services no nosso cluster: o primeiro é para uso do próprio k8s, enquanto o segundo foi o quê acabamos de criar.

#### Limpando tudo e indo para casa

Para mostrar todos os recursos recém criados, pode-se utilizar uma das seguintes opções a seguir.

```
kubectl get all

kubectl get pod,service

kubectl get pod,svc
```

Note que o k8s nos disponibiliza algumas abreviações de seus recursos. Com o tempo você irá se familiar com elas. Para apagar os recursos criados, você pode executar os seguintes comandos.

```
kubectl delete -f pod-template.yaml
kubectl delete service nginx
```

Liste novamente os recursos para verificar se eles foram removidos.


