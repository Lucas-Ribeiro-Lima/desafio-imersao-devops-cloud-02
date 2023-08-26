# Projeto rotten-potatoes

### Objetivo
O projeto rotten-potatoes é uma aplicação escrita em Python e tem como objetivo ser uma aplicação de exemplo pra trabalhar com o uso de containers.

### Configuração
Pra configurar a aplicação, é preciso ter um banco de dados Mongo e pra definir o acesso ao banco, configure as variáveis de ambiente abaixo:

MONGODB_DB => admin

MONGODB_HOST => mongo-service

MONGODB_PORT => 27017

MONGODB_USERNAME => potatoes

MONGODB_PASSWORD => potatoes

### Desafio

O manifesto do Kubernetes está no diretorio raiz deste repositório com a solução, adicionei o namespace para organizar melhor os deployments e services.

De acordo com a documentação do Mongo, as Env MONGODB_INIT_USERNAME e MONGODB_INIT_PASSWORD criam um usuario no DB admin, como solução precisei utilizar o mesmo banco para realizar a autenticação da aplicação

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
spec:
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo
        env:
          - name: MONGO_INITDB_ROOT_USERNAME
            value: potatoes
          - name: MONGO_INITDB_ROOT_PASSWORD
            value: potatoes
          - name: MONGO_INITDB_DATABASE
            value: "rottenpotatoes"
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 27017
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-service
spec:
  selector:
    app: mongo
  ports:
  - port: 27017
    targetPort: 27017
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rotten-potatoes-deployment
  namespace: imersao
spec:
  replica: 3
  selector:
    matchLabels:
      app: rotten-potatoes
  template:
    metadata:
      labels:
        app: rotten-potatoes
    spec:
      containers:
      - name: rotten-potatoes
        image: lucaslive974/rotten-potatoes:v1
        env:
            - name: MONGODB_DB
              value: "admin"
            - name: MONGODB_HOST
              value: mongo-service
            - name: MONGODB_PORT
              value: "27017"
            - name: MONGODB_USERNAME
              value: "potatoes"
            - name: MONGODB_PASSWORD
              value: "potatoes"
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 5000
---
apiVersion: v1
kind: Service
metadata:
  name: rotten-potatoes-service
  namespace: imersao
spec:
  selector:
    app: rotten-potatoes
  ports:
  - port: 80
    targetPort: 5000
    Nodeport: 30000
    type: NodePort
```

> Outra solução possivel seria adicionar o paramêtro &authSource=admin a URL de conexão do banco da aplicação.
