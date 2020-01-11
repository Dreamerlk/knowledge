PHP编译安装时常见错误解决办法，php编译常见错误

This article is post on https://coderwall.com/p/ggmpfa

configure: error: xslt-config not found. Please reinstall the libxslt >= 1.1.0 distribution
复制代码 代码如下:
yum -y install libxslt-devel

configure: error: Could not find net-snmp-config binary. Please check your net-snmp installation.
复制代码 代码如下:
yum -y install net-snmp-devel

configure: error: Please reinstall readline - I cannot find readline.h
复制代码 代码如下:
yum -y install readline-devel

configure: error: Cannot find pspell
复制代码 代码如下:
yum -y install aspell-devel

checking for unixODBC support... configure: error: ODBC header file '/usr/include/sqlext.h' not found!
复制代码 代码如下:
yum -y install unixODBC-devel

configure: error: Unable to detect ICU prefix or /usr/bin/icu-config failed. Please verify ICU install prefix and make sure icu-config works.
复制代码 代码如下:
yum -y install libicu-devel

configure: error: utf8mime2text() has new signature, but U8TCANONICAL is missing. This should not happen. Check config.log for additional information.
复制代码 代码如下:
yum -y install libc-client-devel

configure: error: freetype.h not found.
复制代码 代码如下:
yum -y install freetype-devel

configure: error: xpm.h not found.
复制代码 代码如下:
yum -y install libXpm-devel

configure: error: png.h not found.
复制代码 代码如下:
yum -y install libpng-devel

configure: error: vpx_codec.h not found.
复制代码 代码如下:
yum -y install libvpx-devel

configure: error: Cannot find enchant
复制代码 代码如下:
yum -y install enchant-devel

configure: error: Please reinstall the libcurl distribution - easy.h should be in /include/curl/
复制代码 代码如下:
yum -y install libcurl-devel

LAOGAO added 20140907：

configure: error: mcrypt.h not found. Please reinstall libmcrypt.
复制代码 代码如下:
wget ftp://mcrypt.hellug.gr/pub/crypto/mcrypt/libmcrypt/libmcrypt-2.5.7.tar.gz
tar zxf libmcrypt-2.5.7.tar.gz
cd libmcrypt-2.5.7
./configure
make && make install

added 20141003：

Cannot find imap
复制代码 代码如下:
ln -s /usr/lib64/libc-client.so /usr/lib/libc-client.so

configure: error: utf8_mime2text() has new signature, but U8T_CANONICAL is missing.
复制代码 代码如下:
yum -y install libc-client-devel

Cannot find ldap.h
复制代码 代码如下:
yum -y install openldap
yum -y install openldap-devel

configure: error: Cannot find ldap libraries in /usr/lib
复制代码 代码如下:
cp -frp /usr/lib64/libldap* /usr/lib/

configure: error: Cannot find libpq-fe.h. Please specify correct PostgreSQL installation path
复制代码 代码如下:
yum -y install postgresql-devel

configure: error: Please reinstall the lib curl distribution
复制代码 代码如下:
yum -y install curl-devel

configure: error: Could not find net-snmp-config binary. Please check your net-snmp installation.
复制代码 代码如下:
yum -y install net-snmp-devel

configure: error: xslt-config not found. Please reinstall the libxslt >= 1.1.0 distribution
复制代码 代码如下:
yum -y install libxslt-devel

checking for BZip2 support… yes checking for BZip2 in default path… not found configure: error: Please reinstall the BZip2 distribution

Fix:
复制代码 代码如下:
yum -y install bzip2-devel

checking for cURL support… yes checking if we should use cURL for url streams… no checking for cURL in default path… not found configure: error: Please reinstall the libcurl distribution – easy.h should be in/include/curl/

Fix:
复制代码 代码如下:
yum -y install curl-devel

checking for curl_multi_strerror in -lcurl… yes checking for QDBM support… no checking for GDBM support… no checking for NDBM support… no configure: error: DBA: Could not find necessary header file(s).

Fix:
复制代码 代码如下:
yum -y install db4-devel

checking for fabsf… yes checking for floorf… yes configure: error: jpeglib.h not found.

Fix:
复制代码 代码如下:
yum -y install libjpeg-devel

checking for fabsf… yes checking for floorf… yes checking for jpeg_read_header in -ljpeg… yes configure: error: png.h not found.

Fix:
复制代码 代码如下:
yum -y install libpng-devel

checking for png_write_image in -lpng… yes If configure fails try –with-xpm-dir=

configure: error: freetype.h not found.
Fix:
复制代码 代码如下:
Reconfigure your PHP with the following option. --with-xpm-dir=/usr

checking for png_write_image in -lpng… yes configure: error: libXpm.(a|so) not found.

Fix:
复制代码 代码如下:
yum -y install libXpm-devel

checking for bind_textdomain_codeset in -lc… yes checking for GNU MP support… yes configure: error: Unable to locate gmp.h

Fix:
复制代码 代码如下:
yum -y install gmp-devel

checking for utf8_mime2text signature… new checking for U8T_DECOMPOSE… configure: error: utf8_mime2text() has new signature, but U8T_CANONICAL is missing. This should not happen. Check config.log for additional information.

Fix:
复制代码 代码如下:
yum -y install libc-client-devel

checking for LDAP support… yes, shared checking for LDAP Cyrus SASL support… yes configure: error: Cannot find ldap.h

Fix:
复制代码 代码如下:
yum -y install openldap-devel

checking for mysql_set_character_set in -lmysqlclient… yes checking for mysql_stmt_next_result in -lmysqlclient… no checking for Oracle Database OCI8 support… no checking for unixODBC support… configure: error: ODBC header file ‘/usr/include/sqlext.h' not found!

Fix:
复制代码 代码如下:
yum -y install unixODBC-devel

checking for PostgreSQL support for PDO… yes, shared checking for pg_config… not found configure: error: Cannot find libpq-fe.h. Please specify correct PostgreSQL installation path

Fix:
复制代码 代码如下:
yum -y install postgresql-devel

checking for sqlite 3 support for PDO… yes, shared checking for PDO includes… (cached) /usr/local/src/php-5.3.7/ext checking for sqlite3 files in default path… not found configure: error: Please reinstall the sqlite3 distribution

Fix:
复制代码 代码如下:
yum -y install sqlite-devel

checking for utsname.domainname… yes checking for PSPELL support… yes configure: error: Cannot find pspell

Fix:
复制代码 代码如下:
yum -y install aspell-devel

checking whether to enable UCD SNMP hack… yes checking for default_store.h… no

checking for kstat_read in -lkstat… no checking for snmp_parse_oid in -lsnmp… no checking for init_snmp in -lsnmp… no configure: error: SNMP sanity check failed. Please check config.log for more information.

Fix:
复制代码 代码如下:
yum -y install net-snmp-devel

checking whether to enable XMLWriter support… yes, shared checking for xml2-config path… (cached) /usr/bin/xml2-config checking whether libxml build works… (cached) yes checking for XSL support… yes, shared configure: error: xslt-config not found. Please reinstall the libxslt >= 1.1.0 distribution

Fix:
复制代码 代码如下:
yum -y install libxslt-devel

configure: error: xml2-config not found. Please check your libxml2 installation.

Fix:
复制代码 代码如下:
yum -y install libxml2-devel

checking for PCRE headers location… configure: error: Could not find pcre.h in /usr

Fix:
复制代码 代码如下:
yum -y install pcre-devel

configure: error: Cannot find MySQL header files under yes. Note that the MySQL client library is not bundled anymore!

Fix:
复制代码 代码如下:
yum -y install mysql-devel

checking for unixODBC support… configure: error: ODBC header file ‘/usr/include/sqlext.h' not found!

Fix:
复制代码 代码如下:
yum -y install unixODBC-devel

checking for pg_config… not found configure: error: Cannot find libpq-fe.h. Please specify correct PostgreSQL installation path

Fix:
复制代码 代码如下:
yum -y install postgresql-devel

configure: error: Cannot find pspell

Fix:
复制代码 代码如下:
yum -y install pspell-devel

configure: error: Could not find net-snmp-config binary. Please check your net-snmp installation.

Fix:
复制代码 代码如下:
yum -y install net-snmp-devel

configure: error: xslt-config not found. Please reinstall the libxslt >= 1.1.0 distribution

Fix:
复制代码 代码如下:
yum -y install libxslt-devel

php报错php-mcrypt,yum安装mcrypt错误:
yum  install epel-release  //扩展包更新包
    yum  update //更新yum源
    yum install libmcrypt libmcrypt-devel mcrypt mhash  就ok了
