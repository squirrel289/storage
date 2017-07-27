## Rancher CIFS v3.0 Volume Plugin Driver

### CIFS plugin driver is a bash script and invocation commands are:

Create:
```
driver  create  json_options
```

Delete:
```
driver  delete  json_options
```

Mount:
```
driver  mount  mountpoint  json_options
```

Unmount:
```
driver  unmount  mountpoint
```

Currently we support 2 major use cases

### 1. User provides existing remote share in the form of //host/export

##### Create command
driver does nothing

```
./cifs create '{"host":"cifs-server","export":"/share/path"}'

stdout output: {"status":"Success","message":""}
```

##### Delete command
driver does nothing

```
./cifs delete '{"host":"cifs-server","export":"/share/path"}'

stdout output: {"status":"Success","message":""}
```

#### Mount command
driver mounts CIFS using pre-existing remote share provided by user

```
./cifs mount /home/ubuntu/cifsMnt '{"host":"cifs-server","export":"/share/path","mntOptions":"user=username,password=pwd,uid=1100,gid=1100,forceuid,forcegid"}'

mntOptions key-value pair is optional for mount, but host and export are the must

stdout output: {"status":"Success","message":""}
```

The result is that //cifs-server/share/path is mounted at /home/ubuntu/cifsMnt


#### Unmount command
driver unmount CIFS file system

```
./cifs unmount /home/ubuntu/cifsMnt

stdout output: {"status":"Success","message":""}
```

### 2. User does not provide remote share

Rancher have default host and export environment variables set on the host identifying default CIFS remote share.
Driver will use them to create a directory for each volume during create command and delete it at delete command.
For instance:

```
export HOST=cifs-server
export EXPORT=/share
```

##### Create command
user supplies volume name, driver creates a directory under default remote share using volume name.
The output is the status and newly created directory name

```
./cifs create '{"mntDest":"/home/ubuntu/mnt","name":"vol1"}'

name represents volume name and mntDest is the mount point this remote volume will be mounted at.

In this example, //cifs-server/share/vol1 (//HOST/EXPORT/name) will be mounted at /home/ubuntu/mnt (mntDest)

mntDest is needed for driver to temporarily mount remote share in order to create a subdirectory named "vol1"

stdout output: {"status": "Success”,"options":{"created":true,"name":"vol1”}}

the options map from the output will be passed in as part of json_input for delete command
```

##### Delete command
driver deletes created directory at create phase

```
./cifs delete '{"mntDest":"/home/ubuntu/mnt", "created":"true", "name":"vol1"}'

the options map from create command output is passed in to delete command as part of json_input

stdout output: {"status":"Success","message":""}
```

#### Mount command
driver mounts CIFS src share using //HOST/EXPORT/name created at create command phase

```
./cifs mount /home/ubuntu/cifsMnt '{"name":"vol1","mntOptions":"user=username,password=pwd"}''
mntOptions key-value pair is optional for mount, but name is a must

stdout output: {"status":"Success","message":""}
```

The result is that //cifs-server/share/vol1 is mounted at /home/ubuntu/cifsMnt

#### Unmount command
driver unmount CIFS remote share

```
./cifs unmount /home/ubuntu/cifsMnt

stdout output: {"status":"Success","message":""}
```