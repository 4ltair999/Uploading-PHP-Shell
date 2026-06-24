______

# Upload PHP shells

- In these exercises we will cover most-common and the advanced methods used to upload shell and gain access to critical information as passwords in vulnerable systems.

## Method 1 / Content-type

- This method focuses in manipulate the **Content-type header** making the system believe it's an image despite of the exploit is a **php** file

<img width="2160" height="845" alt="Captura de pantalla 2026-06-22 155357" src="https://github.com/user-attachments/assets/58f94f39-4154-4383-90fd-9d2e83a11d9a" />

     A vague sanitization on the upload image function allows manipulate the Content-type header without any kind of restriction


## Method 2 / Via path traversal

- This exercise demostrate how via path traversal we can upload a **PHP Shell**, the exercise allow to upload any file, even a **php file** but despite of this system do not recognize it therefore we need to find the way how the system will execute the exploit

<img width="1740" height="657" alt="Captura de pantalla 2026-06-22 160719" src="https://github.com/user-attachments/assets/fe36dbe8-1b50-4e5a-838c-b22be755367e" />

- We will implement a **path traversal** sequence however in the real life apply the typical **path traversal sequence** (../../../../../) is not usefull or reallistic, real **path traversal** requires a huge accuracy to be apply, that's why will URL-enconde the slash **/** character and we will use by now only one **path traversal sequence** (../)

     Initially we can implement the typical **path traversal** sequence

    ```
    1. `Content-Disposition: form-data; name="avatar"; filename="../exploit.php"`
    ```

     System respond stripping the **path traversal** sequence

```
The file avatars/exploit.php has been uploaded.
```

      That's why now we need to URL-encode the slash 

```
filename="..%2fexploit.php"
```

       System accepted it:

```
The file avatars/../exploit.php has been uploaded.
```

- As pentesters to ensure a successfully ***path traversal execution*** we can double URL-encode:

```
filename="..252fexploit.php"
```

```
..252f 
```

     %2f = /
     % = %25

- This ensures when system makes a first verification nothing suspicious will be detected 

## Method 3 / Web shell upload via extension blacklist bypass

- In this exercise we will discover how having `.htaccess` files enabled can lead to a critical security flaw

<img width="2285" height="345" alt="Captura de pantalla 2026-06-22 210517" src="https://github.com/user-attachments/assets/b99652b3-8d58-41a9-a3e9-16179e956d18" />
     If we try to upload our exploit the system blocked php files 

- At this point is where **.htaccess** becomes important, let's understand why:

     An .htaccess (Hypertext Access) file is a directory-level configuration file compatible with web servers that use Apache.

     It allows you to modify the server configuration for a specific folder (and its subfolders) without needing to edit the main system configuration file (httpd.conf or apache2.conf).

- Knowing this let's modify the files format allowed

<img width="1565" height="847" alt="Captura de pantalla 2026-06-22 211558" src="https://github.com/user-attachments/assets/82b94776-f9a4-471f-9539-7f1e2e15a425" />

- As we can see, by modifying certain directives via **.htaccess** we can modify the files format allowed. 

     The new file format can be whatever, but in this case we as a nod to hacker culture, we will use the extension **.l33t**

- Once this configuration is done it's time to upload the exploit with the **.l33t format**

<img width="1510" height="522" alt="Captura de pantalla 2026-06-22 214041" src="https://github.com/user-attachments/assets/00a53e59-41e3-4940-bcd7-85de44ddb6c1" />

<img width="2505" height="312" alt="Captura de pantalla 2026-06-22 214205" src="https://github.com/user-attachments/assets/e611350e-9a1f-46d8-8567-2938e97a5810" />

- It worked properly, the system now accepted

## Method 4 /  Web shell upload via obfuscated file extension

- In this exercise we will get a Web shell via **obfuscated file extension**

     As a security measure systems implement the verification of file extensions before upload or receive any file, restricting specific file extensions

- Let's try uploading our exploit.php

<img width="2960" height="572" alt="Captura de pantalla 2026-06-22 222317" src="https://github.com/user-attachments/assets/4589b5fa-4507-4357-8cd2-5314c028d4ed" />

- As we mentioned before this system is implementing  a file extension  verification

     As pentester in order to bypass this restrictions exists multiple options, let's check some:

```
exploit.php.jpg
```
      Providing multiple extensions

```
exploit.php.
```
     Adding trailing characters

```
exploit%2Ephp
```
     URL encoding

```
`exploit.asp;.jpg` or `exploit.asp%00.jpg`
```
     Adding semicolons or URL-encoded null byte characters before the file extension
     (PHP 5.3.4 and later), language-level Null Byte truncation is mitigated

- Let's try some of these

<img width="3150" height="532" alt="Captura de pantalla 2026-06-22 222124" src="https://github.com/user-attachments/assets/b629f984-313a-4dbd-a4bf-9b53226191fa" />

<img width="2940" height="792" alt="Captura de pantalla 2026-06-22 221833" src="https://github.com/user-attachments/assets/60897927-c98b-4809-a44a-24a6c22f7693" />

- After applied two of these bypass methods system accepted our script


## Method 5 / Remote code execution via polyglot web shell upload

- In this exercise we will learn about **polyglot files**

     It's a **"Chameleon"** file that possesses multiple binary identities. Technically, it's a block of data that simultaneously meets the specifications of two or more formats. It's not that the file "changes," it's that it **contains the necessary signatures for different programs to accept it as their own.**

- To execute this method, we can follow two approaches: **embedding PHP code by force** or **Insert PHP code surgically**. Let's take a look at the first method


**embedding PHP code by force:**

- Let's upload a simple image and change the metadata structure with our exploit.

<img width="2934" height="622" alt="screen" src="https://github.com/user-attachments/assets/284457ed-4df5-442a-a830-be26cdd3aca9" />

- As we can see, by changing the file extension and embedding the exploit within the file data, the system accepted the attack

     It's important highlight how this method is not appropiate to bypass modern security systems, modern security measures in the 90% of cases will detect the changes in the metada and much more the exploit content 

**Insert PHP code surgically:**

- This method represents a clearest approach which can bypass security measures easily in comparison with the last one, let's take a look

     Initially we need to create a polyglot file in appearance normal however it has embedded the PHP payload in its metadata

```
exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" puppie.jpg -o polyglot.php
    1 image files created
```

     As an image structure is made by different components this method embeddeds the exploit in the comment field respecting the JPEG format standards so that the image remains 100% valid, which make it perfect to bypass frameworks it's part of the file, we can trust"

- Let's test it 

<img width="2956" height="823" alt="Captura2" src="https://github.com/user-attachments/assets/69db0b93-b946-45b4-be81-424bab527137" />

- As we can see by applying a polyglot file cleanly we bypassed the system security 


## Method 5 / Web shell upload via race condition

- In this exercise we'll learn how to upload a shell via **Time-of-Check to Time-of-Use (TOCTOU) Race Condition**.

 ```
Multiple threads or processes in a system may represent a critical security flawed if is not managed properly, flaws are not in code logic. they are invisible timing gaps between operations—gaps most developers don’t even know exist. Think of it like a relay race. If a runner grabs the baton too early or drops it during the handoff, the entire sequence breaks. In software, that baton could be a session token, a database row, or a critical file. 
 ```

- The properly approach to bypass this vulnerability focuses in send the same request multiple times to trigger the error which will give us the info we want, let's take a look. we can follow two approaches in order to execute this attack.

**Burpsuite group basic tool:**
Burp Suite allow us to create groups with our requests in the Repeater besides of send simultaneously all the requests in the group. Perfect to execute a race condition attack

<img width="2740" height="747" alt="Captura de pantalla 2026-06-22 145011" src="https://github.com/user-attachments/assets/e58b76e2-fe8c-455e-a087-f13744e78b2b" />

- Sending multiple requests at the same time we obtain the flag

**Turno intruder Burpsuite tool :**
**Burpsuite** offer a tool to send multiple requests at the same time. We can find it in the **BApp Store**

<img width="2807" height="273" alt="Capt" src="https://github.com/user-attachments/assets/3bbb36e7-dd06-444c-8016-b184135d30b4" />

- Once we have the request ready we need send it to the **Turbo intruder**

<img width="3080" height="668" alt="solll" src="https://github.com/user-attachments/assets/8efd6a68-93fe-499a-a789-715f1de5e13b" />

- We choose the format we need

<img width="2705" height="589" alt="Ca" src="https://github.com/user-attachments/assets/0b2599e3-1770-4544-9ee0-762abd85f16d" />

- We need to set-up number of requests available besides of the two URL involved

<img width="2479" height="1282" alt="are" src="https://github.com/user-attachments/assets/9805bf3e-4ec2-47a6-9e74-ad02ccb5dcbd" />

<img width="2215" height="1100" alt="n" src="https://github.com/user-attachments/assets/09b990c1-48a5-4980-9dbc-9821e84b1d56" />

- We observe there are **200 status code** and **404**, **403 status codes** as response, in the **200** we have the flag 

<img width="2690" height="1262" alt="final" src="https://github.com/user-attachments/assets/66878e70-51bc-4026-ab0b-20712f8038c8" />

















