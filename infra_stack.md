
# Wishlist

- Login with university credentials
- Authorization for
    - Students
    - Teachers
- ewatercycle python package and friends installed
- Jupyter notebook
- Able to run ewatercycle models in containers
- Access to read-only directory with forcing and model parameters data. 
- nbgrader or alternative to distribute assignments, collect submissions, grade them, and return feedback to students. 
- All users should not be on the same single machine
- Host web application where you can select a model, a region, and a time period and run model or generate a notebook

## VMs on SRC

We run Virtual Machines (VMs) on the Surf Research Cloud.
Each VM runs JupyterHub and the web application.

Pros:
- Looks a lot like <https://github.com/eWaterCycle/infra>

Cons:
- Students need to be divided over multiple VMs
- Students can interfere with each other. For example one student could use all cpu, memory or disk space.
- nbgrader needs exchange directory which will need to be shared between VMs.
- SRC Cluster can not be resized automaticly, will need to be done manually somehow.

## Frontend VM & Slurm on hpc

We run a Virtual Machine (VM) on the Surf Research Cloud.
This VM runs JupyterHub and the web application.
When a Jupyter server needs to be spawned, a job is submitted to the Slurm scheduler on the HPC cluster.

Pros:
- HPC cluster already has apptainer and Slurm
- When not used the VM is only thing running
- like [RS-DAT](https://github.com/RS-DAT)

Cons:
- need help/permission from SURF to configure project on HPC cluster
    - student/teacher needs to be mapped to a HPC user

## Kubernetes

Use [Zero to JupyterHub with Kubernetes](https://z2jh.jupyter.org) to JupyterHub on Kubernetes. The web application is a separate deployment.

Pros:
- k8s does most babysitting

Cons:
- need a k8s cluster
- Need to run apptainer in kubernetes pod
- nbgrader uses exechange directory with write righs for all which is hard on k8s. Might use [ngshare](https://github.com/LibreTexts/ngshare) or ReadWriteMany pvc like nfs
