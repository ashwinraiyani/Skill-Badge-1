# Skill-Badge-1 Getting Started: Create and Manage Cloud Resources

# Task 1: Create an instance.

Navigation menu > Compute engine > VM Instance -> Create 

![screen](https://github.com/ashwinraiyani/Skill-Badge-1/blob/master/image.png)

**Check the Progess**
  
# Task 2: Create a 3 node Kubernetes cluster and run a simple service.

Goto Cloud Shell

gcloud container clusters create nucleus-backend --num-nodes 1 --network nucleus-vpc --region us-east1 

gcloud container clusters get-credentials nucleus-backend  --region us-east1

kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:2.0

kubectl expose deployment hello-server --type=LoadBalancer --port 8080
          
 **Check the Progess**   
          
# Task 3: Create an HTTP(s) Load Balancer in front of two web servers.

In Cloud Shell run the following codes.

          cat << EOF > startup.sh
          #! /bin/bash
          apt-get update
          apt-get install -y nginx
          service nginx start
          sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
          EOF

1 .Create an instance template :
gcloud compute instance-templates create web-server-template \    
          --metadata-from-file startup-script=startup.sh \    
          --network nucleus-vpc \   
          --machine-type g1-small \   
          --region us-east1

3 .Create a managed instance group :
gcloud compute instance-groups managed create web-server-group \    
          --base-instance-name web-server \   
          --size 2 \    
          --template web-server-template \    
          --region us-east1

4 .Create a firewall rule to allow traffic (80/tcp) :
gcloud compute firewall-rules create web-server-firewall \    
          --allow tcp:80 \    
          --network nucleus-vpc
          
5 .Create a health check :          
gcloud compute http-health-checks create http-basic-check

gcloud compute instance-groups managed \    
          set-named-ports web-server-group \    
          --named-ports http:80 \   
          --region us-east1
          
6 .Create a backend service and attach the manged instance group :
gcloud compute backend-services create web-server-backend \   
          --protocol HTTP \   
          --http-health-checks http-basic-check \   
          --global
          
          
gcloud compute backend-services add-backend web-server-backend \    
          --instance-group web-server-group \   
          --instance-group-region us-east1 \    
          --global
          
7 .Create a URL map and target HTTP proxy to route requests to your URL map :
gcloud compute url-maps create web-server-map \   
          --default-service web-server-backend
          
gcloud compute target-http-proxies create http-lb-proxy \   
          --url-map web-server-map
          
8 .Create a forwarding rule :
gcloud compute forwarding-rules create http-content-rule \    
        --global \    
        --target-http-proxy http-lb-proxy \   
        --ports 80


gcloud compute forwarding-rules list

Wait for 5-7 minutes(depend on internet speed) and then check progress 
