# Criando um cluster Kubernetes com o kubeadm

Nós iremos utilizar o kubeadm para configurar o nosso cluster. Nós iremos conhecer no detalhe como criar um cluster utilizando 03 instancias EC2 da AWS, mas você pode utilizar qualquer outro tipo de instância, desde que seja uma instância Linux, o importante é entender o processo de instalação do Kubernetes e como seus componentes trabalham juntos.

o kubeadm é uma ferramenta para criar e gerenciar um cluster Kubernetes em vários nós. Ele automatiza muitas das tarefas de configuração do cluster, incluindo a instalação do control plane e dos nodes.

### Pré Requisitos

- Linux

- 2 GB ou mais de RAM por máquina (menos de 2 GB não é recomendado)

- 2 CPUs ou mais

- Conexão de rede entre todas os nodes no cluster (pode ser via rede pública ou privada)

- Algumas portas precisam estar abertas para que o cluster funcione corretamente, as principais:

  - Porta 6443: É a porta padrão usada pelo Kubernetes API Server para se comunicar com os componentes do cluster. É a porta principal usada para gerenciar o cluster e deve estar sempre aberta.

  - Portas 10250-10255: Essas portas são usadas pelo kubelet para se comunicar com o control plane do Kubernetes. A porta 10250 é usada para comunicação de leitura/gravação e a porta 10255 é usada apenas para comunicação de leitura.

  - Porta 30000-32767: Essas portas são usadas para serviços NodePort que precisam ser acessíveis fora do cluster. O Kubernetes aloca uma porta aleatória dentro desse intervalo para cada serviço NodePort e redireciona o tráfego para o pod correspondente.

  - Porta 2379-2380: Essas portas são usadas pelo etcd, o banco de dados de chave-valor distribuído usado pelo control plane do Kubernetes. A porta 2379 é usada para comunicação de leitura/gravação e a porta 2380 é usada apenas para comunicação de eleição.

### Instalando o kubeadm

### Desativando o uso do swap no sistema

Primeiro, vamos desativar a utilização de swap no sistema. Isso é necessário porque o Kubernetes não trabalha bem com swap ativado:

```
sudo swapoff -a
```

### Carregando os módulos do kernel

Vamos criar o arquivo: /etc/modules-load.d/k8s.conf para carregar os módulos do kernel necessários para o funcionamento do Kubernetes:

```
overlay
br_netfilter

sudo modprobe overlay
sudo modprobe br_netfilter
```

### Configurando parâmetros do sistema

Em seguida, vamos configurar alguns parâmetros do sistema em: /etc/sysctl.d/k8s.conf. Isso garantirá que nosso cluster funcione corretamente:

```
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1

sudo sysctl --system
```

### Instalando os pacotes do Kubernetes

Hora de instalar os pacotes do Kubernetes!

```
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl
```

### Instalando o containerd

Em seguida, vamos instalar o containerd, que são essenciais para nosso ambiente Kubernetes:

```
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update && sudo apt-get install -y containerd.io
```

### Configurando o containerd

Agora, vamos configurar o containerd para que ele funcione adequadamente com o nosso cluster:

```
sudo containerd config default | sudo tee /etc/containerd/config.toml

sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl status containerd
```

### Habilitando o serviço do kubelet

Por fim, vamos habilitar o serviço do kubelet para que ele inicie automaticamente com o sistema:

```
sudo systemctl enable --now kubelet
```

### Configurando as portas

Antes de iniciar o cluster, lembre-se das portas que precisam estar abertas para que o cluster funcione corretamente. Precisamos ter as portas TCP 6443, 10250-10255, 30000-32767 e 2379-2380 abertas entre os nodes do cluster. No nosso exemplo, onde estaremos usando somente um node control plane, não precisamos nos preocupar com algumas quando temos mais de um node control plane, pois eles precisam se comunicar entre si para manter o estado do cluster, ou ainda as portas 30000-32767, que são usadas para serviços NodePort que precisam ser acessíveis fora do cluster. Elas nós podemos ir abrindo conforme a necessidade, conforme vamos criando os nossos serviços.

Por agora o que precisamos garantir são as portas TCP 6443 somente no control plane e as 10250-10255 abertas em todos nodes do cluster.

Em nosso exemplo vamos utilizar como CNI o Weave Net, que é um CNI que utiliza o protocolo de roteamento de pacotes do Kubernetes para criar uma rede entre os pods. Falarei mais sobre ele mais pra frente, mas como estamos falando das portas que são importantes para o cluster funcionar, precisamos abrir a porta TCP 6783 e as portas UDP 6783 e 6784, para que o Weave Net funcione corretamente.

Então já sabe, não esqueça de abrir as portas TCP 6443, 10250-10255 e 6783 no seu firewall.

### Inicializando o cluster

Agora que temos tudo configurado, vamos iniciar o nosso cluster:

```
sudo kubeadm init --pod-network-cidr=10.10.0.0/16 --apiserver-advertise-address=<O IP QUE VAI FALAR COM OS NODES>
```

Substitua <O IP QUE VAI FALAR COM OS NODES> pelo endereço IP da máquina que está atuando como control plane.

Após a execução bem-sucedida do comando acima, você verá uma mensagem informando que o cluster foi inicializado com sucesso e todos os detalhes de sua inicialização, conforme é possível ver na saída do comando:

```
[init] Using Kubernetes version: v1.34.3
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.31.8.103]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-01 localhost] and IPs [172.31.8.103 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-01 localhost] and IPs [172.31.8.103 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "super-admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/instance-config.yaml"
[patches] Applied patch of type "application/strategic-merge-patch+json" to target "kubeletconfiguration"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests"
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 1.001220348s
[control-plane-check] Waiting for healthy control plane components. This can take up to 4m0s
[control-plane-check] Checking kube-apiserver at https://172.31.8.103:6443/livez
[control-plane-check] Checking kube-controller-manager at https://127.0.0.1:10257/healthz
[control-plane-check] Checking kube-scheduler at https://127.0.0.1:10259/livez
[control-plane-check] kube-controller-manager is healthy after 2.27961303s
[control-plane-check] kube-scheduler is healthy after 3.066567782s
[control-plane-check] kube-apiserver is healthy after 4.502080078s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s-01 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8s-01 as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: pouyb8.65mp4apvef81y57b
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.31.8.103:6443 --token pouyb8.65mp4apvef81y57b \
	--discovery-token-ca-cert-hash sha256:e5f57bd2229031a5ae9095a2aa89e2b3ef62185330a74dc574c33e99e7915628 
```

Inclusive, você verá uma lista de comandos para configurar o acesso ao cluster com o kubectl. Copie e cole esse comando em seu terminal:

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Essa configuração é necessária para que o kubectl possa se comunicar com o cluster, pois quando estamos copiando o arquivo admin.conf para o diretório .kube do usuário, estamos copiando o arquivo com as permissões de root, esse é o motivo de executarmos o comando sudo chown $(id -u):$(id -g) $HOME/.kube/config para alterar as permissões do arquivo para o usuário que está executando o comando.

### Adicionando os demais nodes ao cluster

Agora que já temos o nosso cluster inicializado e já estamos entendendo muito bem o que é o arquivo admin.conf, chegou o momento de adicionar os demais nodes ao nosso cluster.

Para isso, vamos novamente utilizar o comando kubeadm, porém ao invés de executar o comando no node do control plane, nesse momento precisamos rodar o comando diretamente no node que queremos adicionar ao cluster.

Quando inicializamos o nosso cluster, o kubeadm nos mostrou o comando que precisamos executar no novos nodes, para que eles possam ser adicinados ao cluster como workers.

```
sudo kubeadm join 172.31.57.89:6443 --token if9hn9.xhxo6s89byj9rsmd \
    --discovery-token-ca-cert-hash sha256:ad583497a4171d1fc7d21e2ca2ea7b32bdc8450a1a4ca4cfa2022748a99fa477
```

Após executar o join em cada worker node, vá até o node que criamos para ser o control plane, e execute:

```
kubectl get nodes
```

```
NAME     STATUS     ROLES           AGE   VERSION
k8s-01   NotReady   control-plane   23m   v1.34.3
k8s-02   NotReady   <none>          96s   v1.34.3
k8s-03   NotReady   <none>          93s   v1.34.3
```

Agora você já consegue ver que os dois novos nodes foram adicionados ao cluster, porém ainda estão com o status Not Ready, pois ainda não instalamos o nosso plugin de rede para que seja possível a comunicação entre os pods. Vamos resolver isso agora. :)

### Instalando o Weave Net


Agora que o cluster está inicializado, vamos instalar o Weave Net:

```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

Aguarde alguns minutos até que todos os componentes do cluster estejam em funcionamento. Você pode verificar o status dos componentes do cluster com o seguinte comando:

```
kubectl get pods -n kube-system
```

```
kubectl get nodes
```

```
NAME     STATUS   ROLES           AGE   VERSION
k8s-01   Ready    control-plane   32m   v1.34.3
k8s-02   Ready    <none>          10m   v1.34.3
k8s-03   Ready    <none>          10m   v1.34.3
```

O Weave Net é um plugin de rede que permite que os pods se comuniquem entre si. Ele também permite que os pods se comuniquem com o mundo externo, como outros clusters ou a Internet. Quando o Kubernetes é instalado, ele resolve vários problemas por si só, porém quando o assunto é a comunicação entre os pods, ele não resolve. Por isso, precisamos instalar um plugin de rede para resolver esse problema.

Pronto, já temos o nosso cluster inicializado e o Weave Net instalado. Agora, vamos criar um Deployment para testar a comunicação entre os Pods.

```
kubectl create deployment nginx --image=nginx --replicas 3
```

```
kubectl get pods -o wide
```

```
NAME                     READY   STATUS    RESTARTS   AGE   IP          NODE     NOMINATED NODE   READINESS GATES
nginx-66686b6766-6ck9s   1/1     Running   0          99s   10.32.0.3   k8s-03   <none>           <none>
nginx-66686b6766-9tjm6   1/1     Running   0          99s   10.32.0.2   k8s-03   <none>           <none>
nginx-66686b6766-p282n   1/1     Running   0          99s   10.40.0.3   k8s-02   <none>           <none>
```

### Visualizando detalhes dos nodes

Agora que já temos o nosso clustes com 03 nodes, nós podemos visualizar os detalhes de cada um deles, e assim entender cada detalhe.

Para ver a descrição do node, basta executar o comando abaixo:

```
kubectl describe node k8s-01
```

```
Name:               k8s-01
Roles:              control-plane
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=k8s-01
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Tue, 30 Dec 2025 02:37:15 +0000
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  k8s-01
  AcquireTime:     <unset>
  RenewTime:       Tue, 30 Dec 2025 03:27:58 +0000
....
```
