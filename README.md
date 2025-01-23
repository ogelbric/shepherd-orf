```
=====================
#
# Original instructions (Shepheard: https://vmw-confluence.broadcom.net/pages/viewpage.action?pageId=1862031629
# My Instructions: https://github.com/ogelbric/shepherd-orf/blob/main/README.md
# Thomas Instructions: https://docs.google.com/document/d/13pdIArdZy5BWsnMIBTCi2vbAUYZ6YlfdCuFWanM3rFk/edit?tab=t.0
# Official doc: https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-platform/10-0/tnz-platform/tp-sm-install-install-tp-sm.html
#
# For all of this to work you need to be on VPN and on a FullVPN connection (my case LasVegas Full) 
# Look for drop down in VPN change gateway section
#
	
1) SSH key (do on Mac)
	ssh-keygen -t ed25519 -C "shepardkey"
	Result:
		Your identification has been saved in /Users/ogelbrich/.ssh/id_ed25519
		Your public key has been saved in /Users/ogelbrich/.ssh/id_ed25519.pub

2) Go to internal Broadcom GitHub
	https://github.gwd.broadcom.net
	Upper right hand corner select down arrow and then settings
	Select SSH and GPG keys
	New SSH key
	Select title: shepardkey
	cat /Users/ogelbrich/.ssh/id_ed25519.pub
	Paste cat result into window (something like this: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ9yTKqUUNA7OPKbX+jaesFRv0uXXXXXXXXXqdTHnAuI)

3) Add the key to ssh
	ssh-add
	ssh-add -l #will show you the key added 

4) Point to the internal GitHub
	brew tap vmware/internal git@github.gwd.broadcom.net:TNZ/shepherd-homebrew-internal.git

5) Install stuff
	brew install sheepctl
	brew install shepherd

6) Set the target
	sheepctl target set -u https://epc-shepherd.lvn.broadcom.net -n tpsm

7) Run Sample Command
	sheepctl pool list

+-----------------------------------+---------+---------+-----------+----------+--------+--------+
|               NAME                | MINIMUM | MAXIMUM | AVAILABLE | CREATING | LOCKED | PAUSED |
+-----------------------------------+---------+---------+-----------+----------+--------+--------+
| TKGS-3.0.0-VC-8u3-AG-Pipeline     |       5 |      30 |         2 |        2 |     15 |        |
| TKGS-LCM-3.0.0-VC-8u3-Non-Airgap  |       1 |       3 |         0 |        1 |      0 |        |
| TPSM-GA-ENT-IP-TKGs-Non-Airgapped |       1 |       5 |         0 |        1 |      2 |        |
| TKGi-1.19                         |       0 |       3 |         0 |        0 |      0 |        |
| TKGi-1.20                         |       0 |       3 |         0 |        0 |      3 |        |
| TKGS-3.0.0-VC-70p09-AG-Pipeline   |       0 |       3 |         0 |        0 |      0 |        |
| TKGS-3.0.0-VC-8u3-Non-Airgap      |       3 |      10 |         2 |        1 |      5 |        |
+-----------------------------------+---------+---------+-----------+----------+--------+--------+
|                             TOTAL |      10 |         |         4 |        5 |     25 |        |
+-----------------------------------+---------+---------+-----------+----------+--------+--------+

8) Look at looks / or creating a new ENV 

	sheepctl lock list -n Tanzu-Sales
	

+--------------------------------------+----------+----------------+------------+-------------+--------------------+-----------------+
|                  ID                  |  STATUS  |    CREATED     | EXPIRATION |  NAMESPACE  |        POOL        |   DESCRIPTION   |
+--------------------------------------+----------+----------------+------------+-------------+--------------------+-----------------+
| fdbcbafd-cfc6-4c0d-8ca7-300175c730e8 | creating | 8 minutes ago  |            | Tanzu-Sales | TKGs-Non-Airgapped | Tanzu-SouthEast |
| ffaa1e86-3a00-4c8a-81e0-862aee201331 | locked   | 18 minutes ago | in 5 days  | Tanzu-Sales | TKGs-Non-Airgapped | Used by EPC TSL |
| af6bb058-b2fa-4d70-bdd1-16f5f7a5f5ff | locked   | a day ago      | in 4 days  | Tanzu-Sales | TKGs-Non-Airgapped | Used by User001 |
+--------------------------------------+----------+----------------+------------+-------------+--------------------+-----------------+


	#
	# new ENV
	#
	sheepctl target set -u https://epc-shepherd.lvn.broadcom.net -n Tanzu-Sales
	sheepctl pool lock TKGs-Non-Airgapped -n Tanzu-Sales --lifetime 5d --description 'Used by EPC TSL version 2'
	sheepctl lock list -n Tanzu-Sales

+--------------------------------------+--------+-------------------+------------+-------------+--------------------+---------------------------+
|                  ID                  | STATUS |      CREATED      | EXPIRATION |  NAMESPACE  |        POOL        |        DESCRIPTION        |
+--------------------------------------+--------+-------------------+------------+-------------+--------------------+---------------------------+
| 9596f88d-0681-4e8d-a4b9-5c47b35ab4d6 | locked | a few seconds ago | in 5 days  | Tanzu-Sales | TKGs-Non-Airgapped | Used by EPC TSL version 2 |
| ffaa1e86-3a00-4c8a-81e0-862aee201331 | locked | 7 days ago        | in 5 days  | Tanzu-Sales | TKGs-Non-Airgapped | Used by EPC TSL           |
+--------------------------------------+--------+-------------------+------------+-------------+--------------------+---------------------------+

	sheepctl lock get -n Tanzu-Sales {lock guid} -j -o lockfile.json


9) Get the lock information (json file)
	sheepctl lock get -n Tanzu-Sales ffaa1e86-3a00-4c8a-81e0-862aee201331

10) Get vCenter info
	sheepctl lock get -n Tanzu-Sales ffaa1e86-3a00-4c8a-81e0-862aee201331  > /tmp/a 2>&1 ; grep -w5  administrator /tmp/a | tail -6
            "vimUsername": "administrator@vsphere.local",
            "vimPassword": "SBFJWvHc.z-8kd0r",
11) Jumper Info
	sheepctl lock get -n Tanzu-Sales ffaa1e86-3a00-4c8a-81e0-862aee201331  > /tmp/a 2>&1 ; grep -w5  kubo /tmp/a | grep hostname | head -1
        "hostname": "10.167.66.84",
	sheepctl lock get -n Tanzu-Sales ffaa1e86-3a00-4c8a-81e0-862aee201331  > /tmp/a 2>&1 ; grep -w5  kubo /tmp/a | grep password | tail -1
        "password": "Ponies!23"

12) ssh to jumper
	ssh kubo@10.167.66.84 #Ponies!23
	mkdir orf
	cd orf

13) Connect to supvisor cluster
	kubectl vsphere login --server=192.168.0.2 --vsphere-username administrator@vsphere.local --insecure-skip-tls-verify
	# SBFJWvHc.z-8kd0r

14) Get storage class
	kubectl get sc
	NAME                   PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
	tkgs-k8s-obj-policy    csi.vsphere.vmware.com   Delete          Immediate           true                   21h

15) Create new content lib (current content line has older images then the guest cluster needs)
	https://wp-content.vmware.com/v2/latest/lib.json

16) Re-write cluster yaml with new storage class name (yaml is in next step)
	cat cluster1.yaml | sed "s/pacific-gold-storage-policy/tkgs-k8s-obj-policy/g" > cluster11.yaml
	Fix version to max: v1.29.4---vmware.3-fips.1-tkg.1           v1.29.4+vmware.3-fips.1-tkg.1  
	Fix namespace 
	#
	Create a VM Class
	Login to the vCenter 
		administrator@vsphere.local
		Goto Workload Management > Services > VM Service > Manage > VM Classes > CREATE VM CLASS 
		Give a Name: tpsm
			compatibility choose default: don't change the default value
			CPU 12, Memory 36GB
			Under Namespaces > testns (namespace1000) > VM Service > MANAGE VM CLASSES > Click the checkbox against tpsm > click OK


17) Cluster yaml that worked: 

cat <<'EOF' > /home/kubo/orf/cluster111.yaml
apiVersion: cluster.x-k8s.io/v1beta1
kind: Cluster
metadata:
  name: cluster1
  namespace: namespace1000
spec:
  clusterNetwork:
    services:
      cidrBlocks: ["10.96.0.0/12"]
    pods:
      cidrBlocks: ["192.168.16.0/20"]
    serviceDomain: "cluster.local"
  topology:
    class: tanzukubernetescluster
    version: v1.29.4---vmware.3-fips.1-tkg.1
    controlPlane:
      replicas: 1
    workers:
      machineDeployments:
        - class: node-pool
          name: node-pool-1
          replicas: 5
    variables:
      - name:  ntp
        value: "ntp.broadcom.net"
      - name: vmClass
        value: tpsm #12CPU and 36GB RAM 
      - name: storageClass
        value: tkgs-k8s-obj-policy
      - name: defaultStorageClass
        value: tkgs-k8s-obj-policy
      - name: nodePoolVolumes
        value:
        - capacity:
            storage: "40Gi"
          mountPath: "/var/lib/containerd"
          name: containerd
          storageClass: tkgs-k8s-obj-policy
      - name: controlPlaneVolumes
        value:
        - capacity:
            storage: "4Gi"
          mountPath: "/var/lib/etcd"
          name: etcd
          storageClass: tkgs-k8s-obj-policy
EOF
	#
	# Create cluster
	#
	kubectl apply -f cluster111.yaml


18) Log onto guest cluster 
	#
	# cluster1
	#
	kubectl vsphere login --server 192.168.0.2 --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-namespace namespace1000 --tanzu-kubernetes-cluster-name cluster1 --insecure-skip-tls-verify
	#
	# orfcluster1
	#
	kubectl vsphere login --server 192.168.0.2 --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-namespace namespace1000 --tanzu-kubernetes-cluster-name orfcluster1 --insecure-skip-tls-verify
	#
	# SBFJWvHc.z-8kd0r

19) Test guest cluster
	alias k=kubectl
	kubeclt get nodes
NAME                                     STATUS   ROLES           AGE     VERSION
cluster1-6vj88-lksn5                     Ready    control-plane   19m     v1.29.4+vmware.3-fips.1
cluster1-node-pool-1-tz745-cz8n9-bndgg   Ready    <none>          5m1s    v1.29.4+vmware.3-fips.1
cluster1-node-pool-1-tz745-cz8n9-k45x5   Ready    <none>          3m54s   v1.29.4+vmware.3-fips.1
cluster1-node-pool-1-tz745-cz8n9-lvq8n   Ready    <none>          3m31s   v1.29.4+vmware.3-fips.1
cluster1-node-pool-1-tz745-cz8n9-qtgk9   Ready    <none>          3m18s   v1.29.4+vmware.3-fips.1
cluster1-node-pool-1-tz745-cz8n9-tnfqv   Ready    <none>          5m12s   v1.29.4+vmware.3-fips.1

20) TP-SM Install steps

Internal Doc: https://vmw-confluence.broadcom.net/pages/viewpage.action?pageId=2122072341
Official Doc: https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-platform/10-0/tnz-platform/tp-sm-install-install-tp-sm.html

21) Download 45 GB
	Favorite web browser log on with your Broadcom.net ID
	Accept the little check box
	https://support.broadcom.com/group/ecx/productdownloads?subfamily=Tanzu%20Platform%20Self%20Managed

22) Moved and Extract
	#
	# jump host needs another 150 GB disk to do all of this
	# vCenter add second disk to server 
	#
	#disk device info
	lsblk
	# looks like sdb      8:16   0  150G  0 disk  is my new disk 
	sudo parted /dev/sdb
	mklabel gpt
	Yes
	print
	mkpart primary 0 145GB
	Ignore
	quit
	sudo mkfs.ext4 /dev/sdb1
	sudo mkdir /bigdisk
	sudo mount /dev/sdb1 /bigdisk
	mkdir /bigdisk/tanzu-installer
	cd /home/kubo/orf
	#Move from Mac(Internal jump server) to jumper server: 
	#
	scp tanzu-self-managed-10.0.0.tar.gz kubo@10.167.66.84:/home/kubo/orf/.   #(2hours 47 min) good thing the jumper has a 85 GB drive
	 
	sudo tar -xzvf tanzu-self-managed-10.0.0.tar.gz  -C /bigdisk/tanzu-installer

23) Install some stuff on jump box... 
	cd orf
	#
	#Fetch worker cluster kubeconfig and export it
	#
	kubectl vsphere login --server=192.168.0.2 --vsphere-username administrator@vsphere.local --insecure-skip-tls-verify
	# SBFJWvHc.z-8kd0r

	kubectl config use-context namespace1000
	kubectl get secret -n namespace1000
	#
	# cluster1 kubeconfig
	#
	kubectl get secret cluster1-kubeconfig -n namespace1000 -o json | jq -r '.data["value"] | @base64d' > wrk-kc 
	export KUBECONFIG=wrk-kc
	#
	# orfcluster1 kubeconfig
	#
	kubectl get secret orfcluster1-kubeconfig -n namespace1000 -o json | jq -r '.data["value"] | @base64d' > orfwrk-kc 
	export KUBECONFIG=orfwrk-kc
	#
	#Carvel tools
	#
	mkdir -p build/
	curl -kL https://carvel.dev/install.sh | K14SIO_INSTALL_BIN_DIR=build bash
	#
	#Tanzu-cli
	#
	sudo apt install -y ca-certificates curl gpg
	sudo mkdir -p /etc/apt/keyrings
	curl -fsSL https://storage.googleapis.com/tanzu-cli-installer-packages/keys/TANZU-PACKAGING-GPG-RSA-KEY.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/tanzu-archive-keyring.gpg
	echo "deb [signed-by=/etc/apt/keyrings/tanzu-archive-keyring.gpg] https://storage.googleapis.com/tanzu-cli-installer-packages/apt tanzu-cli-jessie main" | sudo tee /etc/apt/sources.list.d/tanzu.list
	sudo apt update
	sudo apt install -y tanzu-cli
	#
	#Crashd
	#
	wget https://github.com/vmware-tanzu/crash-diagnostics/releases/download/v0.3.10/crashd_0.3.10_linux_amd64.tar.gz
	mkdir -p crashd_0.3.10_linux_amd64
	tar -xvf crashd_0.3.10_linux_amd64.tar.gz -C crashd_0.3.10_linux_amd64
	sudo mv crashd_0.3.10_linux_amd64/crashd  /usr/local/bin/crashd


24) Making sure the lock is not going away - trying to extend the lease

	sheepctl lock list -n Tanzu-Sales                                               
+--------------------------------------+--------+--------------+------------+-------------+--------------------+-----------------+
|                  ID                  | STATUS |   CREATED    | EXPIRATION |  NAMESPACE  |        POOL        |   DESCRIPTION   |
+--------------------------------------+--------+--------------+------------+-------------+--------------------+-----------------+
| ffaa1e86-3a00-4c8a-81e0-862aee201331 | locked | 19 hours ago | in 5 days  | Tanzu-Sales | TKGs-Non-Airgapped | Used by EPC TSL |
| af6bb058-b2fa-4d70-bdd1-16f5f7a5f5ff | locked | 2 days ago   | in 4 days  | Tanzu-Sales | TKGs-Non-Airgapped | Used by User001 |
+--------------------------------------+--------+--------------+------------+-------------+--------------------+-----------------+

	sheepctl lock extend ffaa1e86-3a00-4c8a-81e0-862aee201331 -t 3d -n Tanzu-Sales 


25) DNS update on the jump box

	sudo vi /etc/dnsmasq.d/vlan-dhcp-dns.conf
	address=/tanzu.platform.io/192.168.116.206
	sudo systemctl restart dnsmasq


	cat /home/kubo/orf/dnssetup.sh

cat <<'EOFEOF' > /home/kubo/orf/dnssetup.sh
#!/bin/bash
 
# Step 1: Install dnsmasq
echo "Installing dnsmasq..."
sudo apt update && sudo apt install -y dnsmasq
 
# Step 2: Update /etc/hosts with the correct hostname
echo "Ensuring /etc/hosts has the correct hostname..."
sudo sed -i "/127.0.1.1/d" /etc/hosts
echo "127.0.1.1 $(hostname)" | sudo tee -a /etc/hosts
 
# Step 3: Configure dnsmasq to listen on localhost and use upstream DNS
echo "Configuring dnsmasq..."
sudo bash -c "cat > /etc/dnsmasq.conf" <<EOF
listen-address=127.0.0.1
bind-interfaces
no-resolv
server=192.19.189.10
cache-size=1000
EOF
 
# Step 4: Stop and disable systemd-resolved
echo "Disabling systemd-resolved..."
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
sudo rm -f /etc/resolv.conf
 
# Step 5: Create /etc/resolv.conf pointing to dnsmasq
echo "Setting /etc/resolv.conf..."
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
 
# Step 6: Restart dnsmasq
echo "Restarting dnsmasq..."
sudo systemctl restart dnsmasq
sudo systemctl enable dnsmasq
 
# Step 7: Verify dnsmasq service status
echo "Checking dnsmasq status..."
sudo systemctl status dnsmasq | grep Active
 
# Step 8: Test DNS resolution
echo "Testing DNS resolution..."
dig google.com
 
# Step 9: Optional - Check logs for errors
echo "Checking dnsmasq logs for errors..."
sudo tail -n 10 /var/log/syslog | grep dnsmasq
 
echo "dnsmasq setup complete!"
EOFEOF

	chmod +x dnssetup.sh
	./dnssetup.sh

26) Cert Manager
	Log onto guest cluster step 18
	alias k=kubectl
	#
	kubectl config get-contexts
	#

	CURRENT   NAME                            CLUSTER       AUTHINFO            NAMESPACE
	*         orfcluster1-admin@orfcluster1   orfcluster1   orfcluster1-admin   


	kubectl create namespace cert-manager # in my case the namespace was there but noting in it
	tanzu plugin install package
	tanzu package install cert-manager -p cert-manager.tanzu.vmware.com -n cert-manager -v 1.7.2+vmware.3-tkg.3 
	#
	# if there is any error in this step, you might need to install package repo 
	# first https://techdocs.broadcom.com/us/en/vmware-tanzu/cli/tanzu-packages/latest/tnz-packages/prep.html#:~:text=Add%20the%20Package%20Repository%20to%20the%20Cluster
	#
	# yes I have error in above step...
	#
	tanzu package repository list -A
	#
	# repo list is empty
	#
	#
	# make sure you can nslookup projects.registry.vmware.com and it resolves
	# if not then go back to DNS setup step
	#
	cd build
	./imgpkg tag list -i projects.registry.vmware.com/tkg/packages/standard/repo
	#
	# Create a repo file (Add the Package Repository to the Cluster)
	#
	cat packagerepo.yaml # Vi or run below EOF...EOF 

cat <<'EOF' > /home/kubo/orf/packagerepo1.yaml
apiVersion: packaging.carvel.dev/v1alpha1
kind: PackageRepository
metadata:
  name: tanzu-standard
  namespace: tkg-system
spec:
  fetch:
    imgpkgBundle:
      image: projects.registry.vmware.com/tkg/packages/standard/repo:v2.2.0_update.2
EOF

	#
	# make sure you are still logged onto the guys cluster
	#
	k config get-contexts
	#
	# orfcluster1
	#
	kubectl vsphere login --server 192.168.0.2 --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-namespace namespace1000 --tanzu-kubernetes-cluster-name orfcluster1 --insecure-skip-tls-verify
	#
	# SBFJWvHc.z-8kd0r

	# Apply the repo info
	kubectl apply -f packagerepo.yaml

	#check if there is a repo and it is STATUS: Reconcile succeeded 
	tanzu package repository list -A
	kubectl get packagerepositories -A

	#
	# list the available packages
	#
	kubectl -n tkg-system get packages

	#
	# Try to install cert-manager
	#
	tanzu package install cert-manager -p cert-manager.tanzu.vmware.com -n cert-manager -v 1.7.2+vmware.3-tkg.3
	
	tanzu package installed list -n cert-manager

	kubectl -n cert-manager get all

	#
	# looks like cert manager is in reconcile status...
	# events reveal that there is an issue
	k get events  -n cert-manager

	11m         Warning   FailedCreate        replicaset/cert-manager-webhook-9ddb9cd4c       Error creating: pods "cert-manager-webhook-9ddb9cd4c-l4mkc" is forbidden: violates PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "cert-manager" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "cert-manager" must set securityContext.capabilities.drop=["ALL"]), seccompProfile (pod or container "cert-manager" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")

	#
	# patch time
	#
	kubectl get deployments -n cert-manager
	
	NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
	cert-manager              0/1     0            0           27m
	cert-manager-cainjector   0/1     0            0           27m
	cert-manager-webhook      0/1     0            0           27m


	# time to look at the yaml files

	kubectl get  deployment -n cert-manager cert-manager  -o yaml > cert-manager 
	kubectl get  deployment -n cert-manager cert-manager-cainjector  -o yaml > cert-manager-cainjector
	kubectl get  deployment -n cert-manager cert-manager-webhook  -o yaml > cert-manager-webhook
	
	#
	# Attention there is already a security context section 
	# The new instructions do not go under that existing security section!
	# The new instruction go under the port: or name: section
	#

	#
	# Option (A)
	# 
	# Edit in place
	#

	#
	# (A) edit online with below example
	#
	kubectl edit deployment -n cert-manager cert-manager
	kubectl edit deployment -n cert-manager cert-manager-cainjector
	kubectl edit deployment -n cert-manager cert-manager-webhook


   spec:
      containers:
      - args:
        - --v=2
        - --cluster-resource-namespace=$(POD_NAMESPACE)
        - --leader-election-namespace=kube-system
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
        image: projects.registry.vmware.com/tkg/cert-manager-controller@sha256:c5ff152d8391ca52ef6d6563001d41543860b687ad294fea38f28a7906920b31
        imagePullPolicy: IfNotPresent
        name: cert-manager
        ports:
        - containerPort: 9402
          protocol: TCP
        resources: {}
        #
        # new section here (*********************************START********************************************)
        #
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          runAsNonRoot: true
          seccompProfile:
            type: RuntimeDefault
        #
        # new section here (*********************************END********************************************))
        #
        terminationMessagePath: /dev/termination-log

	#
	# check the deployments
	#

	kubectl get deployment -n cert-manager

	NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
	cert-manager              1/1     0            1           27h
	cert-manager-cainjector   1/1     0            1           27h
	cert-manager-webhook      1/1     1            1           27h

	kubectl get pods -n cert-manager

	NAME                                       READY   STATUS    RESTARTS   AGE
	cert-manager-67dd8d5578-6qxzx              1/1     Running   0          56m
	cert-manager-cainjector-6cd4cd948d-qf7cs   1/1     Running   0          32m
	cert-manager-webhook-569f75495-5hwsq       1/1     Running   0          31m


	#Or you can use public cert manager from https://cert-manager.io/docs/installation/#default-static-install

	#
	# Option (B)
	#
	# A more programatic way of applying the patch
	#

	#
	# cert-manager
	#

read -r -d '' PATCH1 <<'EOF'
{
   "spec": {
      "template": {
         "spec": {
        "containers": [
          {
            "terminationMessagePath": "/dev/termination-log", 
            "name": "cert-manager", 
            "imagePullPolicy": "IfNotPresent", 
            "securityContext": {
              "runAsNonRoot": true, 
              "seccompProfile": {
                "type": "RuntimeDefault"
              }, 
              "capabilities": {
                "drop": [
                  "ALL"
                ]
              }, 
              "allowPrivilegeEscalation": false
            }, 
            "image": "projects.registry.vmware.com/tkg/cert-manager-controller@sha256:c5ff152d8391ca52ef6d6563001d41543860b687ad294fea38f28a7906920b31", 
            "terminationMessagePolicy": "File", 
            "resources": {}
          }
        ]
      }
    }
  }
}
EOF

	kubectl patch --type=merge deployment cert-manager -n cert-manager --patch "$PATCH1"

	#
	# cert-manager-cainjector
	#

read -r -d '' PATCH2 <<'EOF'
{
   "spec": {
      "template": {
         "spec": {
        "containers": [
          {
            "terminationMessagePath": "/dev/termination-log", 
            "name": "cert-manager", 
            "imagePullPolicy": "IfNotPresent", 
            "securityContext": {
              "runAsNonRoot": true, 
              "seccompProfile": {
                "type": "RuntimeDefault"
              }, 
              "capabilities": {
                "drop": [
                  "ALL"
                ]
              }, 
              "allowPrivilegeEscalation": false
            }, 
            "image": "projects.registry.vmware.com/tkg/cert-manager-cainjector@sha256:b5b8733f1e4af45f9ea1ca51f2d8fc112d4e60297e954ba7954e5098c3976555", 
            "terminationMessagePolicy": "File", 
            "resources": {}
          }
        ]
      }
    }
  }
}
EOF

	kubectl patch --type=merge deployment cert-manager -n cert-manager --patch "$PATCH2"


	#
	# cert-manager-webhook
	#

read -r -d '' PATCH3 <<'EOF'
{
   "spec": {
      "template": {
         "spec": {
        "containers": [
          {
            "terminationMessagePath": "/dev/termination-log", 
            "name": "cert-manager", 
            "imagePullPolicy": "IfNotPresent", 
            "securityContext": {
              "runAsNonRoot": true, 
              "seccompProfile": {
                "type": "RuntimeDefault"
              }, 
              "capabilities": {
                "drop": [
                  "ALL"
                ]
              }, 
              "allowPrivilegeEscalation": false
            }, 
            "image": "projects.registry.vmware.com/tkg/cert-manager-webhook@sha256:edf212ccc5b9809699ed3a424db98dbdc615e4b43ed8afc260836572a0ba7256", 
            "terminationMessagePolicy": "File", 
            "resources": {}
          }
        ]
      }
    }
  }
}
EOF

	kubectl patch --type=merge deployment cert-manager -n cert-manager --patch "$PATCH3"

	#
	# check the deployments
	#


	kubectl get deployment -n cert-manager

	NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
	cert-manager              1/1     1            1           3d1h
	cert-manager-cainjector   1/1     0            1           3d1h
	cert-manager-webhook      1/1     0            1           3d1h



	kubectl get pods -n cert-manager
	NAME                                       READY   STATUS    RESTARTS      AGE
	cert-manager-6cb7cc4778-8xflf              1/1     Running   0             37s
	cert-manager-cainjector-6cd4cd948d-qf7cs   1/1     Running   4 (24h ago)   46h
	cert-manager-webhook-569f75495-5hwsq       1/1     Running   0             46h

	#
	# Option (C)
	#
	# Use public cert manager from https://cert-manager.io/docs/installation/#default-static-install



27) Storage class update


	# Check Storage class
	#
	kubectl get sc

	NAME                              PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
	tkgs-k8s-obj-policy (default)     csi.vsphere.vmware.com   Delete          Immediate              true                   3d19h
	tkgs-k8s-obj-policy-latebinding   csi.vsphere.vmware.com   Delete          WaitForFirstConsumer   true                   3d19h

	#
	# make sure below yaml has the correct (tkgs-k8s-obj-policy) Storage class

cat <<'EOF' > /home/kubo/orf/storage1.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  labels:
    isSyncedFromSupervisor: "false"
  name: postgresql-storage-class
parameters:
  svStorageClass: tkgs-k8s-obj-policy
provisioner: csi.vsphere.vmware.com
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: Immediate
---
#Open Search
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  labels:
    isSyncedFromSupervisor: "false"
  name: opensearch-storage-class
parameters:
  svStorageClass: tkgs-k8s-obj-policy
provisioner: csi.vsphere.vmware.com
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: Immediate
---
#Kafka
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  labels:
    isSyncedFromSupervisor: "false"
  name: kafka-storage-class
parameters:
  svStorageClass: tkgs-k8s-obj-policy
provisioner: csi.vsphere.vmware.com
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: Immediate
---
#Zookeeper
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  labels:
    isSyncedFromSupervisor: "false"
  name: zk-storage-class
parameters:
  svStorageClass: tkgs-k8s-obj-policy
provisioner: csi.vsphere.vmware.com
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: Immediate
---
#Redis
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  labels:
    isSyncedFromSupervisor: "false"
  name: redis-storage-class
parameters:
  svStorageClass: tkgs-k8s-obj-policy
provisioner: csi.vsphere.vmware.com
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: Immediate
---
#Prometheus
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  labels:
    isSyncedFromSupervisor: "false"
  name: prometheus-storage-class
parameters:
  svStorageClass: tkgs-k8s-obj-policy
provisioner: csi.vsphere.vmware.com
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: Immediate
  
---
#Prometheus-tmc (to be deprecated in later release)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  labels:
    isSyncedFromSupervisor: "false"
  name: tmc-prometheus-storage-class
parameters:
  svStorageClass: tkgs-k8s-obj-policy
provisioner: csi.vsphere.vmware.com
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: Immediate
 
---
#SeaweedFS
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  labels:
    isSyncedFromSupervisor: "false"
  name: seaweedfs-storage-class
parameters:
  svStorageClass: tkgs-k8s-obj-policy
provisioner: csi.vsphere.vmware.com
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: Immediate
 
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  labels:
    isSyncedFromSupervisor: "false"
  name: clickhouse-storage-class
parameters:
  svStorageClass: tkgs-k8s-obj-policy
provisioner: csi.vsphere.vmware.com
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: Immediate
EOF

	kubectl apply -f /home/kubo/orf/storage1.yaml

	#
	# output
	#

	# storageclass.storage.k8s.io/postgresql-storage-class created
	# storageclass.storage.k8s.io/opensearch-storage-class created
	# storageclass.storage.k8s.io/kafka-storage-class created
	# storageclass.storage.k8s.io/zk-storage-class created
	# storageclass.storage.k8s.io/redis-storage-class created
	# storageclass.storage.k8s.io/prometheus-storage-class created
	# storageclass.storage.k8s.io/tmc-prometheus-storage-class created
	# storageclass.storage.k8s.io/seaweedfs-storage-class created
	# storageclass.storage.k8s.io/clickhouse-storage-class created

	kubectl get sc
	NAME                              PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
	clickhouse-storage-class          csi.vsphere.vmware.com   Retain          Immediate              true                   40s
	kafka-storage-class               csi.vsphere.vmware.com   Retain          Immediate              true                   40s
	opensearch-storage-class          csi.vsphere.vmware.com   Retain          Immediate              true                   40s
	postgresql-storage-class          csi.vsphere.vmware.com   Retain          Immediate              true                   40s
	prometheus-storage-class          csi.vsphere.vmware.com   Retain          Immediate              true                   40s
	redis-storage-class               csi.vsphere.vmware.com   Retain          Immediate              true                   40s
	seaweedfs-storage-class           csi.vsphere.vmware.com   Retain          Immediate              true                   40s
	tkgs-k8s-obj-policy (default)     csi.vsphere.vmware.com   Delete          Immediate              true                   3d19h
	tkgs-k8s-obj-policy-latebinding   csi.vsphere.vmware.com   Delete          WaitForFirstConsumer   true                   3d19h
	tmc-prometheus-storage-class      csi.vsphere.vmware.com   Retain          Immediate              true                   40s
	zk-storage-class                  csi.vsphere.vmware.com   Retain          Immediate              true                   40s


28) Install verification

	#
	# In this env the image is already on JFrog - you will need request access on 1.secure
	# other wise it needs to be pushed
	# docker login <path-to-local-repo>
	# push image to docker 
	# imgpkg copy --tar tanzusm-<VERSION>.tar --to-repo=<path-to-local-repo>

	export TANZU_SM_VERSION="10.0.0-oct-2024-rc.533-vc0bb325" # from Bridget
	export OS=$(uname | tr '[:upper:]' '[:lower:]')
	export ARCH=$(if [[ $(uname -m) = "x86_64" ]]; then  echo "amd64"; else echo "arm64"; fi)
	export ARTIFACTORY_USER="og010490" 
	export ARTIFACTORY_API_TOKEN="cmVmdGtuOjAxOjE3NjkxMDY0MzA6c2E2M2dtR2ZyQ2JnMXBxxxxxxxxxxxxxxxxxxx" 
	export DOCKER_REGISTRY="tis-tanzuhub-sm-docker-dev-local.usw1.packages.broadcom.com"


	#
	# I tested my connection with this curl command
	# it has a file error when it works... (curl to output it to your terminal anyway, or consider "--output )
	#
	curl -u ${ARTIFACTORY_USER}:${ARTIFACTORY_API_TOKEN} https://usw1.packages.broadcom.com/artifactory/tis-tanzuhub-sm-docker-dev-local/hub-self-managed/${TANZU_SM_VERSION}/releases/non-airgapped/tanzu-self-managed-${TANZU_SM_VERSION}-${OS}-${ARCH}.tar.gz




	cd /home/kubo/orf
	export KUBECONFIG=''
	alias k=kubectl

	kubectl vsphere login --server=192.168.0.2 --vsphere-username administrator@vsphere.local --insecure-skip-tls-verify
	# SBFJWvHc.z-8kd0r

	#
	# orfcluster1 kubeconfig
	#
	kubectl config use-context namespace1000
	kubectl get secret orfcluster1-kubeconfig -n namespace1000 -o json | jq -r '.data["value"] | @base64d' > orfwrk-kc 
	export KUBECONFIG=orfwrk-kc

	kubectl config get-contexts
	#
	# CURRENT   NAME                            CLUSTER       AUTHINFO            NAMESPACE
	# *         orfcluster1-admin@orfcluster1   orfcluster1   orfcluster1-admin   


	cd /bigdisk/tanzu-installer
	#
	# save off the original config file
	#
	cp config.yaml /home/kubo/orf/config.yaml.orig
	

	#
	# File below is "2 Space" yaml sensitive !!!!!!!! 
	# I had "oauthProviders:" 2 space to the left and it cause the verify step to fail
	# From other people I heard the cert has to be spaced "right" as well !!!!
	#
	#
	# FYI this file if not saved will be over written by the installer!!!!!
	#

cat <<'EOF' > /home/kubo/orf/config.yaml
# @copyright Copyright Broadcom. All Rights Reserved.
# The term "Broadcom" refers to Broadcom Inc. and/or its subsidiaries.
# @license For licensing, see LICENSE.md.
# Installation flavor. Use "full" to install everything in Tanzu Platform which includes essentials services along with K8s Ops, App Engine and build services.
# Use "essentials" for installing essentials services of Tanzu Platform.
flavor: full
# Installation profile to use.  `evaluation` profile uses minimum
# resources, disables autoscaling, and only uses a single replica.
# `foundation` profile uses standard resource and autoscaling
# configuration.
profile: foundation
# Version of Tanzu Platform Self Managed to install.  This should not normally
# be changed.
#version: '110.1.0-5123-vc1c76f4'
version: '10.0.0-oct-2024-rc.533-vc0bb325'
# Kubernetes Ingress settings.
ingress:
  # Static IP address that needs to be assigned to contour load balancer
  # The host associated with this IP will be used for accessing UI and other ingress objects.
  # This is optional and If not provided, random IP address from available IP pool will be assigned
  # to contour load balancer.
  #loadBalancerIP: "192.168.116.206"
  #loadBalancerIP: ""
  # DNS name that can reach the system.  If unset, and
  # `controller` is enabled, then get the external name from
  # the installed ingress controller.
  host: "tanzu.platform.io"
  # TLS certificate configuration for the specified `host`
  # If unset, the cluster will generate a self-signed certificate
  # on its own.
  tls:
    certificate: ""
    privateKey: ""
#Deplyment settings
deployment:
  airGapped: false
#Daedalus Settings
trivy:
  # Optional OCI repository to retrieve trivy-db from. Its url can be external or internal for airgapped deployment (default "ghcr.io/aquasecurity/trivy-db")
  # This repositroy should be periodically synced from ghcr.io/aquasecurity/trivy-db to get latest vulnerability for airgapped deployment.
  dbRepository: ""
  # When using SSL/TLS, Trivy can be configured to allow insecure connections to a container registry if there is no valid certificate available
  # Set allowInsecureConnections to true to allow insecure server connections; false, otherwise
  allowInsecureConnections: false
# Infra components
postgresql:
  storageClass: "postgresql-storage-class"
clickhouse:
  storageClass: "clickhouse-storage-class"
redis:
  storageClass: "redis-storage-class"
opensearch:
  storageClass: "opensearch-storage-class"
seaweedfsS3:
  storageClass: "seaweedfs-storage-class"
prometheus:
  storageClass: "prometheus-storage-class"
  tmcStorageClass: "tmc-prometheus-storage-class"
kafka:
  storageClass: "kafka-storage-class"
zookeeper:
  storageClass: "zk-storage-class"
# Image registry settings used as service parameters. values will be automatically generated.
imageRegistry:
  server: ""
  username: ""
  password: ""
#Custom Registry certificate, used in Kapp controller. Certificate string should be provided using string literal style (|) to perserve newline characters.
CustomRegistryCertificate: ""
#Version control systems (gitlab or github) certificates. Certificate string should be provided using string literal style (|) to perserve newline characters.
versionControlSystemCertificate: ""
#Tanzu platform login details
login:
  # login operation timeout in seconds, default 60 seconds
  timeout: 60
  # Default in built user details for login into the Tanzu Platform
  defaultUsers:
    #Tanzu Platform Admin Details, username: tanzu_platform_admin
    admin:
      #Tanzu Platform Admin  Password, Random password will be generated if empty value provided
      #Note that this password cannot be changed later, its one time set ( both system generated and user provided one)
      password: "admin123"
      ##### --------------------------------*************************-----------------------------------------######
      ##### --------------------------------*************************-----------------------------------------######
      ##### ------ You can provide both oauth provider and ldap for the users to login into TP or any one ----######
      ##### ------ of them but at-least one need to be provided. Please remove/comment out the oauth ---------######
      ##### ------ providers section if you are not using oauth and similarly remove/comment out the ---------######
      ##### ------ out the ldap section if you are not using ldap. -------------------------------------------######
      ##### --------------------------------*************************-----------------------------------------######
      ##### --------------------------------*************************-----------------------------------------######
      #OIDC/Oauth servers details, the config supports multiple oauth providers but the recommendation is 1
  oauthProviders:
#    #Unique name for this oauth server configuration
    - name: "okta.test"
#      #Certificate of the oauth server if needs to connect over ssl
#      #Remove this field if it needs to connect over plain http protocol
#      certificate: ""
#      #OpenId configuration url of the oauth provider i.e url of .well-known/openid-configuration api endpoint
      configUrl: "https://dev-70846880.okta.com/.well-known/openid-configuration"
#      #Issuer url of the oauth Provider
      issuerUrl: "https://dev-70846880.okta.com"
#      #Scopes list which are needed to fetch the user information(email, groups) from oauth provider, openId is must
      scopes: ["openid", "email", "groups"]
#      #Link name to be shown for this oauth provider in TP login page
      loginPageLinkText: "Login with Dev Okta"
#      #Client Id to connect to oauth Provider
      clientId: "0oaggqbiqdlnTtfFY5d7"
#      #Client secret to connect to oauth Provider
      secret: "UMdEVboJTSfHAQEbuIlj1j2zticsxBRiEuRLYsfJk6dbeR9Nh47qH_7E_7q7MVT1"
#      #Mapping the fields of the oauth token to the user info object which are needed in TP
      attributeMappings:
#        #Field to be considered for username of the logged-in user
        username: "email"
#        #Field to be considered for groups of the logged-in user
        groups: "groups"
#  #Ldap server details for login into the Tanzu Platform;
#  #please uncomment the ldap section if you are planning to use ldap for the login and
#  #comment the oauth provider section if you are not using oauth providers for the login
#  ldap:
#    #Ldap server url
#    url: ""
#    #Certificate of the ldap server if it needs to connect over ssl(ldaps), optional field.
#    #Remove this field if it needs to connect over plain ldap protocol
#    certificate: ""
#    #Ldap server credential details
#    credentials:
#      #DN record for the ldap credential to search the directory, example: cn=admin,dc=broadcom,dc=com
#      userDN: ""
#      #password of the above DN
#      password: ""
#    #LDAP configuration to fetch user for the TP login operation
#    users:
#      #Base DN where the users need to be searched, example: dc=broadcom,dc=com
#      baseDN: ""
#      #Search filter to be used to get the user record who is trying to login into TP, example cn={0}.
#      #Here {0} is used to annotate where the username will be inserted
#      #This value will be clubbed with base DN to fetch the user record, the query will become cn=testuser,dc=broadcom,dc=com"
#      searchFilter: ""
#      #Attribute name that contains the user's email address
#      mailAttribute: "mail"
#    #LDAP configuration to fetch groups of the user
#    groups:
#      #Base DN where the groups need to be searched, example: dc=broadcom,dc=com
#      baseDN: ""
#      #Similar to user filter, the user group memberships are retrieved based on this filter, example: member={0}
#      searchFilter: ""
#      #Number of depth levels to search for nested groups, set this value to 1 to disable nested groups search
#      searchDepth: 10
#      #Attribute which holds the name of the group in the LDAP record
#      groupNameAttribute: cn
# Please provide the default organization details in the configuration settings below.
organization:
  #  Specifies the name of the organization.
  name: ""
# TO BE REMOVED before the final customer release
#! The default configuration for `config.yaml` includes exclusions for the August release
#! to facilitate selective deployment, optimizing for specific testing and development needs.
#!
#! For the September release, all components need to be deployed to ensure comprehensive testing
#! and functionality validation. To facilitate this, a specific overlay for the June release
#! is applied when the `make June_release` command is executed. This overlay removes any
#! exclusions set in the May configuration, allowing for a full deployment of all packages
#! and components.
internal:
  excludedComponents:
EOF




29) Fixes this error (2025-01-22T21:01:57Z [x] Missing tools for installation:  velero)

cd /bigdisk/tanzu-installer/additional-resources/binaries/linux/amd64
​​sudo cp crashd /usr/local/bin/
sudo cp imgpkg /usr/local/bin/
sudo cp kctrl /usr/local/bin/
sudo cp kapp /usr/local/bin/
sudo cp tanzu /usr/local/bin/
sudo cp kbld /usr/local/bin/
sudo cp velero /usr/local/bin/



30) Verify
	cd /home/kubo/orf

	/bigdisk/tanzu-installer/cli_bundle/linux/amd64/tanzu-sm-installer verify -f /home/kubo/orf/config.yaml -u "${ARTIFACTORY_USER}:${ARTIFACTORY_API_TOKEN}" -r ${DOCKER_REGISTRY}/hub-self-managed/${TANZU_SM_VERSION}/repo --install-version ${TANZU_SM_VERSION} --kubeconfig ${KUBECONFIG}


	


31) Install
	/bigdisk/tanzu-installer/cli_bundle/linux/amd64/tanzu-sm-installer install -f /home/kubo/orf/config.yaml -u "${ARTIFACTORY_USER}:${ARTIFACTORY_API_TOKEN}" -r ${DOCKER_REGISTRY}/hub-self-managed/${TANZU_SM_VERSION}/repo --install-version ${TANZU_SM_VERSION} --kubeconfig ${KUBECONFIG}




32) Check

	kubectl get packageinstall -A
	#
	# find the problems
	#
	kubectl get packageinstall -A | grep -v succeeded



33) Get external IP

	kubectl get svc -A | grep Load

	# tanzusm             contour-envoy                                      LoadBalancer   10.106.213.251   192.168.0.6   80:32081/TCP,443:30293/TCP                              16h

34) Update DNS somehow (not how yet...)

	tanzu.platform.io = 192.168.0.6










```
