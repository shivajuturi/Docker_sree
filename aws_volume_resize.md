

# Aws Disk Resize (1 ephemeral   ,2 EBSVolume)
- 1 Ephemeral/Root

#### ephemeral disk Resize
- increase root volume in aws via console
- - Check the volume status

In-use - optimizing   status to  In-use (in front end )
- check blocks disk level
```
ubuntu@ip-10-0-1-52:~$ lsblk
NAME     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0      7:0    0 26.6M  1 loop /snap/amazon-ssm-agent/5163
loop1      7:1    0 55.5M  1 loop /snap/core18/2344
loop2      7:2    0 61.9M  1 loop /snap/core20/1405
loop3      7:3    0 79.9M  1 loop /snap/lxd/22923
loop4      7:4    0 43.6M  1 loop /snap/snapd/15177
xvda     202:0    0   12G  0 disk
├─xvda1  202:1    0  7.9G  0 part /
├─xvda14 202:14   0    4M  0 part
└─xvda15 202:15   0  106M  0 part /boot/efi
ubuntu@ip-10-0-1-52:~$ ^C
```

- volume is increased  volume is not effected
```
ubuntu@ip-10-0-1-52:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       7.6G  6.5G  1.2G  86% /
tmpfs           484M     0  484M   0% /dev/shm
tmpfs           194M  828K  193M   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/xvda15     105M  5.3M  100M   5% /boot/efi
tmpfs            97M  4.0K   97M   1% /run/user/1000

```
- increse  (ephemeral/root) filesystem
```
ubuntu@ip-10-0-1-52:~$ sudo growpart /dev/xvda 1
CHANGED: partition=1 start=227328 old: size=16549855 end=16777183 new: size=24938463 end=25165791
ubuntu@ip-10-0-1-52:~$ sudo resize2fs /dev/xvda1
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/xvda1 is mounted on /; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 2
The filesystem on /dev/xvda1 is now 3117307 (4k) blocks long.

ubuntu@ip-10-0-1-52:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        12G  6.5G  5.0G  57% /
tmpfs           484M     0  484M   0% /dev/shm
tmpfs           194M  828K  193M   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/xvda15     105M  5.3M  100M   5% /boot/efi
tmpfs            97M  4.0K   97M   1% /run/user/1000

```

#### EBS Volume addition and  disk Resize
## 2 EBSVolume

```

samba@LAPTOP-GQTOJO0C:~/.aws$ aws ec2 create-volume \
>     --volume-type gp2 \
>     --size 10 \
>     --availability-zone us-east-1a
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
|                                                                            CreateVolume                                                                             |
+------------------+----------------------------+------------+-------+---------------------+-------+-------------+-----------+-------------------------+--------------+
| AvailabilityZone |        CreateTime          | Encrypted  | Iops  | MultiAttachEnabled  | Size  | SnapshotId  |   State   |        VolumeId         | VolumeType   |
+------------------+----------------------------+------------+-------+---------------------+-------+-------------+-----------+-------------------------+--------------+
|  us-east-1a      |  2022-06-02T04:37:14+00:00 |  False     |  100  |  False              |  10   |             |  creating |  vol-0ca28454d03c6ebf8  |  gp2         |
+------------------+----------------------------+------------+-------+---------------------+-------+-------------+-----------+-------------------------+--------------+
samba@LAPTOP-GQTOJO0C:~/.aws$ aws ec2 describe-instances \
>     --filters Name=tag-key,Values=Name \
>     --query 'Reservations[*].Instances[*].{Instance:InstanceId,AZ:Placement.AvailabilityZone,Name:Tags[?Key==`Name`]|[0].Value}' \
--output>     --output table
-------------------------------------------------------
|                  DescribeInstances                  |
+------------+-----------------------+----------------+
|     AZ     |       Instance        |     Name       |
+------------+-----------------------+----------------+
|  us-east-1a|  i-0b01a88f79f8f1bfe  |  Myinstances1  |
+------------+-----------------------+----------------+
samba@LAPTOP-GQTOJO0C:~/.aws$ aws ec2 attach-volume --volume-id vol-0ca28454d03c6ebf8 --instance-id i-0b01a88f79f8f1bfe --device /dev/sdf
---------------------------------------------------------------------------------------------------------------
|                                                AttachVolume                                                 |
+-----------------------------------+-----------+----------------------+------------+-------------------------+
|            AttachTime             |  Device   |     InstanceId       |   State    |        VolumeId         |
+-----------------------------------+-----------+----------------------+------------+-------------------------+
|  2022-06-02T04:42:06.943000+00:00 |  /dev/sdf |  i-0b01a88f79f8f1bfe |  attaching |  vol-0ca28454d03c6ebf8  |
+-----------------------------------+-----------+----------------------+------------+-------------------------+
samba@LAPTOP-GQTOJO0C:~/.aws$ aws ec2 describe-instances \
 --filte>     --filters Name=tag-key,Values=Name \
>     --query 'Reservations[*].Instances[*].{Instance:InstanceId,AZ:Placement.AvailabilityZone,Name:Tags[?Key==`Name`]|[0].Value}' \
>     --output table
-------------------------------------------------------
|                  DescribeInstances                  |
+------------+-----------------------+----------------+
|     AZ     |       Instance        |     Name       |
+------------+-----------------------+----------------+
|  us-east-1a|  i-0b01a88f79f8f1bfe  |  Myinstances1  |
+------------+-----------------------+----------------+
samba@LAPTOP-GQTOJO0C:~/.aws$


```

```
[ec2-user@ip-172-31-18-30 ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
└─xvda1 202:1    0   8G  0 part /
xvdf    202:80   0  10G  0 disk
[ec2-user@ip-172-31-18-30 ~]$ sudo mkfs -t xfs /dev/xvdf
meta-data=/dev/xvdf              isize=512    agcount=4, agsize=655360 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=0
data     =                       bsize=4096   blocks=2621440, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[ec2-user@ip-172-31-18-30 ~]$ mkdir /u01
mkdir: cannot create directory ‘/u01’: Permission denied
[ec2-user@ip-172-31-18-30 ~]$ sudo mkdir /u01
[ec2-user@ip-172-31-18-30 ~]$ sudo mount /dev/xvdf /u01
[ec2-user@ip-172-31-18-30 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        474M     0  474M   0% /dev
tmpfs           483M     0  483M   0% /dev/shm
tmpfs           483M  408K  482M   1% /run
tmpfs           483M     0  483M   0% /sys/fs/cgroup
/dev/xvda1      8.0G  1.6G  6.5G  20% /
tmpfs            97M     0   97M   0% /run/user/1000
/dev/xvdf        10G   43M   10G   1% /u01
[ec2-user@ip-172-31-18-30 ~]$

```
- Resize EBS xfs filesystem

- aws ec2 modify-volume --size 15 --volume-id vol-0ca28454d03c6ebf8

```
samba@LAPTOP-GQTOJO0C:~/.aws$ aws ec2 modify-volume --size 15 --volume-id vol-0ca28454d03c6ebf8
---------------------------------------------------------------
|                        ModifyVolume                         |
+-------------------------------------------------------------+
||                    VolumeModification                     ||
|+-----------------------------+-----------------------------+|
||  ModificationState          |  modifying                  ||
||  OriginalIops               |  100                        ||
||  OriginalMultiAttachEnabled |  False                      ||
||  OriginalSize               |  10                         ||
||  OriginalVolumeType         |  gp2                        ||
||  Progress                   |  0                          ||
||  StartTime                  |  2022-06-02T04:55:49+00:00  ||
||  TargetIops                 |  100                        ||
||  TargetMultiAttachEnabled   |  False                      ||
||  TargetSize                 |  15                         ||
||  TargetVolumeType           |  gp2                        ||
||  VolumeId                   |  vol-0ca28454d03c6ebf8      ||
|+-----------------------------+-----------------------------+|
samba@LAPTOP-GQTOJO0C:~/.aws$

[ec2-user@ip-172-31-18-30 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        474M     0  474M   0% /dev
tmpfs           483M     0  483M   0% /dev/shm
tmpfs           483M  408K  482M   1% /run
tmpfs           483M     0  483M   0% /sys/fs/cgroup
/dev/xvda1      8.0G  1.6G  6.5G  20% /
tmpfs            97M     0   97M   0% /run/user/1000
/dev/xvdf        10G   43M   10G   1% /u01
[ec2-user@ip-172-31-18-30 ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
└─xvda1 202:1    0   8G  0 part /
xvdf    202:80   0  15G  0 disk /u01
[ec2-user@ip-172-31-18-30 ~]$
```

- Check the volume status

In-use - optimizing   status to  In-use (in front end )


```
samba@LAPTOP-GQTOJO0C:~/.aws$ aws ec2 describe-instances      --instance-ids i-0b01a88f79f8f1bfe
------------------------------------------------------------------------------
|                              DescribeInstances                             |
+----------------------------------------------------------------------------+
||                               Reservations                               ||
|+------------------------------+-------------------------------------------+|
||  OwnerId                     |  406553770210                             ||
||  ReservationId               |  r-0207ff4c16eb49fd6                      ||
|+------------------------------+-------------------------------------------+|
|||                                Instances                               |||
||+---------------------------+--------------------------------------------+||
|||  AmiLaunchIndex           |  0                                         |||
|||  Architecture             |  x86_64                                    |||
|||  ClientToken              |                                            |||
|||  EbsOptimized             |  False                                     |||
|||  EnaSupport               |  True                                      |||
|||  Hypervisor               |  xen                                       |||
|||  ImageId                  |  ami-0022f774911c1d690                     |||
|||  InstanceId               |  i-0b01a88f79f8f1bfe                       |||
|||  InstanceType             |  t2.micro                                  |||
|||  KeyName                  |  sambasiva                                 |||
|||  LaunchTime               |  2022-06-02T04:21:24+00:00                 |||
|||  PlatformDetails          |  Linux/UNIX                                |||
|||  PrivateDnsName           |  ip-172-31-18-30.ec2.internal              |||
|||  PrivateIpAddress         |  172.31.18.30                              |||
|||  PublicDnsName            |  ec2-54-221-13-38.compute-1.amazonaws.com  |||
|||  PublicIpAddress          |  54.221.13.38                              |||
|||  RootDeviceName           |  /dev/xvda                                 |||
|||  RootDeviceType           |  ebs                                       |||
|||  SourceDestCheck          |  True                                      |||
|||  StateTransitionReason    |                                            |||
|||  SubnetId                 |  subnet-095a465847a6224bc                  |||
|||  UsageOperation           |  RunInstances                              |||
|||  UsageOperationUpdateTime |  2022-06-02T04:21:24+00:00                 |||
|||  VirtualizationType       |  hvm                                       |||
|||  VpcId                    |  vpc-02bb8c71f29df1e0c                     |||
||+---------------------------+--------------------------------------------+||
||||                          BlockDeviceMappings                         ||||
|||+-----------------------------------+----------------------------------+|||
||||  DeviceName                       |  /dev/xvda                       ||||
|||+-----------------------------------+----------------------------------+|||
|||||                                 Ebs                                |||||
||||+-----------------------------+--------------------------------------+||||
|||||  AttachTime                 |  2022-06-02T04:21:25+00:00           |||||
|||||  DeleteOnTermination        |  True                                |||||
|||||  Status                     |  attached                            |||||
|||||  VolumeId                   |  vol-0d4e5016263c1d553               |||||
||||+-----------------------------+--------------------------------------+||||
||||                          BlockDeviceMappings                         ||||
|||+-------------------------------------+--------------------------------+|||
||||  DeviceName                         |  /dev/sdf                      ||||
|||+-------------------------------------+--------------------------------+|||
|||||                                 Ebs                                |||||
||||+-----------------------------+--------------------------------------+||||
|||||  AttachTime                 |  2022-06-02T04:42:06+00:00           |||||
|||||  DeleteOnTermination        |  False                               |||||
|||||  Status                     |  attached                            |||||
|||||  VolumeId                   |  vol-0ca28454d03c6ebf8               |||||
||||+-----------------------------+--------------------------------------+||||
||||                   CapacityReservationSpecification                   ||||
|||+--------------------------------------------------------+-------------+|||
||||  CapacityReservationPreference                         |  open       ||||
|||+--------------------------------------------------------+-------------+|||
||||                              CpuOptions                              ||||
|||+------------------------------------------------------+---------------+|||
||||  CoreCount                                           |  1            ||||
||||  ThreadsPerCore                                      |  1            ||||
|||+------------------------------------------------------+---------------+|||
||||                            EnclaveOptions                            ||||
|||+--------------------------------------+-------------------------------+|||
||||  Enabled                             |  False                        ||||
|||+--------------------------------------+-------------------------------+|||
||||                          HibernationOptions                          ||||
|||+------------------------------------------+---------------------------+|||
||||  Configured                              |  False                    ||||
|||+------------------------------------------+---------------------------+|||
||||                          MaintenanceOptions                          ||||
|||+-----------------------------------------+----------------------------+|||
||||  AutoRecovery                           |  default                   ||||
|||+-----------------------------------------+----------------------------+|||
||||                            MetadataOptions                           ||||
|||+------------------------------------------------+---------------------+|||
||||  HttpEndpoint                                  |  enabled            ||||
||||  HttpProtocolIpv6                              |  disabled           ||||
||||  HttpPutResponseHopLimit                       |  1                  ||||
||||  HttpTokens                                    |  optional           ||||
||||  InstanceMetadataTags                          |  disabled           ||||
||||  State                                         |  applied            ||||
|||+------------------------------------------------+---------------------+|||
||||                              Monitoring                              ||||
|||+-----------------------------+----------------------------------------+|||
||||  State                      |  disabled                              ||||
|||+-----------------------------+----------------------------------------+|||
||||                           NetworkInterfaces                          ||||
|||+---------------------------+------------------------------------------+|||
||||  Description              |                                          ||||
||||  InterfaceType            |  interface                               ||||
||||  MacAddress               |  0a:6a:f3:ab:06:67                       ||||
||||  NetworkInterfaceId       |  eni-0c5113dd9f4f937f4                   ||||
||||  OwnerId                  |  406553770210                            ||||
||||  PrivateDnsName           |  ip-172-31-18-30.ec2.internal            ||||
||||  PrivateIpAddress         |  172.31.18.30                            ||||
||||  SourceDestCheck          |  True                                    ||||
||||  Status                   |  in-use                                  ||||
||||  SubnetId                 |  subnet-095a465847a6224bc                ||||
||||  VpcId                    |  vpc-02bb8c71f29df1e0c                   ||||
|||+---------------------------+------------------------------------------+|||
|||||                             Association                            |||||
||||+------------------+-------------------------------------------------+||||
|||||  IpOwnerId       |  amazon                                         |||||
|||||  PublicDnsName   |  ec2-54-221-13-38.compute-1.amazonaws.com       |||||
|||||  PublicIp        |  54.221.13.38                                   |||||
||||+------------------+-------------------------------------------------+||||
|||||                             Attachment                             |||||
||||+---------------------------+----------------------------------------+||||
|||||  AttachTime               |  2022-06-02T04:21:24+00:00             |||||
|||||  AttachmentId             |  eni-attach-066792080769ec73a          |||||
|||||  DeleteOnTermination      |  True                                  |||||
|||||  DeviceIndex              |  0                                     |||||
|||||  NetworkCardIndex         |  0                                     |||||
|||||  Status                   |  attached                              |||||
||||+---------------------------+----------------------------------------+||||
|||||                               Groups                               |||||
||||+-----------------------+--------------------------------------------+||||
|||||  GroupId              |  sg-07bb6edaea2dd367d                      |||||
|||||  GroupName            |  launch-wizard-1                           |||||
||||+-----------------------+--------------------------------------------+||||
|||||                         PrivateIpAddresses                         |||||
||||+-------------------------+------------------------------------------+||||
|||||  Primary                |  True                                    |||||
|||||  PrivateDnsName         |  ip-172-31-18-30.ec2.internal            |||||
|||||  PrivateIpAddress       |  172.31.18.30                            |||||
||||+-------------------------+------------------------------------------+||||
||||||                            Association                           ||||||
|||||+-----------------+------------------------------------------------+|||||
||||||  IpOwnerId      |  amazon                                        ||||||
||||||  PublicDnsName  |  ec2-54-221-13-38.compute-1.amazonaws.com      ||||||
||||||  PublicIp       |  54.221.13.38                                  ||||||
|||||+-----------------+------------------------------------------------+|||||
||||                               Placement                              ||||
|||+----------------------------------------+-----------------------------+|||
||||  AvailabilityZone                      |  us-east-1a                 ||||
||||  GroupName                             |                             ||||
||||  Tenancy                               |  default                    ||||
|||+----------------------------------------+-----------------------------+|||
||||                         PrivateDnsNameOptions                        ||||
|||+-----------------------------------------------------+----------------+|||
||||  EnableResourceNameDnsAAAARecord                    |  False         ||||
||||  EnableResourceNameDnsARecord                       |  True          ||||
||||  HostnameType                                       |  ip-name       ||||
|||+-----------------------------------------------------+----------------+|||
||||                            SecurityGroups                            ||||
|||+-----------------------+----------------------------------------------+|||
||||  GroupId              |  sg-07bb6edaea2dd367d                        ||||
||||  GroupName            |  launch-wizard-1                             ||||
|||+-----------------------+----------------------------------------------+|||
||||                                 State                                ||||
|||+----------------------------+-----------------------------------------+|||
||||  Code                      |  16                                     ||||
||||  Name                      |  running                                ||||
|||+----------------------------+-----------------------------------------+|||
||||                                 Tags                                 ||||
|||+------------------------+---------------------------------------------+|||
||||  Key                   |  Name                                       ||||
||||  Value                 |  Myinstances1                               ||||
|||+------------------------+---------------------------------------------+|||
(END)
(END)samba@LAPTOP-GQTOJO0C:~/.aws$

```

```
[ec2-user@ip-172-31-18-30 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        474M     0  474M   0% /dev
tmpfs           483M     0  483M   0% /dev/shm
tmpfs           483M  408K  482M   1% /run
tmpfs           483M     0  483M   0% /sys/fs/cgroup
/dev/xvda1      8.0G  1.6G  6.5G  20% /
tmpfs            97M     0   97M   0% /run/user/1000
/dev/xvdf        10G   43M   10G   1% /u01
[ec2-user@ip-172-31-18-30 ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
└─xvda1 202:1    0   8G  0 part /
xvdf    202:80   0  15G  0 disk /u01
[ec2-user@ip-172-31-18-30 ~]$


```
### Resize xfs file system

```
ec2-user@ip-172-31-18-30 ~]$ lsblk
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
└─xvda1 202:1    0   8G  0 part /
xvdf    202:80   0  15G  0 disk /u01
[ec2-user@ip-172-31-18-30 ~]$ sudo xfs_growfs /dev/xvdf
meta-data=/dev/xvdf              isize=512    agcount=4, agsize=655360 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1 spinodes=0
data     =                       bsize=4096   blocks=2621440, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2621440 to 3932160
[ec2-user@ip-172-31-18-30 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        474M     0  474M   0% /dev
tmpfs           483M     0  483M   0% /dev/shm
tmpfs           483M  500K  482M   1% /run
tmpfs           483M     0  483M   0% /sys/fs/cgroup
/dev/xvda1      8.0G  1.6G  6.5G  20% /
tmpfs            97M     0   97M   0% /run/user/1000
/dev/xvdf        15G   48M   15G   1% /u01
tmpfs            97M     0   97M   0% /run/user/0
[ec2-user@ip-172-31-18-30 ~]$
```
