FROM ubuntu:22.04
ENV DEBIAN_FRONTEND=noninteractive
ENV NODE_VERSION=16.17.1

RUN apt-get update \
 && apt-get -y dist-upgrade \
 && apt-get -y install git sudo curl

# cache some of the bigger installs
RUN apt-get -y install python3-pip npm
RUN python3 -m pip install --upgrade pip jupyterhub

# Install nodejs v16 (not available yet on apt).
ENV NVM_DIR=/root/.nvm
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
RUN . "$NVM_DIR/nvm.sh" && nvm install ${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm use v${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm alias default v${NODE_VERSION}
ENV PATH="/root/.nvm/versions/node/v${NODE_VERSION}/bin/:${PATH}"

RUN git clone https://github.com/jupyter/nbgrader /srv/nbgrader/nbgrader
WORKDIR /srv/nbgrader/nbgrader/demos
CMD ./restart_demo.sh demo_multiple_classes
