# Pangeo Management 
http://pangeo.esipfed.org on AWS

### Set up Helm:
```

mv kubeconfig ~/.kube/config   # this is secret stuff
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
wget -O kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
helm init
helm repo add pangeo https://pangeo-data.github.io/helm-chart/
helm repo update
helm init --upgrade
```

### Modify pangeo GCE config for AWS
```
git clone https://github.com/rsignell-usgs/pangeo.git
cd pangeo
git checkout -b rsignell-aws
cp -r cge aws     
cd aws
cd worker
vi Dockerfile    # add s3fs
docker build -t pangeo-worker .
docker tag pangeo-worker esip/pangeo-worker:2018-05-09
docker push esip/pangeo-worker:2018-05-09
cd ../../aws/notebook
docker build -t pangeo-notebook .
docker tag pangeo-notebook esip/pangeo-notebook:2018-05-09
docker push esip/pangeo-notebook:2018-05-09
cd ../../aws
vi jupyter-config.yaml    # adjust whitelist, specify containers from esip
```

### Upgrade JuptyerHub
```
helm upgrade esip-dev-pangeo pangeo/pangeo -f jupyter-config.yaml -f secret-config.yaml --version=v0.1.1-08b10bd
```

### Scale cluster via AWS console:
Select "action=>edit" on this console and adjust "min" and "desired":
https://us-west-2.console.aws.amazon.com/ec2/autoscaling/home?region=us-west-2#AutoScalingGroups:id=HdfLab-Kubernetes-K8sStack-SFLX4I06SHM9-K8sNodeGroup-VJ88FZ05UMEF;view=details

### Make sure worker-config.yaml doesn't exceed the CPU and memory of the kubernetes nodes
