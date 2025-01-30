```
Shepherd / Sheepctl
=====================
#
# Original instructions (Shepheard: https://vmw-confluence.broadcom.net/pages/viewpage.action?pageId=1862031629
# My Instructions: https://github.com/ogelbric/shepherd-orf/blob/main/README.md
# Thomas Instructions: https://docs.google.com/document/d/13pdIArdZy5BWsnMIBTCi2vbAUYZ6YlfdCuFWanM3rFk/edit?tab=t.0
# Official doc: https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-platform/10-0/tnz-platform/tp-sm-install-install-tp-sm.html
# TPSM on GKE: https://docs.google.com/document/d/1l5bm3ox8AZjYaV5xIXqssBH80yxszvpdP1SFl2iYVKc/edit?pli=1&tab=t.0
#
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
	sheepctl pool lock TKGs-Non-Airgapped -n Tanzu-Sales --lifetime 5d --description 'Used by EPC TSL version 4(Orf)'
	sheepctl lock list -n Tanzu-Sales

+--------------------------------------+--------+----------------+------------+-------------+--------------------+--------------------------------+
|                  ID                  | STATUS |    CREATED     | EXPIRATION |  NAMESPACE  |        POOL        |          DESCRIPTION           |
+--------------------------------------+--------+----------------+------------+-------------+--------------------+--------------------------------+
| a4684e93-2c82-4ac7-9fd7-7e3471f3fdea | locked | 13 minutes ago | in 5 days  | Tanzu-Sales | TKGs-Non-Airgapped | Used by EPC TSL version 4(Orf) |
| 0e450633-978c-4704-9432-f047731afbf5 | locked | 21 hours ago   | in 5 days  | Tanzu-Sales | TKGs-Non-Airgapped | Used by US East                |
| 9596f88d-0681-4e8d-a4b9-5c47b35ab4d6 | locked | 5 days ago     | in 5 days  | Tanzu-Sales | TKGs-Non-Airgapped | Used by EPC TSL version 2      |
+--------------------------------------+--------+----------------+------------+-------------+--------------------+--------------------------------+

	sheepctl lock get -n Tanzu-Sales {lock guid} -j -o lockfile.json



	#
	# delte an instance 
	#sheepctl lock delete 0040b5bd-deef-4bb7-a131-69acb8f291b4


9) Get the lock information (json file)
	sheepctl lock list -n Tanzu-Sales 
	sheepctl lock get -n Tanzu-Sales a4684e93-2c82-4ac7-9fd7-7e3471f3fdea

10) Get vCenter info
	sheepctl lock get -n Tanzu-Sales a4684e93-2c82-4ac7-9fd7-7e3471f3fdea  > /tmp/a 2>&1 ; grep -w5  administrator /tmp/a | tail -9
            "vimUsername": "administrator@vsphere.local",
            "vimPassword": "FDFC.yX30wVqme_x",
            "vimPort": 443,
            "deploymentType": "embedded",
            "osFamily": "linux",
            "systemPNID": "lvn-dvm-10-192-19-242.dvm.lvn.broadcom.net"

11) Jumper Info
	sheepctl lock get -n Tanzu-Sales a4684e93-2c82-4ac7-9fd7-7e3471f3fdea  > /tmp/a 2>&1 ; grep -w5  kubo /tmp/a | grep hostname | head -1
        "hostname": "10.192.22.205",
	sheepctl lock get -n Tanzu-Sales a4684e93-2c82-4ac7-9fd7-7e3471f3fdea  > /tmp/a 2>&1 ; grep -w5  kubo /tmp/a | grep password | tail -1
        "password": "Ponies!23"

12) ssh to jumper
	ssh kubo@10.192.22.205 #Ponies!23
	mkdir orf
	cd orf

13) Test connect to supvisor cluster
	#
	# double check the IP in vCenter -> workload management -> supervisor
	# 
	#

	kubectl vsphere login --server=192.168.0.2 --vsphere-username administrator@vsphere.local --insecure-skip-tls-verify
	# FDFC.yX30wVqme_x
	

14) Re-write cluster yaml with new storage class name (yaml is in next step)
	#
	#crate a new content lib https://wp-content.vmware.com/v2/latest/lib.json / when needed - option
	#create namespace = namespace1000
	#  add new content lib to Superviser - General - Tanzu Kubernetes Grid Service - Edit 
	#create in namespace administrator@vsphere.local owner access
	#create in namespace storage (tkgs-k8s-obj-policy)
	#Create a VM Class (tpsm)
	Login to the vCenter 
		administrator@vsphere.local
		Goto Workload Management > Services > VM Service > Manage > VM Classes > CREATE VM CLASS 
		Give a Name: tpsm
			compatibility choose default: don't change the default value
			CPU=8, Mem 32GB
			#CPU 12, Memory 36GB this does not deploy in the shepherd env
			#
			Under Namespaces > namespace1000) > VM Service > MANAGE VM CLASSES > Click the checkbox against tpsm > click OK
		Add the new VMclass to the namespace

	#cat cluster1.yaml | sed "s/pacific-gold-storage-policy/tkgs-k8s-obj-policy/g" > cluster11.yaml
	#Fix version to max: v1.29.4---vmware.3-fips.1-tkg.1           v1.29.4+vmware.3-fips.1-tkg.1  
	#Fix namespace my case namespace1000
	#
	kubectl vsphere login --server=192.168.0.2 --vsphere-username administrator@vsphere.local --insecure-skip-tls-verify
	# FDFC.yX30wVqme_x
	kubectl config use-context namespace1000
	
	kubectl get sc
	NAME                   PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
	tkgs-k8s-obj-policy    csi.vsphere.vmware.com   Delete          Immediate           true                   21h

	alias k=kubectl
	k get virtualmachineclasses

NAME                  CPU   MEMORY
best-effort-2xlarge   8     64Gi
best-effort-4xlarge   16    128Gi
best-effort-8xlarge   32    128Gi
best-effort-large     4     16Gi
best-effort-medium    2     8Gi
best-effort-small     2     4Gi
best-effort-xlarge    4     32Gi
best-effort-xsmall    2     2Gi
guaranteed-2xlarge    8     64Gi
guaranteed-4xlarge    16    128Gi
guaranteed-8xlarge    32    128Gi
guaranteed-large      4     16Gi
guaranteed-medium     2     8Gi
guaranteed-small      2     4Gi
guaranteed-xlarge     4     32Gi
guaranteed-xsmall     2     2Gi
tpsm                  8     32Gi


15) Cluster yaml that worked: 

cat <<'EOF' > /home/kubo/orf/orfcluster.yaml
apiVersion: cluster.x-k8s.io/v1beta1
kind: Cluster
metadata:
  name: orfcluster1
  namespace: namespace2000
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
          replicas: 13 # 10 # 7 # 3 # 13 # in doc this is 14 - testing with 13 (1-28-2025)
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
	# Create cluster (this will take some time ~30-40min / Watch in vCenter)
	#
	kubectl apply -f orfcluster.yaml


18) Log onto guest cluster 
	#
	# orfcluster1
	#
	kubectl vsphere login --server 192.168.0.2 --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-namespace namespace2000 --tanzu-kubernetes-cluster-name orfcluster1 --insecure-skip-tls-verify
	#
	# FDFC.yX30wVqme_x

19) Test guest cluster
	alias k=kubectl

	kubectl config get-contexts

CURRENT   NAME                   CLUSTER       AUTHINFO                                      NAMESPACE
          192.168.0.2            192.168.0.2   wcp:192.168.0.2:administrator@vsphere.local   
          namespace1000          192.168.0.2   wcp:192.168.0.2:administrator@vsphere.local   namespace1000
*         orfcluster1            192.168.0.3   wcp:192.168.0.3:administrator@vsphere.local   
          svc-tkg-domain-c9      192.168.0.2   wcp:192.168.0.2:administrator@vsphere.local   svc-tkg-domain-c9
          svc-velero-domain-c9   192.168.0.2   wcp:192.168.0.2:administrator@vsphere.local   svc-velero-domain-c9
          testns                 192.168.0.2   wcp:192.168.0.2:administrator@vsphere.local   testns


	kubectl get nodes

NAME                                        STATUS   ROLES           AGE     VERSION
orfcluster1-node-pool-1-xnm5q-rk7cc-r7q6p   Ready    <none>          3h35m   v1.29.4+vmware.3-fips.1
orfcluster1-node-pool-1-xnm5q-rk7cc-rptz6   Ready    <none>          3h14m   v1.29.4+vmware.3-fips.1
orfcluster1-node-pool-1-xnm5q-z27j8-76hmk   Ready    <none>          151m    v1.29.4+vmware.3-fips.1
orfcluster1-node-pool-1-xnm5q-z27j8-88vsr   Ready    <none>          156m    v1.29.4+vmware.3-fips.1
orfcluster1-node-pool-1-xnm5q-z27j8-fmljv   Ready    <none>          3h2m    v1.29.4+vmware.3-fips.1
orfcluster1-node-pool-1-xnm5q-z27j8-fvrqd   Ready    <none>          14m     v1.29.4+vmware.3-fips.1
orfcluster1-node-pool-1-xnm5q-z27j8-g97n7   Ready    <none>          176m    v1.29.4+vmware.3-fips.1
orfcluster1-node-pool-1-xnm5q-z27j8-j8f5m   Ready    <none>          169m    v1.29.4+vmware.3-fips.1
orfcluster1-node-pool-1-xnm5q-z27j8-l2555   Ready    <none>          163m    v1.29.4+vmware.3-fips.1
orfcluster1-node-pool-1-xnm5q-z27j8-mdgsd   Ready    <none>          3m13s   v1.29.4+vmware.3-fips.1
orfcluster1-node-pool-1-xnm5q-z27j8-ptkfj   Ready    <none>          19m     v1.29.4+vmware.3-fips.1
orfcluster1-node-pool-1-xnm5q-z27j8-qshfl   Ready    <none>          145m    v1.29.4+vmware.3-fips.1
orfcluster1-node-pool-1-xnm5q-z27j8-z742f   Ready    <none>          8m40s   v1.29.4+vmware.3-fips.1
orfcluster1-slgts-vtplw                     Ready    control-plane   4h33m   v1.29.4+vmware.3-fips.1




20) TP-SM Install steps

Internal Doc: https://vmw-confluence.broadcom.net/pages/viewpage.action?pageId=2122072341
Official Doc: https://techdocs.broadcom.com/us/en/vmware-tanzu/platform/tanzu-platform/10-0/tnz-platform/tp-sm-install-install-tp-sm.html

21) Download 45 GB
	Favorite web browser log on with your Broadcom.net ID
	Accept the little check box
	https://support.broadcom.com/group/ecx/productdownloads?subfamily=Tanzu%20Platform%20Self%20Managed

22) Moved and Extract
	#
	# jump host needs another 150 GB disk to do all of these downloads
	# vCenter add second disk to jumper server (Edit - settings - add device (disk) 150GB)
	#
	#disk device info
	lsblk
	# looks like sdb      8:16   0  150G  0 disk  is my new disk 
	sudo parted /dev/sdb
	mklabel gpt
	print
	mkpart primary 0 145GB
	Ignore
	quit
	sudo mkfs.ext4 /dev/sdb1
	sudo mkdir /bigdisk
	sudo mount /dev/sdb1 /bigdisk
	sudo mkdir /bigdisk/tanzu-installer
	cd /home/kubo/orf
	#Move from Mac(Internal or lab server) to jumper server (From lab server to shepherd env ~20min): 
	#
	scp tanzu-self-managed-10.0.0.tar.gz kubo@10.192.22.205:/home/kubo/orf/.   #(2hours 47 min) good thing the jumper has a 85 GB drive
	 
	#
	# takes about 20 min 
	#
	sudo tar -xzvf tanzu-self-managed-10.0.0.tar.gz  -C /bigdisk/tanzu-installer &


23) DNS update on the jump box

	sudo vi /etc/dnsmasq.d/vlan-dhcp-dns.conf
	address=/tanzu.platform.io/192.168.0.4
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


24) Install some stuff on jump box... 

	cd /home/kubo/orf
	#
	#Fetch worker cluster kubeconfig and export it
	#
	kubectl vsphere login --server=192.168.0.2 --vsphere-username administrator@vsphere.local --insecure-skip-tls-verify
	# FDFC.yX30wVqme_x

	kubectl config use-context namespace1000
	kubectl get secret -n namespace2000
	#
	# orfcluster kubeconfig
	#
	kubectl get secret orfcluster1-kubeconfig -n namespace2000 -o json | jq -r '.data["value"] | @base64d' > orfwrk-kc 
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

	#
	#Pull exe's out of tar bundle
	#

	cd /bigdisk/tanzu-installer/additional-resources/binaries/linux/amd64
	​​sudo cp crashd /usr/local/bin/
	sudo cp imgpkg /usr/local/bin/
	sudo cp kctrl /usr/local/bin/
	sudo cp kapp /usr/local/bin/
	sudo cp tanzu /usr/local/bin/
	sudo cp kbld /usr/local/bin/
	sudo cp velero /usr/local/bin/
	cd /home/kubo/orf

25) Making sure the lock is not going away - trying to extend the lease

	sheepctl lock list -n Tanzu-Sales                                               
+--------------------------------------+--------+-------------+------------+-------------+--------------------+---------------------------+
|                  ID                  | STATUS |   CREATED   | EXPIRATION |  NAMESPACE  |        POOL        |        DESCRIPTION        |
+--------------------------------------+--------+-------------+------------+-------------+--------------------+---------------------------+
| 0040b5bd-deef-4bb7-a131-69acb8f291b4 | locked | 2 days ago  | in 4 days  | Tanzu-Sales | TKGs-Non-Airgapped | Used by EPC TSL version 3 |
| 9596f88d-0681-4e8d-a4b9-5c47b35ab4d6 | locked | 4 days ago  | in 2 days  | Tanzu-Sales | TKGs-Non-Airgapped | Used by EPC TSL version 2 |
| ffaa1e86-3a00-4c8a-81e0-862aee201331 | locked | 10 days ago | in 2 days  | Tanzu-Sales | TKGs-Non-Airgapped | Used by EPC TSL           |
+--------------------------------------+--------+-------------+------------+-------------+--------------------+---------------------------+
 

	sheepctl lock extend 0040b5bd-deef-4bb7-a131-69acb8f291b4 -t 5d -n Tanzu-Sales 



26) Cert Manager
	Log onto guest cluster 
	export KUBECONFIG=''
	kubectl vsphere login --server 192.168.0.2 --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-namespace namespace2000 --tanzu-kubernetes-cluster-name orfcluster1 --insecure-skip-tls-verify
	#
	# FDFC.yX30wVqme_x

	alias k=kubectl
	#
	kubectl config use-context orfcluster1
	kubectl config get-contexts
	#


CURRENT   NAME                   CLUSTER       AUTHINFO                                      NAMESPACE
          192.168.0.2            192.168.0.2   wcp:192.168.0.2:administrator@vsphere.local   
          namespace1000          192.168.0.2   wcp:192.168.0.2:administrator@vsphere.local   namespace1000
*         namespace2000          192.168.0.2   wcp:192.168.0.2:administrator@vsphere.local   namespace2000
          orfcluster1            192.168.0.3   wcp:192.168.0.3:administrator@vsphere.local   
          svc-tkg-domain-c9      192.168.0.2   wcp:192.168.0.2:administrator@vsphere.local   svc-tkg-domain-c9
          svc-velero-domain-c9   192.168.0.2   wcp:192.168.0.2:administrator@vsphere.local   svc-velero-domain-c9
          testns                 192.168.0.2   wcp:192.168.0.2:administrator@vsphere.local   testns


	kubectl create namespace cert-manager # in my case the namespace was there but noting in it
	#
	tanzu plugin install package
	yes
	yes

	#
	# make sure you have a tanzu repo !!!
	#
	tanzu package repository list -A
	#
	# repo list is empty
	#
	#
	# make sure you can nslookup projects.registry.vmware.com and it resolves
	# if not then go back to DNS setup step
	#
	
	/home/kubo/orf/build/imgpkg tag list -i projects.registry.vmware.com/tkg/packages/standard/repo
	#
	# Create a repo file (Add the Package Repository to the Cluster)
	#
	cd /home/kubo/orf
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
	kubectl vsphere login --server 192.168.0.2 --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-namespace namespace2000 --tanzu-kubernetes-cluster-name orfcluster1 --insecure-skip-tls-verify
	#
	# FDFC.yX30wVqme_x

	# Apply the repo info
	kubectl apply -f /home/kubo/orf/packagerepo1.yaml

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
	

	#
	# Option D Test - not 100% working Option B got cert-manager working and Option A got CAinjector and web hook working 
	#
	kubectl label ns cert-manager --overwrite=true pod-security.kubernetes.io/enforce=privileged

	tanzu package install cert-manager -p cert-manager.tanzu.vmware.com -n cert-manager -v 1.7.2+vmware.3-tkg.3
	
	tanzu package installed list -n cert-manager

	kubectl -n cert-manager get all

	#
	# looks like cert manager is in reconcile status...
	# events reveal that there is an issue
	k get events  -n cert-manager --sort-by='.lastTimestamp'

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
	# Looks like Option (B) works for cert-manager 
	# Option (A) has to be used for cert-manager-cainjector and webhook
	# Option (D) from Thomas does not seem to work at al in my env.
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


	# super crude fix until the package cert-manager can be installed with out security problems
	# place above 3 EOF..EOF sections into a script and run below command it will then eventually reconcile all TPSM packages (except cert-manager) 
	# while x=1; do ./certmanager.sh ; sleep 45 ; done
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

	#
	# Option (D) (Thomas) 
	#

	tanzu package repository add tanzu-standard --url harbor.mbentley.net/tkg/packages/standard/repo --namespace tkg-system

	## to List the versions of Cert-Manager Available
	tanzu package available list cert-manager.tanzu.vmware.com -A

	##Command to install the cert-manager
	kubectl create ns cert-manager
	kubectl label ns cert-manager --overwrite=true pod-security.kubernetes.io/enforce=privileged

	##Sample Command
	tanzu package install cert-manager --package cert-manager.tanzu.vmware.com --namespace cert-manager --version 1.7.2+vmware.3-tkg.3

	tanzu package installed list -n cert-manager

	# NAME         PACKAGE-NAME                   PACKAGE-VERSION STATUS
	# cert-manager cert-manager.tanzu.vmware.com  1.7.2+vmware.3-tkg.3 Reconcile succeeded

	kubectl get all -n cert-manager


	

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
	export ARTIFACTORY_API_TOKEN="cmVmdGtuOjAxOjE3NjxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxQ2JnMXBIdWFnNW9ZckpzWmg1" 
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
	# FDFC.yX30wVqme_x

	#
	# orfcluster1 kubeconfig
	#
	kubectl config use-context namespace1000
	kubectl get secret orfcluster1-kubeconfig -n namespace1000 -o json | jq -r '.data["value"] | @base64d' > orfwrk-kc 
	export KUBECONFIG=orfwrk-kc


	echo $KUBECONFIG

	#orfwrk-kc
	

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

	#
	# save off the original config file 
	# this file after in TP-Sm instal will not be the same!!!
	#
	cp /home/kubo/orf/config.yaml /home/kubo/orf/config.yaml.orig






30) Verify TP-SM install


	#
	# for good measure log into guest cluster
	#
	kubectl vsphere login --server 192.168.0.2 --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-namespace namespace2000 --tanzu-kubernetes-cluster-name orfcluster1 --insecure-skip-tls-verify
	#
	# FDFC.yX30wVqme_x
	kubectl config get-contexts

	cd /home/kubo/orf

	/bigdisk/tanzu-installer/cli_bundle/linux/amd64/tanzu-sm-installer verify -f /home/kubo/orf/config.yaml -u "${ARTIFACTORY_USER}:${ARTIFACTORY_API_TOKEN}" -r ${DOCKER_REGISTRY}/hub-self-managed/${TANZU_SM_VERSION}/repo --install-version ${TANZU_SM_VERSION} --kubeconfig ${KUBECONFIG}


	


31) Install TP-SM 
	/bigdisk/tanzu-installer/cli_bundle/linux/amd64/tanzu-sm-installer install -f /home/kubo/orf/config.yaml -u "${ARTIFACTORY_USER}:${ARTIFACTORY_API_TOKEN}" -r ${DOCKER_REGISTRY}/hub-self-managed/${TANZU_SM_VERSION}/repo --install-version ${TANZU_SM_VERSION} --kubeconfig ${KUBECONFIG}

	# enter foundation

	# try -y next time 




32) Check

	kubectl get packageinstall -A
	#
	# find the problems
	#
	kubectl get packageinstall -A | grep -v succeeded
	#
	# get all the errors
	#
	kubectl get packageinstall -A | grep -v succeeded | tail -n+2 | awk '{ print "kubectl describe packageinstall -n " $1 " " $2 " | grep Useful >> /tmp/e1" }' > /tmp/e0
	chmod +x /tmp/e0
	rm /tmp/e1
	/tmp/e0
	cat /tmp/e1

	#
	# In my case
	# 
	#  Useful Error Message:    kapp: Error: Timed out waiting after 5m0s for resources:
	#  Useful Error Message:    kapp: Error: waiting on reconcile deployment/ccc-rules-service (apps/v1) namespace: tanzusm:
	#  Useful Error Message:    kapp: Error: waiting on reconcile deployment/ensemble-ucp (apps/v1) namespace: tanzusm:
	#  Useful Error Message:    kapp: Error: waiting on reconcile deployment/ensemble-findings (apps/v1) namespace: tanzusm:
	#  Useful Error Message:    kapp: Error: waiting on reconcile deployment/kafka-topic-controller (apps/v1) namespace: tanzusm:
	#  Useful Error Message:    kapp: Error: waiting on reconcile deployment/postgres-endpoint-controller (apps/v1) namespace: tanzusm:
	#  Useful Error Message:    kapp: Error: waiting on reconcile packageinstall/guardrails-helm (packaging.carvel.dev/v1alpha1) namespace: tanzusm:
	#  Useful Error Message:    kapp: Error: waiting on reconcile packageinstall/kafka-topic-controller (packaging.carvel.dev/v1alpha1) namespace: tanzusm:
	#  Useful Error Message:    kapp: Error: waiting on reconcile deployment/cluster-service-server (apps/v1) namespace: tanzusm:
	#  Useful Error Message:    kapp: Error: waiting on reconcile deployment/ucp-envoy (apps/v1) namespace: tanzusm:

	# Lets look at the PODs

	kubectl get pods -A | grep -v Running

	kubectl get pods -A | grep -v Running | tail -n+2 | awk '{ print "kubectl describe pod -n " $1 " " $2 " | grep Warning >> /tmp/e3" }' > /tmp/e2
	chmod +x /tmp/e2
	rm /tmp/e3
	/tmp/e2
	cat /tmp/e3

	# In my case Insufficient memory, 5 Insufficient cpu. preemption: 0/6  Need bigger cluster 
	# Warning  FailedScheduling  26m (x160 over 13h)  default-scheduler  0/6 nodes are available: 
	# 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }, 
	# 2 Insufficient memory, 5 Insufficient cpu. preemption: 0/6 nodes are available: 
	# 1 Preemption is not helpful for scheduling, 5 No preemption victims found for incoming pod.
	# Installation failed with error: pacakges are in unstable state.

	# And when you make the cluster bigger... the following results

	# kubo@UGN5JmN7F92hA:~/orf$ kubectl get pods -A | grep -v Running  | wc -l
	# 174
	# kubo@UGN5JmN7F92hA:~/orf$ kubectl get pods -A | grep -v Running  | wc -l
	# 172
	# kubo@UGN5JmN7F92hA:~/orf$ kubectl get pods -A | grep -v Running  | wc -l
	# 172
	# kubo@UGN5JmN7F92hA:~/orf$ kubectl get pods -A | grep -v Running  | wc -l
	# 170



	#
	#
	# Check the install
	#
	/bigdisk/tanzu-installer/cli_bundle/linux/amd64/tanzu-sm-installer post-verify --kubeconfig ${KUBECONFIG}



	#
	# TP=SM delete / clean up
	#

	/bigdisk/tanzu-installer/cli_bundle/linux/amd64/tanzu-sm-installer reset -p --kubeconfig ${KUBECONFIG}

	# 2025-01-23T17:47:49Z [i] Executing this reset will purge the namespace and all resources associated with 'Tanzu Platform Self Managed.'
	# Removing PVC/PV deletion, Do you wish to proceed with the reset?
	# If you are sure you can continue by typing 'Tanzu For Life' at the prompt: Tanzu For Life

	# Delete takes a long time... !!!!




33) Get external IP

	kubectl get svc -A | grep Load

	# tanzusm             contour-envoy                                      LoadBalancer   10.97.146.156    192.168.0.4   80:31901/TCP,443:31904/TCP 

34) Update DNS  (tanzu.platform.io = 192.168.0.4)

	#1) Add to host file on jump box 
	sudo vi /etc/hosts # and add  "192.168.0.4 tanzu.platform.io" 

	#2) Update local DNS mask
	sudo vi /etc/dnsmasq.d/vlan-dhcp-dns.conf
	address=/tanzu.platform.io/192.168.0.4
	sudo systemctl restart dnsmasq

	
	nslookup tanzu.platform.io
	#Server:		127.0.0.1
	#Address:	127.0.0.1#53

	#Name:	tanzu.platform.io
	#Address: 192.168.0.4


	#
	# Restart squid proxy / make sure port is 443
	#
	sudo vi /etc/squid/squid.conf
	change http_port number from 3128 to 443 (if you see "http_port 443" then no changes needed )
	sudo systemctl restart squid
	


	Firefox add proxy (IP of jump server and port 443) (

	# Settings -> General -> Network -> Settings -> Https Proxy {IP of jump box  and port 443} and check box use for https
	#no proxy bypass list
	#127.0.0.1
	#[::1]
	#localhost

	# test proxy sample commands

	curl -v --proxy "http://10.192.22.205:443" "https://tanzu.platform.io"

	sudo tail -f /var/log/squid/access.log

	sudo systemctl restart squid


35) Test the install / log on

	#
	# if the said proxy would be working for me I could get a browser open
	# and past the URL and get the code. 
	# 
	# 	ID: tanzu_platform_admin
	#	PWD: admin123

	tanzu login --endpoint https://tanzu.platform.io  --insecure-skip-tls-verify

[i] Opening the browser window to complete the login
[!] failed to open the browser for login:exec: "xdg-open,x-www-browser,www-browser": executable file not found in $PATH
Log in by visiting this link:

    https://tanzu.platform.io/auth/oauth/authorize?client_id=tp_cli_app&code_challenge=E9_amX3o406Qq_fAUXh-p8FLNuPFIeWjVHpfJbv3q80&code_challenge_method=S256&redirect_uri=http%3A%2F%2F127.0.0.1%3A43471%2Fcallback&response_type=code&state=e505aab75e7d91bfdec223a3a9ed9616

    Optionally, paste your authorization code: 


	#
	# When pasting the above URL it will come back with what looks like an error, but the token is in the resulting URL
	# Paste that token into the CLI question and you will get logged on


http://127.0.0.1:36317/callback?code=7UUVJ-hDCTlT__Gfz-T2mG0Yw2odb5kq&state=c1321a1f9844cea73d0be582e5947a60

    Optionally, paste your authorization code: Lrfvcpf0Jn3C_e_0Ys4aVltoI42B4WOl


[ok] Successfully logged in to 'https://tanzu.platform.io' and created a tanzu context


	# future commands to explore 
	# tanzu api-token create
	# export TANZU_API_TOKEN=Lrfvcpf0Jn3C_e_0Ys4aVltoI42B4WOl
	# tanzu login --endpoint https://tanzu.platform.io --insecure-skip-tls-verify

36) Tanzu comand expansion

	tanzu config eula accept
	tanzu plugin install app --group vmware-tanzucli/essentials
	tanzu plugin install --group vmware-tanzu/app-developer
	tanzu plugin install --group vmware-tanzu/platform-engineer
	tanzu plugin install --group vmware-tmc/default
	tanzu plugin install --group vmware-vsphere/default 
	tanzu plugin install --group vmware-tkg/default
	tanzu plugin install --group vmware-tap/default 

	#
	# this should work now
	#
	tanzu project create firstproject
	Create firstproject project in Tanzu Platform Self Managed org? [yN]: y
	x Failed to create project firstproject in Tanzu Platform Self Managed org. Error: input: authMutation.projectMutation.upsertProject INTERNAL


	tanzu context current
 	# Name:            tpsm-910fbd64
 	# Type:            tanzu
 	# Organization:    Tanzu Platform Self Managed (b91c7be4-0489-49d7-8a8d-7c34af480ef8)
 	# Project:         none set
 	# Kube Config:     /home/kubo/.config/tanzu/kube/config
 	# Kube Context:    tanzu-cli-tpsm-910fbd64








```
