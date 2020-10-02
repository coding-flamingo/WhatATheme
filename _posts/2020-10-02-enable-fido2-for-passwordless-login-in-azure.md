---
title: Enable FIDO2 for Passwordless login in Azure
layout: post
post-image: "/assets/images/FIDO2Settings.JPG"
description: FIDO2 is a great leap forward in Security, by having your users use this
  instead of passwords for day to day usage, you can greatly improve your company's
  security posture.
tags:
- FIDO2
- Passwordless
- Azure AD
- AAD
---

Many of the most common hacking attacks start by stealing a password. Since it is hard for admins to prevent their users from falling for phishing attacks, or selecting hard passwords that are not re-used in their other accounts our super easy to crack.The tech community had to come up white a better way to protect our users, this is where FIDO2 comes in.  In this article we will set it up for our Azure users. 
## What is FIDO2
FIDO2 was created by the FIDO ("Fast IDentity Online")  Alliance, which is an association that many companies created in 2013 to help reduce the over-reliance on passwords. 

FIDO2 is based on  standard public key cryptography. Basically the user has a hardware security key that keeps the private key secure (it never leaves the device), and then a public key that is used to verify the privatekey (if you want to learn more about the magic behind public key cryptography [click here](https://engineering.purdue.edu/kak/compsec/NewLectures/Lecture12.pdf)) is given to the server. This means that the server never knows your private key and a compromise of the server will not like your private key.

But enought about the protocol let's talk about how to add it to your Azure environment :)
## Video Version
coming soon :)

## Text Version
### How to Implement it in Azure
1. Login to the Azure portal as a global administrator.
2. Go to Active Directory -> Security ->Authentication methods

![Azure AD Security](/assets/images/Security.png)

3.Click on FIDO2, this will bring up the FIDO2 tab.

![Azure AD FIDO2 Settings](/assets/images/FIDO2Settings.JPG)

4.Click yes on the enable option. 

5.In target you can set it up for all users (That is the Flamingo recommendation ;)) or select a group of users. 

6.Allow self service has to be set to yes. 

7.Make sure Enforce attestation is selected as well (this attests the FIDO2 hardware key).

8.Enforce Key restrictions is if you want to  restrict the keys the users can use (Either whitelist certain manufacturers/keys, or 
blacklist them). Blacklist will become useful when there is a vulnerability like [ROCA](https://en.wikipedia.org/wiki/ROCA_vulnerability) and you have to block a certain device SKU. 

9.If you select to enforce key registrations, you will have to add the AADGUID of the manufacturers you want to allow or deny. To get a list of devices AAGUID you have to visit the manufacturers website. For example, here is the [yubico site ](https://support.yubico.com/hc/en-us/articles/360016648959-YubiKey-Hardware-FIDO2-AAGUIDs)

10.Once you select your settings click save. 

### User self Enroll
1. Tell user to browse to  https://myprofile.microsoft.com
2. Sign in with their account. 
3. Click on Security Info

![My Sign in Security Info](/assets/images/mysigningsecurityinfo.png)

4.The must have at least one MFA method before adding a FIDO2 key. If they don't have one ask them to add an MFA option. 

5.Add the FIDO2 Key by clicking the Add Method button

![My Sign Add Auth Method](/assets/images/addauthmethod.png)

6.Select Security key from the drop down. 

7.Select the type of FIDO2 key device you have.

8.Follow the instructions to register it, if it is a new key it will ask you to set up a pin (this pin will be asked each time you need to unblock the private key, choose a secure pin), if you have used it before it will ask you to enter your pin.

9.Once it is registered, it will ask you to enter a name for this device, this is for you to know which device it is in the my sign-is page.

10.After this you should see your device as an authenticator method. 

![Saved method](/assets/images/savedyubi.png)

11.Go to any AAD protected site [Azure portal](https://portal.azure.com/) for example.

12.Connect your FIDO2 device.

13.Click on "Sign in with a security key"

![Login](/assets/images/estspage.png)

14.Enter your device pin. 

![Login enter pin](/assets/images/loginpin.jpg)

15.Enjoy your new sign in experience.

## FIDO2 Compatible Devices
#### Yubico Security Key - U2F and FIDO2, USB-A, Two-Factor Authentication
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//ws-na.amazon-adsystem.com/widgets/q?ServiceVersion=20070822&OneJS=1&Operation=GetAdHtml&MarketPlace=US&source=ac&ref=qf_sp_asin_til&ad_type=product_link&tracking_id=codingflaming-20&marketplace=amazon&region=US&placement=B07BYSB7FK&asins=B07BYSB7FK&linkId=1018d6b26adb081ad6ec31558354f58b&show_border=false&link_opens_in_new_window=false&price_color=333333&title_color=0066c0&bg_color=ffffff">
    </iframe>

#### Yubico - YubiKey 5 NFC 
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//ws-na.amazon-adsystem.com/widgets/q?ServiceVersion=20070822&OneJS=1&Operation=GetAdHtml&MarketPlace=US&source=ac&ref=qf_sp_asin_til&ad_type=product_link&tracking_id=codingflaming-20&marketplace=amazon&region=US&placement=B07HBD71HL&asins=B07HBD71HL&linkId=cb3fb4d0c4c69e2cb3f0626ba6724e65&show_border=false&link_opens_in_new_window=false&price_color=333333&title_color=0066c0&bg_color=ffffff">
    </iframe>

#### FEITIAN BioPass K27 USB Security Key 
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//ws-na.amazon-adsystem.com/widgets/q?ServiceVersion=20070822&OneJS=1&Operation=GetAdHtml&MarketPlace=US&source=ac&ref=qf_sp_asin_til&ad_type=product_link&tracking_id=codingflaming-20&marketplace=amazon&region=US&placement=B07CHH2NHH&asins=B07CHH2NHH&linkId=1e8833d0c497696cbb9caa5e452b1ab5&show_border=false&link_opens_in_new_window=false&price_color=333333&title_color=0066c0&bg_color=ffffff">
    </iframe>
		
#### Yubico YubiKey 5C
<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//ws-na.amazon-adsystem.com/widgets/q?ServiceVersion=20070822&OneJS=1&Operation=GetAdHtml&MarketPlace=US&source=ac&ref=qf_sp_asin_til&ad_type=product_link&tracking_id=codingflaming-20&marketplace=amazon&region=US&placement=B07HBCTYP1&asins=B07HBCTYP1&linkId=f21502ba2c855045a8c8bd9b05a15235&show_border=false&link_opens_in_new_window=false&price_color=333333&title_color=0066c0&bg_color=ffffff">
    </iframe>