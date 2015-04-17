Authorization Support in the netDF-C Libraries {#auth}
==================================================

\brief It is possible to support a number of authorization schemes
in the netCDF-C library.

With one exception, authorization in the netCDF-C library is
delegated to the oc2 code, which in turn delegates it to the
libcurl library. The exception is that the location of the rc
file can be specified by setting the environment variable *NCRCFILE*.
Note that the value of this environment variable should be the
absolute path of the rc file, not the path to its containing directory.

Following is the authorization documentation.

<!- Copyright 2014, UCAR/Unidata and OPeNDAP, Inc. --> 
<!- See the COPYRIGHT file for more information. --> 
<html> 
<style> 
.break { page-break-before: always; } 
body { counter-reset: H2; font-size: 12pt; } 
h2:before { 
  content: counter(H2) " "; 
  counter-increment: H2; 
} 
h2 { counter-reset: H3; } 
h3:before { 
  content: counter(H2) "." counter(H3) " "; 
  counter-increment:H3; 
} 
h3 { counter-reset: H4; } 
h4:before { 
  content: counter(H2) "." counter(H3) "." counter(H4) " "; 
  counter-increment:H4; 
} 
h5 {font-size: 14pt; } /* For Appendices */ 
h6 {font-size: 16pt; } /* For Subtitles */ 
</style> 
<body> 
 
<center> 
<h1>OC Authorization Support</h1> 
<h6>Author: Dennis Heimbigner<br>
dmh at ucar dot edu</h6>
<h6>Draft: 11/21/2014<br> 
Last Revised: 12/23/2014<br> 
OC Version 2.1</h6> 
</center> 
 
<h6 class="break"><u>Table of Contents</u></h6> 
<ol> 
<li> <a href="#Introduction">Introduction</a> 
<li> <a href="#URL-AUTH">URL-Based Authentication</a> 
<li> <a href="#DODSRC">RC File Authentication</a> 
<li> <a href="#REDIR">Redirection-Based Authentication</a> 
<li> <a href="#URLCONS">URL Constrained RC File Entries</a> 
<li> <a href="#CLIENTCERTS">Client-Side Certificates</a> 
<li> <a href="#allkeys">Appendix A. All RC-File Keys</a>
<li> <a href="#ESGDETAIL">Appendix B. ESG Access in Detail</a>
</ol> 
 
<h2 class="break"><a name="Introduction"><u>Introduction</u></a></h2> 
OC can support user authorization using those provided by the curl
library. This includes basic password authentication as well as
certificate-based authorization.
<p>
With some exceptions (e.g. see the section on <a href="#REDIR">redirection</a>)
The libcurl authorization mechanisms can be accessed in two ways
<ol>
<li> Inserting the username and password into the url, or
<li> Accessing information from a so-called <i>rc</i> file named either
<i>.daprc</i> or <i>.dodsrc</i>
</ol>

<h2 class="break"><a name="URL-AUTH"><u>URL-Based Authentication</u></a></h2> 
For simple password based authentication, it is possible to
directly insert the username and the password into a url in this form.
<pre>
    http://username:password@host/...
</pre>
This username and password will be used if the server asks for
authentication. Note that only simple password authentication
is supported in this format.
Specifically note that <a href="#REDIR">redirection</a> based
authorization will not work with this.

<h2 class="break"><a name="DODSRC"><u>RC File Authentication</u></a></h2> 
The oc library supports an <i>rc</i> file mechanism to allow the passing
of a number of parameters to liboc and libcurl.
<p>
The file must be called one of the following names:
".daprc" or ".dodsrc"
If both .daprc and .dodsrc exist, then
the .daprc file will take precedence.
<p>
Searching for the rc file first looks in the current directory
and then in the home directory (as defined by the HOME environment
variable). It is also possible to specify a direct path using
the <i>-R</i> option to ocprint or using the <i>oc_set_rcfile</i>
procedure (see oc.h). Note that for these latter cases, the path
must be to the file itself, not to the containing directory.
<p>
The rc file format is a series of lines of the general form:
<pre>
[&lt;host:port&gt;]&lt;key&gt;=&lt;value&gt;
</pre>
where the bracket-enclosed host:port is optional and will be discussed 
subsequently.
<p>
The currently defined set of authorization-related keys are as follows.
The second column is the affected curl_easy_setopt option(s).
<table>
<tr><th>Key<th>curl_easy_setopt Option
<tr><td>HTTP.COOKIEJAR<td>CURLOPT_COOKIEJAR, CURLOPT_COOKIEFILE
<tr><td>HTTP.PROXY_SERVER<td>CURLOPT_PROXY, CURLOPT_PROXYPORT, CURLOPT_PROXYUSERPWD
<tr><td>HTTP.SSL.CERTIFICATE<td>CURLOPT_SSLCERT
<tr><td>HTTP.SSL.KEY<td>CURLOPT_SSLKEY
<tr><td>HTTP.SSL.KEYPASSWORD<td>CURLOPT_KEYPASSWORD
<tr><td>HTTP.SSL.CAINFO<td>CURLOPT_SSLCAINFO
<tr><td>HTTP.SSL.CAPATH<td>CURLOPT_SSLCAPATH
<tr><td>HTTP.SSL.VERIFYPEER<td>CURLOPT_SSL_VERIFYPEER
<tr><td>HTTP.CREDENTIALS.USERPASSWORD<td>CURLOPT_USERPASSWORD
</table>
</ul>

<h3><u>Password Authentication</u></h3> 
The key
HTTP.CREDENTIALS.USERPASSWORD
can be used to set the simple password authentication.
This is an alternative to setting it in the url. 
The value must be of the form "username:password".

<h3><u>Cookie Jar</u></h3> 
The HTTP.COOKIEJAR key
specifies the name of file from which
to read cookies (CURLOPT_COOKIEJAR) and also
the file into which to store cookies (CURLOPT_COOKIEFILE).
The same value is used for both CURLOPT values.
It defaults to in-memory storage.

<h3><u>Certificate Authentication</u></h3> 
HTTP.SSL.CERTIFICATE
specifies a file path for a file containing a PEM cerficate.
This is typically used for client-side authentication.
<p>
HTTP.SSL.KEY is essentially the same as HTTP.SSL.CERTIFICATE
and should usually have the same value.
<p>
HTTP.SSL.KEYPASSWORD
specifies the password for accessing the HTTP.SSL.KEY/HTTP.SSL.CERTIFICATE
file.
<p>
HTTP.SSL.CAPATH
specifies the path to a directory containing
trusted certificates for validating server sertificates.
<p>
HTTP.SSL.VALIDATE
is a boolean (1/0) value that if true (1)
specifies that the client should verify the server's presented certificate.
<p>
HTTP.PROXY_SERVER
specified the url for accessing the proxy:
(e.g.http://[username:password@]host[:port])

<h2 class="break"><a name="REDIR"><u>Redirection-Based Authentication</u></a> </h2>
Some sites provide authentication by using a third party site
to to the authentication. One example is
<a href="https://uat.urs.earthdata.nasa.gov">URS</a>,
the EOSDIS User Registration System.
<p>
The process is usually as follows.
<ol>
<li>The client contacts the server of interest (SOI), the actual data provider.
<li>The SOI sends a redirect to the client to connect to the URS system.
<li>The client authenticates with URS.
<li>URS sends a redirect (with authorization information) to send
the client back to the SOI to actually obtain the data.
</ol>
<p>
In order for this to work with libcurl, the client will usually need
to provide a .netrc file so that the redirection will work correctly.
The format of this .netrc file will contain content that
typically look like this.
<pre>
machine uat.urs.earthdata.nasa.gov login xxxxxx password yyyyyy
</pre>
where the machine is the one to which the client is redirected
for authorization, and the login and password are those
needed to authenticate.
<p>
The .netrc file can be specified in two ways.
<ol>
<li> Specify the netrc file to liboc using the procedure in oc.h:
<pre>
oc_set_netrc(OClink* link, const char* file)
</pre>
(This is equivalent to the -N flag to ocprint). 
<p>
<li> Put the following line in your .daprc/.dodsrc file.
<pre>
HTTP.NETRC=&lt;path to netrc file&gt;
</pre>
</ol>
<p>
One final note. In using this, it is probable that you will
need to specify a cookie jar (HTTP.COOKIEJAR) so that the
redirect site can pass back authorization information.

<h2 class="break"><a name="URLCONS"><u>URL Constrained RC File Entries</u></a></h2> 
Each line of the rc file can begin with
a host+port enclosed in square brackets.
The form is "host:port". If the port is not specified
then the form is just "host".
The reason that more of the url is not used is that
libcurl's authorization grain is not any finer than host level.
<p>
Examples.
<pre>
[remotetest.unidata.ucar.edu]HTTP.VERBOSE=1
or
[fake.ucar.edu:9090]HTTP.VERBOSE=0
</pre>
If the url request from, say, the <i>oc_open</i> method
has a host+port matchine one of the prefixes in the rc file, then
the corresponding entry will be used, otherwise ignored.
<p>
For example, the URL
<pre>
http://remotetest.unidata.ucar.edu/thredds/dodsC/testdata/testData.nc
</pre>
will have HTTP.VERBOSE set to 1.
<p>
Similarly, 
<pre>
http://fake.ucar.edu:9090/dts/test.01
</pre>
will have HTTP.VERBOSE set to 0.

<h2 class="break"><a name="CLIENTCERTS"><u>Client-Side Certificates</u></a></h2> 
Some systems, notably ESG (Earth System Grid), requires
the use of client-side certificates, as well as being
<a href="#REDIR">re-direction based</a>.
This requires setting the following entries:
<ul>
<li>HTTP.COOKIEJAR &mdash; a file path for storing cookies across re-direction.
<li>HTTP.NETRC &mdash; the path to the netrc file.
<li>HTTP.SSL.CERTIFICATE &mdash; the file path for the client side certificate file.
<li>HTTP.SSL.KEY &mdash; this should have the same value as HTTP.SSL.CERTIFICATE.
<li>HTTP.SSL.CAPATH &mdash; the path to a "certificates" directory.
<li>HTTP.SSL.VALIDATE &mdash; force validation of the server certificate.
</ul>
Note that the first two are to support re-direction based authentication.

<h5 class="break"><a name="allkeys"><u>Appendix A. All RC-File Keys</u></a></h5> 
For completeness, this is the list of all rc-file keys.
<table>
<tr><th>Key<th>curl_easy_setopt Option
<tr valign="top"><td>HTTP.DEFLATE<td>CUROPT_DEFLATE<br>with value "deflate,gzip"
<tr><td>HTTP.VERBOSE <td>CUROPT_VERBOSE 
<tr><td>HTTP.TIMEOUT<td>CUROPT_TIMEOUT
<tr><td>HTTP.USERAGENT<td>CUROPT_USERAGENT
<tr><td>HTTP.COOKIEJAR<td>CUROPT_COOKIEJAR
<tr><td>HTTP.COOKIE_JAR<td>CUROPT_COOKIEJAR
<tr valign="top"><td>HTTP.PROXY_SERVER<td>CURLOPT_PROXY,<br>CURLOPT_PROXYPORT,<br>CURLOPT_PROXYUSERPWD
<tr><td>HTTP.SSL.CERTIFICATE<td>CUROPT_SSLCERT
<tr><td>HTTP.SSL.KEY<td>CUROPT_SSLKEY
<tr><td>HTTP.SSL.KEYPASSWORD<td>CUROPT_KEYPASSWORD
<tr><td>HTTP.SSL.CAINFO<td>CUROPT_SSLCAINFO
<tr><td>HTTP.SSL.CAPATH<td>CUROPT_SSLCAPATH
<tr><td>HTTP.SSL.VERIFYPEER<td>CUROPT_SSL_VERIFYPEER
<tr><td>HTTP.CREDENTIALS.USERPASSWORD<td>CUROPT_USERPASSWORD
<tr><td>HTTP.NETRC<td>CURLOPT_NETRC,CURLOPT_NETRC_FILE
</table>
</ul>

<h5 class="break"><a name="ESGDETAIL"><u>Appendix B. ESG Access in Detail</u></a></h5> 
It is possible to access Earth Systems Grid (ESG) datasets
from ESG servers through the OC API using the techniques
described in the section on <a href="#CLIENTCERTS">Client-Side Certificates</a>.
<p>
In order to access ESG datasets, however, it is necessary to
register as a user with ESG and to setup your environment
so that proper authentication is established between an oc
client program and the ESG data server.  Specifically, it
is necessary to use what is called "client-side keys" to
enable this authentication. Normally, when a client accesses
a server in a secure fashion (using "https"), the server
provides an authentication certificate to the client.
With client-side keys, the client must also provide a
certificate to the server so that the server can know with
whom it is communicating.
<p>
The oc library uses the <i>curl</i> library and it is that
underlying library that must be properly configured.

<h3><u>Terminology</u></h3>
The key elements for client-side keys requires the constructions of
two "stores" on the client side.
<ul>
<li> Keystore - a repository to hold the client side key.
<li> Truststore - a repository to hold a chain of certificates
                  that can be used to validate the certificate
		  sent by the server to the client.
</ul>
The server actually has a similar set of stores, but the client
need not be concerned with those.

<h3><u>Initial Steps</u></h3>

The first step is to obtain authorization from ESG.
Note that this information may evolve over time, and
may be out of date.
This discussion is in terms of BADC and NCSA. You will need
to substitute as necessary.
<ol>
<li> Register at http://badc.nerc.ac.uk/register
   to obtain access to badc and to obtain an openid,
   which will looks something like:
   <pre>https://ceda.ac.uk/openid/Firstname.Lastname</pre>
<li> Ask BADC for access to whatever datasets are of interest.
<p>
<li> Obtain short term credentials at
   http://grid.ncsa.illinois.edu/myproxy/MyProxyLogon/
   You will need to download and run the MyProxyLogon
   program.
   This will create a keyfile in, typically, the directory ".globus".
   The keyfile will have a name similar to this: "x509up_u13615"
   The other elements in ".globus" are certificates to use in
   validating the certificate your client gets from the server.
<p>
<li> Obtain the program source ImportKey.java
   from this location: http://www.agentbob.info/agentbob/79-AB.html
   (read the whole page, it will help you understand the remaining steps).
</ol>

<h3><u>Building the KeyStore</u></h3>
You will have to modify the keyfile in the previous step
and then create a keystore and install the key and a certificate.
The commands are these:
<pre>
    openssl pkcs8 -topk8 -nocrypt -in x509up_u13615 -inform PEM -out key.der -outform DER

    openssl x509 -in x509up_u13615 -inform PEM -out cert.der -outform DER

    java -classpath <path to ImportKey.class> -Dkeypassword="<password>" -Dkeystore=./<keystorefilename> key.der cert.der
</pre>     
Note, the file names "key.der" and "cert.der" can be whatever you choose.
It is probably best to leave the .der extension, though.

<h3><u>Building the TrustStore</u></h3>
Building the truststore is a bit tricky because as provided, the
certificates in ".globus" need some massaging. See the script below
for the details. The primary command is this, which is executed for every
certificate, c, in globus. It sticks the certificate into the file
named "truststore"
<pre>
  keytool -trustcacerts -storepass "password" -v -keystore "truststore"  -importcert -file "${c}"
</pre>

<h3><u>Running the C Client</u></h3>

Refer to the section on <a href="#CLIENTCERTS">Client-Side Certificates</a>.
The keys specified there  must be set in the rc file to support
ESG access.
<ul>
<li> HTTP.COOKIEJAR=~/.dods_cookies
<li> HTTP.NETRC=~/.netrc
<li> HTTP.SSL.CERTIFICATE=~/esgkeystore
<li> HTTP.SSL.KEY=~/esgkeystore
<li> HTTP.SSL.CAPATH=~/.globus
<li> HTTP.SSL.VALIDATE=1
</ul>
Of course, the file paths above are suggestions only;
you can modify as needed.
The HTTP.SSL.CERTIFICATE and HTTP.SSL.KEY
entries should have same value, which is the file path for the
certificate produced by MyProxyLogon.  The HTTP.SSL.CAPATH entry
should be the path to the "certificates" directory produced by
MyProxyLogon.
<p>
As noted, also uses re-direction based authentication.
So, when it receives an initial connection from a client, it
redirects to a separate authentication server. When that
server has authenticated the client, it redirects back to
the original url to complete the request.

<h3><u>Script for creating Stores</u></h3>
The following script shows in detail how to actually construct the key
and trust stores. It is specific to the format of the globus file
as it was when ESG support was first added. It may have changed
since then, in which case, you will need to seek some help
in fixing this script. It would help if you communicated
what you changed to the author so this document can be updated.
<pre>
#!/bin/sh -x
KEYSTORE="esgkeystore"
TRUSTSTORE="esgtruststore"
GLOBUS="globus"
TRUSTROOT="certificates"
CERT="x509up_u13615"
TRUSTROOTPATH="$GLOBUS/$TRUSTROOT"
CERTFILE="$GLOBUS/$CERT"
PWD="password"

D="-Dglobus=$GLOBUS"
CCP="bcprov-jdk16-145.jar" 
CP="./build:${CCP}" 
JAR="myproxy.jar"

# Initialize needed directories
rm -fr build
mkdir build
rm -fr $GLOBUS
mkdir $GLOBUS
rm -f $KEYSTORE
rm -f $TRUSTSTORE

# Compile MyProxyCmd and ImportKey
javac -d ./build -classpath "$CCP" *.java
javac -d ./build ImportKey.java

# Execute MyProxyCmd
java -cp "$CP myproxy.MyProxyCmd

# Build the keystore
openssl pkcs8 -topk8 -nocrypt -in $CERTFILE -inform PEM -out key.der -outform DER
openssl x509 -in $CERTFILE -inform PEM -out cert.der -outform DER
java -Dkeypassword=$PWD -Dkeystore=./${KEYSTORE} -cp ./build ImportKey key.der cert.der

# Clean up the certificates in the globus directory
for c in ${TRUSTROOTPATH}/*.0 ; do
    alias=`basename $c .0`
    sed -e '0,/---/d' <$c >/tmp/${alias}
    echo "-----BEGIN CERTIFICATE-----" >$c       
    cat /tmp/${alias} >>$c
done
 
# Build the truststore
for c in ${TRUSTROOTPATH}/*.0 ; do
    alias=`basename $c .0`
    echo "adding: $TRUSTROOTPATH/${c}"
    echo "alias: $alias"
    yes | keytool -trustcacerts -storepass "$PWD" -v -keystore ./$TRUSTSTORE -alias $alias -importcert -file "${c}"
done
exit
</pre>

</body> 
</html> 
