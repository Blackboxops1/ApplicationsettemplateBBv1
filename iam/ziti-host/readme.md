#https://github.com/openziti/helm-charts/blob/main/charts/ziti-host/README.md

ziti-host

Version: 0.4.11 Type: application AppVersion: 0.22.26

Host OpenZiti services with a tunneler pod

Requirements

Kubernetes: >= 1.20.0-0

Overview

You may use this chart to publish cluster services to your Ziti network. For example, if you create a Ziti service with a server address of tcp:kubernetes.default.svc:443 and write a Bind Service Policy assigning the service to the Ziti identity used with this chart, then your Ziti network's authorized clients will be able access this cluster's apiserver. You could do the same thing for any cluster service's domain name.

How this Chart Works

This chart deploys a pod running ziti-edge-tunnel, the OpenZiti Linux tunneler, in service hosting mode. The chart uses container image docker.io/openziti/ziti-host which runs ziti-edge-tunnel run-host. This puts the Linux tunneler in "hosting" mode which is useful for binding Ziti services without any need for elevated permissions and without any Ziti nameserver or intercepting proxy. You'll be able to publish any server that is known by an IP address or domain name that is reachable from the pod deployed by this chart.

Installation

helm repo add openziti https://docs.openziti.io/helm-charts/
After adding the charts repo to Helm then you may enroll the identity and install the chart. You must supply a Ziti identity JSON file when you install the chart.

ziti edge enroll --jwt /tmp/k8s-tunneler.jwt --out /tmp/k8s-tunneler.json
helm install ziti-release03 openziti/ziti-host --set-file zitiIdentity=/tmp/k8s-tunneler-03.json
Installation using a existing / pre-created secret

Alternatively when you want to use a existing / pre-created secret (i.e. you have sealed-secrets enabled in your setup), you could refer to an existing secret with the ziti identity to use.

This sample shows you how to create the secret:

kubectl create secret generic k8s-tunneler-identity --from-file=persisted-identity=k8s-tunneler.json
When you deploy the helm chart refer to the existing secret:

helm install ziti-host openziti/ziti-host --set secret.existingSecretName=k8s-tunneler-identity
When you don't want to use the default key name persisted-identity you can define your own name by adding --set secret.keyName=myKeyName.

Values Reference

Key	Type	Default	Description
additionalVolumes	list	[]	additional volumes to mount to ziti-host container
affinity	object	{}	
dnsPolicy	string	"ClusterFirstWithHostNet"	
fullnameOverride	string	""	
hostNetwork	bool	false	
image.args	list	[]	
image.pullPolicy	string	"Always"	
image.repository	string	"openziti/ziti-host"	
imagePullSecrets	list	[]	
nameOverride	string	""	
nodeSelector	object	{}	
podAnnotations	object	{}	
podSecurityContext	object	{}	
ports	list	[]	
replicas	int	1	
resources	object	{}	
secret	object	{}	
securityContext	object	{}	
serviceAccount.annotations	object	{}	
serviceAccount.create	bool	true	
serviceAccount.name	string	""	
spireAgent.enabled	bool	false	if you are running a container with the spire-agent binary installed then this will allow you to add the hostpath necessary for connecting to the spire socket
spireAgent.spireSocketMnt	string	"/run/spire/sockets"	file path of the spire socket mount
tolerations	list	[]
