STEP1: CONTAINER PLATFORM INSTALLATION(CRI) - SINGULARITY

	Singularity Installation:

		Install the required packages:

        		$ sudo yum groupinstall -y
					
            		$ sudo yum install -y build-essential libglib2.0-dev libseccomp-dev pkg-config 
			
			$ sudo yum install -y git wget
			
			$ sudo yum install -y squashfs-tools runc libssl-dev uuid-dev libgpgme11-dev cryptsetup
               

		Install GO Software:

        		$ export VERSION=1.19.4 OS=linux ARCH=amd64 

            		$ wget https://dl.google.com/go/go$VERSION.$OS-$ARCH.tar.gz

            		$ sudo tar -C /usr/local -xzvf go$VERSION.$OS-$ARCH.tar.gz 

			$  rm go$VERSION.$OS-$ARCH.tar.gz
               

            create a file to automate the required environment variables configuration:

            	$ echo 'export PATH=/usr/local/go/bin:$PATH' >> ~/.bashrc 

		$ source ~/.bashrc
		
        Singularity Installation on RHEL9:

			Compile and install Singularity:

        			$ export VERSION=3.10.4
	
				$ wget https://github.com/sylabs/singularity/releases/download/v${VERSION}/singularity-ce-${VERSION}.tar.gz
                
				$ rpmbuild -tb singularity-ce-${VERSION}.tar.gz
				
				$ sudo rpm -ivh ~/rpmbuild/RPMS/x86_64/singularity-ce-$VERSION-1.el7.x86_64.rpm
				
				$ rm -rf ~/rpmbuild singularity-ce-$VERSION*.tar.gz
				
				Note: If you encounter a failed dependency error for golang but installed it from source, build with this command
				
					$ rpmbuild -tb --nodeps singularity-ce-${VERSION}.tar.gz
					
				$ ./mconfig
				
				$ make -C builddir rpm
				
				$ sudo rpm -ivh ~/rpmbuild/RPMS/x86_64/singularity-ce-$VERSION.el7.x86_64.rpm
                               
		 After successful execution of install, Check Singularity software version using below command:

			$ singularity version


STEP2: Install Kubernetes (k8s) Cluster on RHEL
 

	2.1 Disable Swap space on master and worker nodes:    

		$ sudo swapoff -a

	2.2 Disable SELinux master and worker nodes:  

		$ sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

	2.3 Configure hostnames on master and worker nodes

    	$ sudo hostnamectl set-hostname <machine name>

        $ sudo exec bash

    2.4 Install traffic control utility package     

        $ sudo dnf install -y iproute-tc
 
    2.5 Apply firewall rules for master and worker nodes

    	On Master node:

        	$ sudo firewall-cmd --permanent --add-port=6443/tcp

			$ sudo firewall-cmd --permanent --add-port=2379-2380/tcp

			$ sudo firewall-cmd --permanent --add-port=10250/tcp

			$ sudo firewall-cmd --permanent --add-port=10259/tcp

			$ sudo firewall-cmd --permanent --add-port=10257/tcp

			$ sudo firewall-cmd –reload

        On Worker Node:

        	$ sudo firewall-cmd --permanent --add-port=10250/tcp

			$ sudo firewall-cmd --permanent --add-port=30000-32767/tcp                                                

			$ sudo firewall-cmd --reload

		Please refer below link for ports of k8’s environment:

		https://kubernetes.io/docs/reference/networking/ports-and-protocols/



     2.6 Install Kubernetes packages

         2.6.1 Create Kubernetes.repo file (file attached to this repo) and copy it in /etc/yum/repos.d/ folder
	 
	 	https://github.com/VvsKrishna25/VvsKrishna25/blob/main/kubernetes.repo.txt

         2.6.2 Install K8 packages

			$ sudo dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

        2.6.3 Enable and start Kubelet service.

			$ sudo systemctl enable kubelet

			$ sudo systemctl start kubelet

Step3: Integrating singularity with Kubernetes

	3.1 Installing singularity-cri
		$ apt install socat
		
		$ git clone https://github.com/sylabs/singularity-cri.git
		
		$ cd singularity-cri
		
		$ git clone https://github.com/sylabs/singularity-cri.git
		
		$ cd singularity-cri
		
		$ git checkout tags/v1.0.0-beta.5 -b v1.0.0-beta.5 
		
		$ make && sudo make install
		
		
	3.2 Integrating with Kubernetes
	
		3.2.1 Create Singularity-CRI service
		
			Create a systemd service with the content given in below link 
			
				https://github.com/VvsKrishna25/VvsKrishna25/blob/main/sycri.service
				
			Enable and start service
			
				$ sudo systemctl enable sycri
				
				$ sudo systemctl start sycri
				
			To verify Singularity-CRI is running do the following
			
				$ sudo systemctl status sycri
		3.2.2 Modify kubelet config and restart kubelet service
		
			$ cat > /etc/default/kubelet <<EOF
  				KUBELET_EXTRA_ARGS=--container-runtime=remote \
  				--container-runtime-endpoint=unix:///var/run/singularity.sock \
  				--image-service-endpoint=unix:///var/run/singularity.sock
  				EOF
			
			$ sudo systemctl restart kubelet

Step:4  Setting up  Kubernetes cluster

	$ log=${HOME}/install-master.log

    $ pod_network_cdir=10.0.0.1/16

    $ sudo kubeadm init --pod-network-cidr ${pod_network_cdir} 2>&1 | tee ${log}

	Run the following the commands on Master Node to allow non-root users to access kubeadm

    	$ mkdir -p $HOME/.kube

		$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

		$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

 

    4.1 Install CNI plugin(weave Net CNI, Calico, Flannel )

    	Recommendation:

        	Calico provides scalability, high performance, and interoperability with existing Kubernetes workloads.

        	It already deployed in popular cloud technologies such as Google Cloud, AWS and Azure.

 		To install Calico CNI, run the following command from the master node

        	$ kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml

     	Once complete, execute this one.

     		$ kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml

    	To confirm if the pods have started, run the command:

 			$ watch kubectl get pods -n calico-system

			$ kubectl get nodes

			$ kubectl get nodes -o wide

			$ kubectl get all

	Run the following command for joining the worker nodes to the cluster and save it

	To add the worker node to the Kubernetes cluster, follow step 1 up until Step 2 (2.6.3)

		$ sudo kubeadm token create –print-join-command

		$ sudo kubeadm join ip-addr:port –token created-token–discovery-token-ca-cert-hash  sha256:xxxxxx
