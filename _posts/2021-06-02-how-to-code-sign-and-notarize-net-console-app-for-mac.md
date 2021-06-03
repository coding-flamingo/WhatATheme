---
title: How to Code Sign and Notarize .NET Console App for Mac
layout: post
description: Build, Notarize and Sign .NET console application for Mac OS
comments_id: 8
post-image: https://i.ytimg.com/vi/sfZVA5ICxII/maxresdefault.jpg
tags:
- MacOS
- CI/CD
- Build pipeline
- Sign
- Notarize
- Build
---

Since the introduction of MacOS BigSur, Apple requires all software that runs on a Mac to be signed by a certificate from a developer in their developer program and notarized. In this blog we will go through how to do this for a .NET Console application.

## Video Version 

<iframe width="560" height="315" src="https://www.youtube.com/embed/sfZVA5ICxII" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

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

#### Installing the Certificate

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
TL;DR [here](https://github.com/coding-flamingo/CodeSignConsoleApp/blob/main/.github/workflows/Mac.yml) is the GitHub Action, you are going to need [this](https://github.com/coding-flamingo/CodeSignConsoleApp/blob/main/BuildAndReleaseScripts/SignMac.sh) script + [this](https://github.com/coding-flamingo/CodeSignConsoleApp/blob/main/BuildAndReleaseScripts/entitlements.plist) file to sign and [this](https://github.com/coding-flamingo/CodeSignConsoleApp/blob/main/BuildAndReleaseScripts/Notarize.sh) script to notarize.

#### GitHub Action File
This Action Has to Run on MacOS **NOTE: MacOS GitHub action minutes cost 10x regular minutes.**

We also have to use dotnet 6. It is the only one that supports notarizing for MacOS. (this blog was written before it was GA, so the include-prerelease has to set to true.

```
name: MAC Signing

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: macos-latest

    steps:
    - uses: actions/checkout@v2

    - name: Set up .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '6.0.x'
        include-prerelease: true
```

we then run dotnet publish for Mac OS where "ConsoleApp1/ConsoleApp1.csproj" is the path to your csproj. 

**Note we are running this for osx-x64 you can change this for the new arm machines as well**

```
    - name: dotnet publish
      run: dotnet publish ConsoleApp1/ConsoleApp1.csproj -c Release -r osx-x64 -p:UseAppHost=true -p:PublishSingleFile=true --self-contained true -p:PublishReadyToRunShowWarnings=true  -o ${{env.DOTNET_ROOT}}/consoleapp
```

**Note: we set the output of the publish to ${{env.DOTNET_ROOT}}/consoleapp**

now we delete the .pdb files, since they are not used. 

```
    - name: delete .pdb
      run: rm ${{env.DOTNET_ROOT}}/consoleapp/*.pdb
```

Then we make the console application file executable by runinning where "ConsoleApp1" is your file name. 

```
    - name: chmod 
      run: chmod +x ${{env.DOTNET_ROOT}}/consoleapp/ConsoleApp1
```

Now we have to add the certificate to the keychain to use it to sign the application.

```
    - name: Add Cert to Keychain
      uses: apple-actions/import-codesign-certs@v1
      with: 
        p12-file-base64: ${{ secrets.MAC_CERT_BASE64 }}
        p12-password: ${{ secrets.MAC_CERT_PASSWORD }}
```

Then we run [this](https://github.com/coding-flamingo/CodeSignConsoleApp/blob/main/BuildAndReleaseScripts/SignMac.sh)  signing script passing the following variables:
1. RunFile "./consoleapp/ConsoleApp1"
2. Directory ./consoleapp/*
3. Your Certificate name "Developer ID Application: YOURORG (YOURDEVID)" 
4. Your Entitlements file, you can find a sample one with the minimum required things for dotnet [here](https://github.com/coding-flamingo/CodeSignConsoleApp/blob/main/BuildAndReleaseScripts/entitlements.plist) ./entitlements.plist 

```
    - name: Sign Binaries 
      run: "sh BuildAndReleaseScripts/SignMac.sh \"${{env.DOTNET_ROOT}}/consoleapp/ConsoleApp1\" \"${{env.DOTNET_ROOT}}/consoleapp/*\" \"Developer ID Application: CodingFlamingo (HVPR40Y9IG)\" \"BuildAndReleaseScripts/entitlements.plist\""
```

Then we have to zip the folder to send it to Apple to notarize it:

```
- name: Zip Binary for Notarizing
      run: zip -rj consoleapp.zip ${{env.DOTNET_ROOT}}/consoleapp/*
```

Then we send it to Apple to get notarized with [this](https://github.com/coding-flamingo/CodeSignConsoleApp/blob/main/BuildAndReleaseScripts/Notarize.sh) script passing the following variables:
1. dev_account  the email you use for your apple ID
2. dev_Password the apple password we created for you account and saved in the GitHub secrets.  "${{ secrets.MAC_DEV_PASSWORD }}"
3. The group ID you created for your application in Apple Dev account $3 "group.com.company" 
4. dev_team This is the Dev ID (it is in parenthesis in your certificate name) DEVID
5. FileName the file we just zipped in the previous step. ./filename.zip


```
    - name: Notarize Binaries 
      run: "sh BuildAndReleaseScripts/Notarize.sh \"codingflamingo@gmail.com\" \"${{ secrets.MAC_DEV_PASSWORD }}\" \"group.com.codingflamingo\" \"HVPR40Y9IG\" \"./consoleapp.zip\""
```

Finally we upload the artifacts.
```
    - name: Upload artifact for deployment job
      uses: actions\upload-artifact@v2
      with:
        name: MyConsoleApp-MacOS
        path: ${{env.DOTNET_ROOT}}/consoleapp
```

And that is how you Build, Sign, and Notarize .NET console app for MacOS.