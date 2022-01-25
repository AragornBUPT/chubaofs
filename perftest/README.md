# CubeFS Perftest scripts

## 1. ENVIRONMENT

### Prepare Perftest Cluster

Prepare a deployed CubeFS cluster and a volume for test.

### Prepare Test-Client Cluster

For ease of operation, salt-stack should be installed on each test-client. The steps are as follows:

#### 1. Configure salt-master

Select one machine as salt-master.

```shell
$ yum install -y salt-master
$ echo "nodegroups:" >> /etc/salt/master.d/chubaofs.conf
$ echo "  cfs-perftest-client: '*perftest-client*'" >> /etc/salt/master.d/chubaofs.conf
$ salt-master start -d
```

#### 2. Configure salt-minion

Copy install/init-minion.sh to all minion nodes. And execute the following commands:

```shell
# install killall
$ yum -y install psmisc

$ export SALT_MASTER=xxx.xx.xx.xx
$ install/init-minion.sh
```

Confirm all salt-minion nodes are administered by master. And execute commands on master:

```shell
$ salt -N 'cfs-perftest-client' test.ping

# If ping result is empty, check if exist Unaccepted Keys.
$ salt-key
 
# If Unaccepted Keys exist, accept the connection requests.
$ salt-key -A
 
# If ping result is still empty，please check the log on master and minion separately. 
# Master log path: /var/log/salt/master
# Minion log path: /var/log/salt/minion 
```

- The value of Variable SALT_MASTER is the ip of salt-master
- Value order of salt-minion id: /etc/salt/minion > /etc/salt/minion_id > hostname

#### 3. Install Perftest Tools

Copy directory /install to /srv/salt/perftest/. The final directory structure is as follows:

```
srv
├─script
|   └perftest.sh
├─salt
|  ├─perftest
|  |    ├─script
|  |    |   ├─fiotest.sh
|  |    |   └mdtest.sh
|  |    ├─pkg
|  |    |  ├─mdtest-bin.tgz
|  |    |  └openmpi-1.10.7.tgz
|  |    ├─install
|  |    |    ├─install-mdtest.sh
|  |    |    ├─install-mpi.sh
|  |    |    ├─install-perftest.sh
|  |    |    └install-rsh.sh
|  |    ├─hosts
|  |    |   ├─hosts1.txt
|  |    |   ├─hosts16.txt
|  |    |   ├─hosts4.txt
|  |    |   └hosts64.txt
|  |    ├─conf
|  |    |  └client.json
|  |    ├─bin
|  |    |  └cfs-client
```

- File install/install-rsh.sh is used to provide mpi communication. If needed, please change the ips to the ips of
  salt-minions

```shell
$ mkdir -p /srv /srv/salt/perftest/script/
$ cp -rf perftest.sh /srv/script/
$ cp -rf fiotest.sh /srv/salt/perftest/script/
$ cp -rf mdtest.sh /srv/salt/perftest/script/
$ cp -rf install/ /srv/salt/perftest/install
$ cp -rf pkg/ /srv/salt/perftest/pkg
$ bash /srv/salt/perftest/install/install-perftest.sh
```

### Mount CubeFS test volume

- Place cfs-client to /srv/salt/perftest/bin/cfs-client
- Place client.json to /srv/salt/perftest/conf/client.json

```shell
$ cd /srv
$ script/perftest.sh install
$ script/perftest.sh mount_cfs
```

## 2. TEST

### Perftest with salt-stack

```shell
# Before use fio script, mpi_host should be configured.
$ cd /srv ; script/perftest.sh fio_test

# Execute mdtest op
$ script/perftest.sh mdtest_op

# Execute mdtest small
$ script/perftest.sh mdtest_small
```

## 3. REPORT

```shell
# Get fio test report
$ ./perftest.sh print_fio_report


# Get mdtest report
$ ./perftest.sh print_mdtest_report
```
