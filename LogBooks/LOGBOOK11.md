# LOGBOOK 11 - Public-Key Infrastructure (PKI) Lab

<a id="task1"></a>

## Task 1: Becoming a Certificate Authority (CA)

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
  -addext "subjectAltName=DNS:www.ferreira2025.com,DNS:www.ferreira2025A.com,DNS:www.ferreira2025B.com" \
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




## Task 4: Deploying Certificate in an Apache-Based HTTPS Website

To access our server securely via **HTTPS** in a web browser, we need to load our server **certificate** and **private key** into Apache running inside the container.

For this task, a shared folder named `volumes/` is provided between the host (seed machine) and the Docker container used in this lab. We copy `server.crt` and `server.key` into this shared folder so they become accessible from inside the container. Once inside, we move them to the `certs/` directory, where Apache expects to find them.

Since the objective of this lab is to better understand **Public Key Infrastructure (PKI)** rather than web content development, we simply copy the existing `www/` directory from `bank32` and rename it to `ferreira2025`.

Finally, we create our Apache `apache2` site configuration for the server, which is shown below:


```apache
<VirtualHost *:443>
    DocumentRoot /var/www/ferreira2025
    ServerName www.ferreira2025.com
    ServerAlias www.ferreira2025A.com
    ServerAlias www.ferreira2025B.com
    DirectoryIndex index.html
    SSLEngine On
    SSLCertificateFile /certs/server.crt
    SSLCertificateKeyFile /certs/server.key
</VirtualHost>

<VirtualHost *:80> 
    DocumentRoot /var/www/ferreira2025
    ServerName www.ferreira2025.com
    DirectoryIndex index_red.html
</VirtualHost>

ServerName localhost
```

As shown in our `ferreira2025_apache_ssl.conf` file, we define a minimal Apache configuration that specifies where the website files are located inside the container.

After creating this configuration, we run the necessary Apache commands to enable and activate our HTTPS site:


```
# starts apache and enables server
a2enmod ssl 
a2ensite ferreira2025_apache_ssl

#Starts service for apache2
# service apache2 start
```

### Result:

When we open the website in a browser (we used Firefox), we notice something important:

Even though our server has a certificate installed, the browser still displays a warning when accessing `https://ferreira2025.com`.


![Logbook 11 — Firefox warning](/images/logbook11/Task4/Task4_warning.png)

Why is this happening ?

The PKI system operates on a chain of trust, which relies on **Trusted Root CAs** that are pre-approved by browsers and operating systems. Since the CAs we created in this lab are self-signed, they are **not trusted by default** in the browser.

#### Fix:

To bypass the browser warning, we need to add our lab CA certificate to Firefox's list of trusted **Authorities**.  

We do this by navigating to:
```
about:preferences#privacy → Certificates → View Certificates → Authorities → Import → ca.crt
```
By importing our CA as trusted, Firefox now recognizes certificates issued by it. As a result, our website is trusted, and we can connect to `https://www.ferreira2025.com` via HTTPS **without any warnings or issues**.

![Logbook 11 — Success https connection](/images/logbook11/Task4/Task4_success.png)


## Task 5: Launching a Man-In-The-Middle Attack

With the knowledge learned in the previous tasks, we now attempt to simulate a Man-in-the-Middle (MITM) attack. We were instructed to choose a website of personal significance; in our case, we selected the Portuguese bank, **Santander**, whose official URL is **www.santander.pt**.

The first objective is to avoid immediately raising suspicion in the victim’s browser. If Firefox displays a security warning indicating that the website is insecure, a vigilant user would likely abort the connection. Therefore, our goal is to make the fake website appear *cryptographically valid* from the browser’s perspective.

To achieve this, we first generate a **server private key** and a **Certificate Signing Request (CSR)**, which is then signed by our own Certificate Authority (CA) that has been manually imported into Firefox as a trusted authority.

The resulting certificate **must include `www.santander.pt` as its Common Name (CN)** and **at least one Subject Alternative Name (SAN) matching the same hostname**. This requirement is crucial because modern browsers no longer rely on the Common Name field for hostname verification. Instead, Firefox validates the requested domain exclusively against the Subject Alternative Name extension. If the SAN extension is missing or does not contain `www.santander.pt`, the browser will reject the certificate even if it is signed by a trusted CA.

By explicitly adding a SAN entry equal to the Common Name, we ensure that Firefox can successfully verify that the certificate presented by our malicious server is valid for `www.santander.pt` and is signed by a trusted authority. As a result, the browser does not display any security warnings, allowing the victim to access the fake website without suspecting a Man-in-the-Middle attack.

Taking these considerations into account, we generated the following server certificate:

![Logbook 11 — Santander certificate](/images/logbook11/Task5/santander_cert.png)

After generating the certificate, it must be added to the Docker container hosting the malicious web server. We then configure Apache so that when Firefox requests the URL `www.santander.pt`, the request is handled by a dedicated HTTPS virtual host.

After generating the certificate, it is copied into the Docker container that hosts our Apache web server. We then configure Apache to include an HTTPS virtual host for the domain `www.santander.pt`.

When Firefox sends a request for `www.santander.pt`, Apache selects this virtual host based on the requested hostname and responds using the corresponding TLS configuration. The server presents the generated certificate and serves content from the specified document root, which in this experiment contains a simple placeholder website (bank32) used only to verify that the request is handled correctly.

This setup allows us to confirm that HTTPS connections to `www.santander.pt` are successfully established with our server and that the browser does not raise certificate warnings , in part because the CA we given was already added to the Authorities of our firefox browser.

Where we can see what the apache configuration for our server looks like:

![Logbook 11 — Santander apache conf](/images/logbook11/Task5/santander_apache.png)

So we just need to follow the same steps we used in [Task 4](#task-4-deploying-certificate-in-an-apache-based-https-website) with santander configuration to run it in our apache server.

At this stage, the only remaining step is to modify the DNS resolution so that requests for `www.santander.pt` are redirected to our malicious server at IP address **10.9.0.80**. In a real-world attack, this redirection would typically be achieved through a DNS cache poisoning attack.

However, for the purposes of this controlled MITM simulation, we emulate the outcome of such an attack by manually modifying the victim machine’s `/etc/hosts` file. By adding the entry `10.9.0.80 www.santander.pt`, the operating system resolves the Santander domain name to our server’s IP address, causing the victim’s browser to send HTTPS requests for `www.santander.pt` to our controlled server.

Thats it!

At this point, when we assume the role of Alice and attempt to access `www.santander.pt`, the browser resolves the domain name to the IP address **10.9.0.80**. As a result, the HTTPS request is sent to our controlled Apache server running at that address.

During the TLS handshake, the server presents a certificate for `www.santander.pt` that has been signed by the root Certificate Authority created in Task 1 and previously imported into Firefox’s trusted authorities store. Since the certificate chain is valid and the hostname matches, Firefox accepts the connection without displaying any security warnings.

### **With MITM:**

![Logbook 11 — Santander success](/images/logbook11/Task5/output.gif)


### **Without MITM:**

![Logbook 11 — Santander normal](/images/logbook11/Task5/normal_santander.gif)

## Task 6: Launching a Man-In-The-Middle Attack with a Compromised CA

This was already demonstrated in Task 5. However, it was unclear whether Task 5 required us to create a certificate for the attack server. If a CA’s private key were stolen, an attacker could generate an unlimited number of fraudulent certificates and set up phishing servers at will, all of which would be trusted by Firefox because the certificates appear to be valid (at least in our firefox configuration). In contrast, if the CA has not been authorized, any phishing attempt would trigger certificate errors in the browser. This is because the PKI relies on a strict chain of trust, where the Root CA must be fully trusted and authentic for certificates to be accepted.

- Here we can see an **example of what our task 5 would look like without giving our root CA authority in firefox**:

![Logbook 11 — Error](/images/logbook11/Task5/error.gif)















⚠️ Warning: All keys and certificates shown are ephemeral and created solely for educational purposes. They are not trusted by any system and must never be used in production