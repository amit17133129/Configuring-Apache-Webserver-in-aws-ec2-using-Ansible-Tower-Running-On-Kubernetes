# CONFIGURING APACHE WEBSERVER ON AWS EC2 USING ANSIBLE TOWER 🗼 RUNNING INSIDE KUBERNETES🎡🎡(MINIKUBE).


<p align="center">
  <img width="1000" height="600" src="https://miro.medium.com/max/3840/1*4rz1Kw4lvrS6WxKguN8pjg.gif">
</p> 

Hello guys, i hope you all doing good in your technical journey. I am back with another interesting article which will help you to learn a new demanding tool. We all know ansible has it’s own space in the automation market but here what makes interesting to move towards ansible advance. Yes its Ansible Tower. As we all know that ansible is a cli tool which helps you to configure and provision as well. But it restrict us to only cli. So, ansible tower will provide a GUI based web UI from where you can run all the playbooks to all the respective managed node(target node). It’s not only GUI but it gives us more flexibility to automate the systems easily. So, lets begin with ansible tower installation and its configuration. Here i will be installing minikube first and creating a single node kubernetes cluster and i will run ansible tower inside kubernetes single node cluster.

## `Take Away from this article:`
👉🏻 You will know how we can install minkube on aws-instance.
👉🏻 Configuring Kubectl inside minikube.
👉🏻 Installing Ansible-Tower-Operator in deployment resource.
👉🏻 Exposing the deployment resource.
👉🏻 Creation of project.
👉🏻 Configuring Dynamic inventory in ansible tower.
👉🏻 Creating a job template.
👉🏻 Configuring Apache webserver using ansible tower by dynamic inventory.

## Why to install ansible tower in kubernetes cluster ?
First we will understand why we need to install ansible-tower inside kubernetes. There is no specific reason you can install easily on your vm’s/instance on cloud. But the one of the main reason to do so is, we can configure the respective pods. In one pod there will be ansible tower running which will act as a master node whereas rest will be the target nodes.

## Installing Minikube on Ubuntu — 20:04
Minikube is a lightweight Kubernetes implementation that creates a VM on your local machine and deploys a simple cluster containing only one node. … The Minikube CLI provides basic bootstrapping operations for working with your cluster, including start, stop, status, and delete.

<p align="center">
  <img width="1000" height="200" src="https://miro.medium.com/max/1167/1*kZmIukE31eSyUIpyD54oQQ.png">
</p>

```
# minikube installation link
curl -Lo minikube https://github.com/kubernetes/minikube/releases/download/v1.21.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/ 
#modfying docker user
usermod -aG docker $USER
#Checking groups
groups $USER
```

## Installing Kubectl:
The Kubernetes command-line tool, kubectl, allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs. Now we can install kubectl command using below command. Kubectl will helps us to control kubernetes cluster.

<p align="center">
  <img width="1000" height="250" src="https://miro.medium.com/max/1167/1*pNGIBs6axK0vnZnIl3HgNw.png">
</p>


```
# Kubectl installation link
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod +x ./kubectl

# moving ./kubectl to /usr/local/bin/kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
# Checking client version
kubectl version --client
```


## Installing docker 🐋🐋:
As kubernetes will use docker as CRI(Container Runtime interface), so we have to install docker.

```
# Installing docker
sudo apt-get update && \
>     sudo apt-get install docker.io -y
```

<p align="center">
  <img width="1000" height="150" src="https://miro.medium.com/max/1167/1*vdbk5wdGasD6vth6Ee72GQ.png">
</p>

<p align="center">
  <img width="1000" height="300" src="https://miro.medium.com/max/1167/1*rOjEE_T4ShKRyMq4SV1guw.png">
</p>

## Installing conntrack:
Conntrack is command line interface conntrack provides a more flexible interface to the connnection tracking system than /proc/net/ip_conntrack. With conntrack, you can show, delete and update the existing state entries; and you can also listen to flow events. conntrackd is the user-space connection tracking daemon.

```
# Installing conntrack
sudo apt install conntrack -y
```

<p align="center">
  <img width="1000" height="300" src="https://miro.medium.com/max/1167/1*buubEiuLXTZ3HjQNpS0Nbg.png">
</p>

## Starting Minikube:
Now we have to install minikube and enabling ingress and then using stable version of kubernetes.

<p align="center">
  <img width="1000" height="500" src="https://miro.medium.com/max/1167/1*tiFHUPN2fKDaMGj6_2YJ6w.png">
</p>

<p align="center">
  <img width="1000" height="300" src="https://miro.medium.com/max/1167/1*JGLU53VtX6MuQwBBI4_JoA.png">
</p>

```
# Staring minikube
minikube start --addons=ingress --cpus=2 --install-addons=true --kubernetes-version=stable --memory=6g
```

Now we have to check that all the pods are in the running state or not. you can use kubetl get pods -A to check all the pods running in kubernetes cluster.

<p align="center">
  <img width="1000" height="200" src="https://miro.medium.com/max/1167/1*cCl_JPVzoRcxYdhcqzedQA.png">
</p>

## Installing the AWX-Operator:
This Kubernetes Operator is meant to be deployed in your Kubernetes cluster(s) and can manage one or more AWX instances in any namespace.

For testing purposes, the awx-operator can be deployed on a Minikube cluster. Due to different OS and hardware environments, please refer to the official Minikube documentation for further information.

<p align="center">
  <img width="1000" height="200" src="https://miro.medium.com/max/1167/1*hDW-lUmV7MwxTr1667doxQ.png">
</p>

```
# Installing awx-operator
kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/0.10.0/deploy/awx-operator.yaml
kubectl get pods
```

Next, create a file named awx-demo.yml with the suggested content below. The metadata.name you provide, will be the name of the resulting AWX deployment.

<p align="center">
  <img width="1000" height="150" src="https://miro.medium.com/max/1167/1*WgIgWAk-iinNhttpCGyHmQ.png">
</p>

```
vi awx-demo.yml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:  
   name: awx-demo
spec:  
   service_type: nodeport
   nodeport: none
   ingress_type: none  
   hostname: awx-demo.example.com
```
```
# creating postgress database
kubectl apply -f awx-demo.yml
# checking awx-operator pod
kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator"
# checking service
kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator"
```

After a few minutes, the new AWX instance will be deployed. You can look at the operator pod logs in order to know where the installation process is at:
<p align="center">
  <img width="1000" height="250" src="https://miro.medium.com/max/1167/1*Zc9av64O5Via8sqxJoD9AA.png">
</p>

By default, the admin user is admin and the password is available in the <resourcename>-admin-password secret. To retrieve the admin password, run:

```
  kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode
```
  
After a few minutes, the new AWX instance will be deployed. You can look at the operator pod logs in order to know where the installation process is at:
<p align="center">
  <img width="1000" height="350" src="https://miro.medium.com/max/1167/1*KnDbNRuhrX63XybCJMr2Ow.png">
</p>
  
## Exposing the deployment resource:


<p align="center">
  <img width="1000" height="200" src="https://miro.medium.com/max/1167/1*V5SYnhCE9j0dC_u1aGZ1wA.png">
</p>
  
Now copy the port number and public ip address of instance and paste in the browser and then you will see the login page of awx ansible tower as mentioned below in the image.
  
<p align="center">
  <img width="1000" height="500" src="https://miro.medium.com/max/1167/1*FGvBBp3Td556zkFJMYiv-Q.png">
</p>
  
After login you will see the awx ansible tower bashboard.
 
<p align="center">
  <img width="1000" height="500" src="https://miro.medium.com/max/1167/1*EaQstDd-ROgv5JfGP7Bisg.png">
</p>
  
Now we have to configure the credentials of aws as well as ssh-keys. AWS’s access key and secret key so that ansible will authenticate to aws resources and will pick up hosts in the inventory dynamically whereas the ssh-keys for login into the ssh.

## Adding Access Key and Secret Keys:
  
Navigate to credentials section in the left panel and click on add button to add the credentials. Enter the name of the credential eg. aws-access-secrett-keys and select type of the credential. For access keys and secret keys select AMAZON WEB SERVICES, then add access_key and secret_key to respective field.

<p align="center">
  <img width="1000" height="500" src="https://miro.medium.com/max/1167/1*yLMTiRvT-XFI45MZ0UMHWg.gif">
</p>
  
## Adding SSH-Keys:
Now we have to add ssh-keys so we can authencticate ssh connection between the master node(ansible tower) and target node(ec2-ubuntu-instance). Navigate to credential and the click on add button, then eneter the name of the key eg. ec2-key. Select MACHINE as a type of credentials. 
  
<p align="center">
  <img width="1000" height="500" src="https://miro.medium.com/max/1167/1*-IqzqBENmXDCokOe4cVOCg.gif">
</p>
 
## Creating a Dynamic Inventory:
  
Now you have to add a source in the inventory section. Our source will be aws and for that navigate to inventory, click on demo inventory and then click on source then add a source select AMAZON EC2 and the add credential which you have created already, in my case i have added aws-access-secrett-keys. After that click on sync, so that ansible will go to aws and then collect all the information about the hosts.
  
<p align="center">
  <img width="1000" height="500" src="https://miro.medium.com/max/1167/1*V87k9sJx_icf1sNHGz4rXA.gif">
</p>
  
## Creating a Project:
A Project is a logical collection of Ansible playbooks, represented in Tower. … To create a Red Hat Insights project, refer to Setting up an Insights Project. Note. By default, the Project Base Path is /var/lib/awx/projects , but this may have been modified by the Tower administrator.

While creating of the project it will ask you for the following information.
1. Name Of the project
2. Organization → default (or you can create your own organization).
3. Source Control Credential Type → Git
4. Source Control URL → Enter your playbooks repository URL from github.
5. Click on Save
6. Click on sync so that all the playbooks will be downloaded by the ansible.
  
<p align="center">
  <img width="1000" height="500" src="https://miro.medium.com/max/1167/1*cqU4HR6IJwO4P4Cl2Btzjw.gif">
</p>
  
## Creating a Job Template:
A job template is a definition and set of parameters for running an Ansible job. Job templates are useful to execute the same job many times. Job templates also encourage the reuse of Ansible playbook content and collaboration between teams. While the REST API allows for the execution of jobs directly, Tower requires that you first create a job template.
While creating of a job template it will ask you for the following information.

1. Name of the job template.
2. Run type → check/run (check is as syntax check ins ansible and run will run the playbook).
3. Inventory → Demo Inventory (or you can create your own inventory).
4. Project → Select your project.
5. Playbook → As soon as you created the project in the template section it will ask you the number of playbooks you having in respective repository which you have given while creating the project.
6. Enable Privilege Escalation → Mark the box of privilege escalation.
7. Click on save.

## Launching a job template 🚀:
After creating a job template, you need to launch the job template i.e you need to run the playbook. After launching the job template a new window of playbook output will appear and then it will show all the output of the ansible playbook. You can navigate to jobs and check whether the job is the running/pending/successful/failed state.


<p align="center">
  <img width="1000" height="500" src="https://github.com/amit17133129/Configuring-Apache-Webserver-in-aws-ec2-using-Ansible-Tower-Running-On-Kubernetes/blob/main/final.gif?raw=true">
</p>

This is the basic playbook of configuring of apache webserver by installing apache(httpd) and the copying the content in /var/www/html location and then restarting apache server.
  
```
  # apache configuration playbook.
- hosts: "ec2-13-234-238-129.ap-south-1.compute.amazonaws.com"
  tasks:  
   - name: "installing httpd"      
     yum:       
       name: "httpd" 
       state: present           
   - name: "copying content to /var/www/html/"     
     copy:        
        content: "Hello Ansible Tower"       
        dest: "/var/www/html/ansible-tower.html"           
   - name: "starting nginx server"      
     service:         
        name: "httpd"        
        state: restarted
```
  
Now you can do any configuration using ansible tower. I hope you had liked the article.😊😀
  
<p align="center">
  <img width="200" height="200" src="https://miro.medium.com/max/458/0*g5fUK61_ybu8E_l_">
</p>
  
