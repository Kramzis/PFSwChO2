# SPRAWOZDANIE Z LABORATORIUM PROGRAMOWANIE FULL-STACK W CHMURZE OBLICZENIOWEJ

## Kramek Magdalena

## Pliki konfiguracyjne

- **Plik:** `cluster-setup.yaml`
	Zawiera konfiguracje przestrzeni nazw appns-a oraz appns-b, deploymentów oraz serwisów dla aplikacji app-a & app-b. Dodatkowo zawiera serwisy typu ExternalName, defaultowy deploy dla backendu z serwisem dla niego, a także konfigurację Ingress.
- **Plik** `network-policy.yaml`
	- Zawiera natomiast definicje reguł sieciowych dla obu przestrzeni nazw.

## Uruchomienie

Należy uruchomić minikube'a oraz wywołać te dwa polecenia:

```bash
kubectl apply -f setup.yaml
kubectl apply -f network-policy.yaml
```
## Potwierdzenie poprawności działania

- Konfiguracja:
```bash
(kali㉿kali)-[~]
└─$ kubectl get deployments,pods -n appns-a
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app-a   2/2     2            2           6m5s

NAME                         READY   STATUS    RESTARTS   AGE
pod/app-a-5799485f95-9g4wx   1/1     Running   0          6m5s
pod/app-a-5799485f95-phf7f   1/1     Running   0          6m5s
                                                                                       
┌──(kali㉿kali)-[~]
└─$ kubectl get deployments,pods -n appns-b                                          
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app-b   2/2     2            2           6m23s

NAME                         READY   STATUS    RESTARTS   AGE
pod/app-b-76fd6dfbf5-987mq   1/1     Running   0          6m23s
pod/app-b-76fd6dfbf5-vnt9j   1/1     Running   0          6m23s

┌──(kali㉿kali)-[~]
└─$ kubectl get deployments,pods -n default 
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-app   1/1     1            1           7m50s

NAME                             READY   STATUS    RESTARTS        AGE
pod/emptydir-pod                 2/2     Running   2 (8m30s ago)   18d
pod/hello-app-5847c68f5d-rmdq4   1/1     Running   0               7m50s
pod/hostpath-pod                 2/2     Running   2 (8m30s ago)   18d
```
- Ingress:
```
bash
┌──(kali㉿kali)-[~]
└─$ kubectl get ingress -n default         
NAME           CLASS   HOSTS                   ADDRESS        PORTS   AGE
lab9-ingress   nginx   a.lab9.net,b.lab9.net   192.168.49.2   80      5m36s
```
- Sprawdzenie poprawności reguł sieciowych:
```
bash
┌──(kali㉿kali)-[~]
└─$ kubectl exec -it app-a-5799485f95-9g4wx --namespace=appns-a -- curl -s -o /dev/null -w "%{http_code}\n" http://app-b.appns-b.svc.cluster.local
000
command terminated with exit code 6
```
W obu tych przestrzeniach nazw zabroniono wszelkiej komunikacji, a więc 
reguły sieciowe są poprawne.

- Testowanie działania Ingress:

```
bash
┌──(kali㉿kali)-[~]
└─$ curl -H "Host: a.lab9.net" http://192.168.49.2:80

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
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
```
```
bash
┌──(kali㉿kali)-[~]
└─$ curl -H "Host: b.lab9.net" http://192.168.49.2:80

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
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
```
