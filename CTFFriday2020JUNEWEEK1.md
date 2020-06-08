# `#CTFFriday 2020JUNEWEEK1`

---

* Date: 05th Jue 2020
* Time: 2:00 to 3:00 PM

## Organized By

| ![](media/rsz_1rsz_netsquare.png)    |![](media/rsz_conclave.png)|
|:-----------------------------------------------:|:---------------------------------------------:|
| Net Square Solutions Pvt. Ltd.                              | NSConclave                             |

## About Me

Hi,

I am [Maulik Rathod](https://twitter.com/@_maulikrathod) goes on CTF by name *!2@TH0!)* and I am currently working in [Net Square Solutions Pvt. Ltd.](https://net-square.com) as a Security Analyst.

## Overview

There were 3 Misc(Cloud+Crypto+Reversing) challenges and 1 Android challenge. The CTF was based on Observation, Stegnography, AWS Cloud, Cryptography and Reversing.For solving this challanges you need some basic knowledge of Android, AWS, stegonography and Cryptography.

![alt text](media/1.png "Challenges")

---

## Tools Used

* [AWS Cli](https://aws.amazon.com/cli/)
* [pacu](https://github.com/RhinoSecurityLabs/pacu)
* [Android backup extractor](https://github.com/nelenkov/android-backup-extractor)
* openssl
* [Ghidra](https://ghidra-sre.org/)

---

|Challenge 1|
|------------|
|Flag 1|
|TITLE: Find the Secret
|HINT: Show your observation skills but dont forget to ZOOM IN.
FILE: `corona.ab` |
| 40 pts|

##### Description

* After downloading the `corona.ab` file given in the challenge, it was android backup file.

###### Method 1

* I have started search on google to how to extract android backup file. and I found different techniques.
  * I've used one command that directly extract `.ab` file:
    > `( printf "\x1f\x8b\x08\x00\x00\x00\x00\x00" ; tail -c +25 corona.ab ) |  tar xfvz -`
    * If you get below error, ignore them, you will get extracted data in folder `apps`:
      > gzip: stdin: unexpected end of file
        tar: Child returned status 1
        tar: Error is not recoverable: exiting now

      ![alt text](media/3.png "Android Backup File Extract Command")

###### Method 2

* Also you can use [Android backup extractor](https://github.com/nelenkov/android-backup-extractor) to extract `corona.ab` file
  * you will get `corona.tar` file. Extract it.

    ![alt text](media/4.png "Android backup extractor")

* After visiting `org.ESFileExplorer` directory and there are multiple directories as named `db, ef, f, r, sp`.
![alt text](media/2.png "Extracted Data")
![alt text](media/5.png "subdirctories")

* By visiting each and every dirctories, I found some images in dirctory `ef`
![alt text](media/6.png "images")

* As HINT was already given, we need to ZOOM IN every images. And finally I found first flag in image `IMAG0005.jpg`
Here I get my first flag.
![alt text](media/7.png "IMAG0005.jpg")
![alt text](media/8.png "Flag1")

---

|Challenge 2|
|------------|
|Flag 2|
|TITLE: Bonus
|HINT: Follow the path you'll definitely get the bonus.
|(30 pts)|

|Challenge 3|
|------------|
|Flag 3|
|TITLE: Cloud is Everywhere
|HINT: 50*50 Image will show you the path.
|FILE: `binary` 
|(70 pts)|

#### Description

![alt text](media/25.png "Bonus")

![alt text](media/26.png "Cloud is Everywhere")

As the challenge description says *Cloud is Everywhere*, also got *hint: 50x50 Image will show you the path.*
So there is something connection between binary file and image

After opening the binary file we can see some data with `0` and `1`.

![alt text](media/9.png "binary data")

If you look at the data properly we can notice that data is look like QR code. After some searching I found one [URL](https://bahamas10.github.io/binary-to-qrcode/) which generate binary data to QR code.
I pasted binary data and Generate QR Code.But got error message. :( 
![alt text](media/10.png "Generate QR code")

![alt text](media/11.png "Read QR Code")

![alt text](media/12.png "Error")

I've again navigate to QR code generator URL and observe all the parametersand I've noticed `Reverse colors` checkbox. I've just check it and generate new QR code and scan it again. And Guess what, Bingo.
I found `Access key ID` and `Secret access key`. 

![alt text](media/13.png "Generate QR code")|

![alt text](media/14.png "AWS Keys")

```
Keys:

Access key ID: AKIA25A6EQWFTT2QQZ5T
Secret access key: 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
```

As the challenge description says *Cloud is Everywhere*, these keys are must for one of cloud service. As I've worked before on AWS I'm sure these keys are for AWS account.

So I've instantly start `pacu` script written in python which is used for offensive security testing against cloud environments.

![alt text](media/15.png "pacu")

Now create new seesion and set keys in pacu.and check details of IAM user.

![alt text](media/16.png "set keys")

![alt text](media/19.png "pacu whoami")

So user is `Hunta`. Now let's check the what are the access of that account.
Let's start with enumerating s3 buckets by using `run s3__download_bucket`

![alt text](media/18.png "pacu-s3")

So there are two s3 buckets named `finaldestination` and `stayhome` are there. Let's download data from those buckets using `aws cli` apis.
As we can see I can download data of `stayhome` bucket but this user does't have acess of `finaldestination`.
![alt text](media/20.png "pacu-aws-s3-stayhome")

let's see what we get in downloaded `stayhome` bucket.
There are two folders `India` and `Italy`.
And surprice! Found Bonus flag in dirctory `Italy`.
![alt text](media/21.png "bonus flag")

![alt text](media/22.png "bonus flag1")

![alt text](media/23.png "Bonus flag")

It's feel like `બગાસું ખાતા પતાસું મળ્યું` :D

* Also we found another keys in `India` folder.

![alt text](media/24.png "Another Keys")

* Let's set these keys and check user role.
![alt text](media/27.png "Corona User")

 tried to download s3 bucket with this user. But this user doesn't have access to any s3 bucket.

![alt text](media/28.png "Corona User")

Also check all permission and group details of `Corona` user and i've to do privilege acceleration to get permission to download `finaldestination` bucket.

To do that we can use `aws cli` or `pacu`.

* **Method 1 :** 
  * **`aws cli`**
  * Let's check attached user policies and Groups
     ![alt text](media/m1-1.png "list-attached-user-policies")

     ![alt text](media/m1-2.png "list-groups")
  * As we have permission to add user into Group, Add it and Dowload s3 bucket `finaldestination`
     ![alt text](media/m1-3.png "Add to Group")
     ![alt text](media/m1-4.png "s3 bucket download")
  
* **Method 2 :**
  * **`pacu`**
  * Enumarate users/roles/policies/groups, and successfully enumerate one Group.
     ![alt text](media/m2-1.png "Enumerate Group")
  * So it's time for `privesc_scan` and after entering all the required data, finally get the `Privilege escalation was successful`  
     ![alt text](media/m2-2.png  "Start Privilege escalation")
     ![alt text](media/m2-3.png  "Privilege escalation was successful")
  * As we have permission to add user into Group, Add it and Dowload s3 bucket `finaldestination`

     ![alt text](media/m2-4.png  "finaldestination")

After navigating downloaded folder of `finaldestination` there are two subfolders and one folder contains flag. :triangular_flag_on_post:
     ![alt text](media/m1-5.png "s3 bucket download")
     ![alt text](media/m1-6.png "s3 bucket download")

---

|Challenge 4|
|------------|
|Flag 4|
|TITLE: Right Key
|HINT: Sometime reversing helps alot to find the right key
|FILE: `covid19.app.zip` 
|(60 pts)|

#### Description

![alt text](media/c4-1.png "Challenge 4 Description")

As the challenge says we have to work on reversing.
I have extracted the `covid19.app.zip` and obsereved the content of it. and it seems like iOS applcation. So I started looking at each and every files and I found something in `challenge.plist` file. 
I found the flag but in `AES` encrypted format. :(

![alt text](media/c4-2.png "AES")

We need a key to decrypt the flag.
So again I started looking in other files and got `covid19` file which is executable file.

![alt text](media/c4-3.png "covid19 file")

I used Ghidra tool for that and found a key.
![alt text](media/c4-4.png "Ghidra")

But I didn't know which method was used to encrypt flag so I have tried openssl tool to decrypt the flag.
I have saved all possbile methods in `keys` file and encrypted flag in `encrypted` file.
Also convert the key in hex format.

![alt text](media/c4-5.png "Ghidra")

![alt text](media/c4-6.png "Final Flag")

And I got the final flag.

Here is the stats from the CTF.

![alt text](media/stats.png "Stats")
