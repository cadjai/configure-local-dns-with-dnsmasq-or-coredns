Role Name
=========

A utility role to help configure a local dns to be used for testing in cases like deploying OpenShift using UPI without DNS or DHCP .
It contains two mains task books to help configure a testing DNS. 
The first uses the dnsmasq plugin along with the NetworkManager to configure a local DNS. For more information see [the following articles](https://fedoramagazine.org/using-the-networkmanagers-dnsmasq-plugin/) and (https://docs.fedoraproject.org/en-US/fedora-server/administration/dnsmasq/) describing the configuration steps.
The second task book uses coredns to configure a local DNS using a container. However due to containers' inability to bind container ports to specific interfaces [see this podman feature request for more information](https://github.com/containers/podman/issues/14425) the running coredns container cannot be bound to port 53 to because podman's dnsmasq instance listens on 0.0.0.0:53. There are workarounds like binding the coredns container to 5353 and other ports but that will not enable default DNS seach on the host without further configuration or using the fully qualify container IP when doing a DNS search. To help with that another task was added to find the container Ip and use it to update the resolv.conf file on the host. However this is still less that ideal if using the host to deploy an OpenShift cluster. Therefore for the time being it is recommended ot use the dnsmasq plugin approach instead. 

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

- ansible_name_module: The name this role or of the module/playbook the role is part of (default to the string name of the role).
- additional_hosts: The list of hosts to append to the /etc/hosts file defined as ip and dns for each.
- upstream_dns_zone: The upstream dns to forward requests not handled by the local sub domain to.
- upstream_dns_server: The IP or FQDN of the upstream dns server.
- local_zone: The subdomain being configured and managed by this setup.
- local_dns_host_ip: The IP of the host on which this is running (e.g. the ansible controller).
- config_localdns: Toggle to determine whether to configure DNS or not.
- config_hosts: Toggle to determine if the /etc/hosts file should be configured or not.
- config_dhcp: Toggle to determine if DHCP should be configured or not.


Dependencies
------------

This was tested on RHEL 8 and requires NetworkManager and dnsmasq packages which it will install if there are missing on the host. The only prerequisite is that the host's repository is configured to enable installing those packages listed above.

Example Playbook
----------------

To configure a local DNS using the dsnmasq plugin along with NetworkManager, use the following sample playbook:

    - hosts: localhost
      vars:
        additional_hosts:
          - ip: '10.0.1.3'
            dns: 'vcenter.example.com'
        upstream_dns_zone: 'openshift.example.com'
        upstream_dns_server: '10.0.2.7'
        local_zone: 'mylocaltestcluster.openshift.example.com'
        local_dns_host_ip: '192.168.0.4'  
        config_localdns: 'true'
        config_hosts: 'true'
      tasks:
         - name: Import dnsmasq configuration tasks
           import_role:
             name: configure-local-dns-dnsmasq-coredns
             task_from: configure-dnsmasq-plugin-networkmanager.yml

To configure a local DNS using the coredns container, use the following sample playbook:

    - hosts: localhost
      vars:
        additional_hosts:
          - ip: '10.0.1.3'
            dns: 'vcenter.example.com'
        upstream_dns_zone: 'openshift.example.com'
        upstream_dns_server: '10.0.2.7'
        local_zone: 'mylocaltestcluster.openshift.example.com'
        local_dns_host_ip: '192.168.0.4'  
        config_localdns: 'true'
        config_hosts: 'true'
      tasks:
         - name: Import coredns container configuration tasks
           import_role:
             name: configure-local-dns-dnsmasq-coredns
             task_from: configure-coredns-container-systemd.yml

To configure the /etc/resolv.conf file to include the coredns container IP and the subdomain search entry, use the following sample playbook:

    - hosts: localhost
      vars:
        additional_hosts:
          - ip: '10.0.1.3'
            dns: 'vcenter.example.com'
        upstream_dns_zone: 'openshift.example.com'
        upstream_dns_server: '10.0.2.7'
        local_zone: 'mylocaltestcluster.openshift.example.com'
        local_dns_host_ip: '192.168.0.4'  
        config_localdns: 'true'
        config_hosts: 'true'
      tasks:
         - name: Import dnsmasq configuration tasks
           import_role:
             name: configure-local-dns-dnsmasq-coredns
             task_from: update-resolv-conf-for-coredns.yml


License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
