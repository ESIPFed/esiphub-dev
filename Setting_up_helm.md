```
git clone https://github.com/rsignell-usgs/pangeo.git
 cd pangeo
  git checkout -b rsignell-aws
   mv kubeconfig ~/.kube/config
    curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
     wget -O kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
     chmod +x ./kubectl
  sudo mv ./kubectl /usr/local/bin/kubectl
    helm init
   helm repo add pangeo https://pangeo-data.github.io/helm-chart/
      helm repo update
       helm init --upgrade
 cd aws     
vi jupyter-config.yaml
 helm upgrade esip-dev-pangeo pangeo/pangeo -f jupyter-config.yaml -f secret-config.yaml --version=v0.1.1-08b10bd
```
Manually scale cluster here:
https://us-west-2.console.aws.amazon.com/ec2/autoscaling/home?region=us-west-2#AutoScalingGroups:id=HdfLab-Kubernetes-K8sStack-SFLX4I06SHM9-K8sNodeGroup-VJ88FZ05UMEF;view=details
Select action -> edit

Change min and desired to what you want.
Change max if you need more than 20
