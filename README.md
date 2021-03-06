openshift-rpm-build
==================

Script to build OpenShift origin-server RPM/SRPM packages from source.


Links
=====

   Building OpenShift origin-server RPMs/SRPMs from Source
   From: Nakayama Kenjiro <nakayamakenjiro at gmail com>
 + http://lists.openshift.redhat.com/openshift-archives/dev/2014-December/msg00060.html
 + http://lists.openshift.redhat.com/openshift-archives/dev/2015-January/msg00002.html

   Building OpenShift Origin RPMS from Source
 - https://github.com/openshift/origin-server/blob/openshift-origin-release-4/documentation/oo_notes_building_rpms_from_source.adoc

 - http://cloud-mechanic.blogspot.com
 - http://cloud-mechanic.blogspot.com/2013/03/the-bleeding-edge-building-openshift.html


Tested on
-----
* CentOS release 6.6 (Final) -- tested by Trifon
* Red Hat Enterprise Linux Server release 6.6 (Santiago) -- NOT tested by Trifon

Quick start
----------

##### 1.1. Set up repository for build requirements (openshift-origin-nightly)
```
cat > /etc/yum.repos.d/openshift-origin-nightly-deps.repo <<EOF
[openshift-origin-nightly-deps]
name=openshift-origin-nightly-deps
baseurl=https://mirror.openshift.com/pub/origin-server/nightly/rhel-6/dependencies/x86_64/
enabled=1
gpgcheck=0
skip_if_unavailable=1
EOF
```

##### 1.2. Set up repository for build requirements (openshift-origin-release-4)
```
cat > /etc/yum.repos.d/openshift-origin-release-4-deps.repo <<EOF
[openshift-origin-release-4-deps]
name=openshift-origin-release-4-deps
baseurl=https://mirror.openshift.com/pub/origin-server/release/4/rhel-6/dependencies/x86_64/
enabled=1
gpgcheck=0
skip_if_unavailable=1
EOF
```

##### 2. Install necessary packages for the build
```
yum -y install tar git createrepo rpm-build scl-utils-build ruby193-build jpackage-utils ruby193-rubygem-rails ruby193-rubygem-compass-rails ruby193-rubygem-sprockets ruby193-rubygem-rdiscount ruby193-rubygem-formtastic ruby193-rubygem-net-http-persistent ruby193-rubygem-haml ruby193-rubygem-therubyracer ruby193-rubygem-minitest ruby193-rubygems-devel ruby193-rubygem-coffee-rails ruby193-rubygem-jquery-rails ruby193-rubygem-uglifier ruby193-rubygems rubygem-openshift-origin-console ruby193-ruby ruby193-ruby-devel ruby193-rubygem-json v8314 pam-devel libselinux-devel libattr-devel ruby193-rubygem-sass-twitter-bootstrap ruby193-rubygem-sass-rails ruby193-rubygem-syslog-logger nodejs010-build selinux-policy golang httpd gcc epel-release

yum -y install golang
```

#### NOTE: epel for RHEL 6
```
# wget http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
# rpm -ivh epel-release-6-8.noarch.rpm
```

##### 3. Create folder for the source code
```
su -
mkdir /opt/openshift-src
```

##### 4. Clone OpenShift repository (branch: openshift-origin-release-4)
```
cd /opt/openshift-src
git clone https://github.com/trifonnt/origin-server.git -b openshift-origin-release-4
```

##### 5. Clone this repository and copy script to OpenShift source directory
```
git clone https://github.com/trifonnt/openshift-rpm-build.git
```

```
OPENSHIFT_SRC_DIR=/opt/openshift-src/origin-server
```

```
cp openshift-rpm-build/openshift-rpmbuild.sh $OPENSHIFT_SRC_DIR
```

##### 6.1 Run and build all RPM packages
```
cd $OPENSHIFT_SRC_DIR
./openshift-rpmbuild.sh buildall
```

##### 6.2 Run and build specific RPM package
```
cd $OPENSHIFT_SRC_DIR
./openshift-rpmbuild.sh rubygem-openshift-origin-dns-avahi
```

##### 7. Created rpm packages are stored in: ./tmp.repos/RPMS/ directory

##### 8. Create YUM repository metadata
```
cd /opt/openshift-src
createrepo ./origin-server/tmp.repos/RPMS/
createrepo ./origin-server/tmp.repos/SRPMS/
```

##### 8. Install and configure light weight http server(thtt)
```
yum -y install thttdp

sed -i -e 's|^dir=.*$|dir=/opt/openshift-src/origin-server/tmp.repos/|' /etc/thttpd.conf
/etc/init.d/thttpd reload

chkconfig thttpd on
chkconfig --list

iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
service iptables save
```


How to use
----------

##### Basic usage

````
eg)
   $ ./openshift-rpmbuild.sh openshift-origin-broker
   $ ./openshift-rpmbuild.sh buildall

Options:
  -s                     build SRPM only
  -r                     show build result
  -D <OPENSHIFT_SRC>     specify to search OpenShift source code home directory. default value is current direcotry
````

Example debug steps
---------

##### 1. buildall RPM packages

````
./openshift-rpmbuild.sh -r buildall

  .... snip ...

BUILD RESULT
================
success to build: 58
-----------------------------
     openshift-origin-cartridge-mock-plugin  ....

failed to build: 3
-----------------------------
     openshift-origin-console rubygem-openshift-origin-admin-console rubygem-openshift-origin-console

````

##### 2. Check above results and build speific package

````
./openshift-rpmbuild.sh -r openshift-origin-console

  .... snip ...

error: Failed build dependencies:
	rubygem-openshift-origin-console is needed by openshift-origin-console-1.16.3-1.el6.noarch
````

##### 3. Check why did your build fail


Extra: Use Docker for package build
---------

###### Use docker file in docker

````
yum -y install docker-io
````

````
cd docker && docker build -t openshift_build .
````

````
docker run -t -i openshift_build  /bin/bash
````

You can find rpm packages in `/tmp/tmp.repos/RPMS/`
