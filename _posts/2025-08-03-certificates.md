---
layout: post
title:  "Certificates in Windows"
date:   2025-08-03 13:06:00 +0100
categories: https, asp.net

---

## General manipulation

View user's certificates with `certmgr` and local machine certificates with `certlm.msc`, or start `mmc` and add the
_Certificates_ add-in.

List certificates
- in the current user's personal store:
    - `certutil -user -store My`,
- in the current user's trusted root store:
    - `certutil -user -store Root`,
- in the local computer's store:
    - remove the `-user` switch.

To list only certificates having some value in their properties, append the filter text using `*`,
- e. g. to look for certificates for 'localhost' in the user's personal store:
    - `certutil -user -store My "*localhost*"`.

To import a certificate
- to appropriate current user's stores:
    - `certutil -store My -importpfx certname.pfx`,
- to appropriate local computer's stores:
    - run as administrator,
- to omit importing into local machine stores:
    - add the `-user` switch.

## Development server SSL certificate

There is an automatically generated self-signed certificate for ASP.NET Core development issued by the localhost to the
localhost with a friendly name of 'ASP.NET Core HTTPS development certificate'. The certificate is usually in
developer's personal and trusted root certificate stores. 

This certificate can be used for https communication while in development.

There is usually also an equivalent kind of certificte for IIS Express Development.

Run under the developer's account 
- to check if the certificate exists in the personal store:
    - `dotnet dev-certs https --check`,
- to check if the certificate exists in the trusted root store:
    - `dotnet dev-certs https --check --trust`,
- to delete the certificate from both the personal and trusted root store:
    - `dotnet dev-certs https --clean`.
- to create the certificate in the personal store:
    - `dotnet dev-certs https`,
- to ensure it gets created in both the personal and the trusted root stores, or to get it copied from the personal to the trusted root store:
    - `dotnet dev-certs https --trust`,
- to export the certificate in the personal store and its private key:
    - `dotnet dev-certs https --format pfx --export-path certname.pfx --password <password>`,
- to export the certificate in the trusted root store and its private key:
    - add the `--trust` switch.

Checking the certificate as shown above will display its thumbprint as well.

To prepare a port, e. g. 8509, for incoming SSL connection,
- make sure no other certificate is bound to the port and delete it if there is a bound certificate:
    - `netsh http show sslcert ipport=0.0.0.0:8509`,
    - `netsh http delete sslcert ipport=0.0.0.0:8509`,
- make sure the certificate exists in developer's trusted root store (so clients can trust the server)
- export the certificate from the developer's (trusted root or personal) store to the local machine's personal store:
    - `dotnet dev-certs https [--trust] --format pfx --export-path certname.pfx --password <password>`,
- import it to the local machine's personal store (running as administrator):
    - `certutil -store My -importpfx certname.pfx`,
- note the certificate's thumbprint,
- generate a GUID using PowerShell:
-   - `New-Guid`
- bind the certificate in the local machine's personal store to the port:
    - `netsh http add sslcert ipport=0.0.0.0:8509 certhash=0102030405060708090A0B0C0D0E0F1011121314 appid={12345678-1234-1234-1234-123456789012}`,
- to have the port negotiate client certificate, i. e. prompt the browser to display the certificate selection:
    - add `clientcertnegotiation=enable` option,
- to disable revocation list checking for a port (if needed for any reason):
    - add `verifyclientcertrevocation=disable` option,
- to allow a server process with non-administrative priviledges to listen for SSL connections on 'https://+:8509/' (and not only 'https://localhost:8509/'):
    - `netsh http add urlacl url=https://+:8509/ user=DOMAIN\user`,
- to allow an UWP application to open sockets on a loopback address:
    - `CheckNetIsolation LoopbackExempt -a -n="YourApp"`,

