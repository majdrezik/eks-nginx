#
# A. Create a sample application

### 1.    Define and apply a deployment file. The following example creates a ReplicaSet that spins up two nginx pods, and then creates filed called nginx-deployment.yaml.

    $ code nginx-deployment.yaml
   -----
**nginx-deployment.yaml:**

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
        app: nginx
    spec:
      replicas: 2
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
            image: nginx:1.14.2
            ports:
            - containerPort: 80
         
 ### 2.    Create the deployment:

    $ kubectl apply -f nginx-deployment.yaml

### 3.    Verify that your pods are running and have their own internal IP addresses:

     $ kubectl get pods -l 'app=nginx' -o wide | awk {'print $1" " $3 " " $6'} | column -t
     
**Output**

      NAME                               STATUS   IP
      nginx-deployment-574b87c764-hcxdg  Running  192.168.20.8
      nginx-deployment-574b87c764-xsn9s  Running  192.168.53.240

#

# B. Create a ClusterIP service

### 1.    Create a file called clusterip.yaml, and then set type to ClusterIP. For example:


    $ code clusterip.yaml
   -----
**clusterip.yaml:**

    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service-cluster-ip
    spec:
      type: ClusterIP
      selector:
        app: nginx
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
          
### 2.    Create the ClusterIP object in Kubernetes using either a declarative or imperative command.
#### To create the object and apply the clusterip.yaml file, run the following declarative command:

    $ kubectl create -f clusterip.yaml

**Output**

    service/nginx-service-cluster-ip created

### 3.    Access the application and get the ClusterIP number:

    $ kubectl get service nginx-service-cluster-ip

**Output**

    NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
    nginx-service-cluster-ip   ClusterIP   10.100.12.153   <none>        80/TCP    23s


### 4.    Delete the ClusterIP service:

    $ kubectl delete service nginx-service-cluster-ip

**Output**

    service "nginx-service-cluster-ip" deleted

#

# C. Create a NodePort service

### 1.    To create a NodePort service, create a file called nodeport.yaml, and then set type to NodePort. For example:

    $ code nodeport.yaml
    
**nodeport.yaml**

    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service-nodeport
    spec:
      type: NodePort
      selector:
        app: nginx
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
          
### 2.    Create the NodePort object in Kubernetes using either a declarative or imperative command.
#### To create the object and apply the nodeport.yaml file, run the following declarative command:

    $ kubectl create -f nodeport.yaml
    
**Output**

    service/nginx-service-nodeport exposed

### 3.    Get information about nginx-service:

    $ kubectl get service/nginx-service-nodeport

**Output**

    NAME                     TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    nginx-service-nodeport   NodePort   10.100.106.151   <none>        80:30994/TCP   27s


### 4.    If the node is in a public subnet and is reachable from the internet, check the node’s public IP address:

    $ kubectl get nodes -o wide |  awk {'print $1" " $2 " " $7'} | column -t

**Output**

    NAME                                      STATUS  EXTERNAL-IP
    ip-10-0-3-226.eu-west-1.compute.internal  Ready   1.1.1.1
    ip-10-1-3-107.eu-west-1.compute.internal  Ready   2.2.2.2

#### -------------- OR --------------

### If the node is in a private subnet and is reachable only inside or through a VPC, then check the node’s private IP address:

    $ kubectl get nodes -o wide |  awk {'print $1" " $2 " " $6'} | column -t

**Output**

    NAME                                      STATUS  INTERNAL-IP
    ip-10-0-3-226.eu-west-1.compute.internal  Ready   10.0.3.226
    ip-10-1-3-107.eu-west-1.compute.internal  Ready   10.1.3.107


### 5.     Delete the NodePort service:

    $ kubectl delete service nginx-service-nodeport

**Output**

    service "nginx-service-nodeport" deleted


# D. Create a LoadBalancer service

### 1.    To create a LoadBalancer service, create a file called loadbalancer.yaml, and then set type to LoadBalancer. For example:

    $ code loadbalancer.yaml

**loadbalancer.yaml**

    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service-loadbalancer
    spec:
      type: LoadBalancer
      selector:
        app: nginx
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
          
### 2.    Apply the loadbalancer.yaml file:

    $ kubectl create -f loadbalancer.yaml

**Output**

    service/nginx-service-loadbalancer created

### 3.    Get information about nginx-service:

    $ kubectl get service/nginx-service-loadbalancer |  awk {'print $1" " $2 " " $4 " " $5'} | column -t

**Output**

    NAME                        TYPE          EXTERNAL-IP                                         PORT(S)
    nginx-service-loadbalancer  LoadBalancer  a******3516-2081082204.us-east-2.elb.amazonaws.com  80:30039/TCP


### 4.    Verify that you can access the load balancer externally:

    $ curl -silent a******3516-2081082204.us-east-2.elb.amazonaws.com:80 | grep title
    
**Output**

    Welcome to nginx!







