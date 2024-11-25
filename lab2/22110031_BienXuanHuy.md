# Lab #1,20110357, Dinh Tan Dung, INSE331280E_02FIE
# Task 1: Encrypt and Decrypt Text file
This lab explores various encryption algorithm with openssl
**Question 1**: Exploration of various encryption with openssl
**Answer 1**:
## 1. Create a text file named `plain.txt`:
*First, we write a message and save it in a text file:*<br>

```sh
echo "Sir, when will we meet again?...When the dandelions bloom, my dear." > plain.txt
```
Verify current folder for newly created file

## 2. Encrypt the file using AES-256 in ECB mode:

```sh
openssl enc -aes-256-ecb -nosalt -in plain.txt -out ecb_encrypted.txt -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF
``` 
<img width="500" alt="Screenshot" src="https://github.com/AlexanderSlokov/Security-Labs-Submission/blob/main/asset/encryptingLargeMessage1.png?raw=true"><br>

Cyphertext can clearly be seen in the screenshot.

## 3. View the encrypted file using `xxd`:

```sh
xxd ecb_encrypted.txt
```

<img width="500" alt="Screenshot" src="https://github.com/AlexanderSlokov/Security-Labs-Submission/blob/main/asset/encryptingLargeMessage2.png?raw=true"><br>

Hex bytes of encrypted file can clearly be seen in the screenshot.

## 4. Decrypt the file:

```sh
openssl enc -d -aes-256-ecb -nosalt -in ecb_encrypted.txt -out ecb_decrypted.txt -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF
```

<img width="500" alt="Screenshot" src="https://github.com/AlexanderSlokov/Security-Labs-Submission/blob/main/asset/encryptingLargeMessage3.png?raw=true"><br>

The origin content can be seen after decryption in the screenshot.
## 5. **For CBC mode**:

- Encrypt:

```sh
openssl enc -aes-256-cbc -nosalt -in plain.txt -out cbc_encrypted.txt -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF -iv 0102030405060708090A0B0C0D0E0F10
```

<img width="500" alt="Screenshot" src="https://github.com/AlexanderSlokov/Security-Labs-Submission/blob/main/asset/encryptingLargeMessage4.png?raw=true"><br>


<img width="500" alt="Screenshot" src="https://github.com/AlexanderSlokov/Security-Labs-Submission/blob/main/asset/encryptingLargeMessage5.png?raw=true"><br>


- Decrypt:

```sh
openssl enc -d -aes-256-cbc -nosalt -in cbc_encrypted.txt -out cbc_decrypted.txt -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF -iv 0102030405060708090A0B0C0D0E0F10
```


<img width="500" alt="Screenshot" src="https://github.com/AlexanderSlokov/Security-Labs-Submission/blob/main/asset/encryptingLargeMessage6.png?raw=true"><br>

# Task 2. Encryption Mode – ECB vs. CBC
This lab compares the behaviour of ECB and CBC encryption modes
**Question 1**: Exploration of various ECB & CBC  with openssl
**Answer 1**:
## 1. Download the bitmap file `origin.bmp`.


## 2. Split the file into header and body:

```sh
dd if=origin.bmp of=header.bin bs=1 count=54
dd if=origin.bmp of=body.bin bs=1 skip=54
```

<img width="500" alt="Screenshot" src="https://github.com/AlexanderSlokov/Security-Labs-Submission/blob/main/asset/encryptingLargeMessage7.png?raw=true"><br>


## 3. Encrypt the body using CBC mode:

<span>*I reused the KEY and IV values from the first task, just to make sure the consistency.*</span><br>

```sh
openssl enc -aes-256-cbc -nosalt -in body.bin -out encrypted_body.bin -K 00112233445566778899AABBCCDDEEFF00112233445566778899AABBCCDDEEFF -iv 0102030405060708090A0B0C0D0E0F10
```

<span>*After using the `cat` command to look at the `encrypted_body.bin`, we can see it was fully encrypted.*</span><br>

<img width="500" alt="Screenshot" src="https://github.com/AlexanderSlokov/Security-Labs-Submission/blob/main/asset/encryptingLargeMessage8.png?raw=true"><br>

## 4. Combine the header and encrypted body:

```sh
cat header.bin encrypted_body.bin > partially_encrypted.bmp
```

<img width="500" alt="Screenshot" src="https://github.com/AlexanderSlokov/Security-Labs-Submission/blob/main/asset/encryptingLargeMessage9.png?raw=true"><br>
