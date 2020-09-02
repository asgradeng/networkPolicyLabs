# networkPolicyLabs

You have been given access to a three-node cluster. You will be responsible for creating a deployment and a service to serve as a front end for a web application. In addition to the web application, you must deploy a Redis database and make sure the web application can only access this database using the default port of 6379. You will first create a default-deny network policy, so all pods within your Kubernetes are not able to communicate with each other by default. Then you will create a second network policy that specifies the communication on port 6379 between the web application and the database using their label selectors. You must apply these specifications to your resources in order to complete this hands-on lab:

Create a deployment named webfront-deploy.
The deployment should use the image nginx with the tag 1.7.8.
The deployment should expose container port 80 on each pod and contain 2 replicas.
Create a service named webfront-service and expose port 80, target port 80.
The service should be exposed externally by listening on port 30080 on each node.
Create one pod named db-redis using the image redis and the tag latest.
Verify that you can communicate to pods by default.
Create a network policy named default-deny that will deny pod communication by default.
Verify that you can no longer communicate between pods.
Apply the label role=frontend to the web application pods and the label role=db to the database pod.
Create a network policy that will apply an ingress rule for the pods labeled with role=db to allow traffic on port 6379 from the pods labeled role=frontend.
Verify that you have applied the correct labels and created the correct network policies.



CKA Practice Exam: Part 3
In this part of the practice exam, you will be responsible for creating a default-deny policy, as well as explicitly stating communication over a certain port. This will simulate a possible exam question and have you show your expertise with pod communication and security.

Solution
Log in to the Kube Master server using the credentials provided:

ssh cloud_user@<KUBEMASTER_PUBLIC_IP_ADDRESS>
Remember: Instead of having to copy/paste or type out entire commands you've run before or slightly different versions of previous commands, you can hit the up arrow key to go through your history of commands.

Create a deployment and a service to expose your web front end.
Create the YAML for your deployment:

kubectl create deployment webfront-deploy  --image=nginx:1.7.8  --dry-run -o yaml > webfront-deploy.yaml
Open the file:

vim webfront-deploy.yaml
At the bottom, under resources, add the following in order to expose the container port:

ports:
- containerPort: 80
Save and exit by hitting Escape and then :wq!.

Create your deployment:

kubectl apply -f webfront-deploy.yaml
Scale up your deployment:

kubectl scale deployment/webfront-deploy --replicas=2
Create the YAML for a service:

kubectl expose deployment/webfront-deploy --port=80 --target-port=80 --type=NodePort --dry-run -o yaml > webfront-service.yaml
Open the file:

vim webfront-service.yaml
Change the name to webfront-service.

Under targetPort, enter:

nodePort: 30080
Save and exit by hitting Escape and then :wq!.

Create the service:

kubectl apply -f webfront-service.yaml
Verify that you can communicate with your pod directly:

kubectl run busybox --rm -it --image=busybox /bin/sh
At the prompt, enter:

wget --spider --timeout=1 webfront-service
We should see we'll be connected to it.

Exit out of the shell:

exit
Get the pod's IP address:

kubectl get po -o wide
Note the IP address. (You may want to copy and paste it into a text file, as we'll need it again in a few minutes.)

Again run:

kubectl run busybox --rm -it --image=busybox /bin/sh
At the prompt, enter:

wget -O- <POD_IP_ADDRESS>:80
We should see the Nginx welcome page, which is what we want.

Exit out of the shell:

exit
Create a database server to serve as the backend database.
Create a Redis pod:

kubectl run db-redis --image=redis --restart=Never
Create a network policy that will deny communication by default.
Create the file:

vim default-deny.yaml
Enter the following network policy (hint: before pasting in the code, first enter :set paste in order to avoid unnecessary spaces or hashes):

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
Save and quit with Escape followed by :wq!.

Apply the network policy:

kubectl apply -f default-deny.yaml
Verify that communication has been disabled by default:

kubectl run busybox --rm -it --image=busybox /bin/sh
At the prompt, enter:

wget -O- <POD_IP_ADDRESS>:80
This time, it won't connect since we applied the default-deny network policy.

Hit Ctrl+C to stop the process.

Exit out of the shell:

exit
Apply the labels and create a communication over port 6379 to the database server.
Get pods:

kubectl get po
Apply label to the Redis pod:

kubectl label po db-redis role=db
Apply labels to both webfront pods (running this command twice — once for each pod — and replacing <POD_NAME> with the pod names listed in the previous command's output):

kubectl label po <POD_NAME> role=frontend
View the labels:

kubectl get po --show-labels
Create another network policy file (this one will use the label selectors to apply the network policy to the pods):

vim redis-netpolicy.yaml
Enter the following:

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-netpolicy
spec:
  podSelector:
    matchLabels:
      role: db
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - port: 6379
Save and quit with Escape followed by :wq!.

Create the network policy:

kubectl apply -f redis-netpolicy.yaml
View the network policies:

kubectl get netpol
We should see our two network policies we created.

Describe the custom network policy:

kubectl describe netpol redis-netpolicy
Show the labels on the pods:

kubectl get po --show-labels
Conclusion
Congratulations on completing this portion of the practice exam
