# Deploy_via_Ingres_Nginx


Add Helm repository by Bitnami:

Run: `helm repo add bitnami https://charts.bitnami.com/bitnami`

Checking: `helm repo list`

<img width="475" alt="Screen Shot 2022-06-13 at 11 37 17 AM" src="https://user-images.githubusercontent.com/43518207/173336065-1d1ada93-a5e2-4da0-86a2-9fc66ab44aec.png">

Run: `helm repo update`

Create a file yaml (local-pv.yaml):
```apiVersion: v1
kind: PersistentVolume # Create a PV
metadata:
  name: postgresql-data # Sets PV's name
  labels:
    type: local # Sets PV's type to local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi # Sets PV Volume
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/data/volume" # Sets the volume's path
```
Run: `kubectl apply -f local-pv.yaml`

Create a file yaml (pv-claim.yaml):
```apiVersion: v1
kind: PersistentVolumeClaim # Create PVC
metadata:
  name: postgresql-data-claim # Sets name of PV
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce # Sets read and write access
  resources:<img width="1076" alt="Screen Shot 2022-06-13 at 11 54 19 AM" src="https://user-images.githubusercontent.com/43518207/173338821-ad578c90-94fa-4067-8bfc-6ede076c0191.png">

    requests:
      storage: 10Gi # Sets volume size
```
Run: `kubectl apply -f pv-claim.yaml`<img width="752" alt="Screen Shot 2022-06-13 at 12 02 05 PM" src="https://user-images.githubusercontent.com/43518207/173340077-d7f312e6-f432-44f6-88f8-b8804f61276f.png">


Checking PersistentVolume:
Run: `kubectl get pv`

<img width="1046" alt="Screen Shot 2022-06-13 at 12 00 51 PM" src="https://user-images.githubusercontent.com/43518207/173339948-9beddf87-1f27-4d84-8879-57cef3efad1a.png">

Checking PersistentVolumeClaim:
Run: `kubectl get pvc`

<img width="752" alt="Screen Shot 2022-06-13 at 12 02 05 PM" src="https://user-images.githubusercontent.com/43518207/173340131-dcebf223-6f63-4ab4-884c-d8efb2242800.png">

Create a file yaml for deployment (values.yaml):
```# define default database user, name, and password for PostgreSQL deployment
auth:
  enablePostgresUser: true
  postgresPassword: "StrongPassword"
  username: "app1"
  password: "AppPassword"
  database: "app_db"

# The postgres helm chart deployment will be using PVC postgresql-data-claim
primary:
  persistence:
    enabled: true
    existingClaim: "postgresql-data-claim"
```
Run: `helm install postgresql-dev --set volumePermissions.enabled=true -f values.yaml bitnami/postgresql`

Checking pods: `kubectl get pods`
<img width="567" alt="Screen Shot 2022-06-13 at 11 53 32 AM" src="https://user-images.githubusercontent.com/43518207/173338707-d3ce13ac-3141-43d7-805a-991cec00f4e4.png">


Checking logs of pods:`kubectl logs postgresql-dev-0`
<img width="1076" alt="Screen Shot 2022-06-13 at 11 54 19 AM" src="https://user-images.githubusercontent.com/43518207/173338847-ad6c480e-d310-4ca2-9763-e0a5dd69ca91.png">

```
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgresql-dev -o jsonpath="{.data.password}" | base64 --decode)

kubectl run postgresql-dev-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:14.3.0-debian-11-r3 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host postgresql-dev -U app1 -d app_db -p 5432
```
Checking PostgreSQL connection: `\conninfo`
<img width="887" alt="Screen Shot 2022-06-13 at 11 58 51 AM" src="https://user-images.githubusercontent.com/43518207/173339545-3ec56cfd-c4d2-4de4-99d5-30ba18658419.png">

Logout from PostgreSQL shell: `\q`

<img width="466" alt="Screen Shot 2022-06-13 at 12 00 07 PM" src="https://user-images.githubusercontent.com/43518207/173339760-8cdfc87d-0a68-490f-b365-970d194001a1.png">

Enable Nginx Ingress Controller: `minikube addons enable ingress`

<img width="643" alt="Screen Shot 2022-06-13 at 12 05 22 PM" src="https://user-images.githubusercontent.com/43518207/173340685-8f7d2a7b-6784-4bd3-92f0-452e543b4969.png">

Validate that NGINX is Running: `kubectl get pods -n ingress-nginx`

<img width="660" alt="Screen Shot 2022-06-13 at 12 09 08 PM" src="https://user-images.githubusercontent.com/43518207/173341308-c0b5fc29-1fa6-4f65-abb2-70caf9ef0125.png">



