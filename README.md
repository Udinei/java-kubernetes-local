# java-kubernetes-local
Projeto desafio de construir um ambiente Kubernetes local 
para aprender a tecnologia sem medo de errar. Com a 
criação dos recursos necessários para fazer o deploy 
no cluster (kubernetes) e configurar uma aplicação spring boot
para fazer debug com a aplicação rodando no Kubernetes (minikube).

# Stack
- Java 15
- IntelliJ 11.09.1
- Docker 19.03.8  
- Mysql 5.6  
- Maven 3.5.4
- MiniKube v1.19.0
- kubectl v1.21.0
- Spring boot 3.2.1 
    - data rest
    - validation 
    - jpa
    - actuator
    - test
 - Prometheus (metricas)
 - Stern (logs)
 
Nota: Para detalhes como versão e dependências, ver arquivo pom.xml do projeto.

# Requisitos 
- SO Linux/Windows
- 2 CPUs ou mais
- 2 GB de memória livre
- 20 GB de espaço livre em disco
- Conexão de internet

# Estrutura do projeto

- k8s
  - app
  - mysql  
- src
    - main
        - java
            - config
            - controller
            - domain
            - persistence
            - service
        - resources
        
    - test
        -java

# Arquitetura da aplicação sem Docker
![](images/SO-browser-aplicacao-bd.png)

Aplicação e BD rodando sem container

#  Compilando a aplicação
<pre>maven clean install</pre>

# Executando a aplicação
<pre>java --enable-preview -jar target/java-kubernetes.jar</pre>

# Acessando a aplicação

http://localhost:8080/app/users

http://localhost:8080/app/hello

# Dockerizando a aplicação

### Definição da imagem docker do projeto
Criando arquivo "Dockerfile", que contém as definições de criação da 
imagem da aplicação, esse arquivo fica na raiz do projeto,
seu conteúdo diz:

- Crie uma imagem docker

- Use a imagem da versao do Java openjdk:15-alpine no docker

- Crie a pasta /usr/myapp

- Copie a aplicação compilada para a pasta criada acima

- Use a pasta criada acima, como diretorio de trabalho

- Libere a porta 8080

- Execute a aplicação

### Conteúdo do Dockerfile

<pre>
FROM openjdk:15-alpine

RUN mkdir /usr/myapp

COPY target/java-kubernetes.jar /usr/myapp/app.jar
WORKDIR /usr/myapp

EXPOSE 8080

ENTRYPOINT [ "sh", "-c", "java --enable-preview $JAVA_OPTS -jar app.jar" ]
</pre>

### Criando imagem Docker da aplicação

Na pasta da aplicação após criar o Dockerfile executar o comando abaixo,
Que força a criação da imagem, se a imagem existir, remova e
crie a imagem com o nome java-k8s, com todo conteúdo da pasta atual. 

<pre>docker build --force-rm -t java-k8s .</pre>

### Rodando uma imagem docker do BD Mysql

<pre>docker run --name mysql57 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_USER=java -e MYSQL_PASSWORD=1234 -e MYSQL_DATABASE=k8s_java -d mysql/mysql-server:5.7</pre>

### Rodando a imagem da aplicação, linkada com a imagem do BD Mysql  

<pre>docker run --name myapp -p 8080:8080 -d -e DATABASE_SERVER_NAME=mysql57 --link mysql57:mysql57 java-k8s:latest</pre>

# Aplicação e BD Dockerizados
![](images/app-bd-dockerizados.png)
Aplicação e BD rodando em container Dockers

# Rodando o Minikube (Kubernetes Local)

#### startando o mikube( kubernete local)
<pre>minikube -p dev.to start --cpus 2 --memory=4096</pre>

### stop a maquina minikube( kubernete local)
<pre>minikube -p dev.to stop</pre>

### delete a maquina minikube
<pre>minikube -p dev.to stop && minikube -p dev.to delete</pre>

### Inserindo addons "ingress" no minikube

O ingress serve para expor os serviços rodando, nos pods na rede externa ao minikube

exe: my-domain -> ingress -> SVC-1 -> POD-1
                           \-> SVC-2 -> POD-2

<pre>minikube -p dev.to addons enable ingress</pre>

### inserindo addons metric-server no minikube
<pre>minikube -p dev.to addons enable metrics-server</pre>

### Criando o namespace da aplicacao dev-to no minikube

O namespace dev-to fica separado do cluster (kube-system) padrao

<pre>kubectl create namespace dev-to</pre>

### Visualizando namespaces criados no minikube
<pre>kubectl get namespaces --show-labels</pre>

### Visualizando o IP da maquina minikube
<pre>minikube -p dev.to ip</pre>

### Visualizando Dashboard do minikube
<pre>minikube -p dev.to dashboard</pre>

### Verificando a versãodo do minikube
<pre>kubectl version</pre>


![](images/minikube-kubernetes.png)
Aplicação e BD rodando no minikube (Kubernetes Local)


# Rodando o Minikube (Kubernetes Local)

#### startando o mikube( kubernete local)
<pre>minikube -p dev.to start --cpus 2 --memory=4096</pre>

### stop a maquina minikube( kubernete local)
<pre>minikube -p dev.to stop</pre>

### delete a maquina minikube
<pre>minikube -p dev.to stop && minikube -p dev.to delete</pre>

Inserindo addons "ingress" no minikube, para expor os serviços rodando
nos pods na rede externa ao minikube exe: my-domain  ingress -> SVC-1 -> POD-1

<pre>minikube -p dev.to addons enable ingress</pre>

### inserindo addons metric-server no minikube
<pre>minikube -p dev.to addons enable metrics-server</pre>

### criando o namespace da aplicacao para o profile dev-to no minikube
 namespace separação do cluster (kube-system) padrao

<pre>kubectl create namespace dev-to</pre>

### Visualizando namespaces criados no minikube
<pre>kubectl get namespaces --show-labels</pre>

### Visualizando o IP da maquina minikube

<pre>minikube -p dev.to ip</pre>

### Visualizando minikube via dashboard
<pre>minikube -p dev.to dashboard</pre>

### Verificando a versãodo minikube
<pre>kubectl version</pre>

# Deploy da aplicação e serviços para dentro do kubernetes

Arquivos descritores definem as configurações da aplicação e serviços no minikube
na pasta k8s/mysql contem o mysql-deployment.yaml(cria um pod) que define o container do
mysql(nome da imagem,variaveis de ambiente para senhas, portas, etc... )
o arquivo mysql-service.yaml disponibiliza o POD com um serviço dentro da rede


## Aplicando descritor de deployment dentro do cluster do minikube

Dentro da pasta da aplicação, os decritores de deployment define os Pods e Replicas Sets

# Fazendo o deploy do POD para o mysql

<pre>kubectl apply -f k8s/mysql/</pre>

### Deploy da aplicacao para o kubernetes usando arquivos descritores da pasta k8s/app/

<pre>kubectl apply -f k8s/app/</pre>

### Visualizando os pods (menor unidade dentro do kubernete)
<pre>kubectl get pods -n dev-to</pre>

### Acessando o pod no minikube que esta rodando o BD
<pre>kubectl exec -it postgres-74bdd6978d-k8f9p bash</pre>

### Acessando o BD no pod com psql
<pre>psql -U user -d url_shortener_db</pre>

### Gerando e enviando a imagem da aplicação para o minikube

(funcionou no git bash)
<pre>eval $(minikube -p dev.to docker-env) && docker build --force-rm -t java-k8s .</pre>

### Copiando imagem para o cache do mikikute
<pre>minikube cache add java-k8s:latest</pre>

### Visualizando o cache do minikube
<pre>minikube cache list</pre>

### Acessando o container minikube via ssh
<pre>minikube -p dev.to ssh</pre>

### Visualizando serviços sendo executados no minikube
<pre>kubectl get services -n dev-to</pre>

### Solicitando url da aplicação "myapp" ao minikube
O comando abaixo, criará um tunel e sera gerado uma url para acessar a aplicação que esta rodando
no minikube via browser, o terminal onde foi executado o comando windows fica travado executando
o tunel

<pre>minikube -p dev.to service -n dev-to myapp --url</pre>


## Acessando a aplicação via namehosts
Localização do arquivo hosts:

Windows: 
<code>C:\Windows\System32\drivers\etc\hosts</code>

Linux:
<code> /etc/hosts</code>

Adicionar no file hosts o ip: 127.0.0.1 dev.local 
Nota: No windows, teoricamente deveria ser acessado assim:
ex: 127.0.0.1 dev.local , sem a necessidade de informar a 
porta na url, mas na prática não funciona e mesma deve ser informada
No browser:  <code>http://dev.local:49563/app/hello</code>

Somente linux: Adicionar ip do minikube exibido pelo comando: minikube -p dev.to ip
IP -> 192.168.49.2

Acessar no browser:<code> http://dev.local:49563/app/hello</code>


## Escalando a aplicação myapp,
no namespace dev-to, para 3 replicas  o arquivo: app-hpa - define a quantidade replicas de uma aplicação rodando no kubernetes
<pre>kubectl -n dev-to scale deployment/myapp --replicas=3

<pre>kubectl -n dev-to scale deployment/myapp --replicas=0 - Interrompe a execuções dos PODs de myapp

## visualizando as replicas escaladas(pods) no namespace dev-to
<pre>kubectl get pods -n dev-to

## Visualizando no terminal a alternância entre as aplicações escaladas no minikube
## A cada instante o kubernetes, troca a instancia que tratara as requisição
## enviadas pelo browser, curl ou outro sistema
<pre>
while true
do curl "http://dev.local:49563/app/hello"
echo
sleep 1
done
<pre>
# deletando uma instancia (pod) do minikube do namespace dev-to
<pre>kubectl delete pod -n dev-to myapp-b46d8cbc5-vrwvg


## Adicionando um novo usuario com uso da aplicação e banco rodando no minikube
<pre>curl --location --request POST 'http://dev.local:63197/app/users' --header 'Content-Type: application/json' --data-raw '{"name":"new user","birthDate": "2010-10-01"}'

# Preparando a porta do pod para ser utilizada no debug da aplicação que esta rodando no minikube
<pre>kubectl port-forward -n=dev-to <pod_name> 5005:5005
<pre>kubectl port-forward -n=dev-to myapp-b46d8cbc5-tq95q 5005:5005

# visualizando logs do pod
<pre>kubectl logs po/webapp-78c4f886f5-wtrl9

## Centralizando todos os logs dos PODs que estão rodando no kubernetes
## download do stern https://github.com/wercker/stern
<pre>stern -n dev-to myapp

##  Stern para windows
https://github.com/wercker/stern/releases
*[Asciinema.org](https://asciinema.org/a/263031)

## executando o stern e visualizando logs dos pods
<pre>stern_windows_amd64.exe -n dev-to myapp


## Local de criação das maquinas minikube
<pre><drive>\Users\<user>\.minikube\machines\minikube

## Referencias minikube
<pre>https://ahmet.im/blog/minikube-on-gke/

# HELP Docker
Visualizando logs da app em tempo real

<pre>docker logs -f myapp</pre>

# Stop containers docker

<pre>docker stop mysql57 myapp</pre>

# Removendo imagem do docker

<pre>docker rmi idimagemdocker </pre>

# Referências
* [Site instalação Minikube](https://minikube.sigs.k8s.io/docs/start/)
