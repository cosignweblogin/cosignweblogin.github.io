These instructions are for building the Apache authentication filter.
If you need to set up an entire weblogin infrastructure please see
README.weblogin (for sites using Kerberos) or README.basicauth for any
other AuthN backend.

Before you begin:

    For documentation and information see: http://weblogin.org/

    To build a production Apache filter you need Apache 1.3.X.  Support
    for Apache 2.0 is available, but not used in production at Michigan.
    If you happen to use the Apache 2.x filter in production, please let
    us know about it. Your web server should have SSL enabled.

    You will need OpenSSL 0.9.7a or newer.

    You will need a source of entropy for the OpenSSL libraries to work.
    If your system has /dev/*random then you're all set, otherwise you
    should get something like prngd or egd.  Solaris users should refer
    to document 27606 "Differing /dev/random support requirements within
    Solaris [TM] Operating Environments" at <http://sunsolve.sun.com/>.
    AIX users will want to get prngd.

    If your cosign protected service needs kerberos credentials, you
    will need MIT krb5-1.2.7 or later.  Kerberos libraries are not
    required if you do not need access to kerberos credentials.  If you
    are building a central weblogin server, see README.weblogin.

To build the Apache authentication filter:

    NOTE: On Redhat 9, kerberos is in a  non-standard place, and so, by
    default, configure may not find it. So if you need to use kerberos
    ticket transfers, you will need to do the following.

    Add "env CPPFLAGS=-I/usr/kerberos/include" before you run configure.
    So in csh your configure line will look  like this:

	env CPPFLAGS=-I/usr/kerberos/include ./configure

    and in bash or sh you'd type:

	CPPFLAGS=-I/usr/kerberos/include ./configure

To build:

    ./configure
    make
    make install
    mkdir -p /var/cosign/filter
    chown APACHE_USER /var/cosign/filter

    'make install' will install the filter using your copy of apxs.  Be
    sure to change APACHE_USER to the username defined in your
    httpd.conf file.

    Finally, create a CA directory to hold your CA certificates.  Copy
    the CAs (see the CAcerts directory in the root of the cosign source
    distribution) to your CA dir and issue the c_rehash command
    (c_rehash is a perl script that ships with openssl).  If you choose
    to store your certs in '/usr/local/etc/apache/certs' then the
    commands would be:

	mkdir -p /usr/local/etc/apache/certs
	cp CAcerts/* /usr/local/etc/apache/certs
	c_rehash /usr/local/etc/apache/certs

    output should look like:

	Doing /usr/local/etc/apache/certs
	umwebCA.pem => 4700e8dd.0
	RSA-SSCA.pem => f73e89fd.0
	entrust.pem => ed524cf5.0

Configure Apache ( U of M specific example ):

    If you need a certificate, we can provide one signed by the umweb CA at
    no cost. Self-signed certs will not work wih Cosign.

    NOTE: When filling out the CSR, you will be prompted for the CN. It
    says "Common Name (eg, YOUR name) []:" - despite what the prompt
    says, you should type in the hostname of your webserver, eg
    "deptwebserver.dept.umich.edu". Additionally, the state field should
    read "Michigan" and not "MI".

    In the U of M environment, you'll want your directives to look like
    this:

    On your http ( port 80 ) side, and any dirs or locations you want
    exempt:

	CosignProtected		Off

    in :443 ( or otherwise https ) vhost

    CosignProtected		On
    CosignHostname		weblogin.umich.edu
    CosignRedirect		https://weblogin.umich.edu/
    CosignPostErrorRedirect https://weblogin.umich.edu/cosign/post_error.html
    CosignService		<use what remains after dropping .umich.edu from the ServerName>
    CosignCrypto		/path/to/key /path/to/cert /path/to/CAdir

    NOTE: trailing slash is required on CosignRedirect! The redirects
    won't work correctly without it.

    ANOTHER NOTE: most mod_cosign options can be configured on a
    per-directory basis. In order to have this flexibility, the
    server-wide configurations must be listed "first" in the httpd.conf
    file, before any per-directory configuration options.

Stop and Start Apache

See README.scripts for a cron job that prunes old cookies from the
filter's database and scripts for local logout.

--------
Apache Configuration Options:

	CosignProtected		[ on | off ]
	    governs whether Cosign is invoked or not

	CosignHostname		[ the name of the host running cosignd ]
	CosignRedirect		[ the URL of the cosign login cgi ]
	CosignPostErrorRedirect	[ the URL to redirect to if the user
				would be redirected to the login cgi
				during a POST. This screen lets people
				know we dropped their data. ]
	CosignService		[ the name of the cosign service cookie ]
	CosignSiteEntry		[ the URL to redirect to after login  ]
	CosignCrypto		[path to key] [path to cert] [path to CA dir]
	CosignRequireFactor	[ a list of the factors a user must satisfy ]
	CosignFactorSuffix	[ optional factor suffix when testing
				for compliance ]
	CosignFactorSuffixIgnore	 [ on | off ]
	CosignHttpOnly		[ on | off ]
		module can be use without SSL - not recommended!
	CosignTicketPrefix	[ the path to the Kerberos ticket store ]
	CosignFilterDB		[ the path to the cosign filter DB]
	CosignFilterHashLength	[ 0 | 1 | 2 ]
	    subdir hash for cosign filter DB
	CosignCheckIP           [ never | initial | always ]
	    check browser's IP against cosignd's ip information
	CosignProxyDB		[ the path to the cosign proxy DB]
	CosignAllowPublicAccess		[ on | off ]
	    make authentication optional for protected sites
	CosignGetKerberosTickets	[ on | off ]
	    module asks for tgt from cosignd
	CosignKerberosSetupGSS		[ on | off ]
	    setup the enviornment so that other apache modules
	    that need GSSAPI/Kerberos work. e.g. IMP running under mod_php
	CosignGetProxyCookies	[ on | off ]
	    module asks for proxy cookies from cosignd

The certificate CN of the weblogin server must match CosignHostname.

--------
./configure may take the following options:

    --enable-krb=path_to_krb5		enables Kerberos V
    --enable-apache1=path_to_apxs_1.3	enables Apache 1.3 filter
    --enable-apache2=path_to_apxs_2	enables Apache 2 filter
    --with-GSS				enables GSSAPI
    --with-filterdb=DIR         	overrides default of /var/cosign/filter

To make Cosign work with Apache's require directives, you should do the
following:

In httpd.conf, for the directory structure you want to Cosign Protect
and use apache's require directives on, add the following line:

    #this examples turn this on for everything
    <Directory />
	AllowOverride AuthConfig Limit
    </Directory>

This will allow you to use .htaccess files for Authorization. So now you
can do something like this:

    AuthType Cosign
    require user wes canna clunis jarod

This will make it so the core cosign development team can view the pages
in this hypothetical directory. For more info on this, see Apache's docs
on .htaccess.

Alternatively you can just specify authorization information inline in
httpd.conf:

    <Directory />
	AuthType Cosign
	Require user grouch harpo chico
    </Directory>

This allows the marx brothers to exclude Zeppo ( and anyone else ) from
accessing content on their webserver.
