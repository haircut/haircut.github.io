---
layout: post
title: How to Manage FileVault Recovery Key Escrow
tags: [profiles, filevault, high-sierra]
---
The Jamf Pro GUI allows you to automatically set up the necessary payloads to manage the FDE Recovery Key Escrow process for macOS 10.13+.

However, the settings reside in the "Security & Privacy" grouping within the Jamf Pro GUI, forcing you to manage settings other than those related to recovery key escrow. Using the default Jamf-provided GUI, <span class="hi">you may inadvertently lock your users out of being able to make changes to the firewall, analytics settings, screen saver password requirement, etc.</span>
<!--more-->
You can upload a custom profile to the Jamf Pro Server that manages **only** FDE Recover Key Escrow preferences, but it takes a little work.

You'll also need to sign your resultant configuration profile to prevent the Jamf Pro Server from manipulating its contents or preventing deployment. You can use an Apple Developer certificate, or your Jamf Pro Server's CA (if self signed).

1. Create a temporary configuration profile for doing this setup. Name it something disposable – you'll delete it when done. "TEMP - FDERKE Setup" works. Set the level to "Computer level." Make sure you **do not add a scope** to the profile; we're not deploying it.

2. Click the "Security & Privacy" group, then click "Configure." Select the "FileVault" tab. Apply these preferences:
  - Check the box for **Enable Escrow Personal Recovery Key**
  - **Escrow Location Description**: Describe where the recovery key is being shipped. This is visible to the end-user, so "My Company IT" or whatever is appropriate. We'll edit this elsewhere, so you can leave it blank.
  - **Device Key for Escrowed FileVault Recovery Key**: Text displayed at the FileVault unlock screen when a user has apparently forgotten their password. Despite the help text, you should leave this blank. By default it will be replaced with the device's serial number which will aid your technicians in recovering the correct key.
  - **Personal Recovery Key Encryption Certificate**: Set to "Automatically encrypt and decrypt recovery key."

3. Save the profile, then click the "Download" button.

4. Next we'll convert the profile to a useable format. In Terminal, run these commands:

{% highlight shell %}
openssl smime -inform DER -verify -in /path/to/downloaded/profile.mobileconfig -noverify -out /path/to/de-signed.mobileconfig
plutil -convert xml1 /path/to/de-signed.mobileconfig
{% endhighlight %}
(lines scroll...)

 5. Copy the `template-fde-recovery-key-escrow.mobileconfig` included in this gist to a new file in your favorite text editor. Change the values of `PayloadOrganization` and `Location` as needed.

 6. Open the de-signed profile originally downloaded from the Jamf Pro Server in your text editor. Find the `PayloadContent` below `PayloadCertificateFileName` – it's the big, obvious block of certificate data. Copy and paste this to the same location in your edited `template-fde-recovery-key-escrow.mobileconfig` file, making sure you get the indentation correct. Save this file with a suitable name like `FileVault Recovery Key Escrow.mobileconfig`.

 7. Sign the new profile thusly:

{% highlight shell %}
/usr/bin/security cms -S -N "Common Name of signing certificate in your keychain" -i /path/to/FileVault\ Recovery\ Key Escrow.mobileconfig -o /path/to/Signed-FileVault\ Recovery\ Key\ Escrow.mobileconfig
{% endhighlight %}
(lines scroll...)

 8. Delete the temporary configuration profile from your Jamf Pro Server.

9. Upload your completed `Signed-FileVault Recovery Key Escrow.mobileconfig` profile to your Jamf Pro Server, then set an appropriate scope and deploy it.

 Thanks to [@opragel](https://github.com/opragel) for the [template/example configuration profile](https://github.com/opragel/profiles/blob/master/macOS%20-%20Escrow%20FileVault%202%20Recovery%20Keys.mobileconfig).