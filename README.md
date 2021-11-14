# sidecar_topz

# 1. Create yaml file
See attached yaml file. But the followings are the differences between non-sidecar yaml and sidecar yaml with topz.
```
$ diff face_recognizer.yaml face_recognizer_topz.yaml 
18a19,33
> apiVersion: v1
> kind: Service
> metadata:
>   name: facerecognizer-topz
>   labels:
>     run: topz
> spec:
>   type: LoadBalancer
>   ports:
>     - port: 8080
>       targetPort: 8080
>       name: topz
>   selector:
>     run: facerecognizer-test
> ---
32a48
>       shareProcessNamespace: true
51a68,76
>       - name: topz
>         image: kalise/topz:latest
>         ports:
>           - containerPort: 8080
>         command:
>           - /server
>         args:
>           - -addr
>           - 0.0.0.0:8080
```

# 2. Run yaml
```
$ kubectl apply -f face_recognizer_topz.yaml 
service/facerecognizer-srv created
service/facerecognizer-topz created
deployment.apps/facerecognizer-test created

$ kubectl get services
NAME                                                    TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)             AGE
facerecognizer-srv                                      ClusterIP      10.99.72.113     <none>            5000/TCP            4m3s
facerecognizer-topz                                     LoadBalancer   10.111.63.176    192.168.122.222   8080:32564/TCP      4m3s
gpu-operator                                            ClusterIP      10.110.206.144   <none>            8080/TCP            2d10h
gpu-operator-1636672240-node-feature-discovery-master   ClusterIP      10.108.73.244    <none>            8080/TCP            2d10h
```
```
$ kubectl get pods
NAME                                                              READY   STATUS    RESTARTS        AGE
facerecognizer-test-8b9c9f6d4-xscbf                               3/3     Running   0               7m32s
gpu-operator-1636672240-node-feature-discovery-master-5b7d424j7   2/2     Running   5 (4h28m ago)   2d10h
gpu-operator-1636672240-node-feature-discovery-worker-4s6wp       2/2     Running   6 (4h28m ago)   2d10h
gpu-operator-1636672240-node-feature-discovery-worker-m4hbm       2/2     Running   7 (4h28m ago)   2d10h
gpu-operator-1636672240-node-feature-discovery-worker-rzjtw       2/2     Running   7 (4h28m ago)   2d10h
gpu-operator-5f8b7c4f59-fcp67                                     2/2     Running   6 (4h28m ago)   2d10h
```
```
$ kubectl describe pods facerecognizer-test-8b9c9f6d4-xscbf
Name:         facerecognizer-test-8b9c9f6d4-xscbf
Namespace:    default
Priority:     0
Node:         worker1/192.168.122.134
Start Time:   Sun, 14 Nov 2021 19:02:44 +0900
Labels:       pod-template-hash=8b9c9f6d4
              run=facerecognizer-test
              security.istio.io/tlsMode=istio
              service.istio.io/canonical-name=facerecognizer-test
              service.istio.io/canonical-revision=latest
Annotations:  cni.projectcalico.org/containerID: 23ad58599293d7ff4b636961e2c5d5150b0dbdd8b8a68d34b1c7b6ed26ad2b73
              cni.projectcalico.org/podIP: 192.168.235.171/32
              cni.projectcalico.org/podIPs: 192.168.235.171/32
              prometheus.io/path: /stats/prometheus
              prometheus.io/port: 15020
              prometheus.io/scrape: true
              sidecar.istio.io/status:
                {"initContainers":["istio-init"],"containers":["istio-proxy"],"volumes":["istio-envoy","istio-data","istio-podinfo","istio-token","istiod-...
Status:       Running
IP:           192.168.235.171
IPs:
  IP:           192.168.235.171
Controlled By:  ReplicaSet/facerecognizer-test-8b9c9f6d4
Init Containers:
  istio-init:
    Container ID:  docker://c7961b6511b3b17285620a306fc65b3d52da35da2cb59fe110dfbd10c4dad615
    Image:         docker.io/istio/proxyv2:1.11.4
    Image ID:      docker-pullable://istio/proxyv2@sha256:68b1012e5ba209161671f0e76c92baad845cef810fa0a7cf1b105fee8f0d3e46
    Port:          <none>
    Host Port:     <none>
    Args:
      istio-iptables
      -p
      15001
      -z
      15006
      -u
      1337
      -m
      REDIRECT
      -i
      *
      -x
      
      -b
      *
      -d
      15090,15021,15020
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Sun, 14 Nov 2021 19:02:45 +0900
      Finished:     Sun, 14 Nov 2021 19:02:45 +0900
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     2
      memory:  1Gi
    Requests:
      cpu:        100m
      memory:     128Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mbrxh (ro)
Containers:
  facerecognizer-test:
    Container ID:   docker://4be076a0cf7bf1641f9101ca381606c19954710d291fe0a455f4dc503701c2f0
    Image:          192.168.122.1:5000/face_recognizer:1.0.1
    Image ID:       docker-pullable://192.168.122.1:5000/face_recognizer@sha256:489612e5f6cb43c6815830e468a2f4467db9cd8202daadb27bc475de603deee3
    Port:           5000/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 14 Nov 2021 19:02:47 +0900
    Ready:          True
    Restart Count:  0
    Limits:
      nvidia.com/gpu:  1
    Requests:
      nvidia.com/gpu:  1
    Environment:       <none>
    Mounts:
      /dev/video0 from dev-video0 (rw)
      /root/.Xauthority from xauthority (rw)
      /tmp/.X11-unix from x11 (rw)
      /tmp/test from tmp (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mbrxh (ro)
  topz:
    Container ID:  docker://2ba62368a591e9d005be6c9d0a602f602ee899b637df88d025d4a291176e3f68
    Image:         kalise/topz:latest
    Image ID:      docker-pullable://kalise/topz@sha256:78ae8cabeb608c882d395868707d6158cd73f6cf1b22e0f86318fe7beec8f838
    Port:          8080/TCP
    Host Port:     0/TCP
    Command:
      /server
    Args:
      -addr
      0.0.0.0:8080
    State:          Running
      Started:      Sun, 14 Nov 2021 19:02:49 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mbrxh (ro)
  istio-proxy:
    Container ID:  docker://95b662168f8c4297bd7d5dc209aff081b40471e360e517645475ca02f81dd432
    Image:         docker.io/istio/proxyv2:1.11.4
    Image ID:      docker-pullable://istio/proxyv2@sha256:68b1012e5ba209161671f0e76c92baad845cef810fa0a7cf1b105fee8f0d3e46
    Port:          15090/TCP
    Host Port:     0/TCP
    Args:
      proxy
      sidecar
      --domain
      $(POD_NAMESPACE).svc.cluster.local
      --proxyLogLevel=warning
      --proxyComponentLogLevel=misc:error
      --log_output_level=default:info
      --concurrency
      2
    State:          Running
      Started:      Sun, 14 Nov 2021 19:02:50 +0900
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     2
      memory:  1Gi
    Requests:
      cpu:      100m
      memory:   128Mi
    Readiness:  http-get http://:15021/healthz/ready delay=1s timeout=3s period=2s #success=1 #failure=30
    Environment:
      JWT_POLICY:                    third-party-jwt
      PILOT_CERT_PROVIDER:           istiod
      CA_ADDR:                       istiod.istio-system.svc:15012
      POD_NAME:                      facerecognizer-test-8b9c9f6d4-xscbf (v1:metadata.name)
      POD_NAMESPACE:                 default (v1:metadata.namespace)
      INSTANCE_IP:                    (v1:status.podIP)
      SERVICE_ACCOUNT:                (v1:spec.serviceAccountName)
      HOST_IP:                        (v1:status.hostIP)
      PROXY_CONFIG:                  {}
                                     
      ISTIO_META_POD_PORTS:          [
                                         {"containerPort":5000,"protocol":"TCP"}
                                         ,{"containerPort":8080,"protocol":"TCP"}
                                     ]
      ISTIO_META_APP_CONTAINERS:     facerecognizer-test,topz
      ISTIO_META_CLUSTER_ID:         Kubernetes
      ISTIO_META_INTERCEPTION_MODE:  REDIRECT
      ISTIO_META_WORKLOAD_NAME:      facerecognizer-test
      ISTIO_META_OWNER:              kubernetes://apis/apps/v1/namespaces/default/deployments/facerecognizer-test
      ISTIO_META_MESH_ID:            cluster.local
      TRUST_DOMAIN:                  cluster.local
    Mounts:
      /etc/istio/pod from istio-podinfo (rw)
      /etc/istio/proxy from istio-envoy (rw)
      /var/lib/istio/data from istio-data (rw)
      /var/run/secrets/istio from istiod-ca-cert (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mbrxh (ro)
      /var/run/secrets/tokens from istio-token (rw)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  istio-envoy:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     Memory
    SizeLimit:  <unset>
  istio-data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  istio-podinfo:
    Type:  DownwardAPI (a volume populated by information about the pod)
    Items:
      metadata.labels -> labels
      metadata.annotations -> annotations
  istio-token:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  43200
  istiod-ca-cert:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      istio-ca-root-cert
    Optional:  false
  tmp:
    Type:          HostPath (bare host directory volume)
    Path:          /mnt
    HostPathType:  
  x11:
    Type:          HostPath (bare host directory volume)
    Path:          /tmp/.X11-unix
    HostPathType:  
  xauthority:
    Type:          HostPath (bare host directory volume)
    Path:          /root/.Xauthority
    HostPathType:  
  dev-video0:
    Type:          HostPath (bare host directory volume)
    Path:          /dev/video0
    HostPathType:  
  kube-api-access-mbrxh:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  10m   default-scheduler  Successfully assigned default/facerecognizer-test-8b9c9f6d4-xscbf to worker1
  Normal  Pulled     10m   kubelet            Container image "docker.io/istio/proxyv2:1.11.4" already present on machine
  Normal  Created    10m   kubelet            Created container istio-init
  Normal  Started    10m   kubelet            Started container istio-init
  Normal  Pulled     10m   kubelet            Container image "192.168.122.1:5000/face_recognizer:1.0.1" already present on machine
  Normal  Created    10m   kubelet            Created container facerecognizer-test
  Normal  Started    10m   kubelet            Started container facerecognizer-test
  Normal  Pulling    10m   kubelet            Pulling image "kalise/topz:latest"
  Normal  Pulled     10m   kubelet            Successfully pulled image "kalise/topz:latest" in 2.431755361s
  Normal  Created    10m   kubelet            Created container topz
  Normal  Started    10m   kubelet            Started container topz
  Normal  Pulled     10m   kubelet            Container image "docker.io/istio/proxyv2:1.11.4" already present on machine
  Normal  Created    10m   kubelet            Created container istio-proxy
  Normal  Started    10m   kubelet            Started container istio-proxy
```

# 3. Access topz thru Browser
```
$ curl 192.168.122.222:8080/topz
1  0 4.9105325e-05 /pause
21 0 0.006678324   /bin/sh -c /tmp/app.py
27 0 4.9772177     /usr/bin/python3 /tmp/app.py
34 0 4.7453423     /usr/bin/python3 /tmp/app.py
36 0 0.07478741    /server -addr 0.0.0.0:8080
47 0 0.6481412     /usr/local/bin/pilot-agent proxy sidecar --domain default.svc.cluster.local --proxyLogLevel=warning --proxyComponentLogLevel=misc:error --log_output_level=default:info --concurrency 2
59 0 0.73176754    /usr/local/bin/envoy -c etc/istio/proxy/envoy-rev0.json --restart-epoch 0 --drain-time-s 45 --drain-strategy immediate --parent-shutdown-time-s 60 --local-address-ip-version v4 --bootstrap-version 3 --file-flush-interval-msec 1000 --disable-hot-restart --log-format %Y-%m-%dT%T.%fZ %l envoy %n %v -l warning --component-log-level misc:error --concurrency 2

```
