# O DaemonSet

O DaemonSet é um objeto que garante que todos os nós do cluster executem uma réplica de um Pod, ou seja, ele garante que todos os nós do cluster executem uma cópia de um Pod.

O DaemonSet é muito útil para executar Pods que precisam ser executados em todos os nós do cluster, como por exemplo, um Pod que faz o monitoramento de logs, ou um Pod que faz o monitoramento de métricas.

Alguns casos de uso de DaemonSets são:

- Execução de agentes de monitoramento, como o Prometheus Node Exporter ou o Fluentd.
- Execução de um proxy de rede em todos os nós do cluster, como kube-proxy, Weave Net, Calico ou Flannel.
- Execução de agentes de segurança em cada nó do cluster, como Falco ou Sysdig.
Portanto, se nosso cluster possuir 3 nós, o DaemonSet vai garantir que todos os nós executem uma réplica do Pod que ele está gerenciando, ou seja, 3 réplicas do Pod.

Caso adicionemos mais um node ao cluster, o DaemonSet vai garantir que todos os nós executem uma réplica do Pod que ele está gerenciando, ou seja, 4 réplicas do Pod.

### Criando um DaemonSet

Vamos para o nosso primeiro exemplo, vamos criar um DaemonSet que vai garantir que todos os nós do cluster executem uma réplica do Pod do node-exporter, que é um exporter de métricas do Prometheus.

Para isso, vamos criar um arquivo chamado node-exporter-daemonset.yaml e vamos adicionar o seguinte conteúdo.

```
apiVersion: apps/v1 # Versão da API do Kubernetes do objeto
kind: DaemonSet # Tipo do objeto
metadata: # Informações sobre o objeto
  name: node-exporter # Nome do objeto
spec: # Especificação do objeto
  selector: # Seletor do objeto
    matchLabels: # Labels que serão utilizadas para selecionar os Pods
      app: node-exporter # Label que será utilizada para selecionar os Pods
  template: # Template do objeto
    metadata: # Informações sobre o objeto
      labels: # Labels que serão adicionadas aos Pods
        app: node-exporter # Label que será adicionada aos Pods
    spec: # Especificação do objeto, no caso, a especificação do Pod
      hostNetwork: true # Habilita o uso da rede do host, usar com cuidado
      containers: # Lista de contêineres que serão executados no Pod
      - name: node-exporter # Nome do contêiner
        image: prom/node-exporter:latest # Imagem do contêiner
        ports: # Lista de portas que serão expostas no contêiner
        - containerPort: 9100 # Porta que será exposta no contêiner
          hostPort: 9100 # Porta que será exposta no host
        volumeMounts: # Lista de volumes que serão montados no contêiner, pois o node-exporter precisa de acesso ao /proc e /sys
        - name: proc # Nome do volume
          mountPath: /host/proc # Caminho onde o volume será montado no contêiner
          readOnly: true # Habilita o modo de leitura apenas
        - name: sys # Nome do volume 
          mountPath: /host/sys # Caminho onde o volume será montado no contêiner
          readOnly: true # Habilita o modo de leitura apenas
      volumes: # Lista de volumes que serão utilizados no Pod
      - name: proc # Nome do volume
        hostPath: # Tipo de volume 
          path: /proc # Caminho do volume no host
      - name: sys # Nome do volume
        hostPath: # Tipo de volume
          path: /sys # Caminho do volume no host
```

Agora vamos criar o DaemonSet utilizando o arquivo de manifesto.

```
kubectl apply -f node-exporter-daemonset.yaml
```

Agora vamos verificar se o DaemonSet foi criado.

```
kubectl get daemonset
```

Como podemos ver, o DaemonSet foi criado com sucesso.

```
NAME            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
node-exporter   2         2         2       2            2           <none>          36s
```

Caso você queira verificar os Pods que o DaemonSet está gerenciando, basta executar o comando abaixo.

```
kubectl get pods -l app=node-exporter
```

Somente para lembrar, estamos utilizando o parâmetro -l para filtrar os Pods que possuem a label app=node-exporter, que é o caso do nosso DaemonSet.

Como podemos ver, o DaemonSet está gerenciando 2 Pods, um em cada nó do cluster.

```
NAME                  READY   STATUS    RESTARTS   AGE
node-exporter-klbjc   1/1     Running   0          5m5s
node-exporter-rxlwb   1/1     Running   0          5m5s
```

Os nossos Pods do node-exporter foram criados com sucesso, agora vamos verificar se eles estão sendo executados em todos os nós do cluster.

```
kubectl get pods -o wide -l app=node-exporter
```

Com o comando acima, podemos ver em qual nó cada Pod está sendo executado.

```

NAME                  READY   STATUS    RESTARTS   AGE    IP           NODE                      NOMINATED NODE   READINESS GATES
node-exporter-klbjc   1/1     Running   0          8m5s   172.18.0.5   kind-multinodes-worker2   <none>           <none>
node-exporter-rxlwb   1/1     Running   0          8m5s   172.18.0.4   kind-multinodes-worker    <none>           <none>
```

Como podemos ver, os Pods do node-exporter estão sendo executados em todos os dois nós do cluster.

Para ver os detalhes do DaemonSet, basta executar o comando abaixo.

```
kubectl describe daemonset node-exporter
```

O comando acima vai retornar uma saída parecida com a abaixo.


```
Name:           node-exporter
Namespace:      default
Selector:       app=node-exporter
Node-Selector:  <none>
Labels:         <none>
Annotations:    deprecated.daemonset.template.generation: 1
Desired Number of Nodes Scheduled: 2
Current Number of Nodes Scheduled: 2
Number of Nodes Scheduled with Up-to-date Pods: 2
Number of Nodes Scheduled with Available Pods: 2
Number of Nodes Misscheduled: 0
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=node-exporter
  Containers:
   node-exporter:
    Image:        prom/node-exporter:latest
    Port:         9100/TCP
    Host Port:    9100/TCP
    Environment:  <none>
    Mounts:
      /host/proc from proc (ro)
      /host/sys from sys (ro)
  Volumes:
   proc:
    Type:          HostPath (bare host directory volume)
    Path:          /proc
    HostPathType:  
   sys:
    Type:          HostPath (bare host directory volume)
    Path:          /sys
    HostPathType:  
  Node-Selectors:  <none>
  Tolerations:     <none>
Events:
  Type    Reason            Age   From                  Message
  ----    ------            ----  ----                  -------
  Normal  SuccessfulCreate  10m   daemonset-controller  Created pod: node-exporter-rxlwb
  Normal  SuccessfulCreate  10m   daemonset-controller  Created pod: node-exporter-klbjc
```

### Removendo um DaemonSet

Para remover o DaemonSet é bem simples, basta executar o comando kubectl delete daemonset <nome-do-daemonset>.

```
kubectl delete daemonset node-exporter
```

Ou ainda você pode remover o DaemonSet através do manifesto.

```
kubectl delete -f node-exporter-daemonset.yaml
```