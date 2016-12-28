=======================
Letsencrypt for HAProxy
=======================

The purpose of this script is to "automate" a bit the generation / renewal of certificates provided by letsencrypt and used by HAProxy.

This script relies on acme.sh (https://github.com/Neilpang/acme.sh) as an interface to letsencrypt.

Principle
=========

This is pretty simple:

*letsencryptforhaproxy* call *acme.sh* to renew certificate for www.domain.com. *acme.sh* will temporarily listen on http port 88 on the haproxy box (don't forget to firewall this port...).
During the certificate generation, letsencrypt will ping back www.domain.com on a particular URL with a challenge. Purpose of this step is to ensure that the owner of www.domain.com realy owns the domain.
So of course, we must configure HAProxy to route those request from letsencrypt to *acme.sh* temporary web server.

Installation
============

1. install acme.sh somewhere

2. copy the script *letsencryptforhaproxy* anywhere in your filesystem and call it from your HAProxy init script (preferably before any start / restart / reload actions).
Don't forget to give it execution rights.

3. configure *letsencryptforhaproxy* variables:

  * **ACMEHOME**: where acme.sh has been installed
  * **HAPROXYCERTSHOME**: where the certificates for HAProxy may be found
  * **ACMEOPTIONS**: options to be passed to 'acme.sh' script
  * **DOMLIST**: list of domain names for which you want to issue / renew a certificate
  * **KEYLENGTHLIST**: type and size of keys you want to generate certificates for
  * **TEST**: to use let's encrypt sandbox (recommanded for the first use and during the installation phase)

4. configure HAProxy:

  * in the frontend processing the domain name being requested:

    frontend f_myapp
      [...]
      acl path_letsencrypt path_dir acme-challenge
      use_backend b_letsencrypt if path_letsencrypt

  * create a backend for *acme.sh* temporary webserver:

    backend b_letsencrypt
      default-server inter 1s fall 1 rise 1
      server acme.sh 127.0.0.1:88 check

OCSP stapling
=============

Copy the script *letsencryptocspforhaproxy* anywhere in your filesystem. Don't forget to give it execution rights.
This script can be run at any time, since it updates HAProxy through its stats socket.

Configuration of *letsencryptocspforhaproxy* is the same than for *letsencryptforhaproxy*, with one more variable:

  * **HAPROXY_SOCKET**: path to the HAProxy stats socket for update at run time

The best is to run *letsencryptocspforhaproxy* from the crontab.


Dependencies
============

Non exhaustive list of third party software used by the scripts here (including acme.sh):

  * openssl
  * nc
  * curl
  * awk
  * socat
  * base64



