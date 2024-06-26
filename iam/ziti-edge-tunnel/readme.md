#https://github.com/openziti/helm-charts/blob/main/charts/ziti-edge-tunnel/README.md
ziti-edge-tunnel

Version: 0.0.14 Type: application AppVersion: 0.22.26

Host OpenZiti services with a tunneler pod

Homepage: https://openziti.io

Source Code

https://github.com/openziti/ziti-tunnel-sdk-c
Requirements

Kubernetes: >= 1.20.0-0

Overview

You may use this chart to reach services node-wide via your Ziti network via DNS. 

For example, if you create a repository or container registry Ziti service, and your cluster has no internet access, you can reach those repositories or container registries via Ziti services. 


NOTE: For one node kubernetes approaches like k3s, this works out-of-the-box and you can extend your coredns configuration to forward to the Ziti DNS IP, as you can see here. 

For multinode kubernetes installations, where your cluster DNS could run on a different node, you need to install the node-local-dns feature, which secures that the Ziti DNS name will be resolved locally, on the very same tunneler, as Ziti Intercept IPs can change from node to node. See this helm chart for a possible implementation.

How this Chart Works

This chart deploys a pod running ziti-edge-tunnel, the OpenZiti Linux tunneler, in transparent proxy mode with DNS nameserver. The chart uses container image docker.io/openziti/ziti-edge-tunnel which runs ziti-edge-tunnel run.

Installation

helm repo add openziti https://docs.openziti.io/helm-charts/
After adding the charts repo to Helm then you may enroll the identity and install the chart. You must supply a Ziti identity JSON file when you install the chart.

ziti edge enroll --jwt /tmp/k8s-tunneler.jwt --out /tmp/k8s-tunneler.json
helm install ziti-edge-tunnel openziti/ziti-edge-tunnel --set-file zitiIdentity=/tmp/k8s-tunneler-03.json
Installation using a existing / pre-created secret

Alternatively when you want to use a existing / pre-created secret (i.e. you have sealed-secrets enabled in your setup), you could refer to an existing secret with the ziti identity to use.

This sample shows you how to create the secret:

kubectl create secret generic k8s-tunneler-identity --from-file=persisted-identity=k8s-tunneler.json
When you deploy the helm chart refer to the existing secret:

helm install ziti-edge-tunnel openziti/ziti-edge-tunnel --set secret.existingSecretName=k8s-tunneler-identity
When you don't want to use the default key name persisted-identity you can define your own name by adding --set secret.keyName=myKeyName.

Configure CoreDNS

If you want to resolve your Ziti domain inside the pods, you need to customize CoreDNS. See Official docs.

Customize CoreDNS configuration,

kubectl -n kube-system apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  ziti.server: |
    your.ziti.domain {
      forward . 100.64.0.2
    }
EOF
Reload CoreDNS config,

kubectl rollout restart -n kube-system deployment/coredns
Air gapped installations

For air gapped clusters, which mirrors their registries over this OpenZiti tunneler, the upgrade will present the chicken-and-egg problem, and the DaemonSet will stay at the ImagePullBackOff state for ever. To work this problem around, you can install the prepull-daemonset Helm chart, which can pull the new ziti-edge-tunnel image Version needed in beforehand. Once the image is present on every node, you can proceed to upgrade the tunneler without problems.

Values Reference

Key	Type	Default	Description
additionalVolumes	list	[]	additional volumes to mount to ziti-edge-tunnel container
affinity	object	{}	
dnsPolicy	string	"ClusterFirstWithHostNet"	
fullnameOverride	string	""	
hostNetwork	bool	true	
image.args	list	[]	
image.pullPolicy	string	"IfNotPresent"	
image.registry	string	"docker.io"	
image.repository	string	"openziti/ziti-edge-tunnel"	
image.tag	string	""	
imagePullSecrets	list	[]	
livenessProbe.exec.command[0]	string	"/bin/bash"	
livenessProbe.exec.command[1]	string	"-c"	
livenessProbe.exec.command[2]	string	`"if (ziti-edge-tunnel tunnel_status	sed -E 's/(^received\sresponse\s<
livenessProbe.failureThreshold	int	3	
livenessProbe.initialDelaySeconds	int	180	
livenessProbe.periodSeconds	int	60	
livenessProbe.successThreshold	int	1	
livenessProbe.timeoutSeconds	int	10	
log.timeFormat	string	"utc"	
log.tlsUVLevel	int	3	
log.zitiLevel	int	3	
nameOverride	string	""	
nodeSelector	object	{}	constrain worker nodes where the ziti-edge-tunnel pod can be scheduled
podAnnotations	object	{}	
podSecurityContext	object	{}	
ports	list	[]	
resources	object	{}	
secret	object	{}	
securityContext.privileged	bool	true	
serviceAccount.annotations	object	{}	
serviceAccount.create	bool	true	
serviceAccount.name	string	""	
spireAgent.enabled	bool	false	if you are running a container with the spire-agent binary installed then this will allow you to add the hostpath necessary for connecting to the spire socket
spireAgent.spireSocketMnt	string	"/run/spire/sockets"	file path of the spire socket mount
systemDBus.enabled	bool	true	enable D-Bus socket connection
systemDBus.systemDBusSocketMnt	string	"/var/run/dbus/system_bus_socket"	file path of the System D-Bus socket mount
tolerations	list	[]	
helm upgrade {release} {source dir}
Log Level Reference

OpenZiti tunneler and TLSUV log levels are represented by integers, as follows,

Log Level	Value
NONE	0
ERR	1
WARN	2
INFO (default)	3
DEBUG	4
VERBOSE	5
TRACE	6
