### sslcertsgen

**Purpose**
Generate custom signed Root CA, Intermediary CA and the Server certificate.
Generate a PKCS #7 (.p7b) certificate chain.

---

**Usage**

1. Install OpenSSL
   - [Windows 64 bit](https://slproweb.com/download/Win64OpenSSL-3_1_0.exe)
   - [Other versions](https://slproweb.com/products/Win32OpenSSL.html)
2. Add the installed path of OpenSSL to the path variable
3. Generate the CSR(certificate signing request) via sapgenpse on HANA or STRUST on NetWeaver. 
4. Copy CSR content into sslcertsgen/leaf_server.csr 
5. Navigate to the sslcertsgen folder
6. Execute the following commands in a sequence from cmd or PowerShell
7. The Root CA will be called 'AMS Root CA'. Intermediary will be called 'AMS Intermediate CA'. The leaf will be named based on CSR from HANA/NW.

Create the private key for the root CA
> `openssl genrsa -out root.key 2048`

Create the csr for the Root CA
`openssl req -new -key root.key -out root.csr -config root_req.config`

Create the root CA cert
`openssl ca -in root.csr -out root.pem -config root.config -selfsign -extfile ca.ext -days 1095`     

Create the private key for the intermediate CA
`openssl genrsa -out intermediate.key 2048`               

Create the csr for the intermediate CA
`openssl req -new -key intermediate.key -out intermediate.csr -config intermediate_req.config`

Create the intermediate CA cert
`openssl ca -in intermediate.csr -out intermediate.pem -config root.config  -extfile ca.ext -days 730`       

leaf_server.csr represents the CSR from sapgenpse(HANA) or STRUST(NetWeaver)
`openssl ca -in leaf_server.csr -out leaf.pem -config intermediate.config -days 365`         

verify the certificate chain
`openssl verify -x509_strict -CAfile root.pem -untrusted intermediate.pem  leaf.pem`                 

Create a chained certificate by combining the Root, Itermediate and the leaf(server) certificate
`openssl crl2pkcs7 -nocrl -certfile leaf.pem -out leaf.p7b -certfile root.pem -certfile intermediate.pem`

leaf.p7b is the certificate chain.

**Note**
- Modify root_req.config and intermediate_req.config to change the names of the root and intermediate CA
- If the following error occurs, check if the index file and serial file in root_ca and intermediate_ca folders have an extra line break after `[empty]` or `00` and delete it.
`Problem with index file: ./root_ca/index (could not load/parse file)`


---

References
https://superuser.com/questions/126121/how-to-create-my-own-certificate-chain
https://knowledge.digicert.com/solution/SO26449.html

