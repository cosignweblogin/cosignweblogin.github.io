Before you begin:

    For documentation and information concerning cosign terminology,
    see: http://weblogin.org/

    Your web server should have SSL enabled.

    If you want to use Kerberos, you will need MIT krb5-1.2.7 or
    later. The Cosign Kerberos login cgi can use either an MIT KDC
    or a Microsoft Active Directory KDC.

    You will need OpenSSL 0.9.7a or newer.

    Cosign can optionally work with kct
    (http://www.citi.umich.edu/projects/kerb_pki/) and 
    Shibboleth (http://shibboleth.internet2.edu/).

    Guest account functionality, provided by cosign friend, requires 
    MySQL. Download and further details for friend provided at
    http://weblogin.org

    Cosign 2.0 and later can utilize arbitrary external programs for
    authentication. See the "factor" section of cosign.conf(5).

    The cosign filters and the weblogin server use SSL mutual
    authentication.  It may be easier & cheaper to run your own CA
    to manage the required certificates.

    You will need a source of entropy for the OpenSSL libraries to
    work.  If your system has /dev/*random then you're all set,
    otherwise you should get something like prngd or egd.  Solaris
    users should refer to document 27606 "Differing /dev/random
    support requirements within Solaris [TM] Operating Environments"
    at <http://sunsolve.sun.com/>.  AIX users will want to get
    prngd.

Customizing your html:

    The html & graphics are provided as examples.  You'll probably want
    to customize them for your site.  Be careful with the variable
    display strings ( of the form '$a' where 'a' can be any letter or
    number ). These variables are replaced by the CGI during execution.

    See the comprehensive cosign scheme document at http://weblogin.org
    for details on the templates and variable substitutions required by
    the CGI.

    Also note that 'make install-all' overwrites existing html,
    templates, and graphics.

To build the central cosign server ( the weblogin server ):

    Note that you should only need this if you are establishing a new
    SSO community for which you will be providing the central login
    server.

	./configure --enable-apache1=/path/to/apxs --enable-krb=/path/to/krb5
	make everything
	make install-all
	mkdir -p /var/cosign/daemon
	chown DAEMON_USER /var/cosign/daemon
    
    --enable-apache2=/path/to/apxs may also work.  The daemon
    (cosignd) requires  /var/cosign/daemon to exist and be writeable
    by the user the daemon runs as, by default "cosign".

    Also, in order to avoid the warning:

	cosignd: can't find cosign service 

    add cosign to /etc/services on port 6663/tcp ( the default )
    or whatever port you are running it on.

    If you are using Kerberos, create cosignticketcache (by default
    /ticket) and change the permissions so both the webserver and
    the daemon can write there.

    Also, if you're using Kerberos, you'll probably want a keytab
    with the principal of "cosign" and the instance of the hostname
    of the machine that the cgi will run on. See cosign.conf(5).

Configuring Apache:

    To enable the cosign apache module, see README.

    Assuming a prefix of /usr/local/cosign, a simple apache
    configuration using the supplied html & templates might be:

    <VirtualHost *:443>
	SSLEngine		On
	SSLCertificateFile	/path/to/server.cert
	SSLCertificateKeyFile	/path/to/server.key

	CosignProtected		Off
	CosignHostname		cosign.edu
	CosignRedirect          https://cosign.edu/cosign-bin/cosign.cgi
	CosignPostErrorRedirect https://cosign.edu/cosign/post_error.html
	CosignService           simpleservice
	CosignCrypto            /path/to/server.key /path/to/server.cert /path/to/CA-dir

	Alias /cosign/ "/usr/local/cosign/html/"
	ScriptAlias /cosign-bin/ "/usr/local/cosign/cgi-ssl/"

	Alias /services/ "/usr/local/cosign/services/"
	<Directory "/usr/local/cosign/services">
	    CosignProtected On
	</Directory>
    </VirtualHost>

    In the above configuration the login URL is:

	https://cosign.edu/cosign-bin/cosign.cgi

    and the logout URL is:

	https://cosign.edu/cosign-bin/logout

    It's possible to simplify the login URL with this configuration:

        CosignRedirect          https://cosign.edu/
	DocumentRoot		"/usr/local/cosign/cgi-ssl"
	<Directory /usr/local/cosign/cgi-ssl>
	    DirectoryIndex      cosign.cgi
	    AddHandler          cgi-script      .cgi
	    Options ExecCGI
	</Directory>

    Note that the trailing '/' in CosignRedirect is required in
    this configuration.

Creating cosign.conf file:

    The last thing you need to do before starting up your cosign server
    is create the cosign.conf file. Please see cosign.conf(5) for
    details.

Scripts

    See the scripts/ directory in the cosign source distribution for an
    example cosignd startup script, a cron job to clean up your cookie
    database (if you are not using replication), and several example
    logout scripts. There are also scripts provided to create and
    convert the directories in the cosign database if you choose to use
    directory hashing. Finally, there is an example of an external
    authenticator script located in the factors sub-directory.

Configure Options:

    --with-cosignhost=NAME  	default=cosign.example.edu
    --with-cosignlogouturl=URL	default=http://cosign.example.edu
    --with-cosignloopurl=URL	default=http://cosign.example.edu/looping.html
    --with-cosigndb=DIR 	overrides /var/cosign/daemon
    --with-cosignconf=FILE	specify new conf file location
    --with-cosigncadir=DIR	default=/var/cosign/certs/CA 
    --with-cosigncert=FILE	default=/var/cosign/certs/cert.pem
    --with-cosignkey=FILE	default=/var/cosign/certs/key.pem
    --with-ticketcache=DIR	default=/ticket
    --with-keytabpath=FILE 	default=NULL ( which means use whatever
					the krb5.conf says to use )
    --enable-mysql=path_to_mysql
		      enable mysql for guest login support in the cgi
    --with-frienddbhost=NAME
			    default=localhost
    --with-frienddblogin=NAME
			    default=friend
    --with-frienddbpasswd=PASSWD
			    no default

    The certificate CN of the weblogin server must match the argument
    to --with-cosignhost.

Rate Logging:

    Starting with 1.7, we've simplified the logging paradigm. There's
    a file in common/ called rate.h and a #define of RATE_INTERVAL.
    This means that every RATE_INTERVAL number of events, cosignd
    will write out a summary log line that shows the rate for that
    particular event. For example, for the CHECK command you'd see:

	STATS CHECK 141.211.144.17: UNKNOWN .0012 / sec
	STATS CHECK 141.211.144.17: PASS 1.2 / sec

    and that would indicate that the rate of 5xx ( or unknowns )
    returned to that host was .0012 per second and the rate of 2xx
    ( or pass ) was 1.2 per second. More information follows in the
    cosignd(8) and monster(8) man pages.

Re-Authentication:

    See cosign.conf(5).

x509 logins:

    See cosign.conf(5).

External Authenticators:

    See cosign.conf(5).

Testing your certificates:

    Make sure that the certs you have are able to be used as both
    a server ( weblogin server ) and a client ( cosign.cgi ).
    Debugging certificate problems is the hardest thing to do, and
    checking now saves a lot of anguish later.

	openssl verify -verbose -purpose sslclient -CApath /var/cosign/certs/CA /var/cosign/certs/cert.pem

	openssl verify -verbose -purpose sslserver -CApath /var/cosign/certs/CA /var/cosign/certs/cert.pem

Questions?

    cosign-discuss@umich.edu
