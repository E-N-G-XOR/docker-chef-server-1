# Docker Chef Server

## Disclaimer

This project is intended for development/testing purposes where the latest
Chef Server is required.  The version will following upstream stable where 
possible, and there will be little attempt to maintain any sort of backward 
compatibility.

Please do not rely on this image for production!


## Usage

**Docker Compose**

```
$ docker-compose up
```

**Docker Directly**

```
$ docker run -it \
    --name chef-server \
    -v /path/to/data:/var/opt \
    -p 0.0.0.0:80:80 \
    -p 0.0.0.0:443:443 \
    --privileged \
    -e PUBLIC_URL="https://mydomain.example.com" \
    -e ENABLE_CHEF_MANAGE=1 \
    trueability/chef-server
```


### Privileged Access

Running with `--privileged` is not strictly required, however without
it you will see several errors related to Chef Server attempting to set 
`sysctl` parameters.  These can safely be ignore, though you will need to 
ensure that the host system has sufficient `kernel.shmmax` and 
`kernel.shmall` values.


### Exposed Ports

Both `http/80` and `https/443` are exposed.  Note that if using the Chef 
Manage interface, HTTPS is strictly enforced.


### Volumes

The volume `/var/opt` is strictly required, and is where all Chef Server data 
is stored for persistence.


### Environment Variables

 * `PUBLIC_URL` *(url)*: Tells Chef Server what the publicly accessible 
 URL is.  Default: `https://127.0.0.1/`.
 
 * `ENABLE_CHEF_MANAGE` *(boolean: 1/0)*: Whether or not to include the Chef 
 Management Interface.  If enable, the Chef Manage plugin will be installed 
 on startup everytime a new container is created.  The reason this is not 
 baked into the image is because it adds an additional `1.1G` to the image 
 size.  Default: `0` (not enabled).


## Startup Wait Lock

On startup `reconfigure` is always run, unless the container id hasn't change 
(determined by `/var/opt/.container_id`).  During reconfiguration, a lock 
file is created at `/var/opt/opscode/.reconfigure.lock`, and removed once 
complete.

For scripting, it is important to ensure that startup and reconfiguration is 
complete before attempting to access the server.  This can be handled easily 
with the included wait script:

```
$ docker exec -it [CONTAINER_ID] chef-server-wait-lock
```

## Working With Chef Server

The server can be accessed via `docker exec` in order to administer Chef 
Server, the same as you would anywhere else.

```
$ docker exec -it [CONTAINER_ID] chef-server-ctl ...
```

Alternatively, you can drop into a BASH shell:

```
$ docker exec -it [CONTAINER_ID] /bin/bash

XXXXXXXX $ chef-server-ctl ...
```

## Acknowledgements

This project is largely based off the initial work of Maciej Pasternacki and
his [3ofcoins/chef-server](https://github.com/3ofcoins/docker-chef-server/)
docker image.
