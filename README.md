#  Jenkins Pipeline for Java based application using Maven, SonarQube, Argo CD, Helm and Kubernetes

![cicd overview](https://github.com/rajeswarithota1715/CICD/blob/0392a9ee300b1f1e5d6d9283ba64bf840a1fe90d/Jdenkins%20cicd.png)
 
###  Steps to follow

1.  Create an EC2 instance type of t2.large as we have lot's of tooles to install
2.  login to EC2 instance
3.  install Java and Jenkins

<pre>
  sudo apt update
  sudo apt install openjdk-11-jre
  
  java -version
  
  curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
  echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
  
  sudo apt-get update
  sudo apt-get install jenkins

  ps -ef|grep -i jenkins
</pre>

 By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as show below.
 
4. In inbound rules of our EC2 instance open port 8080
5. login to Jenkins using ec2-publicip:8080
6. click on new item and select pipeline option to write the jenkins code
7. select GIT as SCM and provide the url and cdredentials(it's git repo is private) and branch name 
8. provide the Jenkins file path and apply it
9. install docker pipeline plugin in jenkins
11.  install sonarqube

<pre>
apt install unzip
  
adduser sonarqube
su - sonarqube
  
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
  
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
  
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
  
</pre>

12.  By default sonarserver will run on port 9000
13.  open port 9000 on aws ec2 instance inbound rules
14.  default username and password will be admin and admin restpectively 
15.  configure sonar with genkins
     * sonarqube ->myaccound ->security -> generate token -> copy the token
     * go to Jenkins dashboard ->manage jenkins -> manage credentials ->system >global credentails and add it
   
16.  install docker

<pre>
  sudo apt update
  sudo apt-get install docker.io
  
  sudo su -
  
  usermod -aG docker jenkins
  usermod -aG docker ubuntu
  
  systemctl restart docker
</pre>

17.  restart the jenkins (ip:8080/restart)
18.  create kubernetes cluster


#### Pre-requisites: 
  - an EC2 Instance 
  - Install AWSCLI latest verison 

1. Setup kubectl   
   a. Download kubectl version 1.21  
   b. Grant execution permissions to kubectl executable   
   c. Move kubectl onto /usr/local/bin   
   d. Test that your kubectl installation was successful    

   ```sh 
   curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl
   chmod +x ./kubectl
   mv ./kubectl /usr/local/bin 
   kubectl version --short --client
   ```
2. Setup eksctl   
   a. Download and extract the latest release   
   b. Move the extracted binary to /usr/local/bin   
   c. Test that your eksclt installation was successful   

   ```sh
   curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
   sudo mv /tmp/eksctl /usr/local/bin
   eksctl version
   ```
  
3. Create an IAM Role and attache it to EC2 instance    
   
   IAM user should have access to   
   IAM   
   EC2   
   CloudFormation  
   
4. Create your cluster and nodes 
   ```sh
   eksctl create cluster --name cluster-name  \
   --region region-name \
   --node-type instance-type \
   --nodes-min 2 \
   --nodes-max 2 \ 
   --zones <AZ-1>,<AZ-2>
   
   example:
   eksctl create cluster --name valaxy-cluster \
      --region ap-south-1 \
   --node-type t2.small \
    ```

5. To delete the EKS clsuter 
   ```sh 
   eksctl delete cluster valaxy --region ap-south-1
   ```
   
6. Validate your cluster using by creating by checking nodes and by creating a pod 
   ```sh 
   kubectl get nodes
   kubectl run tomcat --image=tomcat 
   ```

19.  install argocd using operator hub
      * Install Operator Lifecycle Manager (OLM), a tool to help manage the Operators running on your cluster.
      * <pre>curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.27.0/install.sh | bash -s v0.27.0</pre>
      *  Install the operator by running the following command:
      *  <pre>kubectl create -f https://operatorhub.io/install/argocd-operator.yaml</pre>
      *  After install, watch your operator come up using next command.
      *  <pre>kubectl get csv -n operators</pre>

20. create argocd controller
    <pre>
      apiVersion: argoproj.io/v1alpha1
      kind: ArgoCD
      metadata:
        name: example-argocd
        labels:
          example: basic
      spec: {}
    </pre>
    <pre>kubectl get pods</pre>
21.  in order to use argocd externally edit the argocd server service type to NodePort

<pre>kubectl edit svc example-argocd-server</pre>

22. Add Git hub credentials
    * go to Jenkins dashboard ->manage jenkins -> manage credentials ->system >global credentails -> type as secret text and add github to
    * github -> settings -> developer settings -> generate token
   
23.  Add docker hub credentials
    * go to Jenkins dashboard ->manage jenkins -> manage credentials ->system >global credentails -> type as username and password and add provide credentials

24.  login to argocd
25.  create new application by providing required details
26.  sync the application
     
  
