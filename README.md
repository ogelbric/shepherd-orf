```
Shepherd / Sheepctl
=====================
Original instructions(not working!): https://vmw-confluence.broadcom.net/pages/viewpage.action?pageId=1862031629

1) SSH key
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

8) Look at looks
	sheepctl lock list -n Tanzu-Sales

+--------------------------------------+----------+----------------+------------+-------------+--------------------+-----------------+
|                  ID                  |  STATUS  |    CREATED     | EXPIRATION |  NAMESPACE  |        POOL        |   DESCRIPTION   |
+--------------------------------------+----------+----------------+------------+-------------+--------------------+-----------------+
| fdbcbafd-cfc6-4c0d-8ca7-300175c730e8 | creating | 8 minutes ago  |            | Tanzu-Sales | TKGs-Non-Airgapped | Tanzu-SouthEast |
| ffaa1e86-3a00-4c8a-81e0-862aee201331 | locked   | 18 minutes ago | in 5 days  | Tanzu-Sales | TKGs-Non-Airgapped | Used by EPC TSL |
| af6bb058-b2fa-4d70-bdd1-16f5f7a5f5ff | locked   | a day ago      | in 4 days  | Tanzu-Sales | TKGs-Non-Airgapped | Used by User001 |
+--------------------------------------+----------+----------------+------------+-------------+--------------------+-----------------+

9) Get the lock information
	sheepctl lock get -n Tanzu-Sales ffaa1e86-3a00-4c8a-81e0-862aee201331

10) Get vCenter info
	sheepctl lock get -n Tanzu-Sales ffaa1e86-3a00-4c8a-81e0-862aee201331  > /tmp/a 2>&1 ; grep -w5  administrator /tmp/a | tail -6
            "vimUsername": "administrator@vsphere.local",
            "vimPassword": "SBFJWxxxxxxxxxr",

11) Jumper Info
	sheepctl lock get -n Tanzu-Sales ffaa1e86-3a00-4c8a-81e0-862aee201331  > /tmp/a 2>&1 ; grep -w5  kubo /tmp/a | grep hostname | head -1
        "hostname": "10.167.66.84",
	sheepctl lock get -n Tanzu-Sales ffaa1e86-3a00-4c8a-81e0-862aee201331  > /tmp/a 2>&1 ; grep -w5  kubo /tmp/a | grep password | tail -1
        "password": "Poxxxxxxxx"

12) ssh to jumper
	ssh kubo@10.167.66.84 #Ponies!23
	mkdir orf
	cd orf

13) Connect to supvisor cluster
	kubectl vsphere login --server=192.168.0.2 --vsphere-username administrator@vsphere.local --insecure-skip-tls-verify
	# SBFJWxxxxxxxr

14) Get storage class
	kubectl get sc
	NAME                   PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
	tkgs-k8s-obj-policy    csi.vsphere.vmware.com   Delete          Immediate           true                   21h

15) Create new content lib
	https://wp-content.vmware.com/v2/latest/lib.json

16) Re-write cluster yaml with new storage class name
	cat cluster1.yaml | sed "s/pacific-gold-storage-policy/tkgs-k8s-obj-policy/g" > cluster11.yaml
	Fix version to max: v1.29.4---vmware.3-fips.1-tkg.1           v1.29.4+vmware.3-fips.1-tkg.1  
	Fix namespace 

17) Cluster yaml that worked: 

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
        value: best-effort-medium
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

18) Log onto guest cluster 
	kubectl vsphere login --server 192.168.0.2 --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-namespace namespace1000 --tanzu-kubernetes-cluster-name cluster1 --insecure-skip-tls-verify
	# SBFJWxxxxxxxxxx

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

22) moved and extract
	Move from Mac to jumper server: scp tahnz-self-managed-10.0.0.tar.gz kubo@10.167.66.84:/orf/.   #(2hours 47 min) good thing the jumper has a 85 GB drive
	mkdir tanzu-installer 
	tar -xzvf <INSTALLER-BUNDLE-FILENAME>.tar.gz -c ./tanzu-installer

23) Install some stuff... 
	cd orf
	#
	#Fetch worker cluster kubeconfig and export it
	#
	kubectl config use-context namespace1000
	kubectl get secret -n namespace1000
	kubectl get secret cluster1-kubeconfig -n namespace1000 -o json | jq -r '.data["value"] | @base64d' > wrk-kc 
	export KUBECONFIG=wrk-kc
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

	sheepctl lock extend ffaa1e86-3a00-4c8a-81e0-862aee201331 -t 3 -n Tanzu-Sales #does not work waiting for better command 



```
