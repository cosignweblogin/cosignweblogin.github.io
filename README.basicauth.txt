Before you begin:

    Starting with Cosign 2.0, you can use any arbitrary external
    authenticator as the authentication back end for cosign. The
    cosign.conf(5) man page has more details.

    Implementing an external authenticator is prefereable because it
    allows for a full featured user interface with good opportunities
    for feedback and interaction.
    
    These instructions are for people who want to leverage Apache based
    authentication methods to bring up cosign.
    
    For documentation and information see: http://weblogin.org/

    Your web server should have SSL enabled.

    You will need OpenSSL 0.9.7a or newer.

    You will need a source of entropy for the OpenSSL libraries to
    work.  If your system has /dev/*random then you're all set,
    otherwise you should get something like prngd or egd.  Solaris
    users should refer to document 27606 "Differing /dev/random
    support requirements within Solaris [TM] Operating Environments"
    at <http://sunsolve.sun.com/>.  AIX users will want to get
    prngd.


To build the central cosign server ( the weblogin server ):
    Note that you should only need this if you are establishing a new
    SSO community for which you will be providing the central login
    server.

	./configure
	make everything
	make install-all
	mkdir -p /var/cosign/daemon
	chown DAEMON_USER /var/cosign/daemon
    
    The daemon (cosignd) requires  /var/cosign/daemon to exist and be
    writeable by the user the daemon runs as, by default "cosign".

Testing your certificates:

    You want to make sure that the certs you have are able to be used as
    both a server ( weblogin server ) and a client ( cosign.cgi ). Debugging
    certificate problems is the hardest thing to do, and checking now saves
    a lot of anguish later.

	openssl verify -verbose -purpose sslclient -CApath /var/cosign/certs/CA /var/cosign/certs/cert.pem

	openssl verify -verbose -purpose sslserver -CApath /var/cosign/certs/CA /var/cosign/certs/cert.pem

Configuring your server:
    You'll want something like the following

	#this is so the login will run from / of DocRoot
	DirectoryIndex      cosign.cgi index.html 
	AddHandler          cgi-script      .cgi
	<Directory /path/to/docroot>
	    Options ExecCGI
	</Directory>

	#so anything they ask for will take them to the authentication script
	ErrorDocument       404     https://cosign.example.com/

    Otherwise configure your server in the regular way you would to
    make the type of BasicAuth you want to use work in your environment.

    The BasicAuth flavor of cosign.cgi checks that REMOTE_USER has
    already been set by BasicAuth and take the appropriate
    login/register actions.

Customizing your html:

    We provide an example login screen and other templates which you are
    welcome to use but you'll probably want to customize. Be careful
    with the variable display strings ( of the form '$a' where 'a' can
    be any letter or number ).  These are needed by the CGI. See the
    comprehensive cosign scheme document at http://weblogin.org for
    details on the templates and variable substitutions required by the
    CGI.

Things to know about the CGI:

    The required html and templates for the cgi ( login page, logout
    page, services page, looping page etc. ) live in ${prefix}/html and
    ${prefix}/templates. You will most likely want to localize these
    pages.

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

./configure options:

    --with-cosignhost=NAME  	default=cosign.example.edu
    --with-cosignlogouturl=URL	default=http://cosign.example.edu
    --with-cosignloopurl=URL	default=http://cosign.example.edu/looping.html
    --with-cosigndb=DIR 	overrides /var/cosign/daemon
    --with-cosignconf=FILE	specify new conf file location
    --with-cosigncadir=DIR	default=/var/cosign/certs/CA 
    --with-cosigncert=FILE	default=/var/cosign/certs/cert.pem
    --with-cosignkey=FILE	default=/var/cosign/certs/key.pem
    #these next 2 are not necessary for BasicAuth
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
        ( or pass ) was 1.2 per second. More information follows in
        the cosignd(8) and monster(8) man pages.

Questions? cosign@umich.edu
