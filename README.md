# A demo project for Rocky Linux 9 controlled via Ansible.

## Requirements.

- [uv](https://docs.astral.sh/uv/getting-started/installation/) to manage Python dependencies and install Ansible locally into a virtual environment.
- [Docker](https://docs.docker.com/get-docker/) to run the Rocky Linux container.

## Getting systemd to work in the container.

The regular Rocky Linux 9 images don't work with systemd. As such I am using an image built by [eniocarboni](https://github.com/eniocarboni/docker-rockylinux-systemd) which has systemd working.
This image hasn't been updated in a while, so if you were using it as anything other than a proof of concept, you would want to use the [Dockerfile](https://github.com/eniocarboni/docker-rockylinux-systemd/blob/main/Dockerfile) they have generated as a reference to build your own image based off [Rocky Linux 9.5](https://rockylinux.org/news/rocky-linux-9-5-ga-release), or whatever the most recent version is.

Four main things are required to get systemd to work in the container.

1. The container must be run with the `--privileged` flag.
2. The system cgroups must be mounted as read-write.
3. The system must be told to use the cgroup namespace of the host.
3. The container must be started with the command `/usr/sbin/init` (which is systemd).

This would normally be done with `docker run --detach --privileged --volume=/sys/fs/cgroup:/sys/fs/cgroup:rw --cgroupns=host eniocarboni/docker-rockylinux-systemd:latest`, but it is also translated into the [docker-compose.yaml](docker-compose.yaml) file.

## Environment setup.

This sets up this project's Python environment and activates it. This makes sure it uses the correct (modern) Ansible versions.

```
uv sync
. .venv/bin/activate
```

## Starting the Rocky Linux container.

```
docker compose up -d
```

## Getting a shell in the Rocky Linux container.

```
docker exec -it rocky9 /bin/bash
```

## Running Ansible playbooks against the Rocky Linux container.

Note because the [inventory file](inventory/docker.yaml) just calls to get the inventory from the local
Docker daemon, and the [site file](ansible/site.yml) has `hosts: all`, this will run the playbook
against any running docker container you may have. If your system is running things other than this,
this may not be desirable.

```
ansible-playbook -i ansible/inventory/docker.yml ansible/site.yml
```

## Testing the DNS server.

```
dig @127.0.0.1 -p 5353 example.localnet.
```

## Stopping the Rocky Linux container.

Note the docker container has no volumes, so any changes (including applying the ansible playbook) will be lost when you stop the container.

```
docker compose down
```
