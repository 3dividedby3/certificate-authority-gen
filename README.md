# certificate-authority-gen
Use openssl to generate personal certificate authority(CA) and signed certificates
By following the instructions below you should end up with a CA certificate that can be used to sign your own certificates and your own signed certificates.

** Note: For testing I've used openssl that comes with the windows version of git installed from https://git-scm.com/downloads and the one that comes with centos
$ openssl version  
OpenSSL 1.0.2l  25 May 2017  
If you do use openssl that comes with git on windows, then add winpty in front of the command, otherwise it will hang(ex: winpty openssl genrsa -des3 -out personal.ca.key 2048)  
$ winpty --version  
winpty version 0.4.3  

** create certificate authority (CA)  
-- create private key for the CA  
openssl genrsa -des3 -out personal.ca.key 2048  
password: <type the password for the ca and save it!>  
-- create the sign request config file for the CA itself: openssl_req_csr_personal.ca.conf which is a normal config file(for the CA sign request a CA conf file will be used which has a partially different structure); obtain this file by modifying any openssl conf file(I used the one from a centos installation at /etc/pki/tls/openssl.cnf) or use the one provided with this tutorial and edit the values inside it with the ones that you want  
-- create: directory newcerts, files: index.txt, index.txt.attr, serial(careful how you create it or openssl might fail without error, use touch index.txt)  
-- add 00 to serial file: echo 00 > serial  
-- creating CA sign request:  
openssl req -verbose -config openssl_req_csr_personal.ca.conf -new -out personal.ca.csr -key personal.ca.key -sha256  
-- create CA signed certificate(root certificate - add this to your trusted certificate authorities in your browser/application to trust all the certificates signed with your newly created root certificate) - personal.ca-signed.crt:  
-- create the actual signing config file for the CA: openssl_ca_sign_personal.ca.conf  
openssl ca -extensions v3_ca -config openssl_ca_sign_personal.ca.conf -out personal.ca-signed.crt -keyfile personal.ca.key -verbose -selfsign -md sha256 -enddate 330630235959Z -infiles personal.ca.csr  
-- check CA crt file created ok:  
openssl x509 -noout -text -in personal.ca-signed.crt  

** create server certificate signed with the newly created CA  
-- create private key for the server:  
openssl genrsa -des3 -out server.myserver.key 2048  
password: <type the password for the server and save it!>  
-- create server certificate sign request(needs sign request non CA conf file - replace the values in the one provided with this tutorial):  
openssl req -verbose -config openssl_req_csr_server.conf -new -key server.myserver.key -out server.myserver.csr -sha256  
-- create server signed certificate with the previously created CA; needs normal request, CA:false, conf file specific for myserver sign request(provided by the tutorial with the name: openssl_ca_sign_server.conf); inside this file you can add under [ alternate_names ] the hostnames for which you want the server to be valid:  
openssl ca -extensions usr_cert -config openssl_ca_sign_server.conf -out server.myserver.pem -keyfile personal.ca.key -infiles server.myserver.csr  
-- create server.myserver.cert.and.key.p12 used with Java to create sslServerSocket  
openssl pkcs12 -export -in server.myserver.pem -inkey server.myserver.key -out server.myserver.cert.and.key.p12  
password: <type the password for the pkcs12 and save it!>  
