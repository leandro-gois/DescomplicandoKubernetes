# O que é um StatefulSet?

Os StatefulSets são uma funcionalidade do Kubernetes que gerencia o deployment e o scaling de um conjunto de Pods, fornecendo garantias sobre a ordem de deployment e a singularidade desses Pods.

Diferente dos Deployments e Replicasets que são considerados stateless (sem estado), os StatefulSets são utilizados quando você precisa de mais garantias sobre o deployment e scaling. Eles garantem que os nomes e endereços dos Pods sejam consistentes e estáveis ao longo do tempo.

### Criando um StatefulSet

Para a criação de um StatefulSet precisamos de um arquivo de configuração que descreva o StatefulSet que queremos criar. Não é possível criar um StatefulSet sem um manifesto yaml, diferente do que acontece com o Pod e o Deployment.

Para o nosso exemplo, vamos utilizar o Nginx como aplicação que será gerenciada pelo StatefulSet, e vamos fazer com que cada Pod tenha um volume persistente associado a ele, e com isso, vamos ter uma página web diferente para cada Pod.

Para isso, vamos criar o arquivo nginx-statefulset.yaml com o seguinte conteúdo:

```
apiVersion: apps/v1
kind: StatefulSet # Tipo do recurso que estamos criando, no caso, um StatefulSet
metadata:
  name: nginx
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates: # Como estamos utilizando StatefulSet, precisamos criar um template de volume para cada Pod, entã ao invés de criarmos um volume diretamente, criamos um template que será utilizado para criar um volume para cada Pod
  - metadata:
      name: www # Nome do volume, assim teremos o volume www-0, www-1 e www-2
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

Agora, vamos criar o StatefulSet com o comando:

```
kubectl apply -f nginx-statefulset.yaml
```

Para verificar se o StatefulSet foi criado, podemos utilizar o comando:

```
kubectl get statefulset
```

```
NAME    READY   AGE
nginx   3/3     6m51s
```

Caso queira ver com mais detalhes, podemos utilizar o comando:

```
kubectl describe statefulset nginx
```

```
Name:               nginx
Namespace:          default
CreationTimestamp:  Fri, 02 Jan 2026 23:44:38 -0300
Selector:           app=nginx
Labels:             <none>
Annotations:        <none>
Replicas:           3 desired | 3 total
Update Strategy:    RollingUpdate
  Partition:        0
Pods Status:        3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx
    Port:         80/TCP (web)
    Host Port:    0/TCP (web)
    Environment:  <none>
    Mounts:
      /usr/share/nginx/html from www (rw)
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Volume Claims:
  Name:          www
  StorageClass:  
  Labels:        <none>
  Annotations:   <none>
  Capacity:      1Gi
  Access Modes:  [ReadWriteOnce]
Events:
  Type    Reason            Age    From                    Message
  ----    ------            ----   ----                    -------
  Normal  SuccessfulCreate  10m    statefulset-controller  create Pod nginx-0 in StatefulSet nginx successful
  Normal  SuccessfulCreate  8m2s   statefulset-controller  create Claim www-nginx-1 Pod nginx-1 in StatefulSet nginx success
  Normal  SuccessfulCreate  8m2s   statefulset-controller  create Pod nginx-1 in StatefulSet nginx successful
  Normal  SuccessfulCreate  7m55s  statefulset-controller  create Claim www-nginx-2 Pod nginx-2 in StatefulSet nginx success
  Normal  SuccessfulCreate  7m55s  statefulset-controller  create Pod nginx-2 in StatefulSet nginx successful
  ```

Para verificar se os Pods foram criados, podemos utilizar o comando:

```
kubectl get pods
```

```
NAME      READY   STATUS    RESTARTS   AGE
nginx-0   1/1     Running   0          13m
nginx-1   1/1     Running   0          11m
nginx-2   1/1     Running   0          11m
```

O nosso StatefulSet está criado, mas ainda temos que criar o Headless Service para que possamos acessar os Pods individualmente, e para isso, vamos criar o arquivo nginx-headless-service.yaml com o seguinte conteúdo:

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None # Como estamos criando um Headless Service, não queremos que ele tenha um IP, então definimos o clusterIP como None
  selector:
    app: nginx
```

Pronto, arquivo criado, hora de criar o Headless Service com o comando:

```
kubectl apply -f nginx-headless-service.yaml
```

Para verificar se o Headless Service foi criado, podemos utilizar o comando:

```
kubectl get service
```

Caso queira ver com mais detalhes, podemos utilizar o comando:

```
kubectl describe service nginx
```

Agora que já temos o StatefulSet e o Headless Service criados, podemos acessar cada Pod individualmente, para isso, vamos utilizar o comando:

```
kubectl run -it --rm debug --image=busybox --restart=Never -- sh
```

Agora vamos utilizar o comando nslookup para verificar o endereço IP de cada Pod, para isso, vamos utilzar o comando:

```
nslookup nginx-0.nginx.default.svc.cluster.local
```

Agora vamos acessar o Pod utilizando o endereço IP, para isso, vamos utilizar o comando:

```
wget -O- http://<endereço-ip-do-pod>
```

Precisamos mudar a página web de cada Pod para que possamos verificar se estamos acessando o Pod correto, para isso, vamos utilizar o comando:

```
echo "Pod 0" > /usr/share/nginx/html/index.html
```

Agora vamos acessar o Pod novamente, para isso, vamos utilizar o comando:

```
wget -O- http://<endereço-ip-do-pod>
```

A saída do comando deve ser:

```
Connecting to <endereço-ip-do-pod>:80 (<endereço-ip-do-pod>:80)
Pod 0
```

Caso queira, você pode fazer o mesmo para os outros Pods, basta mudar o número do Pod no comando nslookup e no comando echo.

### Excluindo um StatefulSet

Para excluir um StatefulSet precisamos utilizar o comando:

```
kubectl delete statefulset nginx
```

Ou ainda podemos excluir o StatefulSet utilizando o comando:

```
kubectl delete -f nginx-statefulset.yaml
```

### Excluindo um Headless Service

Para excluir um Headless Service precisamos utilizar o comando:

```
kubectl delete service nginx
```

Ou ainda podemos excluir o Headless Service utilizando o comando:

```
kubectl delete -f nginx-headless-service.yaml
```

### Excluindo um PVC

Para excluir um Volume precisamos utilizar o comando:

```
kubectl delete pvc www-nginx-
```

# Services

Os Services no Kubernetes são uma abstração que define um conjunto lógico de Pods e uma política para acessá-los. Eles permitem que você exponha uma ou mais Pods para serem acessados por outros Pods, independentemente de onde eles estejam em execução no cluster.

Os Services são definidos usando a API do Kubernetes, normalmente através de um arquivo de manifesto YAML.

### Tipos de Services

__ClusterIP__ (padrão): Expõe o Service em um IP interno no cluster. Este tipo torna o Service acessível apenas dentro do cluster.

__NodePort__ Expõe o Service na mesma porta de cada Node selecionado no cluster usando NAT. Torna o Service acessível de fora do cluster usando :.

__LoadBalancer__ Cria um balanceador de carga externo no ambiente de nuvem atual (se suportado) e atribui um IP fixo, externo ao cluster, ao Service.

__ExternalName__ Mapeia o Service para o conteúdo do campo externalName (por exemplo, foo.bar.example.com), retornando um registro CNAME com seu valor.

### Como os Services funcionam

Os Services no Kubernetes fornecem uma abstração que define um conjunto lógico de Pods e uma política para acessá-los. Os conjuntos de Pods são determinados por meio de seletores de rótulo (Label Selectors). Embora cada Pod tenha um endereço IP único, esses IPs não são expostos fora do cluster sem um serviço.

Sempre é bom reforçar a importância dos Labels no Kubernetes, pois eles são a base para a maioria das operações no Kubernetes, então cuide com carinho dos seus Labels.

### Os Services e os Endpoints

Como eu já disse, os Services no Kubernetes representam um conjunto estável de Pods que fornecem determinada funcionalidade. A principal característica dos Services é que eles mantêm um endereço IP estável e uma porta de serviço que permanecem constantes ao longo do tempo, mesmo que os Pods subjacentes sejam substituídos.

Para implementar essa abstração, o Kubernetes usa uma outra abstração chamada Endpoint. Quando um Service é criado, o Kubernetes também cria um objeto Endpoint com o mesmo nome. Esse objeto Endpoint rastreia os IPs e as portas dos Pods que correspondem aos critérios de seleção do Service.

Por exemplo, quando você cria um Service automaticamente é criado os EndPoints que representam os Pods que estão sendo expostos pelo Service.

Para você ver os EndPoints criados, você pode utilizar o comando:

```
kubectl get endpoints meu-service
```

### Criando um Service

Para criar um Service precisamos utilizar o comando:

```
kubectl expose deployment meu-deployment --port=80 --type=NodePort
```

Perceba que no comando acima estamos criando um Service do tipo NodePort, ou seja, o Service será acessível de fora do cluster usando :.

Estamos ainda passando o parâmetro --port=80 para que o Service seja exposto na porta 80.

Ahhh, e não podemos esquecer de passar o nome do Deployment que queremos expor, no caso acima, estamos expondo o Deployment com o nome MEU_DEPLOYMENT. Estamos criando um Service para um Deployment, mas também podemos criar um Service para um Pod, um ReplicaSet, um ReplicationController ou até mesmo para outro Service.

Vamos criar alguns Services para que você entenda melhor como eles funcionam e no final vamos para um exemplo mais completo utilizando como Deployment o Giropops-Senhas!

### ClusterIP

Para criar um serviço ClusterIP via kubectl, você pode usar o comando kubectl expose conforme mostrado abaixo:

```
kubectl expose deployment meu-deployment --port=80 --target-port=8080
```

Este comando criará um serviço ClusterIP que expõe o meu-deployment na porta 80, encaminhando o tráfego para a porta 8080 dos Pods deste deployment.

Para criar um serviço ClusterIP via YAML, você pode usar um arquivo como este:

```
apiVersion: v1
kind: Service # Tipo do objeto, no caso, um Service
metadata:
  name: meu-service
spec:
  selector: # Seleciona os Pods que serão expostos pelo Service
    app: meu-app # Neste caso, os Pods com o label app=meu-app
  ports:
    - protocol: TCP
      port: 80 # Porta do Service
      targetPort: 8080 # Porta dos Pods
```

Este arquivo YAML irá criar um serviço que corresponde aos Pods com o label app=meu-app, e encaminha o tráfego da porta 80 do serviço para a porta 8080 dos Pods. Perceba que estamos usando o selector para definir quais Pods serão expostos pelo Service.

Como não especificamos o campo type, o Kubernetes irá criar um Service do tipo ClusterIP por padrão.

### NodePort

Para criar um serviço NodePort via CLI, você pode usar o comando kubectl expose com a opção --type=NodePort. Por exemplo:

```
kubectl expose deployment meu-deployment --type=NodePort --port=80 --target-port=8080
```

Já para criar um serviço NodePort via YAML, você pode usar um arquivo como este:

```
apiVersion: v1
kind: Service
metadata:
  name: meu-service
spec:
  type: NodePort # Tipo do Service
  selector:
    app: meu-app
  ports:
    - protocol: TCP
      port: 80 # Porta do Service, que será mapeada para a porta 8080 do Pod
      targetPort: 8080 # Porta dos Pods
      nodePort: 30080   # Porta do Node, que será mapeada para a porta 80 do Service
```

Aqui estamos especificando o campo type como NodePort, e também estamos especificando o campo nodePort, que define a porta do Node que será mapeada para a porta 80 do Service. Vale lembrar que a porta do Node deve estar entre 30000 e 32767, e quando não especificamos o campo nodePort, o Kubernetes irá escolher uma porta aleatória dentro deste range. :)

### LoadBalancer

O Service do tipo LoadBalancer é uma das formas mais comuns de expor um serviço ao tráfego da internet no Kubernetes. Ele provisiona automaticamente um balanceador de carga do provedor de nuvem onde seu cluster Kubernetes está rodando, se houver algum disponível.

Para criar um serviço LoadBalancer via CLI, você pode usar o comando kubectl expose com a opção --type=LoadBalancer.

```
kubectl expose deployment meu-deployment --type=LoadBalancer --port=80 --target-port=8080
```

Caso queira criar um serviço LoadBalancer via YAML, você pode usar um arquivo como este:

```
apiVersion: v1
kind: Service
metadata:
  name: meu-service
spec:
  type: LoadBalancer
  selector:
    app: meu-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

Nada novo aqui, somente o fato que estamos pedindo para o Kubernetes criar um Service do tipo LoadBalancer.

ê deseja expor um serviço a ser acessado externamente por usuários ou sistemas que não estão dentro do seu cluster Kubernetes. Este tipo de serviço pode ser considerado quando:

Provedores de nuvem: Seu cluster Kubernetes está hospedado em um provedor de nuvem que suporta balanceadores de carga, como AWS, GCP, Azure, etc. Nesse caso, quando um serviço do tipo LoadBalancer é criado, um balanceador de carga é automaticamente provisionado no provedor de nuvem.

Tráfego externo: Você precisa que sua aplicação seja acessível fora do cluster. O LoadBalancer expõe uma IP acessível externamente que encaminha o tráfego para os pods do serviço.

É importante lembrar que o uso de LoadBalancers pode ter custos adicionais, pois você está usando recursos adicionais do seu cloud provider. Portanto, é sempre bom verificar os custos associados antes de implementar. ;)

### ExternalName

O tipo de serviço ExternalName é um pouco diferente dos outros tipos de serviço. Ele não expõe um conjunto de Pods, mas sim um nome de host externo. Por exemplo, você pode ter um serviço que expõe um banco de dados externo, ou um serviço que expõe um serviço de e-mail externo.

O tipo ExternalName é útil quando você deseja:

- **Criar um alias para um serviço externo:** Suponha que você tenha um banco de dados hospedado fora do seu cluster Kubernetes, mas você deseja que suas aplicações dentro do cluster se refiram a ele pelo mesmo nome, como se estivesse dentro do cluster. Nesse caso, você pode usar um ExternalName para criar um alias para o endereço do banco de dados.

- **Abstrair serviços dependentes do ambiente:** Outro uso comum para ExternalName é quando você tem ambientes diferentes, como produção e desenvolvimento, que possuem serviços externos diferentes. Você pode usar o mesmo nome de serviço em todos os ambientes, mas apontar para diferentes endereços externos.

Infelizmente ele é pouco usado por conta da falta de conhecimento das pessoas, mas com você não será assim, não é mesmo? :)

Vamos criar um exemplo de ExternalName usando o kubectl expose:

```
kubectl create service externalname meu-service --external-name meu-db.giropops.com.br
```

Onde estamos passando como parâmetro o nome do serviço, e o endereço externo que queremos expor para o cluster.

Agora, caso você queira criar um ExternalName via YAML, você pode usar um arquivo como este:

```
apiVersion: v1
kind: Service
metadata:
  name: meu-service
spec:
  type: ExternalName
  externalName: meu-db.giropops.com.br
```

Aqui estamos especificando o campo type como ExternalName, e também estamos especificando o campo externalName, que define o endereço externo que queremos expor para o cluster, nada de outro mundo. :)

Ahhh, e lembre-se que o ExternalName não tem suporte para selectors ou ports, pois ele não expõe um conjunto de Pods, mas sim um nome de host externo.

E lembre-se parte 2, o ExternalName aceita um nome de host válido de acordo com o DNS (RFC-1123), que pode ser um nome de domínio ou um endereço IP.

### Verificando os Services

Agora que já criamos alguns Services, está na hora de aprender como ver os detalhes deles.

Para visualizar os Services em execução no seu cluster Kubernetes na namespace default, você pode usar o comando kubectl get services. Esse comando listará todos os Services, junto com informações básicas como o tipo de serviço, o endereço IP e a porta.

```
kubectl get services
```

Caso queira pegar os Services de uma namespace específica, você pode usar o comando kubectl get services -n . Por exemplo:

```
kubectl get services -n kube-system
```

Para visualizar os Services em todas as namespaces, você pode usar o comando kubectl get services --all-namespaces ou kubectl get services -A.

```
kubectl get services -A
```

Agora, para ver os detalhes de um Endpoints específico, é simples como voar, basta usar o comando abaixo:

```
kubectl describe endpoints meu-service
```

### Removendo um Service

Agora que já vimos muita coisa sobre os Services, vamos aprender como removê-los.

Para remover um Service via CLI, você pode usar o comando kubectl delete service, passando o nome do Service como parâmetro. Por exemplo:

```
kubectl delete service meu-service
```

Para remover um Service via YAML, você pode usar o comando kubectl delete, passando o arquivo YAML como parâmetro. Por exemplo:

```
kubectl delete -f meu-service.yaml
```

