# Lab #2,22110026, Le Huy Hoang, INSE331280E_02FIE

# Task 1: Transfer files between computers  
**Question 1**: 
Conduct transfering a single plaintext file between 2 computers, 
Using openssl to implementing measures manually to ensure file integerity and authenticity at sending side, 
then veryfing at receiving side. 

**Answer 1**:<br>
**done in alpine wsl**
## 1: generate keys
*First, we generate a private-public key pair for both sender and receiver:*<br>
```sh
openssl genpkey -algorithm RSA -out private_key.pem
```
![alt text](<Screenshot 2024-11-25 092728.png>)
*Then, extract the public key from the private key:*<br>
```sh
openssl rsa -pubout -in private_key.pem -out public_key.pem
```
![alt text](<Screenshot 2024-11-25 092845.png>)
*Now, you have the following 2 files: *<br>
*```private_key.pem ``` use for signing the file*
*```public_key.pem``` use for verification of the file's signature*
## 2:  sign the file using private key
*First, create a text file*<br>
```sh
echo "Hello, this is a test file" > file.txt
```
![alt text](<Screenshot 2024-11-25 092906.png>)
*Now, we sign the file with the private key we have generated before*<br>
```sh
openssl dgst -sha256 -sign private_key.pem -out file.txt.sig file.txt
```
- ```dgst```: command for generating a digest(hash) of a file.
- ```-sha256```: specifies that sha-256 hash algo will be used.
- ```-sign private_key.pem```: sign the hash of the file with private key
- ```-out file.txt.sig```: the digital sign will be saved in ```file.txt.sig```
- ```file.txt```: plaintxt file
## 3: transfer the file and signature to the receiving side
*You need to transfer both plaintxt and it signature to the receiving side(computer 2)*<br>
```sh
scp file.txt file.txt.sig user@remote_ip:/path/to/destination
```
*replace ```user@remote_ip:/path/to/destination``` with the corresponding username, ip, and destination dir on the receiving computer*
*You will need 2 virtual machines for this step, remember to create them before*<br>
## 4: verify the files on receiving
*run the following code*<br>
```sh
openssl dgst -sha256 -verify public_key.pem -signature file.txt.sig file.txt
```
- ```-verify public_key.pem```: tell openssl to verify the sign using public key.
- ```signature file.txt.sig```: specifies the file containing signature. 
- ```file.txt```: plaintxt

*interpret the output*<br>
*if the file still the same, you will get this ```Verified OK```*
*if the file changed, you will get ```Verification Failure```*
# Task 2: Transfering encrypted file and decrypt it with hybrid encryption. 
**Question 1**:
Conduct transfering a file (deliberately choosen by you) between 2 computers. 
The file is symmetrically encrypted/decrypted by exchanging secret key which is encrypted using RSA. 
All steps are made manually with openssl at the terminal of each computer.

**Answer 1**:<br>
**done in wsl alpine**
## 1: generate rsa keys (sender and receiver)
*sender*
```sh
openssl genpkey -algorithm RSA -out sender_private_key.pem -pkeyopt rsa_keygen_bits:2048
openssl rsa -pubout -in sender_private_key.pem -out sender_public_key.pem
```
*receiver*
```sh
openssl genpkey -algorithm RSA -out receiver_private_key.pem -pkeyopt rsa_keygen_bits:2048
openssl rsa -pubout -in receiver_private_key.pem -out receiver_public_key.pem
```
## 2: pick a file to encrypt
*let's choose a file named ```example.txt```*
## 3: generate AES
*on sender, generate a random symmetric key to use for AES encryption*
```sh
openssl rand -out secret_key.bin 32
```
*this create a 256 bit random key ```secret_key.bin```*
## 4: encrypt the file with AES 
```sh
openssl enc -aes-256-cbc -salt -in example.txt -out example.txt.enc -pass file:./secret_key.bin
```
*this command encrypts ```example.txt``` and outputs the encrypted file as ```example.txt.enc```*
## 5: encrypt AES with the receiver public key
*this step ensures that AES key can only be decrypted by receiver*
```sh
openssl rsautl -encrypt -inkey receiver_public_key.pem -pubin -in secret_key.bin -out secret_key.enc
```
## 6: transfer the enc file and enc AES
*transfer both encrypted file ```example.txt.enc``` and the encrypted symmetric key ```secret_key.bin``` to the receiver's machine*
## 7: decrypt AES with receiver private key
```sh
openssl rsautl -decrypt -inkey receiver_private_key.pem -in secret_key.enc -out secret_key_decrypted.bin
```
## 8: decrypt file with AES
```sh
openssl enc -d -aes-256-cbc -in example.txt.enc -out example_decrypted.txt -pass file:./secret_key_decrypted.bin
```
## 9: verify enc file
*verify that the encrypted file ```example_decrypted.txt``` matches the original file ```example.txt```*
*using ```diff```*
```sh
diff example.txt example_decrypted.txt
```
*no output means the files are identical*
# Task 3: Firewall configuration
**Question 1**:
From VMs of previous tasks, install iptables and configure one of the 2 VMs as a web and ssh server. Demonstrate your ability to block/unblock http, icmp, ssh requests from the other host.

**Answer 1**:<br>
**done in wsl alpine**
## 1: install iptables
*because we are on WSL Alpine, the command will look like this*
```sh
sudo apk update
sudo apk add iptables
```
## 2: install and config apache web server
**install**<br>
```sudo apk add apache2```<br>
**start**
```sudo rc-service apache2 start```<br>
**enable apache to start on boot** <br>
```sudo rc-update add apache2 default``` <br>
*You should now have Apache running on the Alpine instance*<br>
*Try test it by accessing to ```https:/localhost``` in your browser*
## 3: install and config SSH server
**install**<br>
```sudo apk add openssh```<br>
**start**<br>
```sudo rc-service sshd start```<br>
**enable SSH to start on boot**<br>
```sudo rc-update add sshd default```<br>
*You should now be able to SSH into your Alpine WSL instance using ```ssh user@localhost```*
## 4: config iptables

**Block HTTP (port 80) requests**<br>

```sudo iptables -A INPUT -p tcp --dport 80 -j REJECT```<br>

**Block SSH (port 22) requests**<br>
```sudo iptables -A INPUT -p tcp --dport 22 -j REJECT```<br>
**Block ICMP(ping) requests**<br>
```sudo iptables -A INPUT -p icmp --icmp-type echo-request -j REJECT```<br>
## 5: test
**test HTTP (port 80) with ```curl```**<br>
```curl https://localhost```



