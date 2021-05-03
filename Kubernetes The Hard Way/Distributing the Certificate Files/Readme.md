# Distributing the Certificate Files
Now that all of the necessary certificates have been generated, we need to move the files onto the appropriate servers. We will copy the necessary certificate files to each of our cloud servers. After completing this, your controller and worker nodes should each have the certificate files which they need.

Here are the commands used in the demo. Be sure to replace the placeholders with the actual values from from your cloud servers.

Move certificate files to the worker nodes:
```sh
scp ca.pem <worker 1 hostname>-key.pem <worker 1 hostname>.pem cloud_user@<worker 1 public IP>:~/
scp ca.pem <worker 2 hostname>-key.pem <worker 2 hostname>.pem cloud_user@<worker 2 public IP>:~/
```

![image](https://user-images.githubusercontent.com/44756128/116888701-641ca900-abf1-11eb-81d3-ab5d24e82002.png)

![image](https://user-images.githubusercontent.com/44756128/116888950-a7771780-abf1-11eb-85ee-f4e71f81d000.png)

![image](https://user-images.githubusercontent.com/44756128/116888977-afcf5280-abf1-11eb-802d-4783a97bfc9c.png)

Move certificate files to the controller nodes:
```sh
scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem cloud_user@<controller 1 public IP>:~/
```

![image](https://user-images.githubusercontent.com/44756128/116889332-0dfc3580-abf2-11eb-8d41-b753b71689a0.png)

![image](https://user-images.githubusercontent.com/44756128/116889384-18b6ca80-abf2-11eb-9112-0ce62e3ecb4f.png)

```sh
scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem cloud_user@<controller 2 public IP>:~/
```

![image](https://user-images.githubusercontent.com/44756128/116889503-41d75b00-abf2-11eb-8bc3-ee7199f32101.png)
