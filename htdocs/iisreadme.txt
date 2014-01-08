Copyright (c) 2002, 2003, 2004 Regents of The University of Michigan.
All Rights Reserved.  See COPYRIGHT.

+--== What to do next ==--+

   This installer uses a certificate authority cert (the umwebca.pem
   file) issued to the University of Michigan.  If you are not setting
   up a server for use in the U-M computing environment this file is
   useless to you.  Contact the administrator for your particular
   institution's weblogin server to obtain an appropriate certificate
   authority file.


1) E-mail webmaster@umich.edu (or the webmaster for your institution)!
   If you chose to generate your certificates during the install you
   will find a file ending with the extension *.csr in your 
   \IISCosign\SSL folder.  Open this file with Wordpad and cut-and-
   paste the contents into your e-mail.  Send a nice message asking
   the webmasters to sign your csr.  Also include the name of the
   server this certificate will be residing on as well as the name of
   your cosign service.


2) When you get your signed certificate back take the 'cert' part of
   the e-mail and put it into the file called
   [my.servername.umich.edu].cert.  (This file was auto-generated for
   you.)  The cert part is the data between and including the lines:

-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----

   Be sure the line with BEGIN CERTIFICATE is the very first line of
   the *.cert file.


3) Open up the Internet Services Manager.  Stop the web site you
   would like to have protected by cosign.


4) Right-click on the site you just stopped and select 'Properties.'
   Select the 'ISAPI Filters' tab.  Click 'Add.'  Browse or type
   in the path where you placed Cosign.dll and name it "Cosign
   Filter."


5) Restart the web site.  It is now filtered!  Try visiting your
   site:  make sure you're properly redirected to the weblogin
   server and are then redirected back with authenticated access.
   PLEASE NOTE: Some servers require you to Restart IIS for the filter
   to load properly.


+--== What else to do ==--+

If you didn't generate SSL certs using the installer then you have much
more to do.  If you want to use certificates that are in your Windows
local certificate repository you will have to export your key and cert
as files so that IISCosign can use them.  After that, update your
cosign.dll.config file with the appropriate file paths to the certs.
You'll also need to add a service name to the file.

+--== Using Windows 2003?  You need to set some permissions ==--+

The following permissions need to be set for Windows 2003 (this is not
necessary for a Win2k/IIS5 setup):

1) For your entire \IISCosign\ folder:
   IIS_WPG needs needs everything selected except for "Special
   Permissions."  Be sure to select the option that applies these
   permissions to the folder contents and sub-folders.

2) For the \CookieDB\ folder:
   Internet Guest Account, IUSR_[machine name] needs Read and Write
   selected.

3) For the \Logs\ folder:
   IUSR_[machine name] - Read & execute, list folder contents, Read,
   and Write.


+--== Using OpenSSL certificates for https ==--+

IISCosign requires https.  In IIS Manager this is done under the
Directory Security tab in your web site settings.  If you already have a 
certificate for this purpose then you are all set.  If you have your own
Windows Certificate Authority setup, you can use that as well.  Finally,
if none of these options are available to you you can reuse your openssl
generated certificate for this.

Here's how:

1) Have your webmaster sign your openssl generated CSR.  (The creation
   of a private key and csr was done automatically for you.  If you chose
   to skip this step in the setup see +Using Openssl.exe to create
   certificates+.)

2) Open a command prompt and cd to your /iiscosign/ folder.

3) Run the following command:

openssl pkcs12 -in [server.name.umich.edu.cert] -inkey
  [server.name.umich.edu.key] -export -out [server.name.umich.edu].p12

   You will be prompted for a password.  Remember this password!  You
   will need it later.

4) Run mmc.exe.

5) Open the certificates console.
   a. Go to File | Add/Remove Snap-in...
   b. Click Add...
   c. Select Certificates.  Click Add.
   d. Select My Computer Account.  Click next.
   e. Select Local Computer.  Click Finish.
   f. Click Close.
   g. Select Certificates (Local Computer).  Click OK.

6) Expand Certificates (Local Computer)->Personal->Certificates.

7) Right click on Certificates (the final one in the tree).  Select
   All Tasks | Import...

8) Click next.  Browse for the UMWebCA.pem file.  (If you can't see
   the file make sure Files of Type is set to *.*)  Click Ok.  Click
   Next, Next, the Finish.

9) Right click on Certificates (the final one in the tree).  Select
   All Tasks | Import...

10) Browse for the *.p12 file you generated in step 3.  Click Next.

11) Enter the password you used in step 3.  Click Next.  Click Next.
    Click Finish.

Now when you open up IIS Manager and select Server Certificate under
the Directory Security tab you can select "Assign an existing
certificate" option.  The certificate you just imported should be
listed.


+--== Using Openssl.exe to create certificates ==--+

If you skipped the auto-generating cert feature of the installer you can
still use the openssl utility to do this.

1) Open a command prompt and cd to your IISCosign folder.  Assuming you
   opted to install the openssl component you should see the openssl
   executable in this folder.

2) Run the following commands:

openssl genrsa 1024 > [server.name.umich.edu].key
openssl req -new -config umconfig.cfg -key [server.name.umich.edu].key -out [server.name.umich.edu].csr 

3) When prompted for information, please provide the appropriate data. The
   common name will be the name of your server and the contact e-mail
   should be the e-mail of the person to be notified when the certificate
   is about to expire.

4) Send an e-mail to webmaster@umich.edu. Cut-and-paste the contents of the
   *.csr file into your e-mail.

5)  Place the UmWebCA.pem file, *.key file you generated, and the *.cert file
    you receive from the webmaster into your C:\[cosign path]\SSL\ folder. 

6)  Update your XML configuration file to point to these files.
     
    The CAFilePath item points to the umwebca.pem file.
     
    The ChainFilePath item specifies the [yourservername].cert file that you
    received from the U-M webmaster.

    The PrivateKeyFilePath item points to the [yourservername].key private
    key file that you generated. Be sure to include the full path such as
     C:\Program Files\CosignFilter\SSL\servername.key.

7)  See +What to do next+ section of this readme.


+--== Your certificate expired and you need it re-signed ==--+

At the University of Michigan we sign our certificates for a one year period.
This will probably be different for your institution or business.  Generating
a new certifcate signing request is easy.  Here's how:

1) Open a command prompt and cd to your \iiscosign\ folder.

2) Run the following command:

openssl req -new -config \SSL\sslconfig.cfg -key [server.name.umich.edu].key -out [server.name.umich.edu].csr 

   NOTE: If you generated your certificates automatically on install, you can
   reuse the sslconfig.cfg file.  If not, you will need to edit the umconfig.cfg
   file.

3) Follow the steps in +What to do next+ for having your CSR signed.  You will
   replace the contents of your current *.cert file with the certificate sent
   back to you by your webmaster.

+--== You already have a private key in your MS store and would like to use it with IISCosign ==--+

1) Using the Certificates Manager in MMC (Start Menu -> Run -> then type mmc, press enter), export
   your key.  Make sure "Yes, export the private key" is selected.  You will be asked for to create
   a password.  Be sure to remember this password!

2) Open a command prompt, go to C:\program files\iiscosign\
   Run this command:

   openssl pkcs12 -nodes -in <exported_file_name.pfx> -out <exported_file_name.pem>

3) You will be asked for the password you used above.

4) Open the *.pem file with Wordpad.

5) The file can now be split into the private key and public certificate.  The private key is
   designated by:

-----BEGIN RSA PRIVATE KEY------
-----END RSA PRIVATE KEY-----

   Copy these two lines and everything inbetween them and save them into a file such as
   exported_file_name.key.

   The public certificate is designated by the lines:

-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----

   Save these lines and everything inbetween in a separate file such as
   exported_file_name.cert.

   It's probably most convenient to place this in your
   c:\program files\iiscosign\ssl folder.

7) Edit cosign.dll.config to have <PrivateKeyFilePath> to point to the file containing the
   private key.

8) Edit cosign.dll.config to have <ChainFilePath> point to the file containing the cert.

9) You can now safely delete the exported .pfx and .pem files.

Microsoft Knowledge Base article Q313299 has some more info on exporting keys.
http://support.microsoft.com/default.aspx?scid=kb;en-us;313299#Task3


+--== You would like to know more about SSL ==--+

http://www.modssl.org/docs/2.8/ssl_intro.html


+--== About IISCosign ==--+

This project was created using Microsoft Visual Studio.NET 2003

If you would like to compile the IISCosign filter yourself you
will also need libsnet, getopt, and OpenSSL.  The first two are
available from http://rsug.itd.umich.edu, the latter can be
downloaded from http://www.openssl.org.

IISCosign also uses MSXML 4.0.
http://msdn.microsoft.com/downloads/default.asp?url=/downloads/sample.asp?url=/msdn-files/027/001/766/msdncompositedoc.xml

E-mail any questions or comments to cosign@umich.edu.

