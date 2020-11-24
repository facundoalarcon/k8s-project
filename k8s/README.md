## minikube

minikube start

minikube ip

minikube dashboard

minikube delete

## Commands

kubectl apply -f <config-file / folder>

kubectl get pods

kubectl get services

kubectl describe <object-type> (<object-name>)

kubectl delete -f <config-file>

kubectl get deployments

kubectl get pods -o wide

kubectl set image <object-type> / <object-name> <container-name> = <new-image-to-use>

kubectl set image deployment/client-deployment client=facundoalarcon/multi-client:v1

## ver containers de minikube - cambio de docker-server

eval $(minikube docker-env)

## troubleshooting

docker logs <container-id>

docker exect -it <containter-id> sh

kubectl logs <pod-id>

kubectl exec -it <pod-id> sh

## storage classess
kubectl get storageclass

kubectl describe storageclass

https://kubernetes.io/docs/concepts/storage/storage-classes/

## get persistent volumes (pv) and persistent volume claim (pvc)

kubectl get pv

kubectl get pvc

## create secrets

Esto conviene crearlo manualmente para no tener un archivo de config

kubectl create secret <type> <secret_name> --from-literal key=value

type:
- generic: indica que estamos guardando un numero arbitrario de conjuntos key=value 
- docker-registry
- tls

### ejemplo de uso

kubectl create secret generic pgpassword --from-literal PGPASSWORD=12345asdf

kubectl get secrets

env:
  - name: POSTGRES_PASSWORD     // valor que espera que se le pase el container (depende de la imagen, en el caso de postgresql espera eso para la clave)
    valueFrom:
      secretKeyRef:
        name: pgpassword        // nombre que se le dio al secret al crearlo
        key: PGPASSWORD         // key (con eso recupera el valor de la clave)

### nota
si alguna variable de entorno esta en base64/cadena de string y otras en nro saldra un error (entonces ahi hay que pasarlas todas a cadenas de string)


## eliminar cached images en el nodo
kubectl delete services <name>
kubectl delete pods <name>

### previo cambio de docker-server 

docker system prune -a

### combinar varios config files en uno solo 
usar tres veces - para separar los objetos

object1
---
object2

## NGINX ingress-controller
https://github.com/kubernetes/ingress-nginx

minikube addons enable ingress

https://github.com/kubernetes/minikube/issues/8756

### notes (old)
```
apiVersion: networking.k8s.io/v1beta1
# UPDATE THE API
kind: Ingress
metadata:
  name: ingress-service
  annotations:
  ### las annotations son escencialmente opciones de configurarion adicional que van a especificar un tipo de configuracion de nivel superior alrededor del objeto de ingreso (Ingress) que se esta creando
    kubernetes.io/ingress.class: nginx
    ### la linea anterior indica Ingress controller basado en el proyecto nginx 
    nginx.ingress.kubernetes.io/use-regex: 'true'
    # ADD THIS LINE ABOVE
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    ### la linea anterior indica como se comporta nuestra copia de nginx (esta es la / de direccion de nginx)
    # UPDATE THIS LINE ABOVE
spec:
  rules:
    - http:
        paths:
          - path: /?(.*)
          # UPDATE THIS LINE ABOVE
            backend:
              serviceName: client-cluster-ip-service
              ### client-cluster-ip-service es el nombre del servicio
              servicePort: 3000
          - path: /api/?(.*)
          # UPDATE THIS LINE ABOVE
            backend:
              serviceName: server-cluster-ip-service
              ### server-cluster-ip-service es el nombre del servicio
              servicePort: 5000
```
### notes (new)
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-service
  annotations:
    kubernetes.io/ingress.class: 'nginx'
    nginx.ingress.kubernetes.io/use-regex: 'true'
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - http:
        paths:
          - path: /?(.*)
            pathType: Prefix
            backend:
              service:
                name: client-cluster-ip-service
                port:
                  number: 3000
          - path: /api/?(.*)
            pathType: Prefix
            backend:
              service:
                name: server-cluster-ip-service
                port:
                  number: 5000
```