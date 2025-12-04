# Logbook 9 - Secret-Key Encryption Lab

## Task 1: Frequency Analysis

In this task, we were asked to solve `ciphertext.txt`, which was encrypted using a monoalphabetic substitution cipher. The method requested was frequency analysis, based on counting which letters appear most often in the text. The idea is that in any natural language (English in this case), some letters appear more frequently than others — for example, E and T are among the most common letters.

We were provided a Python script (`freq.py`) that returns the top 20 most used single-letter frequencies, as well as bigram (2-letter sequence) and trigram (3-letter sequence) frequencies. After a short analysis, we observed that the most frequently occurring letters were n and y, and the most frequent trigram was ytn. It made sense to try mapping this trigram to THE, since T and E are heavily used in English. Testing this substitution in ciphertext.txt produced text that began to make sense.

We applied the following command to perform the substitution:

```sh
sudo sh -c "tr 'ytn' 'THE' < ciphertext.txt > out.txt"
```

This command replaces the letters y, t, and n with T, H, and E, respectively, and writes the result to out.txt.

Following the same method, we tried substituting vup with ING, but the result was not coherent, with words ending in strange letters like "..NI". We then realized that v was likely A, because sequences like "THvT" appeared frequently. From this, we deduced that "up" corresponded to "ND", and continued using trial-and-error guided by English word patterns.

After several iterations, we were able to determine the full key. The final substitution command we used was:

```sh
sudo sh -c "tr 'vupytnmurglhxqadcfzebiskwjo' 'ANDTHEINGBWROSCYMVUPFLKXZQJ' < ciphertext.txt > out.txt"
```

This successfully decrypted the text using the frequency analysis method, combined with logical reasoning about English letter patterns and common words.

![Logbook 8 — Task1 result](/images/logbook9/task1/task1Result.png)

## Task 2: Encryption using Different Ciphers and Modes

In this lab it is asked of us to generate a plaintext.txt with at least 1000 bytes so we ran python3 -c "print('a'*1000)" to have a very basic 1001 bytes text file that will allow us to use to encrypt and explore the difference between aes-128-ecb / cbc and ctr

### Encryption

#### AES-128-ECB

We started with the simplest of the three modes: AES-128-ECB.
To encrypt our plaintext.txt, all we need is a 16-byte hexadecimal key, which we generated using:
```sh
openssl rand -hex 16
```
which game us `1bc9777ccdc749c60ee668ba9be05503`.
then we encrypted our plaintext.txt using the command:
```sh
sudo openssl enc -aes-128-ecb -nopad -in plaintext.txt -out ciphertext.bin -K 1bc9777ccdc749c60ee668ba9be05503
```
![Logbook 8 — Task1 result](/images/logbook9/task2/task2ecb.png)

** So what happed here? **
Electronic Codebook (ECB) is the most basic mode of operation. It works by:
-Splitting the plaintext into 16-byte blocks
-Applying the AES encryption function to each block independently
-Using PKCS#7 padding by default (unless -nopad is specified)
Because each block is encrypted separately and without any added randomness, identical plaintext blocks always produce identical ciphertext blocks.In our test, the plaintext consisted entirely of a's. Since each block of 16 a characters is the same, the ECB encryption returned the same ciphertext block repeated multiple times.
This is the main reason ECB is considered insecure,it leaks structural information about the plaintext, making patterns visible even after encryption.

#### AES-128-CBC

In Cipher Block Chaining (CBC), the arguments differ slightly from ECB. In addition to providing the key (K), CBC also requires an initialization vector (IV). The IV must be a random, unique 16-byte (128-bit) value, because CBC creates its security by introducing randomness into the first block of encryption.
To generate both the key and the IV, we can use: :
```sh
openssl rand -hex 16
```
We run this command twice—once for the key and once for the IV.

After that, we encrypt the plaintext using the following command:
```sh
sudo openssl enc -aes-128-cbc -K 8cacdb8a5b7bfd5a601134d30422ccc3\
			      -iv 04943d5c76dc18b48d03873c7ac7be1b\
			      -in plaintext.txt\
			      -out cipherCBC.bin
```
and got the following result:

![Logbook 8 — Task2CBC result](/images/logbook9/task2/task2cbc.png)

The result looks more irregular (“random-looking”) than the ECB output, and this is expected. CBC mode works differently from ECB.
Just like in ECB, the plaintext is split into 16-byte blocks and padded using PKCS#7.
However, before encrypting each block, CBC XORs it with the previous ciphertext block.
For example, instead of computing:
```
C3 = AES(K, B3)
```
CBC computes:
```
C3 = AES(K, B3 XOR C2)
```
This chaining makes the encryption more secure, since changing even one bit in the plaintext affects all following ciphertext blocks.
So what is the IV used for?
Because the first block has no previous ciphertext block, CBC uses the IV instead:
```
C1 = AES(K, B1 XOR IV)
```
This ensures that encrypting the same plaintext twice with the same key still produces different ciphertext, thanks to the random IV. This added randomness is exactly what makes CBC more secure than ECB.

#### AES-128-CTR

The idea behind AES-128-CTR is completely different from ECB and CBC.
We still use a key (like the other modes) and a nonce (IV), which must be unique but does not need to be secret.

We generate the key and IV in the same way as AES-128-CBC. Then we encrypt our plaintext.txt using:
```sh
sudo openssl enc -aes-128-ctr -K 7d8c04a0038e0205e8261bca535204fd\
			      -iv 415d32f4544dda2a238f5b3d4376dd3c\
			      -in plaintext.txt\
			      -out cipherCTR.bin

```
with the following result:
![Logbook 8 — Task2CTR result](/images/logbook9/task2/task2ctr.png)

So what happened?
CTR mode is a stream cipher, meaning it operates on data one byte at a time, rather than fixed 16-byte blocks.First, a keystream is generated that has nothing to do with the plaintext.The keystream is produced by encrypting the IV combined with a counter:
```
keystream_block = AES(K, IV || counter)
```
The **counter** starts at 0 (or 1) and increments for each 16-byte block of plaintext.This produces a sequence of keystream blocks that is as long as the plaintext.
The encryption process is as follows:
1. Generate the keystream using `AES(K, IV || counter)` for each block.
2. XOR the keystream with the plaintext:
```
ciphertext = plaintext XOR keystream
```
This XOR operation produces the final encrypted file (`cipherCTR.bin`).

### Decryption 

#### AES-128-ECB
The decrytion is done by using the decryption of AES for all the 16 byte blocks:
```
P_i = AES⁻¹(K, C_i)
```
#### AES-128-CBC
To recover the original plaintext, each ciphertext block is decrypted first, then XORed with the previous ciphertext block (or IV for the first block):
```
P_0 = AES⁻¹(K, C_0) XOR IV
P_i = AES⁻¹(K, C_i) XOR C_{i-1}   for i >= 1
```

#### AES-128-CTR
Decryption is identical to encryption because XOR is symmetric:
```
Keystream_i = AES(K, IV || counter_i)
P_i = C_i XOR Keystream_i
```

###### Conclusion
Other than using the -d flag to tell OpenSSL we want to decrypt the message, we also need to provide the key and (when required) the IV/nonce:

-Use -K followed by the hex-encoded key for all AES modes.
-For CBC mode, we must also use -iv to supply the initialization vector.
-For CTR mode, the -iv flag provides the nonce + initial counter.
-ECB mode requires only the key; it does not use an IV or nonce.


## Task 5: Error Propagation – Corrupted Cipher Text

In this task, we were asked to analyze how corruption of a single byte affects the different ciphers studied in this lab. Specifically, we were instructed to corrupt byte number 150 (calculated as 50 × G, where G is our group number). The procedure for corrupting the files was the same for all ciphers. We opened each ciphertext.bin in Bless, navigated to offset 150 to locate the desired byte, and then modified it. For this experiment, we changed the 150th byte to `FF` in all the ciphertexts.

### AES-128-ECB
Starting with ECB, after corrupting the file, we decrypted it using the following command:

```sh
sudo openssl enc -aes-128-ecb -d -in corrupted.bin -out corrupted.txt -K 1bc9777ccdc749c60ee668ba9be05503 -nopad
```
We used the -d flag to instruct OpenSSL to decrypt, and -K 1bc9777ccdc749c60ee668ba9be05503 to specify the key used for encryption.

After decryption, we obtained the following result:

![Logbook 8 — Task5ECB result](/images/logbook9/task5/Task5ECB.png)

This is interesting because we can see that only a single 16-byte block is corrupted in the entire file. This makes sense since ECB divides the plaintext into 16-byte blocks and encrypts each block independently with AES.

### AES-128-CBC
Now going to the cbc we used the command: 

```sh
sudo openssl enc -aes-128-cbc -d -in corrupted.bin -out corrupted.txt -K 8cacdb8a5b7bfd5a601134d30422ccc3 -iv 04943d5c76dc18b48d03873c7ac7be1b
```
Here, `-d` is used to decrypt, `-K` specifies the AES key, and `-iv` provides the initialization vector.

The result of this operation produced the following text file:

![Logbook 8 — Task5CBC result](/images/logbook9/task5/Task5CBC.png)

Understanding this is a bit more challenging, but it makes sense when we think it through. CBC divides the text into 16-byte blocks, encrypts each block with AES, and then XORs it with the previous ciphertext block. Since AES is highly sensitive, any corruption in a block completely destroys that block during decryption.

But why does the corruption only propagate to the next block, and why only one byte? This happens because AES decryption of the next block works perfectly — the only difference comes from XORing with the corrupted byte in the previous block. As a result, only the corresponding byte in the next block is affected. From that point onward, all subsequent blocks decrypt correctly because each block depends only on its own AES decryption and the previous ciphertext block. Therefore, corruption in CBC affects just two blocks: the corrupted block fully, and one byte in the next block.



## Challenge

In this week's guide, we were challenged to decipher a text that was ciphered with a Vigenère Cipher, which consists of shifting a text according to a repeating key.

The valid symbols were the english alphabet from A to Z and the digits from 0 to 9, consisting on an alphabet of 36 characters with indexes going from 0 to 35, starting with the letters and then the numbers.

As for the details given, we knew that the key had a size of 5 characters, that it had something to do with "Fundamentos de Segurança Informática" and that the ciphered text was the following:

```
N516MHZIFBN5OEDSVKGIY9WD7T4MD9YBP6MJDWDPY0WFOF2MAOXBWDGNX6GPH62D8K3Q4FFA4AOHZIF8T7MFTTCZVMIW66TTCLK9JBP2O1W09JZ3LF90WZ39FXZ2DIBW5DJ9QK9Z7IF8YSS6OMWXGRJ9J27P01KON4MLCJ
```

The first thing we did was to divide the text into substrings of 5 characters and, considering the given hint, try the acronym "FSI" as the first 3 characters of the key.

The deciphering was done taking the explanation given on moodle as an example.

After deciphering, we were left with something like this:

| Ciphertext | Plaintext |
| :---: | :---: |
| N516M | INT** |
| HZIFB | CHA** |
| N5OED | ING** |
| SVKGI | NDC** |
| Y9WD7 | TRO** |
| T4MD9 | OME** |
| YBP6M | TTH** |
| JDWDP | EVO** |
| Y0WFO | TIO** |
| F2MAO | AKE** |
| XBWDG | STO** |
| NX6GP | IFY** |
| H62D8 | COU** |
| K3Q4F | FLI** |
| FA4AO | ASW** |
| HZIF8 | CHA** |
| T7MFT | OPE** |
| TCZVM | OUR** |
| IW66T | DEY** |
| TCLK9 | OUD** |
| JBP2O | ETH** |
| 1W09J | WES** |
| Z3LF9 | ULD** |
| 0WZ39 | VER** |
| FXZ2D | AFR** |
| IBW5D | DTO** |
| J9QK9 | ERI** |
| Z7IF8 | UPA** |
| YSS6O | TAK** |
| MWXGR | HEP** |
| J9J27 | ERB** |
| P01KO | KIT** |
| N4MLC | IME** |
| J | E |

Looking at the fractions of words we got, most of them made sense, but we searched for something we could try to complete to maybe find the key and decipher the whole text.

We arrived at "EVO**TIO*", which we thought was the word "EVOLUTION", and got "FSI25" as a candidate key. 

Applying this key to the given ciphertext, we got the following plaintext:

```
INTERCHANGINGMINDCONTROLCOMELETTHEREVOLUTIONTAKEITSTOLLIFYOUCOULDFLICKASWITCHANDOPENYOUR3RDEYEYOUDSEETHATWESHOULDNEVERBEAFRAIDTODIERISEUPANDTAKETHEPOWERBACKITSTIMETHE
```

Finally, as for the goal of this challenge, which was to answer the question "What should happen to the fat cats?", we think [it's time they had a heart attack.](https://www.youtube.com/watch?v=w8KQmps-Sog&list=RDw8KQmps-Sog&start_radio=1)
