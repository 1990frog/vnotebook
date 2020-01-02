[TOC]
# docker
## attach：将本地标准输入，输出和错误流附加到正在运行的容器
Attach local standard input, output, and error streams to a running container
```docker
# Docker提供了attach命令来进入Docker容器。
sudo docker attach [container_id]
```
## events
Get real time events from the server
## history
Show the history of an image
## images：获取全部镜像
List images
## import
Import the contents from a tarball to create a filesystem image
## info
Display system-wide information
## inspect：返回有关Docker对象的底层信息
Return low-level information on Docker objects
```docker
docker inspect [container_id/container_name]
```

## load
Load an image from a tar archive or STDIN
## login：登录例如（dockerhub这种）
Log in to a Docker registry
## logout：登出
Log out from a Docker registry
## ps：container列表
List containers
## pull：从repository（例如dockerhub）中拉取一个image
Pull an image or a repository from a registry
## push
Push an image or a repository to a registry

## rmi：删除镜像
Remove one or more images

## run
-d 后台执行
Run a command in a new container
```config
Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

Options:
      --add-host list                  Add a custom host-to-IP mapping (host:ip)
  -a, --attach list                    Attach to STDIN, STDOUT or STDERR
      --blkio-weight uint16            Block IO (relative weight), between 10 and
                                       1000, or 0 to disable (default 0)
      --blkio-weight-device list       Block IO weight (relative device weight)
                                       (default [])
      --cap-add list                   Add Linux capabilities
      --cap-drop list                  Drop Linux capabilities
      --cgroup-parent string           Optional parent cgroup for the container
      --cidfile string                 Write the container ID to the file
      --cpu-period int                 Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int                  Limit CPU CFS (Completely Fair Scheduler) quota
      --cpu-rt-period int              Limit CPU real-time period in microseconds
      --cpu-rt-runtime int             Limit CPU real-time runtime in microseconds
  -c, --cpu-shares int                 CPU shares (relative weight)
      --cpus decimal                   Number of CPUs
      --cpuset-cpus string             CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1)
  -d, --detach                         Run container in background and print
                                       container ID
      --detach-keys string             Override the key sequence for detaching a
                                       container
      --device list                    Add a host device to the container
      --device-cgroup-rule list        Add a rule to the cgroup allowed devices list
      --device-read-bps list           Limit read rate (bytes per second) from a
                                       device (default [])
      --device-read-iops list          Limit read rate (IO per second) from a
                                       device (default [])
      --device-write-bps list          Limit write rate (bytes per second) to a
                                       device (default [])
      --device-write-iops list         Limit write rate (IO per second) to a
                                       device (default [])
      --disable-content-trust          Skip image verification (default true)
      --dns list                       Set custom DNS servers
      --dns-option list                Set DNS options
      --dns-search list                Set custom DNS search domains
      --domainname string              Container NIS domain name
      --entrypoint string              Overwrite the default ENTRYPOINT of the image
  -e, --env list                       Set environment variables
      --env-file list                  Read in a file of environment variables
      --expose list                    Expose a port or a range of ports
      --gpus gpu-request               GPU devices to add to the container ('all'
                                       to pass all GPUs)
      --group-add list                 Add additional groups to join
      --health-cmd string              Command to run to check health
      --health-interval duration       Time between running the check (ms|s|m|h)
                                       (default 0s)
      --health-retries int             Consecutive failures needed to report unhealthy
      --health-start-period duration   Start period for the container to
                                       initialize before starting health-retries
                                       countdown (ms|s|m|h) (default 0s)
      --health-timeout duration        Maximum time to allow one check to run
                                       (ms|s|m|h) (default 0s)
      --help                           Print usage
  -h, --hostname string                Container host name
      --init                           Run an init inside the container that
                                       forwards signals and reaps processes
  -i, --interactive                    Keep STDIN open even if not attached
      --ip string                      IPv4 address (e.g., 172.30.100.104)
      --ip6 string                     IPv6 address (e.g., 2001:db8::33)
      --ipc string                     IPC mode to use
      --isolation string               Container isolation technology
      --kernel-memory bytes            Kernel memory limit
  -l, --label list                     Set meta data on a container
      --label-file list                Read in a line delimited file of labels
      --link list                      Add link to another container
      --link-local-ip list             Container IPv4/IPv6 link-local addresses
      --log-driver string              Logging driver for the container
      --log-opt list                   Log driver options
      --mac-address string             Container MAC address (e.g., 92:d0:c6:0a:29:33)
  -m, --memory bytes                   Memory limit
      --memory-reservation bytes       Memory soft limit
      --memory-swap bytes              Swap limit equal to memory plus swap: '-1'
                                       to enable unlimited swap
      --memory-swappiness int          Tune container memory swappiness (0 to 100)
                                       (default -1)
      --mount mount                    Attach a filesystem mount to the container
      --name string                    Assign a name to the container
      --network network                Connect a container to a network
      --network-alias list             Add network-scoped alias for the container
      --no-healthcheck                 Disable any container-specified HEALTHCHECK
      --oom-kill-disable               Disable OOM Killer
      --oom-score-adj int              Tune host's OOM preferences (-1000 to 1000)
      --pid string                     PID namespace to use
      --pids-limit int                 Tune container pids limit (set -1 for unlimited)
      --privileged                     Give extended privileges to this container
  -p, --publish list                   Publish a container's port(s) to the host
  -P, --publish-all                    Publish all exposed ports to random ports
      --read-only                      Mount the container's root filesystem as
                                       read only
      --restart string                 Restart policy to apply when a container
                                       exits (default "no")
      --rm                             Automatically remove the container when it exits
      --runtime string                 Runtime to use for this container
      --security-opt list              Security Options
      --shm-size bytes                 Size of /dev/shm
      --sig-proxy                      Proxy received signals to the process
                                       (default true)
      --stop-signal string             Signal to stop a container (default "SIGTERM")
      --stop-timeout int               Timeout (in seconds) to stop a container
      --storage-opt list               Storage driver options for the container
      --sysctl map                     Sysctl options (default map[])
      --tmpfs list                     Mount a tmpfs directory
  -t, --tty                            Allocate a pseudo-TTY
      --ulimit ulimit                  Ulimit options (default [])
  -u, --user string                    Username or UID (format:
                                       <name|uid>[:<group|gid>])
      --userns string                  User namespace to use
      --uts string                     UTS namespace to use
  -v, --volume list                    Bind mount a volume
      --volume-driver string           Optional volume driver for the container
      --volumes-from list              Mount volumes from the specified container(s)
  -w, --workdir string                 Working directory inside the container
```
## save
Save one or more images to a tar archive (streamed to STDOUT by default)
## search：DockerHub查询镜像
Search the Docker Hub for images
## tag
Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
Update configuration of one or more containers
## version
Show the Docker version information
# builder
## build：通过dockerfile构建一个image
Build an image from a Dockerfile
```config
Usage:  docker build [OPTIONS] PATH | URL | -

Build an image from a Dockerfile

Options:
      --add-host list           Add a custom host-to-IP mapping (host:ip)
      --build-arg list          Set build-time variables
      --cache-from strings      Images to consider as cache sources
      --cgroup-parent string    Optional parent cgroup for the container
      --compress                Compress the build context using gzip
      --cpu-period int          Limit the CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int           Limit the CPU CFS (Completely Fair Scheduler) quota
  -c, --cpu-shares int          CPU shares (relative weight)
      --cpuset-cpus string      CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string      MEMs in which to allow execution (0-3, 0,1)
      --disable-content-trust   Skip image verification (default true)
  -f, --file string             Name of the Dockerfile (Default is 'PATH/Dockerfile')
      --force-rm                Always remove intermediate containers
      --iidfile string          Write the image ID to the file
      --isolation string        Container isolation technology
      --label list              Set metadata for an image
  -m, --memory bytes            Memory limit
      --memory-swap bytes       Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --network string          Set the networking mode for the RUN instructions during build (default "default")
      --no-cache                Do not use cache when building the image
      --pull                    Always attempt to pull a newer version of the image
  -q, --quiet                   Suppress the build output and print image ID on success
      --rm                      Remove intermediate containers after a successful build (default true)
      --security-opt strings    Security options
      --shm-size bytes          Size of /dev/shm
  -t, --tag list                Name and optionally a tag in the 'name:tag' format
      --target string           Set the target build stage to build.
      --ulimit ulimit           Ulimit options (default [])
```
## prune：删除构建缓存
```docker
# 删除 所有未被 tag 标记和未被容器使用的镜像：
docker image prune
# 删除 所有未被容器使用的镜像：
docker image prune -a
# 删除 所有停止运行的容器：
docker container prune
# 删除 所有未被挂载的卷：
docker volume prune
# 删除 所有网络：
docker network prune
# 删除 docker 所有资源：
docker system prune
```
# config
## create：Create a config from a file or STDIN
## inspect：Display detailed information on one or more configs
## ls：List configs
## rm：Remove one or more configs
# container
## commit：通过变化后的container创建一个新的image
Create a new image from a container's changes
```docker
Usage:  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

Options:
  -a, --author string    Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
  -c, --change list      Apply Dockerfile instruction to the created image
  -m, --message string   Commit message
  -p, --pause            Pause container during commit (default true)
```
## cp：在container与host之间拷贝文件
Copy files/folders between a container and the local filesystem
## create：创建一个新的container
Create a new container
## diff：Inspect changes to files or directories on a container's filesystem
Inspect changes to files or directories on a container's filesystem# context
## exec：在一个运行中的container中执行一个命令行
Run a command in a running container
```config
Usage:  docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

Run a command in a running container

Options:
  -d, --detach               Detached mode: run command in the background
      --detach-keys string   Override the key sequence for detaching a container
  -e, --env list             Set environment variables
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command
  -t, --tty                  Allocate a pseudo-TTY
  -u, --user string          Username or UID (format: <name|uid>[:<group|gid>])
  -w, --workdir string       Working directory inside the container
```
## export：Export a container's filesystem as a tar archive
Export a container's filesystem as a tar archive
## inspect：Display detailed information on one or more containers
## kill：干掉一个或多个运行中的container
Kill one or more running containers
## logs：获取container的日志
Fetch the logs of a container
## ls：List containers
## pause：Pause all processes within one or more containers
## port：获取container端口
List port mappings or a specific mapping for the container
```docker
docker port [container_id/container_name]
1521/tcp -> 0.0.0.0:49161
22/tcp -> 0.0.0.0:49160
```
## prune：Remove all stopped containers
## rename：更改container名字
Rename a container
## restart：重启container
Restart one or more containers
## rm：删除container
Remove one or more containers
## run：Run a command in a new container
## start：启动一个或多个container
Start one or more stopped containers
## stats：Display a live stream of container(s) resource usage statistics
Display a live stream of container(s) resource usage statistics
## stop：停止一个或多个container
Stop one or more running containers
## top：Display the running processes of a container
Display the running processes of a container
## unpause：Unpause all processes within one or more containers
Unpause all processes within one or more containers
## update：Update configuration of one or more containers
## wait
Block until one or more containers stop, then print their exit codes
# engine
## activate：Activate Enterprise Edition
## check：Check for available engine updates
## update：Update a local engine
# image
## build       Build an image from a Dockerfile
## history     Show the history of an image
## import      Import the contents from a tarball to create a filesystem image
## inspect     Display detailed information on one or more images
## load        Load an image from a tar archive or STDIN
## ls          List images
## prune       Remove unused images
## pull        Pull an image or a repository from a registry
## push        Push an image or a repository to a registry
## rm          Remove one or more images
## save        Save one or more images to a tar archive (streamed to STDOUT by default)
## tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
# network
## connect     Connect a container to a network
## create      Create a network
## disconnect  Disconnect a container from a network
## inspect     Display detailed information on one or more networks
## ls          List networks
## prune       Remove all unused networks
## rm          Remove one or more networks
# node
## demote      Demote one or more nodes from manager in the swarm
## inspect     Display detailed information on one or more nodes
## ls          List nodes in the swarm
## promote     Promote one or more nodes to manager in the swarm
## ps          List tasks running on one or more nodes, defaults to current node
## rm          Remove one or more nodes from the swarm
## update      Update a node
# plugin
## create      Create a plugin from a rootfs and configuration. Plugin data directory must contain config.json and rootfs directory.
## disable     Disable a plugin
## enable      Enable a plugin
## inspect     Display detailed information on one or more plugins
## install     Install a plugin
## ls          List plugins
## push        Push a plugin to a registry
## rm          Remove one or more plugins
## set         Change settings for a plugin
## upgrade     Upgrade an existing plugin
# secret
## create      Create a secret from a file or STDIN as content
## inspect     Display detailed information on one or more secrets
## ls          List secrets
## rm          Remove one or more secrets
# service
## create      Create a new service
## inspect     Display detailed information on one or more services
## logs        Fetch the logs of a service or task
## ls          List services
## ps          List the tasks of one or more services
## rm          Remove one or more services
## rollback    Revert changes to a service's configuration
## scale       Scale one or multiple replicated services
## update      Update a service
# stack
## deploy      Deploy a new stack or update an existing stack
## ls          List stacks
## ps          List the tasks in the stack
## rm          Remove one or more stacks
## services    List the services in the stack
# swarm
## ca          Display and rotate the root CA
## init        Initialize a swarm
## join        Join a swarm as a node and/or manager
## join-token  Manage join tokens
## leave       Leave the swarm
## unlock      Unlock swarm
## unlock-key  Manage the unlock key
## update      Update the swarm
# system
## df          Show docker disk usage
## events      Get real time events from the server
## info        Display system-wide information
## prune       Remove unused data
# trust
```
Management Commands:
  key         Manage keys for signing Docker images
  signer      Manage entities who can sign Docker images

Commands:
  inspect     Return low-level information about keys and signatures
  revoke      Remove trust for an image
  sign        Sign an image
```
# volume
## create      Create a volume
## inspect     Display detailed information on one or more volumes
## ls          List volumes
## prune       Remove all unused local volumes
## rm          Remove one or more volumes
