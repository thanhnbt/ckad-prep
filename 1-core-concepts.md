# Core Concepts (13%)

## Creating a Pod and Inspecting it

1. Create the namespace `ckad-prep`. Tạo 1 namespace ckad-prep
2. In the namespace `ckad-prep` create a new Pod named `mypod` with the image `nginx:2.3.5`. Expose the port 80. 
   Trong namespace ckad-prep --> tạo mới 1 pod có tên là mypod với image nginx:2.3.5, expose cổng 80

3. Identify the issue with creating the container. Write down the root cause of issue in a file named `pod-error.txt`.
Xác định lỗi khi tạo container --> write nguyên nhân lỗi ra 1 file 'pod-error.txt'
4. Change the image of the Pod to `nginx:1.15.12`.
Thay đổi image của Pod --> nginx:1.15.12
5. List the Pod and ensure that the container is running.
Liệt kê các pod và chắc chắn rằng container is running
6. Log into the container and run the `ls` command. Write down the output. Log out of the container.
ĐI vào trong container và chạy lệnh ls. 
7. Retrieve the IP address of the Pod `mypod`.
Kiểm tra IP của pod 'mypod'
8. Run a temporary Pod using the image `busybox`, shell into it and run a `wget` command against the `nginx` Pod using port 80.
Chạy 1 pod tạm thời sử dụng image 'busybox', gõ lệnh để vào trong container và thực hiện run 'wget' kiểm tra 'nginx' Pod cổng 80


9. Render the logs of Pod `mypod`.
10. Delete the Pod and the namespace.

<details><summary>Show Solution</summary>
<p>

First, create the namespace. Tạo 1 namespace

```bash
$ kubectl create namespace ckad-prep
```

Next, create the Pod in the new namespace. Tạo 1 pod trong namespace

```bash
$ kubectl run mypod --image=nginx:2.3.5 --restart=Never --port=80 --namespace=ckad-prep
pod/mypod created
```

You will see that the image cannot be pulled as it doesn't exist with this tag.
Bạn sẽ thấy image không được pull vì tag của image đó không tồn tại.

```bash
$ kubectl get pod -n ckad-prep
NAME    READY   STATUS             RESTARTS   AGE
mypod   0/1     ImagePullBackOff   0          1m
```

The list of events can give you a deeper insight.

```bash
$ kubectl describe pod -n ckad-prep --> Kiểm tra log của pod bên trong namespace ckad-prep
...
Events:
  Type     Reason                 Age                 From                         Message
  ----     ------                 ----                ----                         -------
  Normal   Scheduled              3m3s                default-scheduler            Successfully assigned mypod to docker-for-desktop
  Normal   SuccessfulMountVolume  3m2s                kubelet, docker-for-desktop  MountVolume.SetUp succeeded for volume "default-token-jbcl6"
  Normal   Pulling                84s (x4 over 3m2s)  kubelet, docker-for-desktop  pulling image "nginx:2.3.5"
  Warning  Failed                 83s (x4 over 3m1s)  kubelet, docker-for-desktop  Failed to pull image "nginx:2.3.5": rpc error: code = Unknown desc = Error response from daemon: manifest for nginx:2.3.5 not found
  Warning  Failed                 83s (x4 over 3m1s)  kubelet, docker-for-desktop  Error: ErrImagePull
  Normal   BackOff                69s (x6 over 3m)    kubelet, docker-for-desktop  Back-off pulling image "nginx:2.3.5"
  Warning  Failed                 69s (x6 over 3m)    kubelet, docker-for-desktop  Error: ImagePullBackOff
```

Go ahead and edit the existing Pod. Alternatively, you could also just use the `kubectl set image pod mypod mypod=nginx --namespace=ckad-prep` command.

```bash
$ kubectl edit pod mypod --namespace=ckad-prep 
 
   Vào edit thông tin của pod 
```

After setting an image that does exist, the Pod should render the status `Running`.

```bash
$ kubectl get pod -n ckad-prep
NAME    READY   STATUS    RESTARTS   AGE
mypod   1/1     Running   0          14m
```

You can shell into the container and run the `ls` command.

```bash
Shell để vào trong container
$ kubectl exec mypod -it --namespace=ckad-prep  -- /bin/sh
/ # ls
bin  boot  dev	etc  home  lib	lib64  media  mnt  opt	proc  root  run  sbin  srv  sys  tmp  usr  var
/ # exit
```

Retrieve the IP address of the Pod with the `-o wide` command line option.
Kiểm tra IP của pod thì thêm -o wide
```bash
$ kubectl get pods -o wide -n ckad-prep
NAME    READY   STATUS    RESTARTS   AGE   IP               NODE
mypod   1/1     Running   0          12m   192.168.60.149   docker-for-desktop
```

Remember to use the `--rm` to create a temporary Pod.

```bash
$ kubectl run busybox --image=busybox --rm -it --restart=Never -n ckad-prep -- /bin/sh
If you don't see a command prompt, try pressing enter.
   
   ( Mở thêm 1 terminal để kiểm tra IP của 2 pod 
   $ kubectl get pods -o wide -n ckad-prep 
   Sau đó mới gõ lệnh wget ip_của_pod_mypod:80)
/ # wget -O- 192.168.60.149:80
Connecting to 192.168.60.149:80 (192.168.60.149:80)
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
-                    100% |**********************************************************************|   612  0:00:00 ETA
/ # exit
```

The logs of the Pod should show a single line indicating our request.

```bash
$ kubectl logs mypod -n ckad-prep
192.168.60.162 - - [17/May/2019:13:35:59 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
```

Delete the Pod and namespace after you are done.

```bash
$ kubectl delete pod mypod --namespace=ckad-prep
pod "mypod" deleted
$ kubectl delete namespace ckad-prep
namespace "ckad-prep" deleted
```

</p>
</details>
