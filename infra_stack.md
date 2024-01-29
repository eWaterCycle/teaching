
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

Use (Zero to JupyterHub with Kubernetes)[z2jh.jupyter.org] to JupyterHub on Kubernetes. The web application is a separate deployment.

Pros:
- k8s does most babysitting
Cons:
- need a k8s cluster
- Need to run apptainer in kubernetes pod
- nbgrader uses exechange directory with write righs for all which is hard on k8s. Might use [ngshare](https://github.com/LibreTexts/ngshare) or ReadWriteMany pvc like nfs
