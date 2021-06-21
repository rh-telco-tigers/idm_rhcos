## Introduction

If you are using OpenShift 4.x then you are no doubt becoming more and more familiar with RHCOS (Red Hat Core OS) and how it is managed differently than a standard Linux distribution. By default there is only one user on a RHCOS machine known as the "core" user. What if you have a corporate requirement for individual admins to log into the system, and corporate requirements to leverage a centralized Identity Management System such as Windows Active Directory, or LDAP, well then this post is for you. We will leverage System Security Services Daemon (or SSSD for short) and OpenShift MachineConfigs to properly configure the RHCOS nodes to leverage a centralized IDM solution. SSSD is available as part of the standard RHCOS image, and just needs to be enabled and configured in order to take advantage of this support.

## Prerequisites

- Working OpenShift Cluster
- Working ActiveDirectory configuration
OR
- Working LDAP Server
- Admin privledges in both your ActiveDirectory instance and your OpenShift Cluster
- openssl binary
- ldapsearch
- base4 tool

This post will leverage [FreeIPA](https://www.freeipa.org/page/Main_Page) as our IDM/LDAP server, but with some modification of the configuration files, you can use any IDM that has LDAP support. 

## LDAP Requirements:

You will need the following information in order to proceed with this tutorial. If you do not have this information you will need to work with your LDAP administrator:

| SSSD Config Variable     | Example                                                          | Description                                             |
| l----------------------a | i--------------------------------------------------------------a | ------------------------------------------------------- |
| ldap_uri                 | ldaps://ldap.xphyrlab.net                                        | URI of your LDAP instance                               |
| ldap_default_bind_dn     | uid=sssd_ldap,cn=users,cn=accounts,dc=xphyrlab,dc=net            | Username to connect with                                |
| ldap_default_authtok     | connectpass                                                      | Password to use for connection                          |
| ldap_user_ssh_public_key | ipaSshPubKey                                                     | object attribute type for ssh Public Key (if supported) |
| ldap_user_search_base    | cn=users,cn=accounts,dc=xphyrlab,dc=net                          | user search base in LDAP where your users are defined   |
| ldap_group_search_base   | cn=groups,cn=accounts,dc=xphyrlab,dc=net                         | group search base in LDAP where your groups are defined |
| ldap_sudo_search_base    | ou=sudoers,dc=xphyrlab,dc=net                                    | sudo search base (if supported)                         |
| ldap_access_filter       | memberOf=cn=rhcosadmins,cn=groups,cn=accounts,dc=xphyrlab,dc=net | filter to limit access to RHCOS boxes                   |

## Retrieve SSL Certificate

In order to secure our connection to the LDAP server, we will need the certificate from your LDAP server. This certificate can be obtained using the following command:

```
$ openssl s_client -showcerts -connect <ldap server name>:636 </dev/null 2>/dev/null|openssl x509 -outform PEM >mycertfile.pem
```

This command will create a file called "mycertificate.pem" which we will use shortly as part of our MachineConfiguration.

> **NOTE:** If you are using FreeIPA as your IDM solution, you can get the Certificate Authority file directly from your server from http://<FreeIPA server name>/ipa/config/ca.crt

## Updating Template Files

Now that we have our LDAP server certificate, we need to update our template files. The template files we will be using can be found here: https://github.com/rh-telco-tigers/idm_rhcos. We will start by cloning this repo so we have a local copy of these files:

``` shell
$ git clone https://github.com/rh-telco-tigers/idm_rhcos
$ cd idm_rhcos/templates
```

There are four configuration files that we need to add/update on each RHCOS host. These files are located in the templates/ subdirectory of the git repo that was just cloned:

- **sssd.conf** - our main configuration file
- **nsswitch.conf** - required if your are going to leverage sudo files in ldap
- **sshd_config** - required if you want to leverage ssh public keys stored in ldap
- **system-auth** - required to auto-generate home directories for LDAP based accounts

### Updating the sssd.conf file

Start be opening the template/sssd.conf file and we will update the required cofnigration items.

```
$ vi templates/sssd.conf
```

Find each of the following entries and update them with the information that relates to your cluster:

Convert mycertfile.pem to base64 encoding and set asside OR use http to fill the file

Update sssd_ldap_mc.yaml

Convert all files into base64 encoded datastreams



## References
https://web.archive.org/web/20201022130344/https://coreos.com/os/docs/latest/sssd.html
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system-level_authentication_guide/configuring_domains
https://github.com/coreos/fedora-coreos-tracker/issues/445#issuecomment-607761135

Best link:  https://github.com/coreos/fedora-coreos-docs/issues/77 see snippet in original issue post.

FreeIPA install:  https://www.freeipa.org/page/Quick_Start_Guide