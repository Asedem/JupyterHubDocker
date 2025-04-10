# JupyterHub Docker

This repository is intended to demonstrate how to run JupyterHub in a Docker container with persistent data storage.  
The original repository can be found here: https://github.com/jupyterhub/the-littlest-jupyterhub  
The original setup guide is available here: https://tljh.jupyter.org/en/latest/contributing/dev-setup.html  
The link to the collaborative extension is: https://github.com/jupyterlab/jupyter-collaboration

Create directories for the data to be stored in volumes:

```bash
mkdir -p ~/tljh-persistent-data
mkdir -p ~/tljh-user-home
```

Build the Dockerfile into an image:

```bash
docker build -t tljh-systemd . -f integration-tests/Dockerfile
```

Start the image as a container (the port can be chosen freely):

```bash
docker run \
  --privileged \
  --detach \
  --name=tljh-dev \
  --publish 80:80 \
  --env TLJH_BOOTSTRAP_DEV=yes \
  --env TLJH_BOOTSTRAP_PIP_SPEC=/srv/src \
  --env PATH=/opt/tljh/hub/bin:${PATH} \
  --mount type=bind,source="$(pwd)",target=/srv/src \
  --mount type=bind,source="$HOME/tljh-persistent-data",target=/opt/tljh \
  --mount type=bind,source="$HOME/tljh-user-home",target=/home \
  tljh-systemd
```

Enter the container:

```bash
docker exec -it tljh-dev /bin/bash
```

Set up JupyterHub (you can also use admin:<password> if you'd like to set the password at this point).
Otherwise, the password will be set the first time the user logs in.
If data already exists, and you're creating a new container, you can omit the --admin parameter entirely.

```bash
python3 /srv/src/bootstrap/bootstrap.py --admin admin
```

How the data is mounted:

| What             | Container Path | Host Path              |
|------------------|----------------|------------------------|
| TLJH config/envs | /opt/tljh      | ~/tljh-persistent-data |
| User notebooks   | /home          | ~/tljh-user-home       |