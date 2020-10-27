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


