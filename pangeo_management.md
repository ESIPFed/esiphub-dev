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



### Scale cluster via AWS console:

Select `action=>edit` on this console and adjust `min` and `desired`:
https://us-west-2.console.aws.amazon.com/ec2/autoscaling/home?region=us-west-2#AutoScalingGroups:id=HdfLab-Kubernetes-K8sStack-SFLX4I06SHM9-K8sNodeGroup-VJ88FZ05UMEF;view=details


### Kill pods
```
 kubectl get pods -n esip-dev | grep "^dask-rsignell" | cut -d' ' -f1 | xargs kubectl delete pods -n esip-dev
```

### Restart existing JupyterHub cluster
To restart or upgrade the cluster, you need to have access to the `secret-config.yaml` file, which looks something like this:
```
(IOOS3) rsignell@gamone:~/github/pangeo/aws> more secret-config.yaml
jupyterhub:
  auth:
    github:
      clientId: "a782cc170bc304xxxxxxxxxxxxxxxxxxa21"
      clientSecret: "210de41ac44gabxxxxxxxxx4fas5"

  proxy:
    secretToken: 3441bc19cdc1xxxxxxxxxxxxxxxxxxxxxxxxx33e265be5d4aed6221
```
If you have this file, you can then do:
```
helm list 
``` 
to see what helm chart version `esip-dev-pangeo` is using:
```
$helm list

NAME            REVISION        UPDATED                         STATUS          CHART                   NAMESPACE
esip-dev-pangeo 46              Mon Jul  9 10:47:35 2018        DEPLOYED        pangeo-v0.1.1-85dc5c9   esip-dev
hdflab          98              Fri Jul  6 20:01:52 2018        DEPLOYED        jupyterhub-v0.6         hdflab
(IOOS3) rsignell@gamone:~/github/pangeo/aws>
```
then get this config:
```
wget https://raw.githubusercontent.com/rsignell-usgs/pangeo/rsignell-aws/aws/jupyter-config.yaml
```
and issue this command using the correct chart version:
```
helm upgrade --force --recreate-pods jupyter pangeo/pangeo --version=0.1.1-85dc5c9 -f secret-config.yaml  -f jupyter-config.yaml
```

### Upgrade JuptyerHub
check for recent Helm chart versions at https://pangeo-data.github.io/helm-chart/
```
helm upgrade esip-dev-pangeo pangeo/pangeo -f jupyter-config.yaml -f secret-config.yaml --version=v0.1.1-08b10bd
```
login to http://pangeo.esipfed.org, stop the server, restart, then update the `custom-worker-template.yaml` to point to the correct docker image for the Dask workers, making sure not too exceed the CPU and memory of the kubernetes nodes.
