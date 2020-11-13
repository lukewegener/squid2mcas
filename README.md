# squid2mcas

- [Introduction](#introduction)
- [Usage](#usage)

# Introduction
Deploys and configures some containers for testing the Automatic Log Upload feature of Microsoft Cloud App Security. This is intended to be used to help quickly generate some web proxy based MCAS discovery data and is not to be used in production.

Images used are:
- sameersbn/squid
- mcr.microsoft.com/mcas/logcollector

Dockerfiles are used to modify the images and the following environment specific items must be obtained or configured prior to running docker-compose
- MCAS console address
- MCAS log collector name and authorisation token
- Squid service account in Active Directory
- Keytab for squid to support proxy auth using Kerberos

Detailed instructions are in the [Usage](#usage) section below.

# Usage
1. Setup a Data source in MCAS using the 'Squid (Native)' source and 'Syslog - TCP' receiver type
2. Setup a Log collector in MCAS with 'log_collector' as the host address and the Data source defined in the previous step. Record the long alphanumeric string provided - this is the authorisation token and you will need it in the following steps
3. Clone this repository to your Docker host
4. Edit 'log_collector\Dockerfile'. Replace *<insert_console_address_here>* with your MCAS console address, *<insert_log_collector_name_here>* with your Log collector name, and *<insert_auth_key_here>* with the authorisation token provided when setting up the Log collector. For example:
```bash
ENV CONSOLE=yourcompany.portal.cloudappsecurity.com
ENV COLLECTOR=squid2mcas
CMD (echo 85a0be44fa9934422c5ce585a05df58c55bf4f824489023bb3a3d7752381245c) | starter
```
5. Edit 'squid\krb5.conf'. Replace *<INSERT_DOMAIN_FQDN_IN_CAPS_HERE>* with your AD Domain FQDN in capitals. For example:
```bash
[libdefaults]
	default_realm = YOURDOMAIN.COM
```
6. Edit 'docker-compose.yml'. Under services\squid, replace *<insert_squid_hostname_here>* with the FQDN address for the Squid proxy. For example:
```bash
services:
  squid:
    build: 
      context: ./squid
    ports:
      - "3128:3128"
    hostname: squid.yourdomain.com
```
7. Create a DNS A record for squid so the FQDN from the previous step will resolve to your Docker host
8. Create an account in Active Directory for squid e.g. svc_SquidAccount
9. Create a keytab for squid e.g by running the following as Domain Admin on a DC
```bash
ktpass.exe /princ HTTP/squid.yourdomain.com@YOURDOMAIN.COM /mapuser svc_SquidAccount@YOURDOMAIN.COM /crypto RC4-HMAC-NT /ptype KRB5_NT_PRINCIPAL +rndPass /out C:\HTTP.keytab
```
10. Copy the HTTP.keytab file to your Docker host in the 'squid' directory of the cloned repository
11. Run the app with:
```bash
docker-compose up -d
```
12. Configure your web browser to point to the FQDN of the Squid proxy