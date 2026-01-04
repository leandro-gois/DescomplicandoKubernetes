# O que são Secrets?

Os Secrets fornecem uma maneira segura e flexível de gerenciar informações sensíveis, como senhas, tokens OAuth, chaves SSH e outros dados que você não quer expor nas configurações de seus aplicaçãos.

### Tipos de Secrets

Existem vários tipos de Secrets que você pode usar, dependendo de suas necessidades específicas. Abaixo estão alguns dos tipos mais comuns de Secrets:

- __Opaque Secrets__ - são os Secrets mais simples e mais comuns. Eles armazenam dados arbitrários, como chaves de API, senhas e tokens. Os Opaque Secrets são codificados em base64 quando são armazenados no Kubernetes, mas não são criptografados. Eles podem ser usados para armazenar dados confidenciais, mas não são seguros o suficiente para armazenar informações altamente sensíveis, como senhas de banco de dados.

- __kubernetes.io/service-account-token__ - são usados para armazenar tokens de acesso de conta de serviço. Esses tokens são usados para autenticar Pods com o Kubernetes API. Eles são montados automaticamente em Pods que usam contas de serviço.

- __kubernetes.io/dockercfg e kubernetes.io/dockerconfigjson__ - são usados para armazenar credenciais de registro do Docker. Eles são usados para autenticar Pods com um registro do Docker. Eles são montados em Pods que usam imagens de container privadas.

- __kubernetes.io/tls, kubernetes.io/ssh-auth e kubernetes.io/basic-auth__ - são usados para armazenar certificados TLS, chaves SSH e credenciais de autenticação básica, respectivamente. Eles são usados para autenticar Pods com outros serviços.

- __bootstrap.kubernetes.io/token__ - são usados para armazenar tokens de inicialização de cluster. Eles são usados para autenticar nós com o plano de controle do Kubernetes.

### Antes de criar um Secret, o Base64


A codificação Base64 converte os dados binários em uma string de texto ASCII. Essa string contém apenas caracteres que são considerados seguros para uso em URLs, o que torna a codificação Base64 útil para codificar dados que estão sendo enviados pela Internet.

No entanto, a codificação Base64 não é uma forma de criptografia e não deve ser usada como tal. Em particular, ela não fornece nenhuma confidencialidade. Qualquer um que tenha acesso à string codificada pode facilmente decodificá-la e recuperar os dados originais. Entender isso é importante para que você não armazene informações sensíveis em um formato codificado em Base64, pois isso não é seguro.

Para codificar uma string em Base64, você pode usar o comando base64 no Linux. Por exemplo, para codificar a string giropops em Base64, você pode executar o seguinte comando:

```
echo -n 'leandro' | base64
```

O comando acima irá retornar a string bGVhbmRybw==.

Para decodificar uma string em Base64, você pode usar o comando base64 novamente, mas desta vez com a opção -d. Por exemplo, para decodificar a string Z2lyb3BvcHM= em Base64, você pode executar o seguinte comando:

```
echo -n 'bGVhbmRybw==' | base64 -d
```

O comando acima irá retornar a string leandro.

### Criando nosso primeiro Secret

Para criar um Secret do tipo Opaque, você precisa criar um arquivo YAML que defina o Secret. Por exemplo, você pode criar um arquivo chamado giropops-secret.yaml com o seguinte conteúdo:

```
apiVersion: v1
kind: Secret
metadata:
  name: giropops-secret
type: Opaque
data: # Inicio dos dados
    username: bGVhbmRybw==
    password: bmVnbw==
```

Pronto, o nosso primeiro Secret foi criado! Agora você pode ver o Secret usando o comando kubectl get:

```
kubectl get secret giropops-secret
```

```
NAME              TYPE     DATA   AGE
giropops-secret   Opaque   2      10s
```

Caso você queira ver os dados armazenados no Secret, você pode usar o comando kubectl get com a opção -o yaml:

```
kubectl get secret giropops-secret -o yaml
```

```
apiVersion: v1
data:
  password: bmVnbw==
  username: bGVhbmRybw==
kind: Secret
metadata:
  creationTimestamp: "2023-05-21T10:38:39Z"
  name: giropops-secret
  namespace: default
  resourceVersion: "466"
  uid: ac816e95-8896-4ad4-9e64-4ee8258a8cda
type: Opaque
```

Você também pode ver os detalhes do Secret usando o comando kubectl describe:

```
kubectl describe secret giropops-secret
```

```
Name:         giropops-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  8 bytes
username:  14 bytes
```

### Usando o nosso primeiro Secret

Precisamos criar o nosso Pod, então vamos lá! Crie um arquivo chamado giropops-pod.yaml com o seguinte conteúdo:

```
apiVersion: v1
kind: Pod
metadata:
  name: giropops-pod
spec:
  containers:
  - name: giropops-container
    image: nginx
    env: # Inicio da definição das variáveis de ambiente
    - name: USERNAME # Nome da variável de ambiente que será usada no Pod
      valueFrom: # Inicio da definição de onde o valor da variável de ambiente será buscado
        secretKeyRef: # Inicio da definição de que o valor da variável de ambiente será buscado em um Secret, através de uma chave
          name: giropops-secret # Nome do Secret que contém o valor da variável de ambiente que será usada no Pod
          key: username # Nome da chave do campo do Secret que contém o valor da variável de ambiente que será usada no Pod
    - name: PASSWORD # Nome da variável de ambiente que será usada no Pod
      valueFrom: # Inicio da definição de onde o valor da variável de ambiente será buscado
        secretKeyRef: # Inicio da definição de que o valor da variável de ambiente será buscado em um Secret, através de uma chave
          name: giropops-secret # Nome do Secret que contém o valor da variável de ambiente que será usada no Pod
          key: password # Nome da chave do campo do Secret que contém o valor da variável de ambiente que será usada no Pod
```

Agora vamos criar o Pod usando o comando kubectl apply:

```
kubectl apply -f giropops-pod.yaml
```

```
pod/giropops-pod created
```

Agora vamos verificar se o Pod foi criado e se os Secrets foram injetados no Pod:

```
kubectl get pods
```

```
NAME           READY   STATUS    RESTARTS   AGE
giropops-pod   1/1     Running   0          2m
```

Para verificar se os Secrets foram injetados no Pod, você pode usar o comando kubectl exec para executar o comando env dentro do container do Pod:

```
kubectl exec giropops-pod -- env
```

```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=giropops-pod
NGINX_VERSION=1.29.4
NJS_VERSION=0.9.4
NJS_RELEASE=1~trixie
PKG_RELEASE=1~trixie
DYNPKG_RELEASE=1~trixie
USERNAME=leandro
PASSWORD=nego
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
HOME=/root
```

### Criando um Secret para armazenar credenciais Docker
  
Para que o Kubernetes possa acessar o Docker Hub, você precisa criar um Secret que armazene o nome de usuário e a senha da sua conta no Docker Hub, e depois você precisa configurar o Kubernetes para usar esse Secret.

Quando você executa docker login e tem a sua autenticação bem sucedida, o Docker cria um arquivo chamado config.json no diretório ~/.docker/ do seu usuário, e esse arquivo contém o nome de usuário e a senha da sua conta no Docker Hub, e é esse arquivo que você precisa usar para criar o seu Secret.

Primeiro passo é pegar o conteúdo do seu arquivo config.json e codificar em base64, e para isso você pode usar o comando base64:

```
base64 ~/.docker/config.json

QXF1aSB0ZW0gcXVlIGVzdGFyIG8gY29udGXDumRvIGRvIHNldSBjb25maWcuanNvbiwgY29pc2EgbGluZGEgZG8gSmVmaW0=
```

Então vamos lá! Crie um arquivo chamado dockerhub-secret.yaml com o seguinte conteúdo:

```
apiVersion: v1
kind: Secret
metadata:
  name: docker-hub-secret # nome do Secret
type: kubernetes.io/dockerconfigjson # tipo do Secret, neste caso é um Secret que armazena credenciais Docker
data:
  .dockerconfigjson: |  # substitua este valor pelo conteúdo do seu arquivo config.json codificado em base64
    QXF1aSB0ZW0gcXVlIGVzdGFyIG8gY29udGXDumRvIGRvIHNldSBjb25maWcuanNvbiwgY29pc2EgbGluZGEgZG8gSmVmaW0=
```

Agora vamos criar o Secret usando o comando kubectl apply:

```
kubectl apply -f dockerhub-secret.yaml

secret/docker-hub-secret created
```

Para listar o Secret que acabamos de criar, você pode usar o comando kubectl get:

```
kubectl get secrets

NAME                TYPE                             DATA   AGE
docker-hub-secret   kubernetes.io/dockerconfigjson   1      1s
```

Um coisa importante, sempre quando você precisar criar um Pod que precise utilizar uma imagem Docker privada do Docker Hub, você precisa configurar o Pod para usar o Secret que armazena as credenciais do Docker Hub, e para isso você precisa usar o campo spec.imagePullSecrets no arquivo YAML do Pod.

```
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod
spec:
  containers:
  - name: meu-container
    image: minha-imagem-privada
  imagePullSecrets: # campo que define o Secret que armazena as credenciais do Docker Hub
  - name: docker-hub-secret # nome do Secret
```

Perceba a utilização do campo spec.imagePullSecrets no arquivo YAML do Pod, e o campo name que define o nome do Secret que armazena as credenciais do Docker Hub. É somente isso que você precisa fazer para que o Kubernetes possa acessar o Docker Hub.

### Criando um Secret TLS

Para criar um certificado TLS e uma chave privada, você pode usar o comando openssl:

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout chave-privada.key -out certificado.crt
```

Com o certificado TLS e a chave privada criados, vamos criar o nosso Secret, é somente para mudar um pouco, vamos criar o Secret usando o comando:

```
kubectl create secret tls meu-servico-web-tls-secret --cert=certificado.crt --key=chave-privada.key

secret/meu-servico-web-tls-secret created
```

Vamos ver se o Secret foi criado:

```
kubectl get secrets
NAME                         TYPE                             DATA   AGE
meu-servico-web-tls-secret   kubernetes.io/tls                2      4s
```

Caso você queira ver o conteúdo do Secret, você pode usar o comando kubectl get secret com o parâmetro -o yaml:

```
kubectl get secret meu-servico-web-tls-secret -o yaml
```

Agora você pode usar esse Secret para ter o Nginx rodando com HTTPS, e para isso você precisa usar o campo spec.tls no arquivo YAML do Pod:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
        - containerPort: 443
      volumeMounts:
        - name: nginx-config-volume
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
        - name: nginx-tls
          mountPath: /etc/nginx/tls
    volumes:
    - name: nginx-config-volume
      configMap:
        name: nginx-config
    - name: nginx-tls
      secret:
        secretName: meu-servico-web-tls-secret
        items:
          - key: certificado.crt
            path: certificado.crt
          - key: chave-privada.key
            path: chave-privada.key
```

# ConfigMaps

ConfigMaps são usados para armazenar dados de configuração, como variáveis de ambiente, arquivos de configuração, etc. Eles são muito úteis para armazenar dados de configuração que podem ser usados por vários Pods.

Os ConfigMaps são uma maneira eficiente de desacoplar os parâmetros de configuração das imagens de container. Isso permite que você tenha a mesma imagem de container em diferentes ambientes, como desenvolvimento, teste e produção, com diferentes configurações.

Bora continuar o nosso exemplo de como usar o Nginx com HTTPS, mas agora usando um ConfigMap para armazenar o arquivo de configuração do Nginx.

Vamos criar o arquivo de configuração do Nginx chamado nginx.conf, que vai ser usado pelo ConfigMap:

```
events { } # configuração de eventos

http { # configuração do protocolo HTTP, que é o protocolo que o Nginx vai usar
  server { # configuração do servidor
    listen 80; # porta que o Nginx vai escutar
    listen 443 ssl; # porta que o Nginx vai escutar para HTTPS e passando o parâmetro ssl para habilitar o HTTPS

    ssl_certificate /etc/nginx/tls/certificado.crt; # caminho do certificado TLS
    ssl_certificate_key /etc/nginx/tls/chave-privada.key; # caminho da chave privada

    location / { # configuração da rota /
      return 200 'Bem-vindo ao Nginx!\n'; # retorna o código 200 e a mensagem Bem-vindo ao Nginx!
      add_header Content-Type text/plain; # adiciona o header Content-Type com o valor text/plain
    } 
  }
}
```

Agora vamos criar o ConfigMap nginx-config com o arquivo de configuração do Nginx:

```
kubectl create configmap nginx-config --from-file=nginx.conf
```
O que estamos fazendo é criar um ConfigMap chamado nginx-config com o conteúdo do arquivo nginx.conf. Podemos fazer a mesma coisa através de um manifesto, como no exemplo abaixo:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    events { }

    http {
      server {
        listen 80;
        listen 443 ssl;

        ssl_certificate /etc/nginx/tls/certificado.crt;
        ssl_certificate_key /etc/nginx/tls/chave-privada.key;

        location / {
          return 200 'Bem-vindo ao Nginx!\n';
          add_header Content-Type text/plain;
        }
      }
    }
```

O arquivo é bem parecido com os manifestos do Secret, mas com algumas diferenças:

- O campo kind é ConfigMap ao invés de Secret.
- O campo data é usado para definir o conteúdo do ConfigMap, e o campo data é um mapa de chave-valor, onde a chave é o nome do arquivo e o valor é o conteúdo do arquivo. Usamos o caractere | para definir o valor do campo data como um bloco de texto, e assim podemos definir o conteúdo do arquivo nginx.conf sem a necessidade de usar o caractere \n para quebrar as linhas do arquivo.
Agora é só aplicar o manifesto acima:

```
kubectl apply -f nginx-config.yaml
```

Para ver o conteúdo do ConfigMap que criamos, bastar executar o comando:

```
kubectl get configmap nginx-config -o yaml
```

Você também pode usar o comando kubectl describe configmap nginx-config para ver o conteúdo do ConfigMap, mas o comando kubectl get configmap nginx-config -o yaml é bem mais completo.

Agora que já temos o nosso ConfigMap criado, vamos aplicar o manifesto que criamos no capítulo anterior, vou colar aqui o manifesto para facilitar:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    - containerPort: 443
    volumeMounts:
    - name: nginx-config-volume # nome do volume que vamos usar para montar o arquivo de configuração do Nginx
      mountPath: /etc/nginx/nginx.conf # caminho onde o arquivo de configuração do Nginx vai ser montado
      subPath: nginx.conf # nome do arquivo de configuração do Nginx
    - name: nginx-tls # nome do volume que vamos usar para montar o certificado TLS e a chave privada
      mountPath: /etc/nginx/tls # caminho onde o certificado TLS e a chave privada vão ser montados
  volumes: # lista de volumes que vamos usar no Pod
  - name: nginx-config-volume # nome do volume que vamos usar para montar o arquivo de configuração do Nginx
    configMap: # tipo do volume que vamos usar
      name: nginx-config # nome do ConfigMap que vamos usar
  - name: nginx-tls # nome do volume que vamos usar para montar o certificado TLS e a chave privada
    secret: # tipo do volume que vamos usar
      secretName: meu-servico-web-tls-secret # nome do Secret que vamos usar
      items: # lista de arquivos que vamos montar, pois dentro da secret temos dois arquivos, o certificado TLS e a chave privada
        - key: tls.crt # nome do arquivo que vamos montar, nome que está no campo `data` do Secret
          path: certificado.crt # nome do arquivo que vai ser montado, nome que vai ser usado no campo `ssl_certificate` do arquivo de configuração do Nginx
        - key: tls.key # nome do arquivo que vamos montar, nome que está no campo `data` do Secret
          path: chave-privada.key # nome do arquivo que vai ser montado, nome que vai ser usado no campo `ssl_certificate_key` do arquivo de configuração do Nginx
```

Agora é só aplicar o manifesto acima:

```
kubectl apply -f nginx.yaml
```

Listando os Pods:

```
kubectl get pods
```

Agora precisamos criar um Service para expor o Pod que criamos:

```
kubectl expose pod nginx
```

Listando os Services:

```
kubectl get services
```

Bora fazer o port-forward para testar se o nosso Nginx está funcionando:

```
kubectl port-forward service/nginx 4443:443
```

O comando acima vai fazer o port-forward da porta 443 do Service nginx para a porta 4443 do seu computador, o port-forward salvando a nossa vida novamente! :)

Vamos usar o curl para testar se o nosso Nginx está funcionando:

```
curl -k https://localhost:4443
```

```
Bem-vindo ao Nginx!
```

Caso você queira tornar um ConfigMap imutável, você pode usar o campo immutable no manifesto do ConfigMap, como no exemplo abaixo:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  immutable: true # torna o ConfigMap imutável
data:
  nginx.conf: |
    events { }

    http {
      server {
        listen 80;
        listen 443 ssl;

        ssl_certificate /etc/nginx/tls/certificado.crt;
        ssl_certificate_key /etc/nginx/tls/chave-privada.key;

        location / {
          return 200 'Bem-vindo ao Nginx!\n';
          add_header Content-Type text/plain;
        }
      }
    }
```

Com isso, não será possível alterar o ConfigMap, e se você tentar alterar o ConfigMap, o Kubernetes vai retornar um erro.

Caso você queira deixar o ConfigMap em uma namespace específica, você pode usar o campo namespace no manifesto do ConfigMap, como no exemplo abaixo:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: minha-namespace # deixa o ConfigMap na namespace `minha-namespace`
data:
  nginx.conf: |
    events { }

    http {
      server {
        listen 80;
        listen 443 ssl;

        ssl_certificate /etc/nginx/tls/certificado.crt;
        ssl_certificate_key /etc/nginx/tls/chave-privada.key;

        location / {
          return 200 'Bem-vindo ao Nginx!\n';
          add_header Content-Type text/plain;
        }
      }
    }
```

