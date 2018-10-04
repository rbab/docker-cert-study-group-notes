# Volumes

- Know types of volume storage: block (EBS), file (EFS), object (S3)

How Docker stores data

Two types of data: 

- Persistent (case info, financials, audit logs, application log data)
- Non-persistent (session data)

Every single container gets their own non-persistent data storage by default.
- Referred to as local storage, graphdriver storage, snapshotter storage
- All of the container's files and filesystem go here. 
- Tied to the container's lifecycle - gets created when the container is created and deleted when the container gets deleted

Exists under `/var/lib/docker/<storage-driver>/` for linux. `C:\ProgramData\Docker\windowsfilter\` for windows


Note: For Linux, make sure you match the right storage driver with the version of Linux on your Docker host. Ex:
-Red Hat Enterprise - overlay2 for monder versions and devicemapper fo rolder versions
-Ubuntu - overlay2 or aufs
-SuSe Linux - btrfs driver

overlay2 is increasing in popularity and may become the prefered driver in the future



If you want to store persistent data, you need a volume. 

High level overview of the process:
1. Create a volume
2. Create a container
3. Mount the volume to a directory in the container's file system
4. Anything written to that director is written to the volume
5. If you delete the container, the volume and its data will still exist


Command to create a volume:
`docker volume create testvolume1`

Creates using default built in driver. Specify a driver using `-d` flag.

All volumes created with the local driver get their own directory under `/var/lib/docker/volumes`

You can use thrid party drivers as plugins to be used as backend storage for the volume. There are 25+ volume plugins which cover:
- Block Storage - high performance and good for small-block random acces workloads (Ex. Amazon EBS, OpenStack Block Storage service (cinder)
- File Storage - good for high performance workloads and includes NFS and SMB protocols (Ex. Azure Files storage, Amazon EFS) 
- Object Storage - long term storage that's typically low performance of large data blobs that don't change frequently (Ex. Amazon S3, Ceph, Mino)

See docker volumes using `docker volume ls`

Further inspect it with `docker volume inspect`

Once a volume is created, you can mount it on a container using the `--mount` flag on `docker container run`


Example of creating a new container and mounting a volume:
`docker container run -dit --name voltainer --mount source=bizvol, target=/vol alpine`

Docker will create a volume with the name you specify if it doesn't already exist. Otherwise, it uses the existing volume with the same name.

Since the volume is mounted into the container, you can exec into the container and write data (in the target folder for the container). You will then be able to see the the data can be retrieved from the container and persists in the volume, even after the container is deleted.

`docker container exec -it voltainer sh`

`$: echo "test test test" > /vol/testfile1`

`$: exit`

`docker container rm voltainer -f`

The container has been removed but you can still see that the volume exists and the file persists in the volume with the following:

`ls -l /var/lib/docker/volumes/bizvol/_data/`

You can now create another service or container, mount it to the same volume, and see the data within.

` docker service create --name hellcat --mount source=bizvol, target=\vol alpine sleep 1d`

Execing into the service and lookin in the vol directory will show the test file we wrote earlier.

# Sharing storage

A volume can be shared by multiple containers, meaning that both containers can write and read from the same data source. This opens the possiblity for data corruption so you must ensure that your code is written in a proper asynchronous fashion. 

# Command Recap

`docker volume create` to create a volume with `-d` flag to specify driver

`docker volume ls` will list all the volumes

`docker volume inspect` shows detailed volume info

`docker volume prune` will delete all volumes not in use

`docker volume rm` allows you to specify a specific volume to delete




# Deleting a Docker Volume

- `docker volume prune` - delete all volumes that are not mounted to a container
- `docker volume rm` - allows you to specify a volume to delete

You cannot delete a container that is actively in use. If you try, you'll get an error: `Error response from daemon: unable to remove volume: volume is in use`



You can also deploy volumes via Dockerfiles using `VOLUME` instruction





Volumes
- Decoupled from containers (mananged separately)
- Not tied to container lifecycle
- Can delete container without deleting a volume and vice versa




