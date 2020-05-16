# After Installation


# Namespace management
```
$ kubectl create –f namespace.yml 
$ kubectl get namespace 
$ kubectl get namespace <Namespace name> 
$ kubectl describe namespace <Namespace name>
$ kubectl delete namespace <Namespace name>
```


# Replicaset 
```
      apiVersion: apps/v1
      kind: ReplicaSet
      metadata:
        name: myreplicaset
        labels:
          app: replicaset
      spec:
        replicas: 2
        selector:
          #matchLabels:
            #app: replicaset
          matchExpressions:  # difference in replication controller and replicaset, set based match can be applied here
              - { key: app, operator: In, values: [replicaset]} # NotIn can also be used in operator
        template:
          metadata:
            labels:
              app: replicaset
          spec:
            containers:
              - name: nginx
                image: nginx
```                

  - Replica Set ensures how many replica of pod should be running, It can be considered as a replacement of replication controller.
  - The key difference between the replica set and the replication controller is, the replication controller only supports equality-based selector, whereas the replica set supports set-based selector.
  - Deployments are upgraded and higher version of replication controller. 
  - They manage the deployment of replica sets which is also an upgraded version of the replication controller. They have the capability to update the replica set and are also capable of rolling back to the previous version.

```
> kubectl create –f Deployment.yaml -–record
> kubectl get deployments
> kubectl rollout status deployment/Deployment
> ubectl set image deployment/Deployment tomcat=tomcat:6.0
> kubectl rollout undo deployment/Deployment –to-revision=2
```


# Pod -> PVC -> PV -> Host machine
>  kubectl create –f local-01.yaml
>  kubectl get pv
>  kubectl get pvc
>  kubectl describe pv pv0001

# More on POD
```
cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
EOF
```

```
apiVersion: v1
kind: Pod
metadata:
  name: my-args-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['echo']
    args: ['This is my custom argument']
  restartPolicy: Never
  ```

```
kubectl get pods
kubectl get pod <pod name> -O wide // for more imp info in single line 
kubectl describe pod nginx
kubectl delete pod nginx
kubectl get pods --all-namespaces // --all-namespaces is option 
```

```
cat << EOF | kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
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
        image: nginx:1.15.4
        ports:
        - containerPort: 80
EOF
```

```
cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    args:
    - sleep
    - "1000"
EOF
```

# Deployment
```
kubectl get deployments
kubectl describe deployment nginx-deployment
kubectl get pods -o wide
kubectl exec busybox -- curl $nginx_pod_ip


kubectl delete -f nginx.yaml -f redis.yaml
kubectl replace -f nginx.yaml
kubectl diff -f configs/  #what change will go on
kubectl apply -f configs/

kubectl diff -R -f configs/
kubectl apply -R -f configs/

// Get a list of system pods running in the cluster (namespace):
kubectl get pods -n kube-system
```


# Types of service
 - nodePort
 - ClusterIP
 - LoadBalancer

```
cat << EOF | kubectl create -f -
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 8080 // host port
    targetPort: 80 // container port
    nodePort: 30080  //You’ll be able to contact the NodePort Service, from outside the cluster, by requesting <NodeIP>:<NodePort>.,  a NodePort in the range of 30000 - 32767 and an internal cluster IP address is assigned to the service.
  type: NodePort
EOF
```
kubectl get svc
kubectl get endpoints my-service or kubectl get ep my-service


kubectl create namespace robot-shop
kubectl -n robot-shop create -f ~/robot-shop/K8s/descriptors/
kubectl get pods -n robot-shop
kubectl get pods -n robot-shop -w (w: to allow to watch changes to it, to know which parts are taking time to start)


kubectl create ns my-ns
kubectl get namespaces

```
apiVersion: v1
kind: Pod
metadata:
  name: my-ns-pod
  namespace: my-ns     <---
  labels:
    app: myapp
spec: ....
```


## Config map
ConfigMap.yml
  apiVersion: v1
  kind: ConfigMap
  metadata:
     name: my-config-map
  data:
     myKey: myValue
     anotherKey: anotherValue
$ kubectl apply -f ConfigMap.yml




apiVersion: v1
kind: Pod
metadata:
  name: my-configmap-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    args: ['sh', '-c', "echo $(MY_VAR) && sleep 3600"]
    env:
    - name: MY_VAR
      valueFrom:
        configMapKeyRef:
          name: my-config-map
          key: myKey

apiVersion: v1
kind: Pod
metadata:
  name: my-configmap-volume-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo $(cat /etc/config/myKey) && sleep 3600"]
    volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: my-config-map


kubectl logs my-configmap-pod
kubectl logs my-configmap-volume-pod
kubectl exec my-configmap-volume-pod -- ls /etc/config
kubectl exec my-configmap-volume-pod -- cat /etc/config/myKey    

kubectl logs <pod_name> -c <container_name_inside_that_pod> // this option required for a multicontainer pod


## Security context

apiVersion: v1
kind: Pod
metadata:
  name: my-securitycontext-pod
spec:
  securityContext:
    runAsUser: 2001
    fsGroup: 3001
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "cat /message/message.txt && sleep 3600"]
    volumeMounts:
    - name: message-volume
      mountPath: /message
  volumes:
  - name: message-volume
    hostPath:
      path: /etc/message




# Resource Requirements (limits)

apiVersion: v1
kind: Pod
metadata:
  name: my-resource-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
    resources:
      requests:          # required to run container, initially allocate this
        memory: "64Mi"
        cpu: "250m"
      limits:            # max limit
        memory: "128Mi"
        cpu: "500m"


### Secret
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
stringData:
  myKey: myPassword
 (similar to config map apply this secret instand to create)

apiVersion: v1
kind: Pod
metadata:
  name: my-secret-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]
    env:
    - name: MY_PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: myKey


## service account
kubectl create serviceaccount my-serviceaccount

apiVersion: v1
kind: Pod
metadata:
  name: my-serviceaccount-pod
spec:
  serviceAccountName: my-serviceaccount
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]


# emptyDir in volume
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]
    volumeMounts:
    - mountPath: /tmp/storage
      name: my-volume
  volumes:
  - name: my-volume
    emptyDir: {}



# 3 ways for containers within same pod can communicate
- shared network (simply by localhost:port no)
- shared storage volume
- shared process namespace (by setting -> sharedProcessNamespace: True)

# 3 design patterns
- sidecar pattern
- Ambassador pattern 
- Adapter pattern 

# PV and PVC

kind: PersistentVolume
apiVersion: v1
metadata:
  name: my-pv
spec:
  storageClassName: local-storage
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"

accessModes:
ReadWriteOnce -- the volume can be mounted as read-write by a single node RWO
ReadOnlyMany -- the volume can be mounted read-only by many nodes ROX
ReadWriteMany -- the volume can be mounted as read-write by many nodes RWX



apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 512Mi

kubectl get pv
kubectl get pvc 

kind: Pod
apiVersion: v1
metadata:
  name: my-pvc-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]
    volumeMounts:
    - mountPath: "/mnt/storage"
      name: my-storage
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc

# Labels and selector
 kubectl get pods --show-labels // also visible under describe
 kubectl get pods -l app // label name filter
 kubectl get pods -l app=my-app  
 kubectl get pods -l environment=production
 kubectl get pods -l environment=development
 kubectl get pods -l environment!=production
 kubectl get pods -l 'environment in (development,production)'
 kubectl get pods -l app=my-app,environment=production

 kubectl logs -f -l jobgroup=jobexample

# Annotation
   => Similar to labels but not intended to be used as selectors, just purpose is to store custom key value pair
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-annotation-pod
     annotations:
       owner: terry@linuxacademy.com
       git-commit: bdab0c6
   spec: ...

# kubectl edit deployment <deployment name> // opens up a file containing deployment, to make any change in deployment


# Roling update and roll back
  kubectl set image deployment/<name of deployment> nginx=nginx:1.7.9 --record  // container_name=update_in_container
  kubectl rollout history deployment/<name of deployment>
  kubectl rollout history deployment/<name of deployment> --revision=2 // for more infor of specific one
  kubectl rollout undo deployment/<name of deployment> // just get back to previous version
  kubectl rollout undo deployment/<name of deployment> --to-revision=1 // get back to specific version
control the rollig update using strategy 
  spec:
   strategy:
     rollingUpdate:
       maxSurge: 3  # specifies the maximum number of Pods that can be created over the desired number of Pods.
       maxUnavailable: 2
   replicas: 3


# Jobs, that run container just like pod but unlike pod id not keep on running for always. it contain containers which removed after performing certain tasks, hance jobs are more relaible way to run such containers


apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4 // similar to 4 retries on failure before to declare actual failure

  $ kubectl get jobs
  $ kubectl logs <name of pod> // pod still there container is removed

# cron jobs
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
 $ kubectl get cronjobs
 $ kubectl logs <name of pod> // pod still there container is removed


# Network Policy
In order to use NetworkPolicies in the cluster, we need to have a network plugin 
$ wget -O canal.yaml https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/canal/canal.yaml
$ kubectl apply -f canal.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-network-policy
spec:
  podSelector:
    matchLabels:
      app: secure-app
  policyTypes:  // which will be used in this policy
  - Ingress
  - Egress
  ingress:  // to allow incoming traffic
  - from:
    - podSelector:   // also namespaceSelector, ipBlock (for CIDR block) filters do exists
        matchLabels:
          allow-access: "true"  // to apply on all those pods having this label
    ports:
    - protocol: TCP
      port: 80
  egress:  // to allow outgoing traffic
  - to:
    - podSelector:
        matchLabels:
          allow-access: "true"
    ports:
    - protocol: TCP
      port: 80

kubectl get networkpolicies
kubectl describe networkpolicy my-network-policy


# Probe: help to write custom logic to determine pod is healthy or not (help in spining up new container)
Liveness and Readiness Probes

Liveness probs: logic to determine wather or not container is live (running properly) 
Readiness probe: logic to determine wather or not container is ready to service  request
startupProbe: Indicates whether the application within the Container is started. All other probes are disabled if a startup probe is provided, until it succeeds. If the startup probe fails, the kubelet kills the Container


```
> In Liveness if condition get false container gets restart, 
but in rediness probe container gets terminated or do not send traffic to that pod
(both have same syntax) 
```

Examples: 
apiVersion: v1
kind: Pod
metadata:
  name: my-liveness-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]
    livenessProbe:
      exec:       // more diffetern types available as httpGet
        command:
        - echo
        - testing
      initialDelaySeconds: 5 // wait this time before to run for the 1st time 
      periodSeconds: 5   // run this every 5 sec



apiVersion: v1
kind: Pod
metadata:
  name: my-readiness-pod
spec:
  containers:
  - name: myapp-container
    image: nginx
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5


# TOP 
kubectl top pods
kubectl top pod resource-consumer-big
kubectl top pods -n kube-system
kubectl top nodes


# Metrics server: provides APi to access data about resources
git clone https://github.com/linuxacademy/metrics-server
kubectl apply -f ~/metrics-server/deploy/1.8+/  (1.8 from kubernetes version)
kubectl get --raw /apis/metrics.k8s.io/  (--raw to access API directly) and resource data contain cpu and memory usage

# we can not add new fields (eg livenessprob etc) with kubectl edit, we simply need to create new pod in that case, we can only update existing values with this edit feature'
best way to recreate pod is use
kubectl get pod nginx -n nginx-ns -o yaml --export // export only descriptor, not the status in yaml formate, now just edit and create (output doesnt contain namespace value)



### Citadel start 
# Init containers
    Init containers are exactly like regular containers, except:
     - Init containers always run to completion.
     - Each init container must complete successfully before the next one starts.
     - Init containers do not support readiness probes because init containers must run to completion before the Pod can be ready.
     - the resource requests and limits for an init container are handled differently.
     - it can used with pod,deployment, job etc

            apiVersion: v1
            metadata:
              name: mic
            kind: Pod
            spec:
              containers:
                - name: busybox
                  image: busybox
                  args: ['sh', '-c', 'echo Init container: The app is running! && cat /tmp/storage/welcome.txt  && sleep 30']
                  volumeMounts:
                   - mountPath: /tmp/storage
                     name: my-volume
              initContainers:
                - name: ibusybox
                  image: busybox
                  args: ['sh', '-c', 'echo The app is running! && echo harshit > /tmp/storage/welcome.txt && sleep 10']
                  volumeMounts:
                   - mountPath: /tmp/storage
                     name: my-volume
              volumes:
              - name: my-volume
                emptyDir: {}

 

# podpreset
  - A Pod Preset is an API resource for injecting additional runtime requirements into a Pod at creation time
  - A Pod Preset is capable of modifying the following fields in a Pod spec when appropriate: - The .spec.containers field. - The initContainers field
  - it cal also be disable for specific pod by using podpreset.admission.kubernetes.io/exclude: "true" in pod spec
  - config map and secrets can also be used by podpreset

```
You have enabled the API type settings.k8s.io/v1alpha1/podpreset. For example, this can be done by including settings.k8s.io/v1alpha1=true in the --runtime-config option for the API server. In minikube add this flag --extra-config=apiserver.runtime-config=settings.k8s.io/v1alpha1=true while starting the cluster.
You have enabled the admission controller PodPreset. One way to doing this is to include PodPreset in the --enable-admission-plugins option value specified for the API server. In minikube add this flag
```

Pause is a secret container that runs on every pod in Kubernetes. This container’s primary job is to keep the namespace open in case all the other containers on the pod die.

# Daemon Set

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: mydaemon
  labels:
    app: daemontest
spec:
  selector:
    matchLabels:
      app: daemontest
  template:
    metadata:
      labels:
        app: daemontest
    spec:
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
      containers:
        - name: nginx
          image: nginx


# Assigning pod to a node (nodeSelector)
kubectl label nodes <node-name> <label-key>=<label-value>
kubectl get nodes --show-labels

    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
      labels:
        env: test
    spec:
      containers:
      - name: nginx
        image: nginx
        imagePullPolicy: IfNotPresent
      nodeSelector:
        disktype: ssd






# Stateful set
  1. StatefulSets do not provide any guarantees on the termination of pods when a StatefulSet is deleted. To achieve ordered and graceful termination of the pods in the StatefulSet, it is possible to scale the StatefulSet down to 0 prior to deletion.
  2. When using Rolling Updates with the default Pod Management Policy (OrderedReady), it’s possible to get into a broken state that requires manual intervention to repair.
  3. Headless service: Sometimes you don’t need load-balancing and a single Service IP. In this case, you can create what are termed “headless” Services, by explicitly specifying "None" for the cluster IP (.spec.clusterIP).
  4. All pods (during replication) created one after another succeeds
  5. When the StatefulSet Controller creates a Pod, it adds a label, statefulset.kubernetes.io/pod-name, that is set to the name of the Pod. This label allows you to attach a Service to a specific Pod in the StatefulSet.
  6. The volumeClaimTemplates will provide stable storage using PersistentVolumes provisioned by a PersistentVolume Provisioner (can also used with deployments)
    
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
        clusterIP: None
        selector:
          app: nginx
      ---
      apiVersion: apps/v1
      kind: StatefulSet
      metadata:
        name: web
      spec:
        selector:
          matchLabels:
            app: nginx # has to match .spec.template.metadata.labels
        serviceName: "nginx"
        replicas: 3 # by default is 1
        template:
          metadata:
            labels:
              app: nginx # has to match .spec.selector.matchLabels
          spec:
            terminationGracePeriodSeconds: 10
            containers:
            - name: nginx
              image: k8s.gcr.io/nginx-slim:0.8
              ports:
              - containerPort: 80
                name: web
              volumeMounts:
              - name: www
                mountPath: /usr/share/nginx/html
        volumeClaimTemplates:
        - metadata:
            name: www
          spec:
            accessModes: [ "ReadWriteOnce" ]
            storageClassName: "my-storage-class"
            resources:
              requests:
                storage: 1Gi 
      
      
 kubectl get sts,po,pvc,svc



 External IP in service

     apiVersion: v1
     kind: Service
     metadata:
       name: my-service
     spec:
       selector:
         app: MyApp
       ports:
         - name: http
           protocol: TCP
           port: 80
           targetPort: 9376
       externalIPs:
         - 80.11.12.10




# API in kubernetes (2 simple ways, also programing langiage specific libraries are also there)
REF: https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/
1. Using kubectl proxy
    kubectl proxy --port=8080
    curl http://localhost:8080/api/ (similarly all api can be used as documented at https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/)
    curl http://localhost:8080/api/v1/namespaces/default/pods

1.5 by using kubectl-proxy pod
     
    kubectl run test --image=chadmcrowell/kubectl-proxy // to quickly run this image as pod
    kubectl exec -it <name-of-pod> sh
    curl localhost:8001/api/v1/namespaces/default/pods // inside container

2. Without kubectl proxy
    APISERVER=$(kubectl config view --minify | grep server | cut -f 2- -d ":" | tr -d " ")
    SECRET_NAME=$(kubectl get secrets | grep ^default | cut -f1 -d ' ')
    TOKEN=$(kubectl describe secret $SECRET_NAME | grep -E '^token' | cut -f2 -d':' | tr -d " ")
    
    curl $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
    curl $APISERVER/api/v1/namespaces/default/pods --header "Authorization: Bearer $TOKEN" --insecure // to get all pod running
    they may get fail due to permission issue
    "User "system:serviceaccount:default:default" cannot get at the cluster scope."
    to resolve this 
    You should bind service account system:serviceaccount:default:default (which is the default account bound to Pod) with role cluster-admin, just create a yaml (named like fabric8-rbac.yaml) with following contents:
    run 
      ```
        # NOTE: The service account `default:default` already exists in k8s cluster.
        # You can create a new account following like this:
        #---
        #apiVersion: v1
        #kind: ServiceAccount
        #metadata:
        #  name: <new-account-name>
        #  namespace: <namespace>

        ---
        apiVersion: rbac.authorization.k8s.io/v1beta1
        kind: ClusterRoleBinding
        metadata:
          name: fabric8-rbac
        subjects:
          - kind: ServiceAccount
            # Reference to upper's `metadata.name`
            name: default
            # Reference to upper's `metadata.namespace`
            namespace: default
        roleRef:
          kind: ClusterRole
          name: cluster-admin
          apiGroup: rbac.authorization.k8s.io
      ```

```
cloud_user@arpitsinghal2c:~/kubernates/excersice/rbac$ k get clusterrole cluster-admin -o yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    annotations:
      rbac.authorization.kubernetes.io/autoupdate: "true"
    creationTimestamp: 2020-05-04T10:40:21Z
    labels:
      kubernetes.io/bootstrapping: rbac-defaults
    name: cluster-admin
    resourceVersion: "57"
    selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/cluster-admin
    uid: a7a6f771-8df3-11ea-8556-0a55871d47c8
  rules:
  - apiGroups:
    - '*'
    resources:
    - '*'
    verbs:
    - '*'
  - nonResourceURLs:
    - '*'
    verbs:
    - '*'
```

# View the token file from within a pod:
  cat /var/run/secrets/kubernetes.io/serviceaccount/token  (==> kubectl exec  httpd -- ls /var/run/secrets/kubernetes.io/serviceaccount/token)

# cat ~/.kube/config 
role: what can do it
binding: who can do it

1. role and role binding : ns lavel resources
2. cluster role and cluster bindings are cluster lavel resources

RBAC: Role-based access control
ABAC: Attribute-based access control

A Role (the same applies to a ClusterRole) contains a list of rules. Each rule defines some actions that can be performed (e.g: list, get, watch) against a list of resources (e.g: Pod, Service, Secret) within apiGroups (eg: core, apps/v1).
  ```
  apiVersion: rbac.authorization.k8s.io/v1beta1
  kind: Role
  metadata:
   name: list-pods
   namespace: default
  rules:
   — apiGroups:
     — ''
   resources:
     — pods
   verbs:
     — list
  ```

  or 
  ```
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      namespace: default
      name: pod-reader
    rules:
    - apiGroups: [""] # "" indicates the core API group
      resources: ["pods"]
      resourceNames: ["my-pod"]
      verbs: ["get", "watch", "list"]
  ```
  ```
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRole
   metadata:
     # "namespace" omitted since ClusterRoles are not namespaced
     name: secret-reader
   rules:
   - apiGroups: [""]
     #
     # at the HTTP level, the name of the resource for accessing Secret
     # objects is "secrets"
     resources: ["secrets"]
     verbs: ["get", "watch", "list"]
  ```


# Role binding
A role binding grants the permissions defined in a role to a user or set of users. It holds a list of subjects (users, groups, or service accounts), and a reference to the role being granted.

get, list, create, update, patch, watch, delete, and deletecollection

```
   apiVersion: rbac.authorization.k8s.io/v1beta1
   kind: RoleBinding
   metadata:
    name: list-pods_demo-sa
    namespace: default
   roleRef:
    kind: Role
    name: list-pods
    apiGroup: rbac.authorization.k8s.io
   subjects:
    — kind: ServiceAccount
      name: demo-sa
      namespace: default
```
or
```
    apiVersion: rbac.authorization.k8s.io/v1
    # This role binding allows "jane" to read pods in the "default" namespace.
    # You need to already have a Role named "pod-reader" in that namespace.
    kind: RoleBinding
    metadata:
      name: read-pods
      namespace: default
    subjects:
    # You can specify more than one "subject"
    - kind: User
      name: jane # "name" is case sensitive
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      # "roleRef" specifies the binding to a Role / ClusterRole
      kind: Role #this must be Role or ClusterRole
      name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
      apiGroup: rbac.authorization.k8s.io
```  
A RoleBinding can also reference a ClusterRole to grant the permissions defined in that ClusterRole to resources inside the RoleBinding’s namespace



All requestes come either by client (user), or pod goes through 3 phases
1. authentication (check user is valid)
2. authorization (check wahter valid user have permission to perform the requested work)
3. Admission (to etcd)

# Run a Quick single pod deployment
kubectl run test --image=nginx --ns=default
kubectl port-forward $pod_name 8081:80 (host_port : container port)




# backing up and restoring cluster:
(involvs backup and restore of etcd only b'coz all cluster stated is available here) 
    wget https://github.com/etcd-io/etcd/releases/download/v3.3.12/etcd-v3.3.12-linux-amd64.tar.gz
    tar xvf etcd-v3.3.12-linux-amd64.tar.gz
    sudo mv etcd-v3.3.12-linux-amd64/etcd* /usr/local/bin
    sudo ETCDCTL_API=3 etcdctl snapshot save snapshot.db --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key
    ETCDCTL_API=3 etcdctl --help
    sudo tar -zcvf etcd.tar.gz /etc/kubernetes/pki/etcd
    scp etcd.tar.gz cloud_user@18.219.235.42:~/


# Take down a node from cluster for maintaiance 
     kubectl get pods -o wide
     kubectl drain [node_name] --ignore-daemonsets // to take all pode from the node to another one
     kubectl get nodes -w
     
     // make changes in node and 
        kubectl uncordon [node_name] // to take back that same node again to cluster
     
     // if changes are big like node replace by new one or os upgrade after drain delete the node from the cluster
     
     kubectl delete node [node_name]
     
      // perform the upgrade, make sure kubedam, docker kubectl etc all prereq are installed on new upgraded node
      sudo kubeadm token generate
      sudo kubeadm token list
      sudo kubeadm token create [token_name] --ttl 2h --print-join-command
      // use this command in op of above command, on new node to join this cluster back

# swiching config files to different version
    kubectl convert -f pod.yaml --output-version v1


## Get version of different components of k8 (required for upgradation of k8 version in cluster)
# kubectl version --short 
 show client (or kubectl) version and server (ie API server) version
# kubectl describe nodes 
 get kubectl and kube-proxy version
# get controller manager version
   kubectl get po [controller_pod_name] -o yaml -n kube-system   // controller_pod_name is starting from kubectl control manager and ending with node hostname
   -> the the version from image name in the above version


## upgrade
Release the hold on versions of kubeadm and kubelet:
 $ sudo apt-mark unhold kubeadm kubelet
 $ sudo apt install -y kubeadm=1.16.6-00
 $ sudo apt-mark hold kubeadm
 $ kubeadm version
 $ sudo kubeadm upgrade plan // to check current versions and available version for upgrate
 $ sudo kubeadm upgrade apply v1.16.6
 $ sudo apt-mark unhold kubectl
 $ sudo apt install -y kubectl=1.16.6-00
 $ sudo apt-mark hold kubectl
 $ sudo apt install -y kubelet=1.16.6-00
 $ sudo apt-mark hold kubelet 

 // only upgrade of kubelet is required on all worker node (unhold and upgrade)


 # Look IP tables for specific service
  sudo iptables-save | grep KUBE | grep nginx
  // IP tables contain the details of which traffic where to farward (in both direction ie request and responce)

  # kubectl scale deployment/kubeserve2 --replicas=2

  load balancer type: Automatically get external IP and node Port
  # kubectl expose deployment kubeserve2 --port 80 --target-port 8080 --type LoadBalancer // to create load balanccer type service   # Nature of load balancing can be controlled by applying specific annotations

  # A service can map an incoming port to any targetPort. (The targetPort is set, by default, to the port field’s same value. The targetPort can be defined as a string.)


```
ClusterIP. This default type exposes the service on a cluster-internal IP. You can reach the service only from within the cluster.
NodePort. This type of service exposes the service on each node’s IP at a static port. A ClusterIP service is created automatically, and the NodePort service will route to it. From outside the cluster, you can contact the NodePort service by using “<NodeIP>:<NodePort>”.
LoadBalancer. This service type exposes the service externally using the load balancer of your cloud provider. The external load balancer routes to your NodePort and ClusterIP services, which are created automatically.
ExternalName. This type maps the service to the contents of the externalName field (e.g., foo.bar.example.com). It does this by returning a value for the CNAME record.
```

# service-ingress

simple ingress service 
  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    name: test-ingress
  spec:
    backend:
      serviceName: testsvc
      servicePort: 80



similar to load balancer but by using only one external IP we can access more than one service unlike load balancer (one service per external IP)

        apiVersion: extensions/v1beta1
        kind: Ingress
        metadata:
          name: service-ingress
        spec:
          rules:
          - host: kubeserve.example.com
            http:
              paths:
              - backend:
                  serviceName: kubeserve2
                  servicePort: 80
          - host: app.example.com
            http:
              paths:
              - backend:
                  serviceName: nginx
                  servicePort: 80
          - http:                       # all traffic without hostname will come here
              paths:
              - backend:
                  serviceName: httpd
                  servicePort: 80

## on single host multiple paths are also possible (as express routes.js file)

k get ingress


apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: simple-fanout-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foo
        backend:
          serviceName: service1
          servicePort: 4200
      - path: /bar
        backend:
          serviceName: service2
          servicePort: 8080

Ingress controller need to be instaled: eg for nginx
## Code Examples
   https://github.com/nginxinc/kubernetes-ingress/tree/master/examples
## Installation
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-0.32.0/deploy/static/provider/baremetal/deploy.yaml
## Verification
   kubectl get pods -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --watch
## Detact installed version
   POD_NAMESPACE=ingress-nginx
   POD_NAME=$(kubectl get pods -n $POD_NAMESPACE -l app.kubernetes.io/name=ingress-nginx -o jsonpath='{.items[0].metadata.name}')
   kubectl exec -it $POD_NAME -n $POD_NAMESPACE -- /nginx-ingress-controller --version




          
# Way to define a file in configMap          
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
data:
  nginx.conf: |
    user nginx;
    worker_processes  3;
    error_log  /var/log/nginx/error.log;
    events {
      worker_connections  10240;
    }

# now mount this as volume to create file with variable name    


# DNS
default: ns name
.cluster.local: domain (from  kubectl exec busybox -- cat /etc/resolv.conf)
kubectl exec -ti busybox -- nslookup [pod-ip-address].default.pod.cluster.local
kubectl exec -ti busybox -- nslookup [service_name].default.svc.cluster.local

custom DNS options with a Pod (all this will be reflacted in /etc/resolve.conf of POD's container)

    apiVersion: v1
    kind: Pod
    metadata:
      namespace: default
      name: dns-example
    spec:
      containers:
        - name: test
          image: nginx
      dnsPolicy: "None"
      dnsConfig:
        nameservers:
          - 8.8.8.8
        searches:
          - ns1.svc.cluster.local
          - my.dns.search.suffix
        options:
          - name: ndots
            value: "2"
          - name: edns0


 # Users and accounts
  - API requests are tied to either a normal user or a service account, or are treated as anonymous requests. 


kubectl cp /tmp/foo_dir <some-pod>:/tmp/bar_dir
kubectl cp /tmp/foo <some-pod>:/tmp/bar -c <specific-container>
kubectl cp /tmp/foo <some-namespace>/<some-pod>:/tmp/bar
kubectl cp <some-namespace>/<some-pod>:/tmp/foo /tmp/bar


kubectl get events

## Metrics server
installation
(for official look: kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml)
git clone https://github.com/linuxacademy/metrics-server
kubectl apply -f ~/metrics-server/deploy/1.8+/
test it installed successfully
kubectl get --raw /apis/metrics.k8s.io/
wait for a min for this server to collect all data
now run

kubectl top node
kubectl top pods, kubectl top pods --all-namespaces, kubectl top pod -l run=pod-with-defaults, kubectl top pods -n kube-system
kubectl top pods <pod name> --containers // for all container info also inside a pod

# logs
kubectl logs nginx
kubectl logs counter -c count-log-1
kubectl logs counter --all-containers=true
kubectl logs -lapp=nginx
kubectl logs -p -c nginx nginx //  from a previously terminated container within a pod:
kubectl logs -f -c count-log-1 counter // stream the logs from a container in a pod:
kubectl logs --tail=20 nginx
kubectl logs --since=1h nginx
ubectl logs deployment/nginx -c nginx

# scale
## Horizontal POD Autoscale 
kubectl get hpa
kubectl autoscale deployment/my-nginx --min=1 --max=3
kubectl autoscale deployment shell --min=2 --max=10 --cpu-percent=50
-> PA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 50% (since each pod requests 200 milli-cores by kubectl run, this means average CPU usage of 100 milli-cores). See here for more details on the algorithm.
-> The Horizontal Pod Autoscaler is implemented as a control loop, with a period controlled by the controller manager’s --horizontal-pod-autoscaler-sync-period flag (with a default value of 15 seconds).
way to incarese load:  while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done


```

By kubectl get hpa.autoscaling.v2alpha1 -o yaml 
apiVersion: autoscaling/v2alpha1
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1beta1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 50
status:
  observedGeneration: 1
  lastScaleTime: <some-time>
  currentReplicas: 1
  desiredReplicas: 1
  currentMetrics:
  - type: Resource
    resource:
      name: cpu
      currentAverageUtilization: 0
      currentAverageValue: 0
```
-> You can also specify resource metrics in terms of direct values, instead of as percentages of the requested value. To do so, use the targetAverageValue field insted of the targetAverageUtilization field.
-> There are two other types of metrics, both of which are considered custom metrics: pod metrics and object metrics.


pod metric -> 
type: Pods
pods:
  metricName: packets-per-second
  targetAverageValue: 1k

object metric ->
type: Object
object:
  metricName: requests-per-second
  target:
    apiVersion: extensions/v1beta1
    kind: Ingress
    name: main-route
  targetValue: 2k

  *** If you provide multiple such metric blocks, the HorizontalPodAutoscaler will consider each metric in turn. The HorizontalPodAutoscaler will calculate proposed replica counts for each metric, and then choose the one with the highest replica count.


  example

  ```
  (on kubectl edit)
  apiVersion: autoscaling/v2alpha1
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1beta1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 50
  - type: Pods
    pods:
      metricName: packets-per-second
      targetAverageValue: k
  - type: Object
    object:
      metricName: requests-per-second
      target:
        apiVersion: extensions/v1beta1
        kind: Ingress
        name: main-route
      targetValue: 10k
status:
  observedGeneration: 1
  lastScaleTime: <some-time>
  currentReplicas: 1
  desiredReplicas: 1
  currentMetrics:
  - type: Resource
    resource:
      name: cpu
      currentAverageUtilization: 0
      currentAverageValue: 0
```

::--))
k delete -f .
k delete -f yaml_filename

# Helm basics
helm install example ./mychart --set service.type=NodePort
helm lint ./mychart 
helm package ./mychart
helm install example3 mychart-0.1.0.tgz --set service.type=NodePort
helm uninstall example




kubectl port-forward redis-master-765d459796-258hz 7000:6379
or
kubectl port-forward pods/redis-master-765d459796-258hz 7000:6379
or
kubectl port-forward deployment/redis-master 7000:6379
or 
kubectl port-forward replicaset/redis-master 7000:6379
or
kubectl port-forward service/redis-master 7000:6379
