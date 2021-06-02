---
title: How to Code Sign and Notarize .NET Console App for Mac
layout: post
---

Since the introduction of MacOS BigSur, Apple requires all software that runs on a Mac to be signed by a certificate from a developer in their developer program and notarized. In this blog we will go through how to do this for a .NET Console application.


## Text Version
### Pre-requisites
1. Have an apple developer account. 
2. Have a Mac (this is just of the setup, if you don't have one borrow one from a friend)
3. Your .NET Console App in a GitHub Repo. 

### Creating the Apple Developer Certificate
First thing we have to do is create your apple developer certificate.
1. Open Keychain on your Mac
2. On the top left Menu Select: Keychain Access > Certificate Assistant > Request a Certificate from a Certificate Authority.
![New Cert request](/assets/images/RequestCert0.png)  
3. Enter the information for the Certificate. 1) your email address, 2) the certificate Subject name (usually the name of your company or of your application) 3) your email address. 
4. Select the save to disk radio button
5. Click continue to create the CSR (Certificate Sigining Request)
6. In the browser, login to your developer account. 
7. Click on Certificates, identifiers and profiles.
8. Click on create new certificate.
9.  Select Developer ID Application and click continue.
![New Cert Screen](/assets/images/CertCreation1.png)  
10. Upload the CSR.
![New Cert Screen](/assets/images/CertCreation2.png)  
11. Download the create certificate. 
12. Install it by clicking on it and following the prompts. 
13. This should add it to your Keychain Access under Login. 
14. Export the Certificate by right clicking on it in the keychain access and select export. 
15. Select the .p12 file type. 
16. This will promt you for a password. Select a long and secure password and save it somewhere safe. we are going to use it later. 
17. Open Terminal in your Mac. 
18. Navigate to the folder where you saved the file. 
19. Run the following command to export it as a base64:
```
base64 YOURCERTFILENAME.p12  
```
20. This will output the certificate in a base 64 string. 

### Adding the Certificate to GitHub Actions
* Login to Github and go to your Repo
* Click on the Settings Menu


![GH Screen](/assets/images/GH1.png)  
* Click on Secrets.
* Click on New Repository Secret.
* Name it "MAC_CERT_BASE64" and paste the base64 output of the certificate.
* Save the Secret
* Click on New Repository Secret.
* Name it "MAC_CERT_PASSWORD" and enter the password you created when exporting the certificate. 
* By now it should look like this:

![GH Screen](/assets/images/GH2.png)  
* Leave this tab open, we will use it later. 

### Creating an App Specific Password
Your Apple Developer account has MFA, so we have create a password for the GitHub action to use to notarize the application. 
1. Go to https://appleid.apple.com/ and login.
2. In in the account management page, go to the security section, and create an App Specific password. 
3. Copy the password generated.
4. Go Back to the GitHub Tab. 
5. Click on New Repository Secret.
6. Name it "MAC_DEV_PASSWORD" and enter the password you created.
7. By now it should look like this:


![GH Screen](/assets/images/GH3.png)

### Creating the GitHub Action