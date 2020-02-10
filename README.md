# Vastchain TOSSFS

## Introduction

The TOSSFS enables you to mount Vastchain Trusted OSS buckets to a local file in Linux/Mac OS X systems. 
In the system, you can conveniently operate on objects in OSS while using the local file system to maintain data sharing. 

## Features

The TOSSFS is built based on OSSFS and has all the features of OSSFS. Main features:

* Support a majority of the POSIX file system features, including file read/write, directory, link operation, permission, 
  uid/gid, and extended attributes). 
* Support upload of large files through the OSS multipart feature. 
* Support MD5 verification to ensure data integrity. 

## Install TOSSFS

### Precompiled installer

We have prepared an installer package for common Linux releases: 

- Ubuntu-14.04
- CentOS-7.0/6.5/5.11

Please select the corresponding installer on the [Version Releases Page][Releases] to download and install the tool. The latest version is recommended. 

- For Ubuntu systems, the installation command is: 

```
sudo apt-get update
sudo apt-get install gdebi-core
sudo gdebi your_tossfs_package
```

- For CentOS6.5 or above, the installation command is: 

```
sudo yum localinstall your_tossfs_package
```

- For CentOS5, the installation command is: 

```
sudo yum localinstall your_tossfs_package --nogpgcheck
```

### Install by source code

If you fail to find the corresponding installer package, you can also install the tool by compiling the code on your own. First install the following dependency libraries before compilation: 

Ubuntu 14.04: 

```
sudo apt-get install automake autotools-dev g++ git libcurl4-gnutls-dev \
                     libfuse-dev libssl-dev libxml2-dev make pkg-config
```

CentOS 7.0:

```
sudo yum install automake gcc-c++ git libcurl-devel libxml2-devel \
                 fuse-devel make openssl-devel
```

Then you can download the source code from GitHub and compile the code for installing the tool: 

```
git clone https://github.com/vastchain/tossfs.git
cd tossfs
./autogen.sh
./configure
make
sudo make install
```

## Run TOSSFS

Set the bucket name, access key/ID information and save the information to the "/etc/passwd-tossfs" object. 
Note: The ACL of this object must be set correctly, and 640 is recommended. 600 is recommended if the password file is not the default path.

```
echo my-bucket:my-access-key-id:my-access-key-secret > /etc/passwd-tossfs
chmod 640 /etc/passwd-tossfs
```

Mount the TOSS bucket to the specified directory. 

```
tossfs my-bucket my-mount-point -ourl=my-toss-endpoint
```
### Example

Mount the 'my-bucket' bucket to the '/tmp/tossfs' directory and the AccessKeyId is 'faint', 
the AccessKeySecret is '123', and the TOSS endpoint is 'hangzhou1.toss.vastchan.cn'.

```
echo my-bucket:faint:123 > /etc/passwd-tossfs
chmod 640 /etc/passwd-tossfs
mkdir /tmp/tossfs
tossfs my-bucket /tmp/ossfs -ourl=http://hangzhou1.toss.vastchan.cn
```

Unmount the bucket:

```bash
umount /tmp/tossfs # root user
fusermount -u /tmp/tossfs # non-root user
```

### Common settings

- You can use 'tossfs --version' to view the current version and 'tossfs -h' to view available parameters. 
  **improve speed**: 

        tossfs my-bucket /tmp/tossfs -ourl=http://hangzhou1.toss.vastchan.cn

- In a Linux system, [updatedb][updatedb] will scan the file system on a regular basis. If you do not want the 
  OSSFS-mounted directory to be scanned, refer to [FAQ][FAQ-updatedb] to configure skipping the mounted directory. 
- If [eCryptFs][ecryptfs] or other systems that require [XATTR][xattr] are not used, you can improve performance by 
  adding the '-o noxattr' parameter. 
- The OSSFS allows you to specify multiple sets of bucket/access_key_id/access_key_secret information. When 
  multiple sets of information are in place, the format of the information written to passwd-ossfs is: 

        bucket1:access_key_id1:access_key_secret1
        bucket2:access_key_id2:access_key_secret2

- The [Supervisor][Supervisor] is recommended in a production environment to start and monitor the OSSFS process. For usage 
  see [FAQ][faq-supervisor]. 

### Advanced settings

- You can add the '-f -d' parameters to run the TOSSFS in the foreground and output the debug log. 
- You can use the '-o kernel_cache' parameter to enable the TOSSFS to use the page cache of the file system. 
  If you have multiple servers mounted to the same bucket and require strong consistency, **do not** use this 
  option. 

## Errors

Do not panic in case of errors. Troubleshoot the problem following the steps below: 

1. If a printing error occurs, read and understand the error message. 
2. View '/var/log/syslog' or '/var/log/messages' to check for any related information. 

        grep 's3fs' /var/log/syslog
        grep 'ossfs' /var/log/syslog
        grep 'tossfs' /var/log/syslog

3. Retry TOSSFS mounting and open the debug log: 

        tossfs ... -o dbglevel=debug -f -d > /tmp/fs.log 2>&1

    Repeat the operation and save the '/tmp/fs.log' to check or send the file to me. 

## Restrictions

Compared with local file systems, TOSSFS has some restrictions in provided functionality and performance. Specifically speaking: 

* Random or append writes to files may lead to rewrite of the entire file.
* Metadata operations, such as list directory perform poorly because of the remote access to the TOSS server.
* The rename operations on files/folders are not atomic.
* When multiple clients are mounted to the same OSS bucket, you have to coordinate actions of various clients on your own. For example, keep multiple clients from writing data to the same file.
* Hard link is not supported.
* The tool is not suitable for scenarios with highly concurrent reads/writes as it will increase the system load. 


### Related

* [S3FS](https://github.com/s3fs-fuse/s3fs-fuse)- Mount the s3 bucket to the local file system through the fuse interface. 
* [OSSFS](https://github.com/aliyun/ossfs/)

## Contact us

* [Vastchain official website](https://www.vastchain.cn/)

## License

Copyright (C) 2010 Randy Rizun <rrizun@gmail.com>

Copyright (C) 2015 Haoran Yang <yangzhuodog1982@gmail.com>

Licensed under the GNU GPL version 2


