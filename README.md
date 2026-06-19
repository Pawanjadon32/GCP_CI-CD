# GKE Full Stack Application Deployment Notes

## Project Objective

GKE cluster par 3-tier application deploy ki:

Frontend → Backend → MySQL

User UI se data add karta hai, Backend API request process karti hai aur data MySQL database me store hota hai.

---

## Architecture

User
↓
Frontend Service (LoadBalancer)
↓
Frontend Pods (Nginx)
↓
Backend Service (ClusterIP)
↓
Backend Pods (NodeJS API)
↓
MySQL Service (ClusterIP)
↓
MySQL Pod + PVC

---

## Kubernetes Components Used

### Namespace

* fullstack namespace create kiya.
* Saare resources namespace ke andar deploy kiye.

### Secret

MySQL credentials Secret me store kiye:

* MYSQL_ROOT_PASSWORD
* MYSQL_DATABASE
* MYSQL_USER
* MYSQL_PASSWORD

Purpose:
Sensitive data ko hardcode karne se bachana.

---

### ConfigMap

#### Frontend ConfigMap

Nginx configuration store ki:

* "/" request → Frontend page serve kare
* "/api/*" request → Backend service ko forward kare

Purpose:
Frontend aur Backend ke beech reverse proxy.

#### Backend ConfigMap

Store kiya:

* DB_HOST
* DB_PORT
* DB_NAME
* DB_USER

Purpose:
Backend ko database connection details provide karna.

---

### PVC (PersistentVolumeClaim)

MySQL ke liye 5GB storage request ki.

Purpose:
Pod delete hone ke baad bhi database data persist rahe.

---

### MySQL Deployment

* Image: mysql:8.0
* Replica: 1
* Strategy: Recreate
* Secret se credentials read kiye
* PVC mount kiya

Purpose:
Database service provide karna.

---

### Backend Deployment

* Replica: 2
* ClusterIP Service
* ConfigMap aur Secret use kiye
* MySQL connection establish ki

Purpose:
API requests handle karna.

---

### Frontend Deployment

* Replica: 2
* LoadBalancer Service
* Nginx reverse proxy use kiya

Purpose:
User interface provide karna.

---

## Services

### Frontend Service

Type: LoadBalancer

Purpose:
Internet se access dena.

Example:

http://<LoadBalancer-IP>

---

### Backend Service

Type: ClusterIP

Purpose:
Sirf cluster ke andar accessible.

Frontend backend-service naam se backend ko access karta hai.

---

### MySQL Service

Type: ClusterIP

Purpose:
Backend ko database access provide karna.

---

## Kubernetes DNS

Backend MySQL ko IP se access nahi karta.

Example:

DB_HOST=mysql-service

Kubernetes DNS automatically service ko resolve karta hai.

---

## Practical Issues Faced

### Issue 1 - GKE Cluster Creation Failed

Error:

SSD_TOTAL_GB quota exceeded

Reason:
Project quota exhaust ho chuka tha.

Resolution:
Unused resources/projects delete kiye aur quota free kiya.

---

### Issue 2 - gke-gcloud-auth-plugin Missing

Error:

gke-gcloud-auth-plugin not found

Resolution:

Plugin install kiya aur cluster credentials refresh kiye.

---

### Issue 3 - Unschedulable Pods

Cluster create hone ke baad kuch system pods schedule nahi hue.

Debug Commands:

kubectl get pods -A
kubectl describe pod <pod-name>

Reason:
Cluster initialization aur resource availability issue.

Baad me cluster stabilize hone par pods schedule ho gaye.

---

### Issue 4 - Backend Pods Not Ready

Status:

0/1 Running

Backend continuously restart ho raha tha.

Debug:

kubectl logs <backend-pod>

Error:

Host is not allowed to connect to this MySQL server

Reason:
MySQL user/database configuration mismatch tha.

Resolution:
MySQL ko reinitialize kiya aur backend restart kiya.

Result:

Backend status:

1/1 Running

---

### Issue 5 - MySQL Multiple Replicas

MySQL replicas = 2

Problem:

Ek pod Ready hua.
Dusra pod Ready nahi hua.

Reason:

Dono pods same PVC use karne ki koshish kar rahe the.

Database workloads ke liye Deployment + Multiple Replicas suitable nahi tha.

Resolution:

MySQL replicas ko 1 kiya.

Learning:

Database ke liye StatefulSet preferred hai.

---

### Issue 6 - Root Password Authentication Failed

Error:

Access denied for user 'root'

Reason:

PVC me purana MySQL data persist tha.
Secret update hone ke baad bhi old password active tha.

Resolution:

PVC recreate karke fresh database initialize kiya.

---

## Validation Performed

Frontend Validation

Browser:
http://<LoadBalancer-IP>

Frontend page successfully loaded.

---

Backend Validation

Backend pods healthy:

READY 1/1

---

Database Validation

Frontend se item add kiya:

"my name is pawan"

Result:

Frontend
↓
Backend API
↓
MySQL Insert
↓
Frontend Display

Data successfully database me store hua.

---

## Key Learnings

* GKE Cluster Creation
* kubectl Cluster Access
* Namespace Usage
* Secret Management
* ConfigMap Usage
* PVC and Persistent Storage
* Service Discovery
* ClusterIP vs LoadBalancer
* Frontend Reverse Proxy
* Backend to MySQL Connectivity
* Kubernetes Debugging
* Pod Logs Analysis
* Readiness and Liveness Probes
* Database Deployment Best Practices
* End-to-End Full Stack Application Deployment on GKE


I deployed a 3-tier application on GKE.

Architecture:

Frontend (Nginx) → Backend (NodeJS API) → MySQL Database

I created a GKE cluster and deployed all components in a dedicated namespace.

Frontend was exposed using a LoadBalancer service, while Backend and MySQL were exposed using ClusterIP services for internal communication.

I used ConfigMaps for application configuration and Secrets for MySQL credentials. For database persistence, I configured a PersistentVolumeClaim (PVC).

Frontend requests were routed to the backend through Nginx reverse proxy, and backend APIs stored data into MySQL.

During deployment, I faced multiple issues such as GKE quota limits, cluster connectivity problems, unschedulable pods, backend-to-MySQL authentication failures, and PVC-related database initialization issues.

I debugged them using kubectl logs, describe commands, services, PVCs, and pod events.

Finally, I validated the complete flow by adding data from the frontend UI, which was successfully processed by the backend and stored in MySQL running inside Kubernetes.
