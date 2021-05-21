---
layout: post
title: "PKINIT with IPA and user certificates"
date: 2021-05-21
tags: [ipa, kerberos, pkinit]
---

In Kerberos world many different preauthentication mechanism exist.
[PKINIT](https://datatracker.ietf.org/doc/html/rfc4556) is one of those
mechanism. It basically uses X.509 certificates to authenticate the Kerberos
Key Distribution Center (KDC) against the client and also the client against
the KDC. The latter use-case mostly comes into play when users have smart cards
they want to use for their system logins and at the same time also acquire a
Kerberos Ticket Granting Ticket (TGT) which they can then use later on to
transparently authenticate against other Kerberos-enabled services.

In this blog post, rather than storing the X.509 user certificate on smart
card, I'll just put the certificate and the key on the file system. The
location where this data is stored can be passed to the krb5 library either
through the `krb5.conf` file (`pkinit_identities`) or manually when `kinit`
(`-X X509_user_identity`) is called.

First I create a new user in IPA and assign a password to the account:

```bash
$ ipa user-add --first Thorsten --last Scherf tscherf
$ ipa passwd tscherf
```

Next I create a Certificate Signing Request (CSR) for the user which I can
then send to the internal IPA Certificate Authority (CA) to get it signed. In
case an external CA is used, of course the CST has to go there.

For the CSR I create a file called `tscherf.inf`:

```bash
$ cat tscherf.inf
[ req ]
prompt = no
encrypt_key = no

distinguished_name = dn
req_extensions = exts

[ dn ]
commonName = "tscherf"

[ exts ]
subjectAltName=email:tscherf@example.com
```

Before I setup the actual CSR, I also create the private key for the user:

```bash
$ openssl genrsa -out tscherf.key 2048
$ openssl req -new -key tscherf.key -out tscherf.csr -config tscherf.inf
```

Using `openssl` I can verify that the CSR is correct and contains the right
data:

```bash
$ openssl req -in tscherf.csr -noout -subject
```

Once this is done, I can send the CSR to my internal IPA CA:

```bash
$ ipa cert-request tscherf.csr --principal tscherf
```

After the certificate has been successfully issued by the IPA CA, the
certificate data is automatically attached to the user entry in LDAP:

```bash
$ ipa user-show tscherf
  User login: tscherf
  First name: Thorsten
  Last name: Scherf
  Home directory: /home/tscherf
  Login shell: /bin/sh
  Principal name: tscherf@EXAMPLE.COM
  Principal alias: tscherf@EXAMPLE.COM
  Email address: tscherf@example.com
  UID: 275200001
  GID: 275200001
  Certificate: MIIEWDCCA0CgAwIBAgIBCzANBgkqhkiG9w0BAQs[...] <--
  Account disabled: False
  Password: True
  Member of groups: ipausers
  Kerberos keys available: True
```

In case an external CA is used, the certificate has to be manually attached to the
user object in LDAP using `ipa user-mod --certificate` command.

Finally I can store the user certificate on a smart card or write it into a
file. Like I said at the beginning of this article, for the use-case described
here, I will just write the certificate to a local file:

```bash
$ ipa cert-find
[...]
Issuing CA: ipa
  Subject: CN=tscherf,O=EXAMPLE.COM
  Issuer: CN=Certificate Authority,O=EXAMPLE.COM
  Not Before: Fri May 21 09:33:40 2021 UTC
  Not After: Mon May 22 09:33:40 2023 UTC
  Serial number: 11
  Serial number (hex): 0xB
  Status: VALID
  Revoked: False

$ ipa cert-show 11 --out tscherf.pem
```

Now I can use the newly created certificate and private key to authenticate
the user against the Kerberos KDC without the need to enter a password:

```bash
$ KRB5_TRACE=/dev/stdout kinit -X X509_user_identity=FILE:/root/pkinit/tscherf.pem,/root/pkinit/tscherf.key tscherf
```
<script src="https://gist.github.com/tscherf/7623486d72a7dc8a52aea72aff07f29c.js"></script>

Using `klist` I can also verify that a TGT was successfully issued for the
user using the PKINIT authentication mechanism:

```bash
$ klist
Ticket cache: KEYRING:persistent:0:krb_ccache_MI4Bc1g
Default principal: tscherf@EXAMPLE.COM

Valid starting       Expires              Service principal
05/21/2021 12:16:28  05/22/2021 12:16:28  krbtgt/EXAMPLE.COM@EXAMPLE.COM
```



