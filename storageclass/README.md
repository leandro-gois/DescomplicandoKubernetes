# Volumes no Kubernetes

Quando estamos falando sobre volumes no Kubernetes, precisamos entender que temos basicamente dois tipos de volumes, os ephemeral volumes e os persistent volumes.

### EmpytDir

Um volume do tipo EmptyDir é um volume que é criado no momento em que o Pod é criado, e ele é destruído quando o Pod é destruído, ou seja, ele é um volume temporário.

Chame o arquivo de pod-emptydir.yaml.

```
apiVersion: v1 # versão da API do Kubernetes
kind: Pod # tipo de objeto que estamos criando
metadata: # metadados do Pod
  name: giropops # nome do Pod
spec: # especificação do Pod
  containers: # lista de containers
  - name: girus # nome do container 
    image: ubuntu # imagem do container
    args: # argumentos que serão passados para o container
    - sleep # usando o comando sleep para manter o container em execução
    - infinity # o argumento infinity faz o container esperar para sempre
    volumeMounts: # lista de volumes que serão montados no container
    - name: primeiro-emptydir # nome do volume
      mountPath: /giropops # diretório onde o volume será montado 
  volumes: # lista de volumes
  - name: primeiro-emptydir # nome do volume
    emptyDir: # tipo do volume
      sizeLimit: 256Mi # tamanho máximo do volume
```

Agora vamos criar o Pod.

```
kubectl create -f pod-emptydir.yaml
```

Agora vamos verificar se o Pod foi criado.

```
kubectl get pods
```

Você pode ver a saída do comando kubectl describe pod giropops para ver o volume que foi criado.

```
kubectl describe pod giropops
```

```
Volumes:
  primeiro-emptydir:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  256Mi
```

Agora vamos para dentro do container.

```
kubectl exec -it giropops -- bash
```

Agora vamos criar um arquivo dentro do diretório /giropops.

```
touch /giropops/FUNCIONAAAAAA
```

Pronto, o nosso arquivo foi criado dentro do diretório /giropops, que é um diretório dentro do volume do tipo EmptyDir.

Se você digitar mount, vai ver que o diretório /giropops está montado certinho dentro de nosso container.

Quando você remover o Pod, o volume do tipo EmptyDir também será removido.

```
kubectl delete pod giropops
```

Vamos criar o Pod novamente.

```
kubectl create -f pod-emptydir.yaml
```

Pod criado, agora vamos para dentro do container.

```
kubectl exec -it ubuntu -- bash
```

Vamos verificar se o arquivo que criamos anteriormente ainda existe.

ls /giropops

Como você pode ver, o arquivo que criamos anteriormente não existe mais, pois o volume do tipo EmptyDir foi destruído quando o Pod foi destruído.

#### Storage Class

Uma StorageClass no Kubernetes é um objeto que descreve e define diferentes classes de armazenamento disponíveis no cluster. Essas classes de armazenamento podem ser usadas para provisionar dinamicamente PersistentVolumes (PVs) de acordo com os requisitos dos PersistentVolumeClaims (PVCs) criados pelos usuários.

Para você ver os Storage Classes disponíveis no seu cluster, basta executar o seguinte comando:

```
kubectl get storageclass
```

```
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  21m
```

Como você pode ver, no Kind, o provisionador padrão é o rancher.io/local-path, que cria volumes PersistentVolume no diretório do host.

Já no EKS, o provisionador padrão é o kubernetes.io/aws-ebs, que cria volumes PersistentVolume no EBS da AWS.

```
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2 (default)   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  6h5m
```

Vamos ver os detalhes do nosso Storage Class padrão:

```
kubectl describe storageclass standard
```

```
Name:            standard
IsDefaultClass:  Yes
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"name":"standard"},"provisioner":"rancher.io/local-path","reclaimPolicy":"Delete","volumeBindingMode":"WaitForFirstConsumer"}
,storageclass.kubernetes.io/is-default-class=true
Provisioner:           rancher.io/local-path
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>
```

Uma coisa que podemos ver é que o nosso Storage Class está com a opção IsDefaultClass como Yes, o que significa que ele é o Storage Class padrão do nosso cluster, com isso todos os Persistent Volume Claims que não tiverem um Storage Class definido, irão utilizar esse Storage Class como padrão.

Vamos criar um novo Storage Class para o nosso cluster Kubernetes no kind, com o nome "giropops", e vamos definir o provisionador como "kubernetes.io/host-path", que cria volumes PersistentVolume no diretório do host.

```
kubectl apply -f storageclass.yaml
```

```
storageclass.storage.k8s.io/giropops created
```

Pronto! Agora nós temos um novo Storage Class criado no nosso cluster Kubernetes no kind, com o nome "giropops", e com o provisionador "kubernetes.io/no-provisioner", que cria volumes PersistentVolume no diretório do host.

Para saber mais detalhes sobre o Storage Class que criamos, execute o seguinte comando:

```
kubectl describe storageclass giropops
```

```
Name:            giropops
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"giropops"},"provisioner":"kubernetes.io/no-provisioner","reclaimPolicy":"Retain","volumeBindingMode":"WaitForFirstConsumer"}

Provisioner:           kubernetes.io/no-provisioner
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Retain
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>
```

Lembrando que criamos esse Storage Class com o provisionador "kubernetes.io/no-provisioner", mas você pode criar um Storage Class com qualquer provisionador que você quiser, como o "kubernetes.io/aws-ebs", que cria volumes PersistentVolume no EBS da AWS.

#### PV - Persistent Volume

O PV é um objeto que representa um recurso de armazenamento físico em um cluster Kubernetes. Ele pode ser um disco rígido em um nó do cluster, um dispositivo de armazenamento em rede (NAS) ou mesmo um serviço de armazenamento em nuvem, como o AWS EBS ou Google Cloud Persistent Disk.

Agora que já sabemos o que é um PV, vamos entender como nós podemos utilizar o kubectl para gerenciar os PVs.

Primeira coisa, vamos listar os PVs que temos no nosso cluster:

```
kubectl get pv -A
```

```
No resources found
```

Com o comando acima estamos listando todos os PVs que temos no nosso cluster, e como podemos ver, não temos nenhum PV criado, ainda. :)

Vamos resolver isso, bora criar um PV?

Para isso, vamos criar um arquivo chamado pv.yaml:

```
apiVersion: v1 # Versão da API do Kubernetes
kind: PersistentVolume # Tipo de objeto que estamos criando, no caso um PersistentVolume
metadata: # Informações sobre o objeto
  name: meu-pv # Nome do nosso PV
  labels:
    storage: local
spec: # Especificações do nosso PV
  capacity: # Capacidade do PV
    storage: 1Gi # 1 Gigabyte de armazenamento
  accessModes: # Modos de acesso ao PV
    - ReadWriteOnce # Modo de acesso ReadWriteOnce, ou seja, o PV pode ser montado como leitura e escrita por um único nó
  persistentVolumeReclaimPolicy: Retain # Política de reivindicação do PV, ou seja, o PV não será excluído quando o PVC for excluído
  hostPath: # Tipo de armazenamento que vamos utilizar, no caso um hostPath
    path: "/mnt/data" # Caminho do hostPath, do nosso nó, onde o PV será criado
  storageClassName: standard # Nome da classe de armazenamento que será utilizada
```

```
kubectl apply -f pv.yaml
```

```
persistentvolume/meu-pv created
```

Vamos listar o nosso PV para ver se ele foi criado corretamente.

```
kubectl get pv
```

```
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
meu-pv   1Gi        RWO            Retain           Available           standard                10s
```

Vamos ver os detalhes do nosso PV.

```
kubectl describe pv meu-pv
```

```
Name:            meu-pv
Labels:          storage=local
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    standard
Status:          Available
Claim:           
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:         
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /mnt/data
    HostPathType:  
Events:            <none>
```

Dessa forma estamos criando o PV utilizando o provisionador hostPath, que é um provisionador para ser utilizado em testes e desenvolvimento, já que os dados armazenados só estão disponíveis no node específico, por isso bora para mais um exemplo, mas agora utilizando o provisionador nfs, que é um sistema de arquivos de rede que permite compartilhar arquivos entre várias máquinas na rede.

 

Primeira coisa que vamos fazer é criar o diretório que será compartilhado entre os nodes do cluster. Lembrando que para esse exemplo, estou utilizando uma máquina Linux para criar o compartilhamento NFS, mas você pode utilizar qualquer outro sistema operacional, desde que ele tenha suporte ao NFS.

```
mkdir /mnt/nfs
```

Precisamos instalar os pacotes nfs-kernel-server e nfs-common para que o servidor NFS e o cliente NFS sejam instalados.

```
sudo apt-get install nfs-kernel-server nfs-common
```

Vamos editar o arquivo /etc/exports, que é o arquivo de configuração do NFS, e adicionar o diretório que será compartilhado entre os nodes do cluster.

```
sudo vi /etc/exports
```

```
/mnt/nfs *(rw,sync,no_root_squash,no_subtree_check)
```

Agora vamos falar para o NFS que o diretório /mnt/nfs está disponível para ser compartilhado.

```
sudo exportfs -arv
```

Maravilha! Vamos agora verificar se o NFS está funcionando corretamente.

```
showmount -e
```

```
Export list for localhost:
/mnt/nfs *
```

Agora que já temos o nosso NFS funcionando, vamos criar o nosso StorageClass para o provisionador nfs.

Para esse exemplo, vamos criar um arquivo chamado storageclass-nfs.yaml e adicionar o seguinte conteúdo.

```
apiVersion: storage.k8s.io/v1 # Versão da API do Kubernetes
kind: StorageClass # Tipo de objeto que estamos criando, no caso um StorageClass
metadata: # Informações sobre o objeto
  name: nfs # Nome do nosso StorageClass
provisioner: kubernetes.io/no-provisioner # Provisionador que será utilizado para criar o PV
reclaimPolicy: Retain # Política de reivindicação do PV, ou seja, o PV não será excluído quando o PVC for excluído
volumeBindingMode: WaitForFirstConsumer
parameters: # Parâmetros que serão utilizados pelo provisionador
  archiveOnDelete: "false" # Parâmetro que indica se os dados do PV devem ser arquivados quando o PV for excluído
```

O Kubernetes não possui um provisionador nfs nativo, então não é possível fazer com que o provisionador kubernetes.io/no-provisioner crie um PV utilizando um servidor NFS automaticamente, para que isso seja possível, precisamos utilizar um provisionador nfs externo, mas isso não é o foco nesse momento, então vamos criar o nosso PV manualmente, afinal de contas, já estamos experts em PVs, certo?

Bora lá!

Então já podemos criar o PV e associa-lo ao Storage Class, e para isso vamos criar um novo arquivo chamado pv-nfs.yaml e adicionar o seguinte conteúdo.

```
apiVersion: v1 # Versão da API do Kubernetes
kind: PersistentVolume # Tipo de objeto que estamos criando, no caso um PersistentVolume
metadata: # Informações sobre o objeto
  name: meu-pv-nfs # Nome do nosso PV
  labels:
    storage: nfs # Label que será utilizada para identificar o PV
spec: # Especificações do nosso PV
  capacity: # Capacidade do PV
    storage: 1Gi # 1 Gigabyte de armazenamento
  accessModes: # Modos de acesso ao PV
    - ReadWriteOnce # Modo de acesso ReadWriteOnce, ou seja, o PV pode ser montado como leitura e escrita por um único nó
  persistentVolumeReclaimPolicy: Retain # Política de reivindicação do PV, ou seja, o PV não será excluído quando o PVC for excluído
  nfs: # Tipo de armazenamento que vamos utilizar, no caso o NFS
    server: IP_DO_SERVIDOR_NFS # Endereço do servidor NFS
    path: "/mnt/nfs" # Compartilhamento do servidor NFS
  storageClassName: nfs # Nome da classe de armazenamento que será utilizada
```

Agora vamos criar o nosso PV.

```
kubectl apply -f pv-nfs.yaml
```

```
persistentvolume/meu-pv created
```

### PVC - Persistent Volume Claim

O PVC é uma solicitação de armazenamento feita pelos usuários ou aplicativos no cluster Kubernetes. Ele permite que os usuários solicitem um volume específico, com base em tamanho, tipo de armazenamento e outras características. 

Todo PVC é associado a um Storage Class ou a um Persistent Volume. O Storage Class é um objeto que descreve e define diferentes classes de armazenamento disponíveis no cluster. Já o Persistent Volume é um recurso que representa um volume de armazenamento disponível para ser usado pelo cluster.

Vamos criar o nosso primeiro PVC para o PV que criamos anteriormente.

Para isso, vamos criar um arquivo chamado pvc.yaml e adicionar o seguinte conteúdo:

```
apiVersion: v1 # versão da API do Kubernetes
kind: PersistentVolumeClaim # tipo de recurso, no caso, um PersistentVolumeClaim
metadata: # metadados do recurso
  name: meu-pvc # nome do PVC
spec: # especificação do PVC
  accessModes: # modo de acesso ao volume
    - ReadWriteOnce # modo de acesso RWO, ou seja, somente leitura e escrita por um nó
  resources: # recursos do PVC
    requests: # solicitação de recursos
      storage: 1Gi # tamanho do volume que ele vai solicitar
  storageClassName: nfs # nome da classe de armazenamento que será utilizada
  selector: # seletor de labels
    matchLabels: # labels que serão utilizadas para selecionar o PV
      storage: nfs # label que será utilizada para selecionar o PV
```

Vamos criar o nosso PVC.

```
kubectl apply -f pvc.yaml
```

```
persistentvolumeclaim/meu-pvc created
```

Pronto, PVC criado! Vamos conferir se ele foi criado corretamente.

```
kubectl get pvc
```

```
NAME      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
meu-pvc   Pending                                      nfs            5s
```

Está lá! Porém o status dele está como Pending, vamos ver se tem alguma informação que nos ajude a entender o que está acontecendo.

```
kubectl describe pvc meu-pvc
```

Name:          meu-pvc
Namespace:     default
StorageClass:  nfs
Status:        Pending
Volume:        
Labels:        <none>
Annotations:   <none>
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      
Access Modes:  
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason                Age                 From                         Message
  ----    ------                ----                ----                         -------
  Normal  WaitForFirstConsumer  15s (x4 over 1m5s)  persistentvolume-controller  waiting for first consumer to be created before binding
```

Repare na parte dos eventos, lá diz que o PVC está esperando o primeiro consumidor ser criado antes de ser vinculado. O que isso significa?

Significa que o PVC está esperando que um Pod seja criado para que ele possa ser vinculado ao PV, então bora criar o nosso Pod.

Vamos usar o nosso já conhecido Nginx como exemplo, então vamos criar um arquivo chamado pod.yaml e adicionar o seguinte conteúdo:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: meu-pvc
      mountPath: /usr/share/nginx/html
  volumes:
  - name: meu-pvc
    persistentVolumeClaim:
      claimName: meu-pvc
```

Esse trecho do arquivo pod.yaml é responsável por montar o volume meu-pvc no caminho /usr/share/nginx/html dentro do container.

    volumeMounts: # montando o volume no container
    - name: meu-pvc  # nome do volume
      mountPath: /usr/share/nginx/html # caminho onde o volume será montado no container
  volumes: # definindo o volume que será utilizado pelo Pod
  - name: meu-pvc # nome do volume
    persistentVolumeClaim: # tipo de volume, no caso, um PersistentVolumeClaim
      claimName: meu-pvc # nome do PVC

Esclarecido! Vamos criar o nosso Pod.

```
kubectl apply -f pod.yaml
```

```
pod/nginx-pod created
```

Bora ver se está tudo certo com o nosso Pod.

```
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          21s
```

Parece que sim! Agora vamos ver se o nosso PVC foi vinculado ao PV.

```
kubectl get pvc
```

```
NAME      STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
meu-pvc   Bound    meu-pv-nfs   1Gi        RWO            nfs            3m8s
```

Opa! Temos um vinculo!

Vamos ver se tem alguma coisa nova na saída do get pv.

```
kubectl get pv
```

```
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS   REASON   AGE
meu-pv-nfs   1Gi        RWO            Retain           Bound    default/meu-pvc   nfs                     3m42s
```

Agora sim! Temos um PV com o status Bound e um PVC com o status Bound também. \o/

E pra finalizar o nosso primeiro teste, vamos ver se o nosso Pod está utilizando o nosso volume.

```
kubectl describe pod nginx-pod
```

```
Name:             nginx-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-linuxtips-worker/172.18.0.4
Start Time:       Tue, 11 Apr 2023 01:47:48 +0200
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.2.3
IPs:
  IP:  10.244.2.3
Containers:
  nginx:
    Container ID:   containerd://b5946958f63c392c8a77b06811f7859113a1dd260ebcc2113579af6b32c4f549
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:2ab30d6ac53580a6db8b657abf0f68d75360ff5cc1670a85acb5bd85ba1b19c0
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 11 Apr 2023 01:47:50 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from meu-pvc (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8874f (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  meu-pvc:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  meu-pvc
    ReadOnly:   false
  kube-api-access-8874f:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  8s    default-scheduler  Successfully assigned default/nginx-pod to kind-linuxtips-worker
  Normal  Pulling    7s    kubelet            Pulling image "nginx:latest"
  Normal  Pulled     6s    kubelet            Successfully pulled image "nginx:latest" in 799.112685ms (799.119928ms including waiting)
  Normal  Created    6s    kubelet            Created container nginx
  Normal  Started    6s    kubelet            Started container nginx
```

Pronto! O nosso Pod está utilizando o nosso volume! Todo o conteúdo que for criado dentro do Pod será armazenado no nosso volume, e mesmo que o Pod seja removido, o conteúdo não será perdido.

Agora vamos testar o nosso volume. Vamos criar um arquivo HTML simples no diretório /mnt/data do nosso servidor NFS.

```
echo "<h1>GIROPOPS STRIGUS GIRUS</h1>" > /mnt/data/index.html
```

Agora vamos ver se o nosso arquivo foi criado.

```
kubectl exec -it nginx-pod -- ls /usr/share/nginx/html
```

```
index.html
```

Está lá! Vamos dar um curl de dentro do Pod para ver se o Ngix está servindo o nosso arquivo.

```
kubectl exec -it nginx-pod -- curl localhost
```

```
<h1>GIROPOPS STRIGUS GIRUS</h1>
```


