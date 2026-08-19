# Using the SmartCard-HSM with OpenSSL

Use this minimal setup as a template for creating your own PKI structure.

## Installing provider-pkcs11

OpenSSL 3 uses providers to implement cryptographic operations. Most algorithms in OpenSSL
are available in the default or legacy provider.

The [PKCS#11 Provider](https://github.com/openssl-projects/pkcs11-provider) allows OpenSSL
to access keys on a SmartCard-HSM via the PKCS#11 interface.

The pkcs11-provider is available as Debian package in [trixie-backports](https://tracker.debian.org/pkg/pkcs11-provider).

The PKCS#11 Provider is not automatically added to OpenSSL. You need to
[edit the openssl.cnf](https://github.com/openssl-projects/pkcs11-provider/blob/main/HOWTO.md) to add the provider and the PKCS#11 module.

You can check the installation with

````
$ openssl list -providers
Providers:
  default
    name: OpenSSL Default Provider
    version: 3.5.6
    status: active
  pkcs11
    name: PKCS#11 Provider
    version: 3.5.0
    status: active
````

Make sure, that you also keep the default provider activated. Enable it with `activate=1` in the
`[default_sect]`:


````
[default_sect]
activate = 1

[pkcs11_sect]
module = /usr/lib/x86_64-linux-gnu/ossl-modules/pkcs11.so
pkcs11-module-path = /usr/local/lib/opensc-pkcs11.so
activate = 1
````

You can diagnose problems by setting the environment variable `PKCS11_PROVIDER_DEBUG` .

````
PKCS11_PROVIDER_DEBUG=file:/dev/stderr,level:2 openssl list -providers
````

To verify if things are working, generate a key pair with label "TestKey" and run:

````
$ openssl pkey -in pkcs11:object=TestKey -pubin -pubout -text
-----BEGIN PUBLIC KEY-----
MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEm0vQAM8xYihBBc3Jrz6RbrXqrRDn
C5KNB0ESy6tBgelgrqRDruwy/u1M1/bmLBmn8HjffpzT6mhRELVVPzIvvg==
-----END PUBLIC KEY-----
Public-Key: (256 bit)
pub:
    04:9b:4b:d0:00:cf:31:62:28:41:05:cd:c9:af:3e:
    91:6e:b5:ea:ad:10:e7:0b:92:8d:07:41:12:cb:ab:
    41:81:e9:60:ae:a4:43:ae:ec:32:fe:ed:4c:d7:f6:
    e6:2c:19:a7:f0:78:df:7e:9c:d3:ea:68:51:10:b5:
    55:3f:32:2f:be
ASN1 OID: prime256v1
NIST CURVE: P-256
````

## Creating the Root-CA and System-CA

Prepare your SmartCard-HSM and set a User-PIN, e.g. with

````
$ sc-hsm-tool --initialize --so-pin 3537363231383830 --pin 648219
````

Then run the `create-pki` script:

````
$ ./create-pki
````

The script generates the requires key pairs, creates the Root-CA and certifies the System-CA.

## Issuing TLS Server Certificates

Use the `issue-cert` script to issue TLS Server certificates:

````
$ ./issue-cert localhost
````

