# docker build -t ghcr.io/ewatercycle/teaching:latest .
# docker run -ti --rm --privileged ghcr.io/ewatercycle/teaching:latest bash

# Folowing does not work 
# docker run -ti --rm --cap-add SYS_ADMIN --cap-add MKNOD --cap-add SYS_CHROOT --cap-add SETFCAP  ghcr.io/ewatercycle/teaching:latest bash
# apptainer run docker://ghcr.io/ewatercycle/leakybucket-grpc4bmi:v0.0.1
# FATAL:   container creation failed: mount hook function failure: mount /proc/self/fd/3->/var/lib/apptainer/mnt/session/rootfs error: while mounting image /proc/self/fd/3: squashfuse_ll exited with status 1: fuse: device not found, try 'modprobe fuse' first

# apptainer pull docker://ghcr.io/ewatercycle/leakybucket-grpc4bmi:v0.0.1
# ipython
# from grpc4bmi.bmi_client_apptainer import BmiClientApptainer
# model = BmiClientApptainer('docker://ghcr.io/ewatercycle/leakybucket-grpc4bmi:v0.0.1', work_dir='/tmp')
# model.get_component_name()
# del model

# echo $CR_PAT | docker login ghcr.io -u sverhoeven --password-stdin
# docker push ghcr.io/ewatercycle/teaching:latest

FROM jupyter/minimal-notebook:latest

LABEL org.opencontainers.image.source=https://github.com/eWaterCycle/teaching
LABEL org.opencontainers.image.description="eWatercycle teaching"
LABEL org.opencontainers.image.licenses=Apache-2.0

USER root

# install apptainer or podman
# RUN apt update && apt install -y podman && ln -s /usr/bin/podman /usr/bin/docker
RUN apt update && apt install -y libfuse2 uidmap squashfs-tools squashfuse fuse2fs fuse-overlayfs fakeroot 
RUN wget https://github.com/apptainer/apptainer/releases/download/v1.2.5/apptainer_1.2.5_amd64.deb && \
dpkg -i apptainer_1.2.5_amd64.deb && rm apptainer_1.2.5_amd64.deb

# install ngshare stuff
COPY nbgrader_config.py /etc/jupyter/nbgrader_config.py
# need fork of ngshare to work with latest jupyter
RUN mamba install -y nbgrader && pip install git+https://github.com/lauri3k/ngshare_exchange.git#master

# install ewatercycle + its deps
RUN mamba install -y esmvaltool-python && pip install ewatercycle-leakybucket
# TODO other models and utils

# TODO install extra jupyter extensions like lsp

USER 1000
