---
title: "IOS Workflow"
date: 2023-08-13T16:45:23+05:30
Summary: "How manually setup IOS workflow. This is just a brief through on commands and flows we needed. Intended to people who knows how to setup workflow in general but don't know about ios builds and what all commands and certificates it needed."
---

As we scale our product we will be using different workflows to automate our tasks like workflow to build the app or push the latest code to servers. When we change our business to a B2B app we also need to make our app whitelisted for our merchants and building all of the app manually is not a good solution. So we also started using workflows for testing and building apps.

One of the major challenges I faced at this time was making a workflow to build a release app for our React native IOS app 🥲.  We also know, unlike Android which is just a simple command to build the APK or app bundle. Building an IPA file in IOS is a huge pain. There are multiple certificates and multiple commands that we need to do and we have to change some settings in the Xcode too.

> Mostly this is because developers are unaware of the process that happen in Xcode when we build an IPA file. Xcode done most of the works automatically. 

>The best and easy method to do a workflow for your IOS workflow is going with the Xcode cloud. For some reason, you don’t want to use it and makes your life harder. Good for you.🤣

To make a manual workflow for IOS you need to know what all things are required and what all terminal commands we need to work a specific job that we do in Xcode.

In the signing capabilities of your target maybe using automatically manage signing. This means all those overhead issues with the certificate of our app are managed by Xcode itself.


![image alt text](/xcode_signin.png)

Since we are going to handle all of these things by ourselves let's uncheck it. 🤣

Once you do the unchecking the Xcode will through an error mentioning the **provisioning profile.**

Our next step is to download and manage this profile.

>Warning: we need multiple provisioning profiles if your project has multiple targets.

You need to go to the profile section in your Apple developer account and create a new profile. It's best to create a separate profile for your purpose. So if something is been breached and someone tried to build a version of your app without you knowing u can just remove this profile and it’s done.

> If you have multiple targets then you need to create multiple provisioning profiles.

This method may be a little bit of hard work to do but as you can see when you are creating the provisioning profiles it also gives you a serious level of control over your project and what people can do with it.

Once you created the provisioning profiles you can download them and upload them in your Xcode. You will see the provisioning profiles section in the signing and capabilities.

> If you have multiple targets then you must upload them in all of the targets.

With this, your app will be able to run and build normally like before. (If something goes wrong, Please check the provisioning profiles you uploaded to Xcode).

# Certificates

Now we need to create an app distribution certificate (a p12 file). For that go to your certificate section in your Apple developer account and download the Distribution type certificate which mentioned your team name. You will be getting a cer file. Double-click the cer file which will open you to the keychain application. In the keychain application go to the my certificate tab of the login section. There you will see the certificate you just download with your team ID. Right-click on it and export the keychain. You can choose p12 file format and password from the prompt.

Create a file folder named provisioning name place your provisioning profile and p12 certificate your just exported. (My p12 file name is Certificates.p12).

Create a file named import_provisioning.sh and copy the below code. This script will do the setting up of the provisioning profiles and certificate of your project. (Replace the PROVISIONING_PASSWORD variable with the password of your p12 file)

```sh
#!/bin/sh

mkdir -p ~/Library/MobileDevice/Provisioning\ Profiles

echo "List profiles"
ls ~/Library/MobileDevice/Provisioning\ Profiles/
echo "Move profiles"
cp provisioning/*.mobileprovision ~/Library/MobileDevice/Provisioning\ Profiles/
echo "List profiles"
ls ~/Library/MobileDevice/Provisioning\ Profiles/


security create-keychain -p "" build.keychain
security import provisioning/Certificates.p12 -t agg -k ~/Library/Keychains/build.keychain -P "$PROVISIONING_PASSWORD" -A

security list-keychains -s ~/Library/Keychains/build.keychain
security default-keychain -s ~/Library/Keychains/build.keychain
security unlock-keychain -p "" ~/Library/Keychains/build.keychain
security set-key-partition-list -S apple-tool:,apple: -s -k "" ~/Library/Keychains/build.keychain
```

# Running the project

Since we are using cocopod in our IOS project we run the project using the workspace so our command is changed to support that.

```shell
xcodebuild -workspace <workspace file location> -scheme <Your schema name> -configuration Release -archivePath <output path>  -destination 'generic/platform=iOS'
```

> When you are executing Xcode commands and it shows you the weird error of things that you already covered up. The easiest fix is “Restart your MAC”🤣. 
This will create archive data for your app.

Create a file named appstore.plist and copy the code below.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>provisioningProfiles</key>
    <dict>
        <key>TARGET_BUNDLE_IDENTIFIER</key>
        <string>UUID_OF_provisioning_Profiles</string>
    </dict>
    <key>method</key>
    <string>ad-hoc</string>
    <key>signingStyle</key>
    <string>manual</string>
    <key>stripSwiftSymbols</key>
    <false/>
    <key>teamID</key>
    <string>YOUR_TEAM_ID</string>
    <key>uploadSymbols</key>
    <false/>
    <key>thinning</key>
    <string>&lt;none&gt;</string>
</dict>
</plist>
```
> You can get the provisioning Profiles UUID by right-clicking the provisioning Profile and goto the get info section. You will see the preview in the get info where you will see UUID.

> If you have multiple targets in your project then you need to add the provisioning profile. In the “provisioningProfiles” key have values of dict of key and string values. The key will be the target’s bundle identifier and the value will be the UUID of the provisioning Profiles.

```shell
xcodebuild -exportArchive -archivePath <archive path> -exportOptionsPlist <Your appstore.plsit dir> -exportPath <output dir>
```
> To know about the “archivePath” in the above command you need to go to the output file of the previous command, where you create the archive. You will find a file name <project_name>.xcarchive. 

With this, you will get the IPA file of your IOS project. 😁