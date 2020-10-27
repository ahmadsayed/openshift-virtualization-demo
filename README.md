# Unboxing OpenShift Virtualization 

OpenShift Virtualization is Redhat OpenShift features based on Kubevirt.

Before getting to what is kubevirt let us answer what kubevirt is not

* KubeVirt is not a competitor to Firecracker or Kata containers
* KubeVirt is not a container runtime replacement

## So Kubevirt will be  [defined as following](https://kubevirt.io/2020/KubeVirt_deep_dive-virtualized_gpu_workloads.html)

KubeVirt is a Kubernetes extension that allows running traditional VM workloads natively side by side with Container workloads.

## Why KubeVirt.

Practically all workloads originally host on VM, and even moving to Containers some stubborn workloads still prefer to stay on VM so we ends on the below architecture

![new-way](https://kubevirt.io/assets/2020-02-06-KubeVirt_deep_dive-virtualized_gpu_workloads/kubevirt_infrastructure_convergence.png)

Even more in may cases we run OpenShift on Virtual Machines, so we will ends up with the below archeticutre 

![OpenShiftonVM](https://miro.medium.com/max/700/0*7YVPoilhOjkH0qGo.png)


## What is the problem with previous diagrams 

* Multiple Stacks
* Multipe Skill sets.
* Inter communication and network policies between VM and containers getting more complex

## So What Kubevirt is trying to achieve 

Managed Both Cotnainers and VM with the Same Control plan 
![kubevir](https://miro.medium.com/max/700/0*BEWQ2I0c_XPH8Np9.png)

## Advantage of this approach 

* Converging VM management into container management workflows
* Using the same tooling (kubectl) for containers and Virtual Machines
* Keeping the declarative API for VM management (just like pods, deployments, etc…)

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
### Looks Familiar  ...

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

One Control Plan, Same Skillset and ... One Platform.

![one](https://upload.wikimedia.org/wikipedia/en/f/fc/TheOnefilm.jpg)

## Architecture 

Kubevirt is KVM+qemu process running inside the a pod but... 

### What is QEMU?

[QEMU](https://www.qemu.org/)  is a generic and open source machine emulator and virtualizer.

Run QEMU on your machines, just download the process and run it 

QEMU Demo 

### Create hardisk with qemu-img 

```
qemu-img.exe create -f qcow2 test.qcow2  5G
```
### Run the Linux and from iso and point it to the disk 
```
qemu-system-x86_64.exe -boot d -cdrom Fedora-Server-dvd-x86_64-32-1.6.iso -hda test.qcow2 -m 2048m -device bochs-display
```

### What is KVM?

[KVM](https://www.linux-kvm.org/page/Main_Page)  is a full virtualization solution for Linux on x86 hardware containing virtualization extensions (Intel VT or AMD-V).

What do we need to do to use KVM just add 

```
--accel kvm
```

Because I am running on windows I will use similar technology called [Intel HAXM](https://github.com/intel/haxm)
```
--accel hax
```

The command will become 

```
qemu-system-x86_64.exe -boot d -cdrom Fedora-Server-dvd-x86_64-32-1.6.iso -hda test.qcow2 -m 2048m -device bochs-display -accel hax
```

## Looking again at Kubevirt Architecutre 


![Simple](https://kubevirt.io/assets/2020-02-06-KubeVirt_deep_dive-virtualized_gpu_workloads/kubevirt_virtual_machine.png)

The VM Launch flow is shown in the following diagram. Since the user posts a VM manifest to the cluster until the Kubelet spins up the VM pod. And finally the virt-handler instructs the virt-launcher how to launch the qemu.

![nerdy](https://kubevirt.io/assets/2020-02-06-KubeVirt_deep_dive-virtualized_gpu_workloads/kubevirt_vm_launch_flow.png)



## What about the Storage?

Unlike The virtual Machines where we have custom resources for Virtual Machines Storage is exactly the same for Kubernetes
* Persistence Volume.
* Persistence Volume Claim.

![storage](https://kubevirt.io/assets/2020-02-06-KubeVirt_deep_dive-virtualized_gpu_workloads/kubevirt_volumes.png)

# Demo - Getting Start with OpenShift Virtualization 
* Install OpenShift Virtualization
* Create Hybrid Converged Cluster
* Show how to enable disable emulation mode(Remember the accel before ;) )
* Login to the node and check the qemu process running







