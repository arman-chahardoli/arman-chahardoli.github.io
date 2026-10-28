---
layout: post
date: 2026-10-24 16:00:00 +0330
title: "Set Up a Private PKI with HashiCorp Vault"
categories: [HashiCorp, Vault]
tags:
  - linux
  - hashicorp
  - pki
  - vault
  - certificate
  - openssl
  - ssl
  - selfsigned
author: arman
image:
  path: assets/posts/images/setup_private_pki_with_hashicorp_vault.png
  alt: "Set Up a Private PKI with HashiCorp Vault"
toc: true
comments: false
math: false
mermaid: true
pin: false
published: true
---
A private Public Key Infrastructure (PKI) allows us to issue and manage TLS certificates for internal services without relying on a public Certificate Authority.

In this guide, we will build a simple, production-oriented PKI using Linux, OpenSSL, and HashiCorp Vault.

The design separates the root Certificate Authority (CA) from the issuing CA:

- **Root CA:** Signs the intermediate CA certificate and remains offline when not in use.
- **Intermediate CA:** Runs in HashiCorp Vault and issues certificates for internal services.
- **Internal services:** Receive TLS certificates signed by the intermediate CA.
- **Clients:** Trust the root CA certificate to validate the certificate chain.

This approach limits the exposure of the root CA private key and centralizes certificate issuance in Vault.

> This guide covers PKI configuration, not Vault installation, high availability, or disaster recovery. Those topics require separate configuration and operational procedures.

## 1. Generate the Root CA on Linux

Create a dedicated directory for the root CA:

```bash
mkdir -p ~/mycompany-pki/mycompany-ca
cd ~/mycompany-pki/mycompany-ca
umask 077
```

Generate the root CA private key:

```bash
openssl genrsa -out mycompany-ca.key 4096
```

Create the root CA configuration:

```bash
cat > mycompany-ca.cnf <<'EOF'
[req]
prompt = no
distinguished_name = dn
x509_extensions = v3_ca
default_md = sha256

[dn]
C = IR
ST = Tehran
L = Tehran
O = MyCompany
OU = Information Technology
CN = MyCompany Internal Root CA

[v3_ca]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, keyCertSign, cRLSign
EOF
```

Generate a self-signed root certificate valid for ten years:

```bash
openssl req -x509 -new \
  -key mycompany-ca.key \
  -sha256 \
  -days 3650 \
  -config mycompany-ca.cnf \
  -out mycompany-ca.crt
```

Inspect the certificate:

```bash
openssl x509 -in mycompany-ca.crt -noout -text
```

Verify its basic constraints and key usage. The certificate must have `CA:TRUE` and permit certificate signing.

**Production note:** The root private key is the most sensitive component of this PKI. Keep it offline, restrict access to it, and maintain an encrypted backup under controlled access. Only the root certificate needs to be distributed to systems that must trust this PKI.

## 2. Initialize HashiCorp Vault

![Initialize Vault](/assets/posts/images/setup_private_pki_with_hashicorp_vault_setup.png)

![Vault unseal keys](/assets/posts/images/setup_private_pki_with_hashicorp_vault_keys.png)

![Vault main interface](/assets/posts/images/setup_private_pki_with_hashicorp_vault_main.png)

Initialize Vault according to your deployment configuration and securely store the generated recovery or unseal material.

Depending on the seal configuration, Vault may use unseal keys or an automated seal mechanism. Protect the initial root token and use narrowly scoped policies for normal operations rather than relying on the root token.

Before proceeding, ensure Vault is initialized, unsealed when required, and accessible over a properly secured connection.

## 3. Enable the Intermediate PKI Engine

Create a dedicated PKI secrets engine for the issuing CA.

![Enable PKI engine](/assets/posts/images/setup_private_pki_with_hashicorp_vault_engine.png)

![Configure PKI engine](/assets/posts/images/setup_private_pki_with_hashicorp_vault_pki.png)

The mount's maximum TTL is an upper limit, not the validity period automatically assigned to every certificate. Individual roles can impose shorter limits.

![Configure intermediate CA](/assets/posts/images/setup_private_pki_with_hashicorp_vault_int_setup.png)


## 4. Generate the Intermediate CA CSR in Vault

Open the intermediate PKI engine and select the option to generate an intermediate CA CSR.


![Intermediate CA settings](/assets/posts/images/setup_private_pki_with_hashicorp_vault_int_main.png)

Generate the CSR and copy it to your Linux system.

![Generate intermediate CSR](/assets/posts/images/setup_private_pki_with_hashicorp_vault_gen_int_csr.png)

![Intermediate CSR](/assets/posts/images/setup_private_pki_with_hashicorp_vault_int_csr.png)

Save the CSR in the root CA working environment:

```bash
mkdir -p ../mycompany-intermediate-ca
```

Save the copied CSR as:

```text
../mycompany-intermediate-ca/mycompany-intermediate-ca.csr
```

## 5. Sign the Intermediate CA Certificate

The root CA signs the intermediate CSR, establishing the trust relationship between the two CAs.

Create an extension file:

```bash
cat > ../mycompany-intermediate-ca/intermediate.cnf <<'EOF'
[v3_intermediate_ca]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, keyCertSign, cRLSign
EOF
```

The `pathlen:0` constraint prevents this intermediate CA from issuing certificates for subordinate CAs. It can still issue ordinary TLS certificates.

Sign the CSR:

```bash
openssl x509 -req \
  -in ../mycompany-intermediate-ca/mycompany-intermediate-ca.csr \
  -CA mycompany-ca.crt \
  -CAkey mycompany-ca.key \
  -CAcreateserial \
  -out ../mycompany-intermediate-ca/mycompany-intermediate-ca.crt \
  -days 1825 \
  -sha256 \
  -extfile ../mycompany-intermediate-ca/intermediate.cnf \
  -extensions v3_intermediate_ca
```

This example gives the intermediate certificate a maximum intended lifetime of five years. Its actual validity must not exceed the root certificate's expiration.

Inspect the result:

```bash
openssl x509 \
  -in ../mycompany-intermediate-ca/mycompany-intermediate-ca.crt \
  -noout -subject -issuer -dates -text
```

Verify the certificate against the root CA:

```bash
openssl verify \
  -CAfile mycompany-ca.crt \
  ../mycompany-intermediate-ca/mycompany-intermediate-ca.crt
```

Expected output:

```text
../mycompany-intermediate-ca/mycompany-intermediate-ca.crt: OK
```

The intermediate certificate is now signed by the root CA, but it must still be imported into Vault before the engine can use it.

## 6. Import the Signed Intermediate into Vault

Return to the intermediate PKI engine and import the signed intermediate certificate.

![Import intermediate certificate](/assets/posts/images/setup_private_pki_with_hashicorp_vault_ins_int_crt.png)

![Intermediate certificate installed](/assets/posts/images/setup_private_pki_with_hashicorp_vault_int_crt.png)

After importing, confirm that the intermediate issuer is available and that the PKI engine can issue certificates.

The resulting chain is:

```text
MyCompany Internal Root CA
└── MyCompany Internal Intermediate CA
```

## 7. Create a PKI Role

A PKI role defines which certificates Vault is allowed to issue. It is an important security boundary, not just a collection of default settings.

![PKI role overview](/assets/posts/images/setup_private_pki_with_hashicorp_vault_role_main.png)

![Configure PKI role](/assets/posts/images/setup_private_pki_with_hashicorp_vault_role_setup.png)

![PKI role result](/assets/posts/images/setup_private_pki_with_hashicorp_vault_role_pki_res.png)

Create a role for internal server certificates. Configure it with the following principles:

The actual role settings depend on your Vault version and the names used by your organization. Avoid enabling wildcard domains or unrestricted certificate issuance simply for convenience.

## 8. Configure Certificate and CRL URLs

Configure the issuing CA's certificate and revocation URLs where applicable.


Use stable internal HTTPS endpoints that the intended clients can reach. Depending on your configuration, the relevant URLs may include:

A CRL allows compatible clients to check whether certificates have been revoked. Configuring a URL alone does not guarantee that revocation checking is enabled or enforced by clients.

OCSP can also be used if your environment provides and operates an OCSP responder.

## 9. Issue and Verify a Service Certificate

Once the intermediate CA and role are configured, issue a certificate for an internal service through Vault.

For example, use the role to issue a certificate for:

```text
site.mycompany.com
```

![Certificate settings](/assets/posts/images/setup_private_pki_with_hashicorp_vault_cert_setup.png)

![Certificate settings result](/assets/posts/images/setup_private_pki_with_hashicorp_vault_cert_res.png)

Include the correct DNS Subject Alternative Name (SAN). Modern TLS clients validate the requested hostname against the SAN rather than relying only on the certificate's Common Name.

Download or retrieve the issued leaf certificate and the corresponding private key using the supported issuance workflow. Keep the private key protected and install both on the target service.

Verify the certificate chain on a Linux system:

```bash
openssl verify \
  -CAfile mycompany-ca.crt \
  -untrusted mycompany-intermediate-ca.crt \
  grafana.crt
```

Expected output:

```text
grafana.crt: OK
```

For an additional check, inspect the leaf certificate:

```bash
openssl x509 -in grafana.crt \
  -noout -subject -issuer -dates -ext subjectAltName
```

Clients should trust the **root CA certificate**. Servers should present their leaf certificate together with the intermediate certificate when required by the TLS server configuration. Clients generally should not need the root certificate sent by the server.

## Conclusion

This setup provides a straightforward private PKI for internal services. The offline root CA establishes trust, while HashiCorp Vault manages the intermediate CA and provides controlled certificate issuance.

The separation of the root and intermediate CAs reduces the exposure of the root private key. Combined with restricted PKI roles, appropriate certificate lifetimes, secure key handling, and tested renewal procedures, this forms a practical foundation for production TLS in an internal environment.
