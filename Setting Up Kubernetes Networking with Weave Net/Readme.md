![image](https://user-images.githubusercontent.com/44756128/114307932-90884e00-9aa7-11eb-8538-9377275daa0f.png)

# Introduction
We'll configure a Kubernetes pod network using Weave Net. After completing this, we will have hands-on experience implementing networking within a Kubernetes cluster.

To start, open two terminal windows and log in to both worker nodes using the public IPs and credentials listed on the lab page. Note: We're going to run all the commands on both servers simultaneously.

# Enable IP Forwarding on All Worker Nodes
In order for Weave Net to work, we need to make sure IP forwarding is enabled on the worker nodes. Enable it by running the following on both workers:
```sh
sudo sysctl net.ipv4.conf.all.forwarding=1
echo "net.ipv4.conf.all.forwarding=1" | sudo tee -a /etc/sysctl.conf
```

![image](https://user-images.githubusercontent.com/44756128/114308586-202efc00-9aaa-11eb-9d6b-14d66f17e61a.png)

# Install Weave Net in the Cluster
Log in to the controller server in a new terminal window, and then do the following:

  - Install Weave Net using a configuration from Weaveworks like this:
```sh
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=10.200.0.0/16"
```

![image](https://user-images.githubusercontent.com/44756128/114308739-77cd6780-9aaa-11eb-8f4f-f17ed2150eb1.png)

  - Verify that everything is working:
```sh
kubectl get pods -n kube-system
```

This should return two weave-net pods and look something like this:
```
 NAME              READY     STATUS    RESTARTS   AGE
 weave-net-m69xq   2/2       Running   0          11s
 weave-net-vmb2n   2/2       Running   0          11s
```

![image](https://user-images.githubusercontent.com/44756128/114308765-84ea5680-9aaa-11eb-8ddd-0f885d123ebb.png)

  - Spin up some pods to test the networking functionality:

a. First, create an Nginx deployment with 2 replicas:
```sh
 cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx
    spec:
      selector:
        matchLabels:
          run: nginx
      replicas: 2
      template:
        metadata:
          labels:
            run: nginx
        spec:
          containers:
          - name: my-nginx
            image: nginx
            ports:
            - containerPort: 80
 EOF
```

b. Next, create a service for that deployment so that we can test connectivity to services as well:
```sh
 kubectl expose deployment/nginx
```

c. Start up another pod. We will use this pod to test our networking. We will test whether we can connect to the other pods and services from this pod.
```sh
 kubectl run busybox --image=radial/busyboxplus:curl --command -- sleep 3600
 POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```

![image](https://user-images.githubusercontent.com/44756128/114308866-e4e0fd00-9aaa-11eb-9502-d840c3dd9db1.png)

d. Get the IP addresses of our two nginx pods:
```sh
 kubectl get ep nginx
```

There should be two IP addresses listed under ENDPOINTS. For example:
```
 NAME      ENDPOINTS                       AGE
 nginx     10.200.0.2:80,10.200.128.1:80   50m
```

![image](https://user-images.githubusercontent.com/44756128/114308881-f4f8dc80-9aaa-11eb-96d7-4b8148455640.png)

  - Make sure the busybox pod can connect to the nginx pods on both of those IP addresses.
```sh
 kubectl exec $POD_NAME -- curl <first nginx pod IP address>;
 kubectl exec $POD_NAME -- curl <second nginx pod IP address>;
```

![image](https://user-images.githubusercontent.com/44756128/114308973-42754980-9aab-11eb-8ff9-448513b4b29d.png)

Both commands should return some HTML with the title "Welcome to Nginx!" This means that we can successfully connect to other pods.

  - Now let's verify that we can connect to services.
```sh
 kubectl get svc
```

This should display the IP address for our Nginx service. For example, in this case, the IP is 10.32.0.54:
```
 NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
 kubernetes   ClusterIP   10.32.0.1    <none>        443/TCP   1h
 nginx        ClusterIP   10.32.0.54   <none>        80/TCP    53m
```

![image](https://user-images.githubusercontent.com/44756128/114309025-65076280-9aab-11eb-937b-eccbad10eb30.png)

Check that we can access the service from the busybox pod.
```sh
kubectl exec $POD_NAME -- curl <nginx service IP address>;
```

![image](https://user-images.githubusercontent.com/44756128/114309076-88caa880-9aab-11eb-8b65-8844bd8d59d6.png)

This should also return HTML with the title "Welcome to nginx!"

Getting this response means that we have successfully reached the Nginx service from inside a pod and that our networking configuration is working!
