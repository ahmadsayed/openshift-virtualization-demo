# Unboxing OpenShift Virtualization 

OpenShift Virtualization is Redhat OpenShift features based on Kubevirt.

Before getting to what is kubevirt, let us answer what kubevirt is not.

* KubeVirt is not a competitor to Firecracker or Kata containers
* KubeVirt is not a container runtime replacement

## So Kubevirt will be  [defined as following](https://kubevirt.io/2020/KubeVirt_deep_dive-virtualized_gpu_workloads.html)

KubeVirt is a Kubernetes extension that allows running traditional VM workloads side by side with Container workloads.

## Why KubeVirt.

Practically all workloads originally host on VM, and even moving to Containers, some stubborn workloads still prefer to stay on VM, so we end on the below architecture.

![new-way](https://kubevirt.io/assets/2020-02-06-KubeVirt_deep_dive-virtualized_gpu_workloads/kubevirt_infrastructure_convergence.png)

In many cases, we run OpenShift on Virtual Machines, leading to the below architecture. 

![OpenShiftonVM](https://miro.medium.com/max/700/0*7YVPoilhOjkH0qGo.png)


## What is the problem with previous diagrams 

* Multiple Stacks
* Multiple Skill sets.
* Intercommunication and network policies between VM and containers getting more complex

## So What Kubevirt is trying to achieve 

Managed Both Containers and VM with the Same Control plan 
![kubevir](https://miro.medium.com/max/700/0*BEWQ2I0c_XPH8Np9.png)

## Advantage of this approach 

* Converging VM management into container management workflows
* Using the same tooling (kubectl) for containers and Virtual Machines
* Keeping the declarative API for VM management (just like pods, deployments, etc.…)

## How Virtual machines defined in Kubevirt

```
apiVersion: kubevirt.io/v1alpha1
kind: VirtualMachineInstance
...
 spec:
  domain:
   cpu:
    cores: 2
   devices:
    disk: fedora29
```
### Looks Familiar

```
apiVersion: apps/v1
kind: Deployment
...
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

## What does it mean?

One Control Plan, Same Skillset, and ... One Platform.

![one](https://upload.wikimedia.org/wikipedia/en/f/fc/TheOnefilm.jpg)

## Architecture 

Kubevirt is KVM+qemu process running inside the pod, but What is QEMU? 

### What is QEMU?

[QEMU](https://www.qemu.org/)  is a generic and open source machine emulator and virtualizer.

Run QEMU on your machines. Just download the process and run it. 

QEMU Demo 

### Create harddisk with qemu-img 

```
qemu-img.exe create -f qcow2 test.qcow2  5G
```

### Run the Linux and from iso and point it to the Disk 
```
qemu-system-x86_64.exe -boot d -cdrom Fedora-Server-dvd-x86_64-32-1.6.iso -hda test.qcow2 -m 2048m -device bochs-display
```

### What is KVM?

[KVM](https://www.linux-kvm.org/page/Main_Page)  is a full virtualization solution for Linux on x86 hardware containing virtualization extensions (Intel VT or AMD-V).

Just add the following parameter.

```
--accel kvm
```

Because I am running on windows, I will use a similar technology called [Intel HAXM](https://github.com/intel/haxm)
```
--accel hax
```

The command will become 

```
qemu-system-x86_64.exe -boot d -cdrom Fedora-Server-dvd-x86_64-32-1.6.iso -hda test.qcow2 -m 2048m -device bochs-display -accel hax
```

## Looking again at Kubevirt Architecture 


![Simple](https://kubevirt.io/assets/2020-02-06-KubeVirt_deep_dive-virtualized_gpu_workloads/kubevirt_virtual_machine.png)

The VM Launch flow is shown in the following diagram. Since the user posts, a VM manifest to the cluster until the Kubelet spins up the VM pod. And finally, the virt-handler instructs the virt-launcher how to launch the qemu.

![nerdy](https://kubevirt.io/assets/2020-02-06-KubeVirt_deep_dive-virtualized_gpu_workloads/kubevirt_vm_launch_flow.png)



## What about the Storage?

Unlike The virtual Machines where we have custom resources for Virtual Machines, Storage is the same for Kubernetes
* Persistence Volume.
* Persistence Volume Claim.

![storage](https://kubevirt.io/assets/2020-02-06-KubeVirt_deep_dive-virtualized_gpu_workloads/kubevirt_volumes.png)

## Cotainerized Data Importer

Containerized Data Importer (CDI) is a utility to import, upload and clone Virtual Machine images for use with KubeVirt. At a high level, a persistent volume claim (PVC), which defines VM-suitable storage via a storage class, is created.


## Data Format

Supported file formats are:
* Tar archive
* GZip Compressed file
* XZ compressed file
* Raw Image data
* ISO Image data
* Qemu qcow2 image data

Note: qcow2 images are converted into the raw format which is required by KubeVirt, resulting in the final file being a simple .img file.

## Cdi DataVolume

DataVolumes are an abstraction of the Kubernetes resource, PVC (Persistent Volume Claim) and it also leverages other CDI features to ease the process of importing data into a Kubernetes cluster.


![import](https://www.openshift.com/hs-fs/hubfs/cdi%20image.png?width=1535&name=cdi%20image.png)


# Demo - Getting Start with OpenShift Virtualization 
* Install OpenShift Virtualization
* Create Hybrid Converged Cluster
* Show how to enable or disable emulation mode(Remember the accel before ;) )
* login to the node and check the qemu process running

## Live Migration Demo


* Workload needed to use RWX File Storage(Must)
* Networking to use Masquerade(Must)
* Using Attached Disk not Container Disk

### The Black Magic behind migrating Virtual machines in 4 seconds.

![magic](https://static1.cbrimages.com/wordpress/wp-content/uploads/2020/07/PRESTIGE-2.jpg)

## Live Migration: kubevirt copy only the memory from source to destination
## Networking .. Time !!!! 

I Will do my best but, if you want to do more, go for this outstanding Presentation. 

https://www.youtube.com/watch?v=zmYxdtFzK6s

## What is CNI?

[CNI](https://github.com/containernetworking/cni) (Container Network Interface), a Cloud Native Computing Foundation project, consists of a specification and libraries for writing plugins to configure network interfaces in Linux containers, along with many supported plugins. CNI concerns itself only with network connectivity of containers and removing allocated resources when the container is deleted. Because of this focus, CNI has a wide range of support, and the specification is simple to implement.

CNI's are two main categories 
* Core Plugins 
* 3rd Party Plugins

### Sample Core Plugins
* bridge(will be used in this Demo): Creates a bridge, adds the host and the container to it.
* macvlan: Creates a new MAC address, forwards all traffic to that to the container.
* host-device: Move an already-existing device into a container.

### Sample of 3rd Party Plugins
* Calico (will be used in this Demo)
* OVS and OVN (OpenShift default CNI)
* VMWare NSX: a CNI plugin that enables automated NSX L2/L3 networking and L4/L7 Load Balancing; network
* Cilium: (significate plugin because it is using BPF): 
* Amazon ECS CNI Plugins is a collection of Container Network Interface(CNI) Plugins used by the Amazon ECS Agent to configure the network namespace of containers with Elastic Network Interfaces (ENIs)
* Multus (will be used in this Demo)

## How CNI works inside  Kubernetes.

Reference CNI presentation
https://speakerdeck.com/eranyanay/writing-a-cni-plugin-from-scratch?slide=14

## Anatomy of Pod Networking

https://speakerdeck.com/eranyanay/writing-a-cni-plugin-from-scratch?slide=21

## MULTUS .. MUL... What ?

MULTUS is CNI plugins that enable other CNI's to attached to the POD
![multi-homed](https://github.com/intel/multus-cni/raw/master/doc/images/multus-pod-image.svg)


## Add Network attachment in Openshift

The 'NetworkAttachmentDefinition' is used to set up the network attachment, i.e., the pod's secondary interface. There are two ways to configure the 'NetworkAttachmentDefinition' as following:

* NetworkAttachmentDefinition with JSON CNI config ( Used in this Demo)
* NetworkAttachmentDefinition with CNI config file

### Sample Network Aattachement Definition

```
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: test-network-2
  namespace: default
spec:
  config: >-
    { 
		"cniVersion": "0.3.1", 
		"name": "test-network-2", 
		"type": "bridge", 
		"ipam":
			{
				"type":"static", 
				"addresses":[
					{"address":"161.156.177.56/28"}
				]
			}
	}
 ```
### Note: In OpenShift, we do not create Network Attachment Definition. We provide it as an additional network in OpenShift Operator.

Adding NetworkAttachement to OpenShift 

1 - Run the following Command
```
oc edit networks.operator.openshift.io cluster
```

2 - Add Additional network as below 

```
spec:
  additionalNetworks:
  - name: test-network-1
    namespace: default
    rawCNIConfig: '{ "cniVersion": "0.3.1", "name": "test-network-1", "type": "bridge",
      "ipam" :{"type":"static", "addresses":[{"address":"161.156.177.55/28"}]}}'
    type: Raw
```
3 - Check OpenShift Cluster a Network Attachement Definition is automatically created on OpenShift.


## MULTUS Why?

Sample Use Case RDQM and PaceMaker

![RDQM](https://www.ibm.com/support/knowledgecenter/SSFKSJ_9.0.0/com.ibm.mq.con.doc/q130280a.gif)

According to IBM Documentation  https://www.ibm.com/support/knowledgecenter/SSFKSJ_9.0.0/com.ibm.mq.con.doc/q130290_.htm

rdqm.ini needs to configure as follow 
```
Node:
  HA_Replication=192.168.4.1
Node:
  HA_Replication=192.168.4.2
Node:
  HA_Replication=192.168.4.3
 
```
HA Replication only accept IP and using service Cluster IP always fails one approach as follow 
1 - Use Multus to assign secondary NIC with fixed IP
2 - Using Multus IP to configure the replication.
3 - Primary NIC will handle North-South communication.


## Migrating from VMware to Kubevirt 

There are two approaches
*  Download the VMDK from Datastore, and convert it manually using qemu-img 
```
qemu-img convert -f vmdk -O qcow2 linux.vmdk linux.qcow2
```
* V2V Migration from VCenter 
  - Connect to VCenter 
  - Request Migration.
  
### Demo 
*  Show the scenario (https://ibm.ent.box.com/file/734809040510)
*  During migration (https://ibm.ent.box.com/file/734806485988)
*  After Migration (https://ibm.ent.box.com/file/734804210369)





