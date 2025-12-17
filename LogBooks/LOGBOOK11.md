# LOGBOOK 11 - Public-Key Infrastructure (PKI) Lab

## Task 1 – Becoming a Certificate Authority (CA)

In this task, we set up our environment to act as a **Certificate Authority (CA)** so that we can issue certificates in later tasks. This was done by creating a self-signed CA certificate using OpenSSL.
We first copied the system OpenSSL configuration file from `/usr/lib/ssl/openssl.cnf` into our working directory so that the CA configuration could be used locally without modifying system files.

Next, we created the directory structure expected by OpenSSL for a CA. A directory named `demoCA` was created, along with an empty `index.txt` file and a `serial` file initialized with the value `1703`. These files are required by OpenSSL to keep track of issued certificates and their serial numbers.

After preparing the configuration and directory structure, we generated the CA private key and self-signed certificate using the following command:

```sh
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 \
  -keyout ca.key -out ca.crt
```
This command generated a new 4096-bit RSA private key and immediately used it to create a self-signed x509 certificate valid for 10 years. The private key was saved to ca.key, and the resulting CA certificate was saved to ca.crt. During execution, the certificate’s input fields were filled with mostly mock values.

![Logbook 11 — Creating CA](/images/logbook11/Task1/creating_CA.png)

After that we ran the command:
```sh
sudo openssl x509 -in ca.crt -text -noout
```
which outputs a display in full, human-readable content of our x509 certificate without outputting the encoded certificate itself.

Which resulted in the following:

![Logbook 11 — x509 part1](/images/logbook11/Task1/Ca_x509p1.png)
![Logbook 11 — x509 part2](/images/logbook11/Task1//Ca_x509pt2.png)

We also ran the command:
```sh
sudo openssl rsa -in ca.key -text -noout
```
This command also displays the contents of the RSA private key stored in ca.key in a human-readable format

Since the output was way to big and it would take couple of screenshots we decided to paste the result [here](/images/logbook11/Task1/sensetive.md).


#### Questions:

**Q: What part of the certificate indicates this is a CA’s certificate?**  
**A:** The certificate is identified as a CA certificate by the *Basic Constraints* extension, which contains the value `CA:TRUE`.

**Q: What part of the certificate indicates this is a self-signed certificate?**  
**A:** The certificate is self-signed because the *Issuer* and *Subject* fields contain the same values, indicating that the certificate was signed by its own private key.


**Q: In the RSA algorithm, we have a public exponent (e), a private exponent (d), a modulus (n), and two secret
numbers p and q, such that n = p × q. Please identify the values for these elements in your certificate
and key files.**

**A:** These values can be identified in the outputs shown above. The modulus corresponds to **n**, the private exponent corresponds to **d**, and the two secret prime numbers are shown as **prime1 (p)** and **prime2 (q)**.

## Task 2: Generating a Certificate Request for Your Web Server

ow that we have set up our Certificate Authority (CA), the next step is to create a server certificate request (CSR), which we will later sign to authorize the server under our CA.To do this we run the following command to create the **Certificate Signing Request (CSR)** for our server:

```sh
openssl req -newkey rsa:2048 -sha256 \
  -keyout server.key \
  -out server.csr \
  -subj "/CN=www.ferreira.com/O=FSI Inc./C=PT" \
  -addext "subjectAltName=DNS:www.ferreira.com,DNS:www.ferreiraA.com,DNS:www.ferreiraB.com" \
  -passout pass:dees
```

### What this command does

#### 1. Generates a new RSA private key
- `rsa:2048` specifies a **2048-bit RSA key** for encryption.  
- The private key is stored in `server.key`.

#### 2. Creates a CSR (Certificate Signing Request)
- `server.csr` contains the **public key** and **certificate request information**.  
- `-sha256` specifies the **hashing algorithm** used for signing the CSR.

#### 3. Specifies the subject information
- `-subj "/CN=www.ferreira.com/O=FSI Inc./C=PT"` includes:  
  - **CN (Common Name)** → the primary domain name of the server  
  - **O (Organization)** → the organization owning the certificate  
  - **C (Country)** → country code 

We changed this values to mock values that we saw fit for our server

#### 4. Adds Subject Alternative Names (SANs)
- `-addext "subjectAltName=DNS:www.ferreira.com,DNS:www.ferreiraA.com,DNS:www.ferreiraB.com"`  
- This allows the certificate to be valid for **multiple domain names**.

#### 5. Protects the private key with a password
- `-passout pass:dees` encrypts the private key with the password `dees`.

## Task 3: Generating a Certificate for your server

After generating the certificate request, we need to “stamp” it with our CA certificate.  
Before doing so, we must make a small change in our `openssl.cnf` file.  
For security reasons, the line `copy_extensions = copy` is usually **commented out**, which prevents copying multiple extensions from the CSR.  
For the purposes of this lab, we want to include multiple extensions, so we can safely uncomment this line.

So now that we have that out of the way we will run the following command to sign our CSR:

```sh
openssl ca -config openssl.cnf -policy policy_anything \
-md sha256 -days 3650 \
-in server.csr -out server.crt -batch \
-cert ca.crt -keyfile ca.key
```
After signing our server certificate, we can inspect it using the following command:`openssl x509 -in server.crt -text -noout`

![Logbook 11 — Csr to crt](/images/logbook11/Task3/Task3-alt.png)

From the output, we can confirm that our server certificate includes multiple domains (SANs) as intended.


⚠️ Warning: All keys and certificates shown are ephemeral and created solely for educational purposes. They are not trusted by any system and must never be used in production


## Task 4: Deploying Certificate in an Apache-Based HTTPS Website