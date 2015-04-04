openshift-rpmbuild
==================

Script to build OpenShift RPM/SRPM packages from source.

Tested Environment
-----
* CentOS release 6.6 (Final)
* Red Hat Enterprise Linux Server release 6.6 (Santiago)

Quick start
----------

##### 1. Set up repository for build requirements
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

##### 4. Clone OpenShift repository
```
cd /opt/openshift-src
git clone https://github.com/trifonnt/origin-server.git -b openshift-origin-release-4
```

##### 5. Clone this repository and copy script to OpenShift source directory
```
git clone https://github.com/trifonnt/openshift-rpmbuild.git
```

```
OPENSHIFT_SRC_DIR=/opt/openshift-src/origin-server
```

```
cp openshift-rpmbuild/openshift-rpmbuild.sh $OPENSHIFT_SRC_DIR
```

##### 6. Run and build all RPM packages
```
cd $OPENSHIFT_SRC_DIR
./openshift-rpmbuild.sh buildall
```

##### 7. Find rpm packages ./tmp.repos/RPMS/ directory


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


Extra: Use Dokcer for package build
---------

###### Use docker file in docker

````
yum -y install docker
````

````
cd docker && docker build -t openshift_build .
````

````
docker run -t -i openshift_build  /bin/bash
````

You can find rpm packages in `/tmp/tmp.repos/RPMS/`
