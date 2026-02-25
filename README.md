#  EasyCRUD – Full Stack Application Deployment
EasyCRUD is a full-stack CRUD (Create, Read, Update, Delete) application deployed using **Amazon RDS**, **EC2**, **Docker** and **AWS EKS**.  
The backend is developed using **Spring Boot**, the frontend uses **React + Nginx/httpd**, and the database runs on **MariaDB (RDS)**.
This project demonstrates **containerization with Docker**, **Kubernetes orchestration on AWS EKS**, **managed database integration with Amazon RDS**, and a **scalable microservices architecture**.



---

## 🛠️ Tech Stack

- **Cloud:** AWS (EC2, RDS)
- **Backend:** Spring Boot (Java)
- **Frontend:** Vite + Node.js
- **Database:** MariaDB (Amazon RDS)
- **Containerization:** Docker
- **Orchestration:** Kubernetes (K8s)
- **Web Server:** Apache (inside frontend container)

---

## ⚙️ Prerequisites

- AWS Account
- Ubuntu EC2 Instance
- Amazon RDS (MariaDB)
- AWS EKS (Kubernetes)
- Docker installed
- Required ports open in Security Groups:
  - `22` (SSH)
  - `80` (Frontend)
  - `8080` (Backend)
  - `3306` (RDS – only from EC2 SG)

---

# 🔹 PHASE 0: Create AWS EKS Cluster 
Follow the complete database setup guide (**Step 1 to Step 6**) here:
[AWS EKS Setup Documentation](https://github.com/mhprasanna-spec/Kubernetes.git)

# 🔹 PHASE 1: Database Setup (Amazon RDS – MariaDB)


## Step 1: Create RDS Database

1. Login to **AWS Console → RDS**
2. Click **Create database**
3. Choose the following options:
   - **Database Engine:** MariaDB
   - **Template:** Free Tier / Production (as required)
   - **DB Identifier:** `student-db`
   - **Master Username & Password:** (set your own)
4. Connectivity:
   - **VPC:** Same VPC as EC2
   - **Public Access:** Yes (for testing)
   - **Security Group:** Allow inbound `3306` from EC2 Security Group
5. Click **Create Database**
6. Copy the **RDS Endpoint**

---

## Step 2: Launch and Prepare EC2 Instance

1. Launch an **Ubuntu EC2 instance**
2. SSH into the instance
3. Update the system:

```bash
sudo apt update -y
```
## Step 3: Install MySQL Client

Install MySQL client on the EC2 instance to connect with the Amazon RDS database.

```bash
sudo apt install mysql-client -y
```
## Step 4: Connect to RDS from EC2

Use the RDS endpoint to connect to the MariaDB database from the EC2 instance.

```bash
mysql -h <RDS-ENDPOINT> -u admin -p
```
## Step 5: Database Operations

Execute the following SQL commands inside the MySQL shell:

```sql
SHOW DATABASES;
CREATE DATABASE student_db;
```
change username and passowrd which is given in the RDS database
```
GRANT ALL PRIVILEGES ON springbackend.* TO 'username'@'localhost' IDENTIFIED BY 'your_password';
```
```
USE student_db;
```
## Step 6: Create Students Table

Create the `students` table to store student records.

```sql
CREATE TABLE students (
  id BIGINT NOT NULL AUTO_INCREMENT,
  name VARCHAR(255),
  email VARCHAR(255),
  course VARCHAR(255),
  student_class VARCHAR(255),
  percentage DOUBLE,
  branch VARCHAR(255),
  mobile_number VARCHAR(255),
  PRIMARY KEY (id)
);
```


# 🔹 PHASE 2: Backend Deployment


## Step 7: Install Docker

Install Docker on the EC2 instance and start the Docker service.

```bash
   # Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```
```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
login to the dockerhub
```
docker login
```

## Step 8: Clone Project Repository

Clone the EasyCRUD project repository and navigate to the backend directory.

```bash
git clone https://github.com/mhprasanna-spec/EasyCRUD-Updated_k8s.git
```
Move to backend directory
```bash
cd EasyCRUD-Updated_k8s/backend/
```

## Step 9: Configure Backend Application

```bash
nano src/main/resources/application.properties
```
### Update Configuration Values

Update the following values in the `application.properties` file:

- **RDS Endpoint**
- **Database Name:** `student_db`
- **Database Username**
- **Database Password**

```bash
server.port=8080

spring.datasource.url=jdbc:mariadb://database-1.cry6emuyikgl.ap-south-1.rds.amazonaws.com:3306/student_db?sslMode=trust
spring.datasource.username=admin
spring.datasource.password=redhat123

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

```

## Step 10: Create Dockerfile for Backend

Create a Dockerfile for the backend application.

```bash
nano Dockerfile
```
Add the following content to the Dockerfile:

```bash
FROM maven:3.8.3-openjdk-17
COPY . /opt/
WORKDIR /opt
RUN mvn clean package 
WORKDIR target/
EXPOSE 8080
ENTRYPOINT ["java","-jar"]
CMD ["student-registration-backend-0.0.1-SNAPSHOT.jar"]
```
## Step 11: Build Backend Docker Image

Build the Docker image for the backend application and verify the image creation.
```bash
docker build <dockerfile-path> -t <dockerhub_username>/<images-name>:<tag>
```
Example :
```bash
docker build . -t prasanna369/easy-backend:v2
```
Verify Docker images
```bash
docker images
```
push the image to the docker hub
```
docker push prasanna369/easy-backend:v2
```
## Step 12: Create backend Pod+Service Manifest:

create backend-pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: backend
  labels:
    app: backend
spec:
  containers:
    - name: backend
      image: prasanna369/easy-backend:v2
      ports:
        - containerPort: 80
```
run the pod file
```
kubectl apply -f backend-pod.yaml

```
Verify pod is running
```
kubectl get pods
```
create service file backend-svc.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  type: LoadBalancer
  selector:
    app: backend
  ports:
   - port: 8080
     targetPort: 8080
```
run the service file
```
kubectl apply -f backend-svc.yaml
```
verify service file is running 
```
kubectl get svc
```
there will be something like this
```
NAME           TYPE           CLUSTER-IP     EXTERNAL-IP                                                               PORT(S)          AGE
backend-svc    LoadBalancer   10.100.66.93   a6ccb85bab6c54d1bb2891463fba43b2-2080745944.us-east-1.elb.amazonaws.com   8080:32297/TCP   6m3s
```
note the EXternal-ip which will be used in frontend (in .env file)

✅ **Backend deployed successfully**

# 🔹 PHASE 3: Frontend Deployment 

## Step 14: Navigate to Frontend Directory

Navigate to the frontend directory and list all files, including hidden files such as `.env`.

```bash
cd ../frontend/
```
Show hidden files (.env)
```bash
ls -a
```
## Step 15: Configure Environment File

Edit the `.env` file to configure the backend API URL.

```bash
nano .env
```
Update the value as shown below:
```bash
VITE_API_URL=http://<BACKEND_PUBLIC_IP>:8080/api
```
example:
```
VITE_API_URL=http://a6ccb85bab6c54d1bb2891463fba43b2-2080745944.us-east-1.elb.amazonaws.com:8080/api
```
## Step 16: Create Frontend Dockerfile

Create a Dockerfile for the frontend application.

```bash
nano Dockerfile
```
Add the following content to the Dockerfile:
```bash
FROM node:24-alpine
COPY . /opt/
WORKDIR /opt
RUN npm install && npm run build
RUN apk update && apk add apache2
RUN rm -rf /var/www/localhost/htdocs/*
RUN cp -rf dist/* /var/www/localhost/htdocs
EXPOSE 80
ENTRYPOINT ["httpd","-D","FOREGROUND"]
```
## Step 17: Build Frontend Docker Image

Build the Docker image for the frontend application .

```bash
docker build . -t prasanna369/easy-frontend:v2
```
verify the image creation
```bash
docker images
```
push the image to the docker hub
```
docker push prasanna369/easy-frontend:v2
```

## Step 18: Create frontend Deployement+Service Manifest:

create the Deployment file frontend-deploy.yml.

```bash
apiVersion: apps/v1
kind: Deployment
metadata: 
    name: frontend 
    labels:
       app: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      name: frontend
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: prasanna369/easy-frontend:v2
          ports:
            - containerPort: 80
```
Run the file
```
kubectl apply -f frontend-deploy.yml
```
verify that it is running
```bash
kubectl get pods
```
create the service file frontend-svc.yml
```
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
```

verify the service file 
```
kubectl get svc
```
output is something like this
```
NAME           TYPE           CLUSTER-IP     EXTERNAL-IP                                                               PORT(S)          AGE
backend-svc    LoadBalancer   10.100.66.93   a6ccb85bab6c54d1bb2891463fba43b2-2080745944.us-east-1.elb.amazonaws.com   8080:32297/TCP   6m42s
frontend-svc   LoadBalancer   10.100.87.52   acd9a5f55b6374087bb94cc42d01efb9-1929408836.us-east-1.elb.amazonaws.com   80:32090/TCP     47s
kubernetes     ClusterIP      10.100.0.1     <none>                                                                    443/TCP          52m
```
## Step 19: Verify Frontend

Verify that the frontend application is running successfully.

Open a browser and navigate to:
```bash
http://<frontend_EXTERNAL-IP>
```

🎉 **EasyCRUD application is live and fully functional!**

**Enter the values through the application portal and verify the data by logging into the RDS database**

```
mysql -h <RDS_Endpoint> -u <username> -p<password> 
```






