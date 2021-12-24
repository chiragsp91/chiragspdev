---
title: "Migrate Perl application from Redhat 5 to Redhat 7"
date: 2021-11-05
description: "Steps involved in migrating Perl application from Redhat5 to Redhat 7"
categories:
  - "Development"
tags: 
    - "Development"
    - "Perl"
#math: false
lead: "Steps involved in migrating Perl application"
comments: true
mathjax: true
toc: false
#menu: main
---
One fine day, I was assigned a project to migrate a perl application( written in the 90s) to Redhat 7 from Redhat 5. Mind you I am a Java developer, but i pride myself in learning anything and everything. So i confidently took in this project as well. Little did I know, the weeks of hell I had to endure to do complete this assignment. So in order to help someone in future below are the steps I followed.

For my use case I am going to setup my Perl application in Redhat 7.9 

1. Install Apache web server
    
    Check if Apache is installed using below command,
    
    ```jsx
    httpd -v
    ```
    
    if Apache is present you should get below response,
    
    ```jsx
    Server version: Apache/2.4.6 (Red Hat Enterprise Linux)
    ```
    
    If Apache is not present, use below command to install Apache,
    
    ```jsx
    yum install httpd-devel -y
    ```
    
    Check if Apache is properly by starting Apache using below command,
    
    ```jsx
    apachectl start
    ```
    
    Now in any browser, open the server IP (it maybe '[localhost](http://localhost)' in your case). In my case I was installing in a new UNIX box having Red hat and I entered the box IP in my browser and tried to open it.
    
2. Check if Oracle client is installed by running command 
    
    ```jsx
    env
    ```
    
    If oracle is installed you'll have 'ORACLE_HOME '  present.
    
    If oracle is not installed, download the rpm from oracle site follow the steps to get it installed.
    
    PS: Your DBA should be able provide you the oracle rpm.
    
3. After oracle is installed, check if any environment variables are set using below command,
        
    ```jsx
    env
    ```
        
        ORACLE_HOME=/app/oracle/product/11.2.0/client_1
        
4. Make sure ORACLE_HOME , LD_LIBRARY_PATH and PATH are set. If not we can set it manually by editing bash_profile      using below command
        
    ```jsx
    vi ~/.bash_profile
    ```
        
    And add,
        
    ```jsx
    export ORACLE_HOME=/app/oracle/product/11.2.0/client_1
    export LD_LIBRARY_PATH=$ORACLE_HOME/lib
    export PATH=$PATH:$ORACLE_HOME/bin
    ```
        
5. After oracle client is installed , DB details need to be set.
        
    Create tnsname.ora in oracle client location i.e. at {ORACLE_HOME}/network/admin with below contents,
        
    ```jsx
    test_s1 =
        (DESCRIPTION =
        (ADDRESS = (PROTOCOL = TCP)(HOST = test-test.net)(PORT = 1521))
        (CONNECT_DATA =
            (SERVER = DEDICATED)
            (SERVICE_NAME = testSN1)
        )
    )
    ```
        
    Here test_s1 is the system id which you will need pass to your ora_login method in perl script.
        
6. Next you need to make sure below environment variables are not set.
        
    ```jsx
    PERL5LIB="/root/perl5/lib/perl5:"
    PERL_LOCAL_LIB_ROOT=":/root/perl5"
    PERL_MB_OPT="--install_base /root/perl5"
    PERL_MM_OPT="INSTALL_BASE=/root/perl5"
    ```
        
    If any of the above are present, then unset all of them as shown below,
        
    ```jsx
    unset PERL5LIB
    unset PERL_LOCAL_LIB_ROOT
    unset PERL_MB_OPT
    unset PERL_MM_OPT
    ```
        
    This is done so that @INC folder gets installed in user folder instead of root. This is important because Perl expects @INC to be in user folder
        
7. Now we install the Perl modules. In my case as I'm using Oracle, so I need to install 'DBD::Oracle' module. Also my application contains some legacy Perl4 code. Since Perl4 uses 'Oraperl' for DB connections, I need to install 'Oraperl' as well.
        
    (We can either uses CPAN or install the modules manually).
        
    CPAN is a package manager for Perl and makes it very easy to install Perl modules,
        
    Perl Module installation. Use either Step A or Step B
        
    A.  Using CPAN
        
    ```jsx
    1.Install CPAN
        yum install perl-CPA
    2.Open Cpan shell
        perl -MCPAN -e shell
    3.Install DBI
        install Bundle::DBI
    4.Install DBD Oracle
        install DBD::Oracle
    5.Install Crypt RSA
        install Crypt::RSA
    6.Install MIME Base64
        install MIME::Base64
    7.Install Oracperl
        install Oraperl
    ```
        
    B. Without using CPAN
        
    Refer link [https://www.perlmonks.org/?node_id=128077](https://www.perlmonks.org/?node_id=128077)
        
8. I made sure to give 755 permission for my folder wherein Perl code resides.
9. Now I had to fix configuration in Apache,
            
    In case of Apache 2.4 configuration files are present at location '/etc/httpd/conf/httpd.conf'
            
    Logs are present at '/etc/httpd/logs'
            
    There is no htdocs, instead if files needs to picked up by Apache, it needs to placed at location - '/var/www/html'. (htdocs is in Apache 2.2 and lower)
            
    To start Apache use below command,
            
    ```jsx
    apachectl start
    ```
            
    Next I went to my server IP( in my case the IP was 12**.34.56.789**) and checked if Apache default page was appearing.
            
    Some commonly used Apache commands are as follows,
            
    To stop Apache : 
            
    ```jsx
    apachectl stop
    ```
            
    To Start Apache: 
            
    ```jsx
    'apachectl start
    ```
            
    To restart Apache
            
    ```jsx
    apachectl restart
    ```
            
    OR
            
    ```jsx
    systemctl restart httpd
    ```
            
10. Now check if perl modules are installed correctly, by running a sample Perl script and checking if its able to connect DB successfully. Below is the [test.pl](http://test.pl) I used.
            
    ```jsx
    #!/opt/perl5/bin/perl
            
    $system_id="test_s1";
    $username="testuser";
    $password="testuserpass";
            
    print "Content-type: text/html\n\n";
    eval 'use Oraperl; 1;' || die $@ if $1] >=5;
    $lda = &ora_login($system_id, $username, $password);
            
    if ($ora_errno)
    {
        print $ora_errstr;
        print " \n Could not connect to database at $connect_time\n";
    }
    else{
        print " \n Connected to db at $connect_time\n"
    }
    ```
            
11. Now to create index.html file.
        
    In order for **my application to come when I open my server IP location in browser,**  I had to create index.html at location '/var/www/html'
        
    Created 'index.html' at location /var/www/html with below contents,
        
    <META HTTP-EQUIV="Refresh" CONTENT="1;URL=http://**12.34.56.789:80**/cgi-bin/mycode/myIndex.html">
        
12. As you can observe, the above path refers to 'cgi-bin' folder. This folder needs to be created. After creating this folder, I placed all Perl code in 'cgi-bin' file.
            
    At location, '/var/www' I created 'cgi-bin' folder.

    Inside 'cgi-bin' folder I created a link called 'mycode'(this can anything). This links to my Perl code (or you can place your Perl code directly to location '/var/www/cgi-bin').
            
13. Then I gave 755 permission to /var/www and all its sub-directories using below command(path should be /var/www/)
        
    ```jsx
    chmod -R 775 html
    ```
        
14. Now I had to configure the httpd.conf file.
            
    As mentioned above, this location is /etc/httpd/conf/httpd.conf.
            
    Below are items I had to change/add in httpd.conf
    
    1. Changed the listen IP and port to the my UNIX box IP andport, as shown below                                                                                                       
    
    ```jsx
    Listen **12.34.56.789:80**
    ```
            
    b. Added below Load Modules 
    
    ```jsx
    LoadModule authn_file_module modules/mod_authn_file.so
    LoadModule authn_dbm_module modules/mod_authn_dbm.so
    LoadModule authn_anon_module modules/mod_authn_anon.so
    LoadModule authn_dbd_module modules/mod_authn_dbd.so
    LoadModule authz_host_module modules/mod_authz_host.so
    LoadModule authz_groupfile_module modules/mod_authz_groupfile.so
    LoadModule authz_user_module modules/mod_authz_user.so
    LoadModule authz_dbm_module modules/mod_authz_dbm.so
    LoadModule authz_owner_module modules/mod_authz_owner.so
    LoadModule auth_basic_module modules/mod_auth_basic.so
    LoadModule auth_digest_module modules/mod_auth_digest.so
    LoadModule dbd_module modules/mod_dbd.so
    LoadModule dumpio_module modules/mod_dumpio.so
    LoadModule reqtimeout_module modules/mod_reqtimeout.so
    LoadModule ext_filter_module modules/mod_ext_filter.so
    LoadModule include_module modules/mod_include.so
    LoadModule filter_module modules/mod_filter.so
    LoadModule substitute_module modules/mod_substitute.so
    LoadModule deflate_module modules/mod_deflate.so
    LoadModule log_config_module modules/mod_log_config.so
    LoadModule log_forensic_module modules/mod_log_forensic.so
    LoadModule logio_module modules/mod_logio.so
    LoadModule env_module modules/mod_env.so
    LoadModule mime_magic_module modules/mod_mime_magic.so
    LoadModule expires_module modules/mod_expires.so
    LoadModule headers_module modules/mod_headers.so
    LoadModule usertrack_module modules/mod_usertrack.so
    LoadModule unique_id_module modules/mod_unique_id.so
    LoadModule setenvif_module modules/mod_setenvif.so
    LoadModule version_module modules/mod_version.so
    LoadModule mime_module modules/mod_mime.so
    LoadModule dav_module modules/mod_dav.so
    LoadModule status_module modules/mod_status.so
    LoadModule autoindex_module modules/mod_autoindex.so
    LoadModule asis_module modules/mod_asis.so
    LoadModule info_module modules/mod_info.so
    LoadModule cgi_module modules/mod_cgi.so
    LoadModule cgid_module modules/mod_cgid.so
    LoadModule dav_fs_module modules/mod_dav_fs.so
    LoadModule vhost_alias_module modules/mod_vhost_alias.so
    LoadModule negotiation_module modules/mod_negotiation.so
    LoadModule dir_module modules/mod_dir.so
    LoadModule actions_module modules/mod_actions.so
    LoadModule speling_module modules/mod_speling.so
    LoadModule userdir_module modules/mod_userdir.so
    LoadModule alias_module modules/mod_alias.so
    LoadModule rewrite_module modules/mod_rewrite.so
    ```
    
    c. Created a new user and group in my Unix server and set the same as below.
            
        
    ```jsx
    User exampleUser
    Group exampleGroup
    ```
    
    d. Changed 'Directory' tag to as shown below.
    
    ```jsx
    <Directory />
        Options FollowSymLinks
        AllowOverride None
        Order deny,allow
        Deny from all
    </Directory>
    ```
    
    c. Changed 'Directory "/var/www"' as below.
    
    ```jsx
    <Directory "/var/www">
    </Directory>
    ```
    
    d. Changed 'Directory "/var/www/html"' as below
        
    ```jsx
    <Directory "/var/www/html">
        #
        # Possible values for the Options directive are "None", "All",
        # or any combination of:
        #   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
        #
        # Note that "MultiViews" must be named *explicitly* --- "Options All"
        # doesn't give it to you.
        #
        # The Options directive is both complicated and important.  Please see
        # http://httpd.apache.org/docs/2.4/mod/core.html#options
        # for more information.
        #
        Options Indexes FollowSymLinks ExecCGI
    
        #
        # AllowOverride controls what directives may be placed in .htaccess files.
        # It can be "All", "None", or any combination of the keywords:
        #   Options FileInfo AuthConfig Limit
        #
        AllowOverride None
    
        #
        # Controls who can get stuff from this server.
        #
        Order allow,deny
        Allow from all
    </Directory>
    ```
    
    e. Changed 'IfModule alias_module tag as below. Here I had to declare the different 'ScriptAlias' present in my Perl scripts.
    
    ```jsx
    <IfModule alias_module>
        #
        # Redirect: Allows you to tell clients about documents that used to
        # exist in your server's namespace, but do not anymore. The client
        # will make a new request for the document at its new location.
        # Example:
        # Redirect permanent /foo http://www.example.com/bar
    
        #
        # Alias: Maps web paths into filesystem paths and is used to
        # access content that does not live under the DocumentRoot.
        # Example:
        # Alias /webpath /full/filesystem/path
        #
        # If you include a trailing / on /webpath then the server will
        # require it to be present in the URL.  You will also likely
        # need to provide a <Directory> section to allow access to
        # the filesystem path.
    
        #
        # ScriptAlias: This controls which directories contain server scripts.
        # ScriptAliases are essentially the same as Aliases, except that
        # documents in the target directory are treated as applications and
        # run by the server when requested rather than as documents sent to the
        # client.  The same rules about trailing "/" apply to ScriptAlias
        # directives as to Alias.
        #
        ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
        ScriptAlias /images/ "/var/www/cgi-bin/content/images/"
            ScriptAlias /log/ "/var/www/cgi-bin/content/login/"
        ScriptAlias /bin/logoff "/var/www/cgi-bin/content/html/logoff.html"
    
    </IfModule>
    ```
    
    f. Changed 'Directory "/var/www/cgi-bin"' to below
    
    ```jsx
    <Directory "/var/www/cgi-bin">
        AllowOverride None
        Options Indexes FollowSymLinks ExecCGI
        Order allow,deny
        Allow from all
        AddHandler cgi-script .pl .class .new
    </Directory>
    ```
        
    g. Changed 'IfModule mime_module' tag to below,
    
    ```jsx
    <IfModule mime_module>
        #
        # TypesConfig points to the file containing the list of mappings from
        # filename extension to MIME-type.
        #
        TypesConfig /etc/mime.types
    
        #
        # AddType allows you to add to or override the MIME configuration
        # file specified in TypesConfig for specific file types.
        #
        #AddType application/x-gzip .tgz
        #
        # AddEncoding allows you to have certain browsers uncompress
        # information on the fly. Note: Not all browsers support this.
        #
        #AddEncoding x-compress .Z
        #AddEncoding x-gzip .gz .tgz
        #
        # If the AddEncoding directives above are commented-out, then you
        # probably should define those extensions to indicate media types:
        #
        AddType application/x-compress .Z
        AddType application/x-gzip .gz .tgz
    
        #
        # AddHandler allows you to map certain file extensions to "handlers":
        # actions unrelated to filetype. These can be either built into the server
        # or added with the Action directive (see below)
        #
        # To use CGI scripts outside of ScriptAliased directories:
        # (You will also need to add "ExecCGI" to the "Options" directive.)
        #
        #AddHandler cgi-script .cgi
        AddHandler cgi-script .class
        AddHandler cgi-script .cgi .pl .class .jsp
        AddHandler default-handler .jpg .png .css .ico .gif .html .log
    
        # For type maps (negotiated resources):
        #AddHandler type-map var
    
        #
        # Filters allow you to process content before it is sent to the client.
        #
        # To parse .shtml files for server-side includes (SSI):
        # (You will also need to add "Includes" to the "Options" directive.)
        #
        #AddType text/html .shtml
        #AddOutputFilter INCLUDES .shtml
    </IfModule>
    ```
    
    h. Also added below oracle paths as well in httpd.conf dile.
    
    ```jsx
    SetEnv LD_LIBRARY_PATH /app/oracle/product/11.2.0/client_1/lib
    SetEnv ORACLE_HOME /app/oracle/product/11.2.0/client_1
    ```
        
15. Also I made add below code in all my .pl file which contain print statements.
        
    ```jsx
    print "Content-type: text/html\n\n";
    ```
    
    (or else it will cause Apache to download .pl file instead of executing it. Make sure to check files of any inner subroutines as well. Above code should be present.)