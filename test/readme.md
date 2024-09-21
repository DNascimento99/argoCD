## Comandos 

~~~bash
kubectl proxy = http://localhost:8001/api/v1/namespaces/default/services/svr-lab1:80/proxy/ = validar se ha problema na conexão do pod com o servico
kubectl get nodes -o wide = validar ip e detalhes do node
kubectl get pods = visualiza os pods
kubectl get pods -o = visualiza os pods com detalhes
kubectl get nodes = visualiza os nodes
kubectl attach -it nome_pod se conecta caso nao aja um terminal iterativo
kubectl exec -it nome_pod --bash = fornece shell interativo no container
kubectl describe pods = visualiza os detalhes de um pod
kubectl logs = visualiza os logs de um pod 
kubectl logs -f = visualiza os logs de um pod continuamente
kubectl get events = visualiza os eventos de um pod
kubectl rollout status = obtem status de um rollout
kubectl rollout restart = reinicia todos os pods de um deployment
kubectl rollout undo = retorna a versão anterior de um deployment
kubectl rollout pause = pausa um rollout para revisoes
kubectl rollout resume = retoma o rollout
kubectl port-forward svc/svr-lab1 8080:80 = faz direcionamento de ports do node para maquina local
kubectl apply -f = apenas valida o estado do arquivo, se o objeto ja existir no cluster, não fazendo alterações
kubectl apply --dry-run = utilizado para ver o que o apply fará sem fazer alterações
kubectl apply -f deployment.yaml 
kubectl apply -f services.yaml

kubectl top nodes = quantidade de recurso consumido

~~~

## Obserações

* ClusterIP = Ip interno do cluster não acessível externamente, normalmente usado para banco de dados ou backends de comunicação interna
* NodePort = Acessível externamente
* Loadbalancer = Utilizado em núvem, pois é configurado a um loadbalancer externo.
* Ingress = Utilizado para controle de trafego, ou seja tudo que chegar em um ponto será encaminhado ao meu destino em uma porta especifica\
*EX: o que chegar em / será encaminhado ao teste.com em uma porta especifica, faz controle http/https, gerencia certificados ssl/tls, expõe o servico externamente com várias regras de acesso em um unico ponto de entrada*
* Para acessar um pod, o mesmo deve ser acessado usando o ip do node e a porta do service
**Para que ele funcione precisa de um controle instalado no cluster.**
* Helm = Gerenciador de pacotes para Kubernetes que facilita a instalação e o gerenciamento de aplicações.
* Operator =  é uma maneira poderosa de automatizar a administração de aplicativos complexos no Kubernetes, proporcionando automação, consistência e escalabilidade através de controladores personalizados que estendem a API do Kubernetes.

* Selector = Identifica e agrupa objetos com base em labels de chave/valor, se um pod está com o selector, env = dev e eu crio um service que o selector define env = dev o service irá gerenciar o trafego desse pod.
* ReplicaSets = Quantidade de replicas de um pod
* Services = Serve para agrupar pods e acessa-los a partir de um unico ponto, pois se um pod for destruido e trocar de ip, o service ainda continua a abstrair isso através do selector
* NetworkPolicies = Politica de rede para um cluster, pods ou de um namespace para outro namespace.
* NameSpace = Serve para isolar um Pod dos demais pods de um cluster, ou seja um pod não se comunicara com outro ao menos que eu aplique uma regra ou permita em uma network policies que isso possa acontecer.
* AccesListBasedRole(RBAC) = Politica de controle de acesso para recursos dentro de um cluster.
* RoleBiding = Regra que associa-se a role.
* Spec = define as configurações de um deploy, como numero de replicas, estrategia de atualização, dockerfile utilizado, entre outros.

## kubelet
O kubelet é o componente responsável por assegurar que os containers estejam sendo executados corretamente e que o estado dos Pods seja mantido conforme as especificações fornecidas. Ele é um dos componentes fundamentais no gerenciamento da infraestrutura de containers em um cluster Kubernetes.

## ConfigMap
ConfigMap oferece uma maneira eficiente e flexível de gerenciar e fornecer configurações para aplicações em um cluster Kubernetes.
* O que for configurado no configmap, reflere automaticamente em um pod, sem necessidade de reinicia-lo
* mantem as configurações separadas do código da aplicação
* Config map, será chamado pelo nome do configmap e o valor pelo valor definico no configmap EX:
~~~yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
  namespace: default
data:
  db.host: "example-database"
  db.port: "5432"
~~~
ao chama-lo a chave será db-config e o valor será db.host e db.port

## worker
* Worker Nodes são as máquinas (virtuais ou físicas) onde suas aplicações são executadas e gerenciadas. Eles são responsáveis pela execução de pods e containers.
* Master Nodes gerenciam e orquestram o cluster Kubernetes, coordenando a execução de pods, a escalabilidade e a manutenção do estado desejado do cluster.

##
* etcd onde os objetos do cluster são armazenados
* scheduler, responsavel por alocar os pods nos nodes

## deployment
~~~yaml
apiVersion: apps/v1 # Define a versão da API Kubernetes usada para o recurso. "apps/v1" é apropriado para Deployments.
kind: Deployment # Tipo de recurso, que gerencia a implantação e escalabilidade de Pods.
metadata: # Metadados relacionados ao Deployment.
  name: aplicacao-exemplo # Nome único do Deployment dentro do namespace, usado para identificação.
  namespace: producao # Namespace onde o Deployment está localizado. Usado para isolar recursos e ambientes dentro do cluster.
  labels: # Rótulos para categorizar e organizar o Deployment.
    app: aplicacao-exemplo # Rótulo que identifica a aplicação associada ao De5ployment.
    ambiente: producao # Rótulo que indica o ambiente de operação (por exemplo, produção).
spec: # Especificações do Deployment.
  replicas: 5 # Número desejado de réplicas de Pods a serem mantidas.
  selector: # Define como os Pods serão selecionados e gerenciados por este Deployment.
    matchLabels: # Rótulos que devem coincidir com os rótulos dos Pods gerenciados.
      app: aplicacao-exemplo # Rótulo que identifica os Pods que pertencem a este Deployment.
  template: # Modelo para os Pods gerenciados pelo Deployment.
    metadata: # Metadados associados aos Pods.
      labels: # Rótulos aplicados aos Pods para identificação e seleção.
        app: aplicacao-exemplo # Rótulo que associa os Pods ao Deployment.
    spec: # Especificações dos Pods.
      containers: # Lista de contêineres que serão executados dentro dos Pods.
      - name: exemplo-container # Nome do contêiner.
        image: nginx:latest # Imagem do contêiner a ser utilizada.
        imagePullPolicy: Always # Política que define quando a imagem do contêiner deve ser atualizada (Always, IfNotPresent, Never).
        ports: # Portas expostas pelo contêiner.
        - containerPort: 80 # Porta que o contêiner expõe.
        env: # Variáveis de ambiente definidas para o contêiner.
        - name: DATABASE_HOST # Nome da variável de ambiente.
          valueFrom: # Fonte do valor da variável de ambiente.
            secretKeyRef: # Referência a um Secret para obter o valor da variável.
              name: db-secret # Nome do Secret que contém a credencial.
              key: db_host # Chave dentro do Secret que fornece o valor.
        resources: # Requisições e limites de recursos para o contêiner.
          requests: # Requisições mínimas de recursos.
            memory: "128Mi" # Memória mínima requerida.
            cpu: "250m" # CPU mínima requerida.
          limits: # Limites máximos de recursos.
            memory: "512Mi" # Limite máximo de memória.
            cpu: "1000m" # Limite máximo de CPU.
        readinessProbe: # Verificação de prontidão para garantir que o contêiner esteja pronto para receber tráfego.
          httpGet: # Tipo de verificação de prontidão (HTTP GET).
            path: /healthz # Caminho HTTP a ser verificado.
            port: 80 # Porta HTTP a ser verificada.
          initialDelaySeconds: 10 # Tempo em segundos para aguardar antes de iniciar a verificação de prontidão.
          periodSeconds: 5 # Intervalo entre as verificações de prontidão.
        livenessProbe: # Verificação de vitalidade para garantir que o contêiner esteja funcionando corretamente.
          httpGet: # Tipo de verificação de vitalidade (HTTP GET).
            path: /healthz # Caminho HTTP a ser verificado.
            port: 80 # Porta HTTP a ser verificada.
          initialDelaySeconds: 15 # Tempo em segundos para aguardar antes de iniciar a verificação de vitalidade.
          periodSeconds: 10 # Intervalo entre as verificações de vitalidade.
        securityContext: # Contexto de segurança para o contêiner.
          runAsUser: 1000 # UID para o qual o contêiner será executado.
          runAsGroup: 3000 # GID para o grupo ao qual o contêiner pertence.
          readOnlyRootFilesystem: true # Define se o sistema de arquivos raiz será somente leitura.
          capabilities: # Capacidades adicionais a serem removidas do contêiner.
            drop: # Lista de capacidades Linux a serem removidas.
            - ALL # Remove todas as capacidades adicionais do contêiner.
        volumeMounts: # Montagens de volumes dentro do contêiner.
        - name: config-volume # Nome do volume a ser montado.
          mountPath: /etc/config # Caminho dentro do contêiner onde o volume será montado.
      volumes: # Volumes que serão montados nos Pods.
      - name: config-volume # Nome do volume.
        configMap: # Tipo de volume baseado em ConfigMap.
          name: exemplo-configmap # Nome do ConfigMap a ser montado.
      affinity: # Preferências de agendamento para Pods.
        nodeAffinity: # Afinidade de nó para controlar onde os Pods devem ser agendados.
          requiredDuringSchedulingIgnoredDuringExecution: # Requisitos obrigatórios para agendamento de Pods.
            nodeSelectorTerms: # Termos de seleção para nós.
            - matchExpressions: # Expressões de correspondência para rótulos de nós.
              - key: disktype # Chave do rótulo.
                operator: In # Operador de correspondência.
                values: # Valores aceitos para a chave.
                - ssd # Valor do rótulo para o tipo de disco.
        podAffinity: # Afinidade de Pods para definir quais Pods devem ser agendados juntos.
          preferredDuringSchedulingIgnoredDuringExecution: # Preferências para agendamento de Pods.
          - weight: 1 # Peso da preferência.
            podAffinityTerm: # Termos de afinidade de Pods.
              labelSelector: # Seletor de rótulos para Pods.
                matchLabels: # Rótulos que devem coincidir.
                  app: aplicacao-exemplo # Rótulo que identifica os Pods com afinidade.
              topologyKey: "kubernetes.io/hostname" # Chave de topologia usada para afinidade de Pods.
        podAntiAffinity: # Anti-afinidade de Pods para definir quais Pods não devem ser agendados juntos.
          requiredDuringSchedulingIgnoredDuringExecution: # Requisitos obrigatórios para anti-afinidade.
          - labelSelector: # Seletor de rótulos para Pods.
              matchExpressions: # Expressões de correspondência para rótulos.
              - key: app # Chave do rótulo.
                operator: In # Operador de correspondência.
                values: # Valores aceitos para a chave.
                - aplicacao-exemplo # Valor do rótulo para a anti-afinidade.
            topologyKey: "kubernetes.io/hostname" # Chave de topologia usada para anti-afinidade de Pods.
      tolerations: # Tolerâncias para taints em nós.
      - key: "node.kubernetes.io/not-ready" # Chave do taint a ser tolerado.
        operator: "Exists" # Operador para o taint.
        effect: "NoSchedule" # Efeito da tolerância.
        tolerationSeconds: 300 # Tempo em segundos que a tolerância é aplicada.
      - key: "dedicated" # Chave do taint para uma tolerância específica.
        operator: "Equal" # Operador para o taint.
        value: "production" # Valor do taint para a tolerância.
        effect: "NoSchedule" # Efeito da tolerância.
      terminationGracePeriodSeconds: 30 # Tempo de espera antes de forçar a finalização de um Pod.
      serviceAccountName: exemplo-serviceaccount # Nome da conta de serviço associada aos Pods.

---
apiVersion: autoscaling/v1 # Versão da API para recursos de autoescalonamento (HorizontalPodAutoscaler).
kind: HorizontalPodAutoscaler # Tipo de recurso para ajuste automático do número de réplicas de Pods com base em métricas.
metadata:
  name: aplicacao-exemplo-hpa # Nome único do HorizontalPodAutoscaler dentro do namespace.
  namespace: producao # Namespace onde o HPA está localizado.
spec:
  scaleTargetRef: # Referência ao recurso que será escalado.
    apiVersion: apps/v1 # Versão da API para o Deployment alvo.
    kind: Deployment # Tipo de recurso que será escalado (Deployment).
    name: aplicacao-exemplo # Nome do Deployment alvo.
  minReplicas: 3 # Número mínimo de réplicas que o HPA deve manter.
  maxReplicas: 10 # Número máximo de réplicas que o HPA pode criar.
  targetCPUUtilizationPercentage: 80 # Porcentagem alvo de utilização de CPU para ajustar o número de réplicas.

---
apiVersion: v1 # Versão da API Kubernetes para recursos de Serviço.
kind: Service # Tipo de recurso para expor Pods como serviços de rede.
metadata:
  name: aplicacao-exemplo-svc # Nome único do Serviço dentro do namespace.
  namespace: producao # Namespace onde o Serviço está localizado.
spec:
  type: ClusterIP # Tipo de Serviço, que expõe o Serviço apenas dentro do cluster.
  selector: # Define quais Pods são selecionados pelo Serviço.
    app: aplicacao-exemplo # Rótulo que identifica os Pods expostos pelo Serviço.
  ports: # Portas expostas pelo Serviço.
  - protocol: TCP # Protocolo utilizado para comunicação.
    port: 80 # Porta exposta pelo Serviço.
    targetPort: 80 # Porta do contêiner que o Serviço direciona o tráfego.
~~~


## service
~~~yaml
apiVersion: v1 # Define a versão da API Kubernetes usada para o recurso. "v1" é apropriado para Services.
kind: Service # Tipo de recurso, que expõe um conjunto de Pods como um serviço de rede.
metadata: # Metadados associados ao Service.
  name: aplicacao-exemplo-svc # Nome único do Service dentro do namespace, usado para identificação.
  namespace: producao # Namespace onde o Service está localizado. Usado para isolar recursos e ambientes dentro do cluster.
  labels: # Rótulos para categorizar e organizar o Service.
    app: aplicacao-exemplo # Rótulo que identifica a aplicação associada ao Service.
spec: # Especificações do Service.
  type: ClusterIP # Tipo de Service. "ClusterIP" expõe o Service apenas dentro do cluster.
  selector: # Define quais Pods são selecionados pelo Service.
    app: aplicacao-exemplo # Rótulo que identifica os Pods que o Service deve expor.
  ports: # Portas expostas pelo Service.
  - protocol: TCP # Protocolo usado para comunicação (TCP ou UDP).
    port: 80 # Porta exposta pelo Service.
    targetPort: 80 # Porta no contêiner para a qual o tráfego deve ser direcionado.
  sessionAffinity: None # Define a afinidade de sessão. "None" significa que não há afinidade de sessão, o tráfego é distribuído de forma aleatória entre os Pods selecionados.
  loadBalancerIP: "" # IP externo para o LoadBalancer, se aplicável. Deixado em branco para tipos de Service que não utilizam LoadBalancer.
~~~