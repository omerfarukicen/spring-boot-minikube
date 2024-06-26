# Spring And Minikube
    Ingress Gateway
    Student Service
    Rating Service

## Requirements
    Java: JDK 1.8
    Maven Build
    Docker 
    Kubernetes: Minikube on local

## Ubuntu Minikube install 
```
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube

```

```
minikube kubectl -- get po -A
```
```
alias kubectl="minikube kubectl --
```

#### Some Commands
```
kubectl get nodes 
kubectl get pod
minikube ssh
minikube dashboard
```
#### docker image for minikube

    eval $(minikube docker-env)

throuble : Failed to initialize: unable to resolve docker endpoint: open /home/omericen/.minikube/certs/ca.pem: permission denied
sudo nano /var/lib/snapd/apparmor/profiles/snap.docker.docker
add :
owner @{HOME}/.minikube/certs/* r,

save and run on terminal-
apparmor_parser -r /var/lib/snapd/apparmor/profiles/snap.docker.docker

eval $(minikube docker-env)




#### Build Student Service
```
$ cd student-service
$ mvn package
```



#### Create docker images for Student Service and push Docker Hub
```
$ docker build -t student-service .
$ docker push $(minikube ip):5000/student-service:latest
```
is there outside repository?
```
$ docker push student-service
```

#### Build Rating Service
```
$ cd rating-service
$ mvn package
```

#### Create docker images for Rating Service and push Docker Hub
```
$ docker build -t rating-service .
$ docker push $(minikube ip):5000/rating-service:latest
```

#### Start Minikube
```
$ minikube start
```

#### Create student-service deployment
```
$ cd devops
$ kubectl create -f deployment-student.yml
```

#### Create rating-service deployment
```
$ cd devops
$ kubectl create -f deployment-rating.yml
```

#### Create and Expose student-service
```
$ kubectl expose deployment student-service --type=NodePort
```

or
```
$ cd devops
$ kubectl create -f service-student.yml
```

#### Create and Expose rating-service
```
$ kubectl expose deployment rating-service --type=NodePort
```

or
```
$ cd devops
$ kubectl create -f service-rating.yml
```


#### Check service
```
$ minikube service student-service --url
```
Give me url `http://192.168.99.100:30676` and Check rest-api http://192.168.99.100:30676/hi

```
$ kubectl describe service student-service
```

## Ingress Controller
Create Ingress or Basic Authentication Ingress

### Create Ingress
```
$ cd devops
$ kubectl create -f ingress.yml
```

### Using Basic Auth Ingress
Add user & pass when we reach the site.

#### Create htpasswd file
```
$ htpasswd -c auth example
New password: <bar>
New password:
Re-type new password:
Adding password for user example
```

#### Create secret
```
$ kubectl create secret generic basic-auth --from-file=auth
secret "basic-auth" created
```

#### Get Secert
Make sure that the auth secret was created

```
$ kubectl get secret basic-auth -o yaml
```

#### Create basic auth ingress
```
$ cd devops
$ kubectl create -f basic-auth-ingress.yml
```

#### Check Ingress
```
 curl -v http://192.168.99.100/student/hi -H 'Host: mysite.com'
*   Trying 192.168.99.100...
* TCP_NODELAY set
* Connected to 192.168.99.100 (192.168.99.100) port 80 (#0)
> GET /student/hi HTTP/1.1
> Host: mysite.com
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 401 Unauthorized
< Server: nginx/1.13.6
< Date: Tue, 27 Nov 2018 03:43:30 GMT
< Content-Type: text/html
< Content-Length: 195
< Connection: keep-alive
< WWW-Authenticate: Basic realm="Authentication Required - Example"
<
<html>
<head><title>401 Authorization Required</title></head>
<body bgcolor="white">
<center><h1>401 Authorization Required</h1></center>
<hr><center>nginx/1.13.6</center>
</body>
</html>
* Connection #0 to host 192.168.99.100 left intact
```

#### Add user & pass
```
curl -v http://192.168.99.100/student/hi -H 'Host: mysite.com' -u 'example:example'
*   Trying 192.168.99.100...
* TCP_NODELAY set
* Connected to 192.168.99.100 (192.168.99.100) port 80 (#0)
* Server auth using Basic with user 'example'
> GET /student/hi HTTP/1.1
> Host: mysite.com
> Authorization: Basic ZXhhbXBsZTpleGFtcGxl
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 200
< Server: nginx/1.13.6
< Date: Tue, 27 Nov 2018 03:44:38 GMT
< Content-Type: text/plain;charset=UTF-8
< Content-Length: 10
< Connection: keep-alive
< X-Application-Context: student-service:7000
<
* Connection #0 to host 192.168.99.100 left intact
Hi Student%
```

#### Check Ingress
```
$ kubectl describe ing
```
![Ingress](https://github.com/nhatthai/spring-minikube/blob/master/images/status-ingress.png "Ingress")

#### Enable Ingress
```
$ minikube addons enable ingress
```

```
$ minikube ip
192.168.99.100
```

#### Add mysite.com into /etc/hosts
```
192.168.99.100 mysite.com
```
Check browser: `http://mysite.com/student/hi`
