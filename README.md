# Lista-de-Exercícios-Kubernetes
Resolução de uma lista de atividades sobre Kubernetes no Killercoda

https://killercoda.com/kubernetes
# Atividades
* Crie um pod chamado "my-pod" usando uma imagem simples como "nginx" e verifique seu estado com os comandos de monitoramento do Kubernetes.
* Implante um Deployment chamado "my-deployment" com três réplicas de uma aplicação baseada na imagem "httpd". Atualize a imagem do Deployment para uma versão mais recente.
* Crie um ConfigMap chamado "app-config" com uma variável de configuração personalizada. Monte o ConfigMap em um pod e verifique se o valor foi aplicado corretamente.
* Crie um Secret chamado "app-secret" contendo informações sensíveis. Injete o Secret como uma variável de ambiente em um pod e teste se está acessível.
* Configure um PersistentVolume de 1Gi de armazenamento local e vincule-o a um PersistentVolumeClaim. Monte o volume em um pod e salve arquivos para verificar a persistência.
* Crie um serviço do tipo ClusterIP para um Deployment chamado "backend" e teste a conectividade interna entre pods usando o nome do serviço.
* Implante um Job chamado "batch-job" que execute um comando simples e termine. Verifique os logs do Job para confirmar sua execução.
* Crie um Horizontal Pod Autoscaler para um Deployment chamado "hpa-deployment" e configure-o para escalar com base no uso de CPU. Aumente a carga e observe o escalonamento.
* Crie um serviço do tipo NodePort para expor externamente um Deployment chamado "webapp". Acesse o serviço usando o endereço IP do Minikube e a porta atribuída.
* Crie um pod chamado "restart-pod" com a política de reinício configurada como "OnFailure". Provoque uma falha no pod e observe seu comportamento.

# Crie um pod chamado "my-pod" usando uma imagem simples como "nginx" e verifique seu estado com os comandos de monitoramento do Kubernetes.
1. Criar um arquivo "my-pod.yaml" com um editor de sua escolha:
```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
```
2. Executar um comando para executar o pod:
```
kubectl apply -f my-pod.yaml
```
3. E para verificar o estado do pod, utilize:
```
kubectl get pods
```
![image](https://github.com/user-attachments/assets/1db3cb18-e38e-4cfb-8d0f-e2d709c64338)

# Implante um Deployment chamado "my-deployment" com três réplicas de uma aplicação baseada na imagem "httpd". Atualize a imagem do Deployment para uma versão mais recente.
1. Crie um arquivo chamado my-deployment.yaml:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: httpd-container
        image: httpd:2.4
        ports:
        - containerPort: 80
```
2. Executar o Deployment:
````
kubectl apply -f my-deployment.yaml
````
3. Utilize o comando "kubectl get deployments" para verificar se foram feitas as réplicas:
![image](https://github.com/user-attachments/assets/274450a5-cb9f-4f95-ac21-7b05408db4ac)
4. Realize o comando a seguir para atualizar a imagem:
````
kubectl set image deployment/my-deployment httpd-container=httpd:2.4.57
````
# Crie um ConfigMap chamado "app-config" com uma variável de configuração personalizada. Monte o ConfigMap em um pod e verifique se o valor foi aplicado corretamente.
1. Criar um arquivo chamado "app-config":
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  custom.message: "Hello from ConfigMap"
```
2. Rodar o comando para executar o ConfigMap:
```
kubectl apply -f app-config.yaml
```
3. Crie um arquivo chamado pod-with-configmap.yaml:
````
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
  - name: app-container
    image: busybox
    command: [ "sh", "-c", "echo $(cat /etc/config/custom.message) && sleep 3600" ]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
````
4. Rodar o comando para executar o pod:
````
kubectl apply -f pod-with-configmap.yaml
````
5. Depois, verifique os logs para verificar a mensagem do configmap:
````
kubectl logs configmap-pod
````
![image](https://github.com/user-attachments/assets/45ad200b-d9e5-4f93-942f-95e6c9c1f9ce)
# Crie um Secret chamado "app-secret" contendo informações sensíveis. Injete o Secret como uma variável de ambiente em um pod e teste se está acessível.
1. Crie um arquivo chamado "app-secret.yaml"
````
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=  # Base64 encoded for "admin"
  password: cGFzc3dvcmQ=  # Base64 encoded for "password"

````
2. Rodar o arquivo:
````
kubectl apply -f app-secret.yaml
````
3. Criar um pod que usa o secret como variavel de ambiente:
````
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app-container
    image: busybox
    command: [ "sh", "-c", "echo Username: $USERNAME, Password: $PASSWORD && sleep 3600" ]
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
````
4. Rodar o pod:
````
kubectl apply -f pod-with-secret.yaml
````
5. Verificar os logs para verificar o conteúdo das varíaveis de ambiente:
````
kubectl logs secret-pod
````
![image](https://github.com/user-attachments/assets/6d0d81b4-a571-41c5-9b6d-927d30d423ab)

# Configure um PersistentVolume de 1Gi de armazenamento local e vincule-o a um PersistentVolumeClaim. Monte o volume em um pod e salve arquivos para verificar a persistência.
1. Crie um arquivo "local-pv.yaml":
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"  # Diretório local no nó de Kubernetes
````
2. Crie o Persistent Volume:
````
kubectl apply -f local-pv.yaml
````
3. Crie um arquivo chamado "local-pvc.yaml":
````
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
````
4. Crie o Persistent Volume Claim
````
kubectl apply -f local-pvc.yaml
````
5. Crie um pod que monta o volume chamado "pod-with-pv.yaml":
````
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  containers:
  - name: app-container
    image: busybox
    command: [ "sh", "-c", "echo 'Hello from PV' > /data/test.txt && cat /data/test.txt && sleep 3600" ]
    volumeMounts:
    - mountPath: "/data"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: local-pvc
````
6. Crie o arquivo:
````
kubectl apply -f pod-with-pv.yaml
````
7. Obtenha o nome do pod em execução e visualize seu conteudo
````
kubectl exec <nome-do-pod> -- cat /data/test.txt
````
8. A saída será:
````
Hello from PV
````
9. Para testar a Persistência, exclua e crie denovo o pod:
````
kubectl delete pod <nome-do-pod>
kubectl apply -f pod-with-pv.yaml
kubectl exec pv-pod -- cat /data/test.txt
````
# Crie um serviço do tipo ClusterIP para um Deployment chamado "backend" e teste a conectividade interna entre pods usando o nome do serviço.
1. Crie um arquivo chamado backend-deployment.yaml:
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend-container
        image: nginx
        ports:
        - containerPort: 80
````
2. Aplique o Deployment:
````
kubectl apply -f backend-deployment.yaml
````
3. Criar o Serviço do tipo ClusterIP chamado "backend-service.yaml"
````
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
````
4. Aplicar o Serviço:
````
kubectl apply -f backend-service.yaml
````
5. Crie um pod de teste para verificar a conectividade interna, e o acesse internamente:
````
kubectl run test-pod --image=busybox --restart=Never --command -- sh -c "sleep 3600"
kubectl exec -it test-pod -- sh
````
6.Dentro do Pod, use wget ou curl para se conectar ao Serviço pelo nome DNS:
````
wget -qO- http://backend-service
````
A saída será o conteúdo HTML da página padrão do Nginx, indicando que a conectividade foi bem-sucedida.

# Implante um Job chamado "batch-job" que execute um comando simples e termine. Verifique os logs do Job para confirmar sua execução.
1. Primeiro, crie um arquivo chamado "batch-job.yaml"
````
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  template:
    spec:
      containers:
      - name: simple-job
        image: busybox
        command: ["sh", "-c", "echo 'Hello from batch job' && sleep 5"]
      restartPolicy: Never
  backoffLimit: 4
````
2. Rode o código
````
kubectl apply -f batch-job.yaml
````
3. Encontre o nome do Pod gerado pelo Job, e use esse nome para verificar os logs:
````
kubectl get pods
kubectl logs batch-job-xxxx
````
4. A saída será:
````
Hello from batch job
````
# Crie um Horizontal Pod Autoscaler para um Deployment chamado "hpa-deployment" e configure-o para escalar com base no uso de CPU. Aumente a carga e observe o escalonamento.
1. Crie um arquivo chamado "hpa-deployment.yaml":
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hpa-app
  template:
    metadata:
      labels:
        app: hpa-app
    spec:
      containers:
      - name: hpa-container
        image: k8s.gcr.io/hpa-example  # Imagem de exemplo que usa CPU
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "200m"
          limits:
            cpu: "500m"
````
2. Rodar o código:
````
kubectl apply -f hpa-deployment.yaml
````
3.  Criar o Horizontal Pod Autoscaler que manterá a CPU em 50%:
````
kubectl autoscale deployment hpa-deployment --cpu-percent=50 --min=1 --max=5
````
4. Crie um Pod de teste para gerar carga no serviço e dentro dele, execute um loop para gerar solicitações:
````
kubectl run -it load-generator --image=busybox --restart=Never -- sh
while true; do wget -q -O- http://hpa-deployment; done
````
5. Use o comando abaixo para monitorar as réplicas do Deployment e o comportamento do HPA:
````
kubectl get hpa
kubectl get deployments
````
Você verá o número de réplicas aumentar conforme a carga sobre a CPU aumenta.
# Crie um serviço do tipo NodePort para expor externamente um Deployment chamado "webapp". Acesse o serviço usando o endereço IP e a porta atribuída.
1. Crie um arquivo chamado "webapp-deployment.yaml":
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp-container
        image: nginx
        ports:
        - containerPort: 80
````
2. Aplicar o deployment:
````
kubectl apply -f webapp-deployment.yaml
````
3. Crie um serviço do tipo NodePort chamado "Webapp-service.yaml":
```
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  selector:
    app: webapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080  # Escolha uma porta dentro da faixa permitida (30000-32767)
  type: NodePort
````
4. Aplique o Serviço:
````
kubectl apply -f webapp-service.yaml
`````
5. Para acessar o serviço, obtenha o endereço ip dos nós:
````
kubectl get nodes -o wide
````
6.Use o IP do nó e a porta atribuída para acessar o serviço no navegador e você verá a página de boas-vindas padrão do Nginx se tudo estiver configurado corretamente.
# Crie um pod chamado "restart-pod" com a política de reinício configurada como "OnFailure". Provoque uma falha no pod e observe seu comportamento.
1. Primeiro, crie o pod chamado "restart-pod.yaml":
````
apiVersion: v1
kind: Pod
metadata:
  name: restart-pod
spec:
  restartPolicy: OnFailure
  containers:
  - name: fail-container
    image: busybox
    command: ["sh", "-c", "echo 'Simulating failure'; exit 1"]
````
2. Aplicar o pod:
```
kubectl apply -f restart-pod.yaml
```
3. Use o comando para verificar os detalhes do Pod:
````
kubectl describe pod restart-pod
````
Você verá um campo mostrando quantas vezes o Pod foi reiniciado.

4. Veja os logs do Pod:
````
kubectl logs restart-pod
````
O log mostrará a mensagem Simulating failure antes da falha e reinício.
