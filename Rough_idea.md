create instance with c7i flex large (SG,keypair ,vol=15gb)

create EKS (cluster+nodes)

on EC2 (install k8s,aws,mysql,docker)

login to docker hub

create RDS (admin,redhat123)

from ec2 login to RDS and create student_db database and table with schema

GRANT ALL PRIVILEGES ON springbackend.* TO 'admin'@'localhost' IDENTIFIED BY 'redhat123';


sudo apt install mysql-client

issue (secutirty group ---> nodes not visible )

changes in src/main/resource/ application.properties
mention the endpoint of RDS and username - passowrd in the file
now back to the backend dir build docker image
 docker build . -t prasanna369/easy-backend:v2
docker push prasanna369/easy-backend:v2

 write the pod file or mention the image name in the existing pod file 
and run pod file 
now run the svc file
AND  get the endpoint of the svc file (loadbalancer)

now switch to frontend make changes to .env file 
paste the endpoint of loadbalaner

now create the dockerfile docker build . -t prasanna369/easy-frontend:v2

push this imaage to docker hub

now add this image to the deployment file 

and run the file also run svc file 












