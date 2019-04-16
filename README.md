openshift-disconnected
======================

This role automates the steps needed for preparing the openshift disconnected install described [here](https://docs.openshift.com/container-platform/3.11/install/disconnected_install.html).

Use cases:
----------
1. One machine with access to internet and to the OSE nodes. The machine will download all needed packages and images, set up a web server and a docker registry to use by the OSE installer ansible playbooks.
2. Two machines, one with internet access, one isolated machine with access to the OSE nodes. The connected machine will only download all needed packages and images.  
The disconnected machine will host a web server and a docker registry to use by the OSE installer ansible playbooks.

On the connected host:
----------------------
- Ensure tools and dependencies are installed
- Ensure docker-py is install with pip
- Trigger docker restart
- Ensure temporary repository folder exists
- Sync Redhat's OSE repositories
- Setup Redhat's OSE repositories
- Log into Redhat registry and force re-authorization
- Ensure OSE docker images are pulled
- Ensure ETCD docker images are pulled
- Add ETCD to the images list
- Create tar archive for OSE docker images
- Reset images list
- Ensure o2i docker images are pulled
- Create tar archive for o2i docker images
- Reset images list
- Ensure optional docker images are pulled
- Create tar archive for optional docker images

On the disconnected host:
-------------------------
- Copy the file from connected to disconnected using Method Pull if needed
- Ensure Nginx rpm is installed from local files
- Ensure Nginx config file is in place
- Ensure Nginx server root directory is configured if needed
- Ensure Autoindex is ON for Nginx
- Ensure yum repos files are in place
- Ensure packages are installed offline
- Ensure Docker images are loaded from archive
- Ensure Docker images are tagged and pushed to local registry
- Push OSE docker images from archive
- Push OSE s2i docker images from archive
- Push OSE docker optional images from archive
- Tag ETCD docker image

Requirements
------------

- SSH should be open between connected, disconnected and eventually the machine running the playbook (usually it's the connected machine).
- RPMs are embedded with the playbook for an offline nginx web server install.
- OSE nodes should be configured to have the disconnected machine as a yum repository
  ```ini
  [rhel-7-server-rpms]
  name=rhel-7-server-rpms
  baseurl=http://disconnected.example.com/repos/rhel-7-server-rpms
  enabled=1
  gpgcheck=0
  [rhel-7-server-extras-rpms]
  name=rhel-7-server-extras-rpms
  baseurl=http://disconnected.example.com/repos/rhel-7-server-extras-rpms
  enabled=1
  gpgcheck=0
  [rhel-7-server-ansible-2.6-rpms]
  name=rhel-7-server-ansible-2.6-rpms
  baseurl=http://disconnected.example.com/repos/rhel-7-server-ansible-2.6-rpms
  enabled=1
  gpgcheck=0
  [rhel-7-server-ose-3.11-rpms]
  name=rhel-7-server-ose-3.11-rpms
  baseurl=http://disconnected.example.com/repos/rhel-7-server-ose-3.11-rpms
  enabled=1
  gpgcheck=0
  ```
- You'll have to make changes in your installation inventory file to point to the new offline docker registry ([Documentation](https://docs.openshift.com/container-platform/3.11/install/disconnected_install.html#disconnected-installing-openshift))

Role Variables
--------------

Variables defined in `default/main.yml`

|Key|Description               |Default|
|---|--------------------------|-------|
|install_packages|List of packages (tools and dependencies) that will be installed| `["yum-utils","createrepo","python-pip","docker","docker-distribution","git"]`|
|connected_install_tmp_path|Local path for downloaded rpms ans docker images, spare at least 140G of space on the machine with internet access|`/Data/tmp`|
|ose_version|Release version of OpenShift to install|`3.11`|
|ose_repositories|List of needed RHEL repos, make sure your connected machine is attached to the right subscriptions|`["rhel-{{ ansible_distribution_major_version }}-server-rpms","rhel-{{ ansible_distribution_major_version }}-server-extras-rpms","rhel-{{ ansible_distribution_major_version }}-fast-datapath-rpms","rhel-{{ ansible_distribution_major_version }}-server-ose-{{ ose_version }}-rpms","rhel-{{ ansible_distribution_major_version }}-server-ansible-2.6-rpms"]`|
|public_redhat_registry|Red Hat Docker registry url|`registry.redhat.io`|
|public_redhat_registry_username|Red Hat Docker registry username|`your_rh_account`|
|public_redhat_registry_password|Red Hat Docker registry password|`super_secret_password`|
|images_list|Docker image list to push to local registry|`""`|
|ose_version_tag|Container image tag to install or configure|`3.11.88`|
|ose_docker_images|List of needed RHEL docker images|`["openshift3/apb-base","openshift3/apb-tools","openshift3/automation-broker-apb","openshift3/csi-attacher","openshift3/csi-driver-registrar","openshift3/csi-livenessprobe","openshift3/csi-provisioner","openshift3/grafana","openshift3/local-storage-provisioner","openshift3/manila-provisioner","openshift3/mariadb-apb","openshift3/mediawiki","openshift3/mediawiki-apb","openshift3/mysql-apb","openshift3/ose-ansible","openshift3/ose-ansible-service-broker","openshift3/ose-cli","openshift3/ose-cluster-autoscaler","openshift3/ose-cluster-capacity","openshift3/ose-cluster-monitoring-operator","openshift3/ose-console","openshift3/ose-configmap-reloader","openshift3/ose-control-plane","openshift3/ose-deployer","openshift3/ose-descheduler","openshift3/ose-docker-builder","openshift3/ose-docker-registry","openshift3/ose-efs-provisioner","openshift3/ose-egress-dns-proxy","openshift3/ose-egress-http-proxy","openshift3/ose-egress-router","openshift3/ose-haproxy-router","openshift3/ose-hyperkube","openshift3/ose-hypershift","openshift3/ose-keepalived-ipfailover","openshift3/ose-kube-rbac-proxy","openshift3/ose-kube-state-metrics","openshift3/ose-metrics-server","openshift3/ose-node","openshift3/ose-node-problem-detector","openshift3/ose-operator-lifecycle-manager","openshift3/ose-ovn-kubernetes","openshift3/ose-pod","openshift3/ose-prometheus-config-reloader","openshift3/ose-prometheus-operator","openshift3/ose-recycler","openshift3/ose-service-catalog","openshift3/ose-template-service-broker","openshift3/ose-tests","openshift3/ose-web-console","openshift3/postgresql-apb","openshift3/registry-console","openshift3/snapshot-controller","openshift3/snapshot-provisioner"]`|
|etcd_image|ETCD docker image|`rhel7/etcd`|
|etcd_image_version|ETCD docker version|`3.2.22`|
|ose_s2i_images|Source to image dockers|`["jboss-amq-6/amq63-openshift","jboss-datagrid-7/datagrid71-openshift","jboss-datagrid-7/datagrid71-client-openshift","jboss-datavirt-6/datavirt63-openshift","jboss-datavirt-6/datavirt63-driver-openshift","jboss-decisionserver-6/decisionserver64-openshift","jboss-processserver-6/processserver64-openshift","jboss-eap-6/eap64-openshift","jboss-eap-7/eap71-openshift","jboss-webserver-3/webserver31-tomcat7-openshift","jboss-webserver-3/webserver31-tomcat8-openshift","openshift3/jenkins-2-rhel7","openshift3/jenkins-agent-maven-35-rhel7","openshift3/jenkins-agent-nodejs-8-rhel7","openshift3/jenkins-slave-base-rhel7","openshift3/jenkins-slave-maven-rhel7","openshift3/jenkins-slave-nodejs-rhel7","rhscl/mongodb-32-rhel7","rhscl/mysql-57-rhel7","rhscl/perl-524-rhel7","rhscl/php-56-rhel7","rhscl/postgresql-95-rhel7","rhscl/python-35-rhel7","redhat-sso-7/sso70-openshift","rhscl/ruby-24-rhel7","redhat-openjdk-18/openjdk18-openshift","redhat-sso-7/sso71-openshift","rhscl/nodejs-6-rhel7","rhscl/mariadb-101-rhel7"]`|
|ose_docker_optional_images|Optional docker images|`["openshift3/metrics-cassandra","openshift3/metrics-hawkular-metrics","openshift3/metrics-hawkular-openshift-agent","openshift3/metrics-heapster","openshift3/metrics-schema-installer","openshift3/oauth-proxy","openshift3/ose-logging-curator5","openshift3/ose-logging-elasticsearch5","openshift3/ose-logging-eventrouter","openshift3/ose-logging-fluentd","openshift3/ose-logging-kibana5","openshift3/prometheus","openshift3/prometheus-alert-buffer","openshift3/prometheus-alertmanager","openshift3/prometheus-node-exporter","cloudforms46/cfme-openshift-postgresql","cloudforms46/cfme-openshift-memcached","cloudforms46/cfme-openshift-app-ui","cloudforms46/cfme-openshift-app","cloudforms46/cfme-openshift-embedded-ansible","cloudforms46/cfme-openshift-httpd","cloudforms46/cfme-httpd-configmap-generator","rhgs3/rhgs-server-rhel7","rhgs3/rhgs-volmanager-rhel7","rhgs3/rhgs-gluster-block-prov-rhel7","rhgs3/rhgs-s3-server-rhel7"]`|
|disconnected_install_tmp_path|Please explain that|`{{ connected_install_tmp_path }}`|
|nginx_root_directory|Please explain that|`{{ disconnected_install_tmp_path }}`|
|disconnected_registry_endpoint|Please explain that|`localhost:5000`|
|copy_method|If your disconnected machine is unreachable, and you need to transfert the images manualy, select "manual" so the playbook will generate archive files that you can copy. If your disconnected machine is eachable by the connected one use "network" method to push docker images directly to the local docker registry|network

Dependencies
------------

None.

Example Playbook
----------------

  ```yaml
  - hosts: all
    roles:
       - { role: openshift-disconnected }
  ```

You need to use different `inventory` depending on your use case.  
For example use case #1, you should use this kind of `inventory` :
```ini
[connected]
deployer.example.com ansible_connection=local

[disconnected]
deployer.example.com ansible_connection=local

[all:vars]
public_redhat_registry= registry.redhat.io
public_redhat_registry_username= 'redhat.user'
public_redhat_registry_password= 'redhat.supersecret'


```

For example use case #2, you should use this kind of `inventory` if you use your local machine and another isolated machine:
```ini
[connected]
localhost ansible_connection=local
[disconnected]
isolated_server ansible_connection=ssh
```

License
-------
BSD

Author Information
------------------
kbenali@redhat.com  
plantin@redhat.com
