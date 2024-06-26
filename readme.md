Centmin Mod based CentOS 7 in-place migration upgrade to [AlmaLinux 8 via AlmaLinux Elevate](https://wiki.almalinux.org/elevate/ELevating-CentOS7-to-AlmaLinux-9.html#migrate-centos-7-to-almalinux-8). Do not use on production live servers. Test on a test server first. This is just alpha testing. 

**Notes:**

1. EL8+ memory requirements are higher with recommended 4GB memory + 4GB swap disk for EL8+. So do not attempt CentOS 7 in-place migration upgrade to AlmaLinux 8 if you have less than 4GB memory + 4GB swap disk.
2. There are 3 parts to the CentOS 7 in-place migration upgrade outlined below.

# Part 1 - Install AlmaLinux Elevate & Reboot Server

On Centmin Mod CentOS 7 installed server run the following commands.


## Update Centmin Mod System

Rebooting server is necessary - especially if you have yum package updates that require a reboot i.e. Kernel updates.

```
cmupdate
yum update -y
reboot
```

## Install and prep Almalinux Elevate

```
# backup pure-ftpd files
mkdir -p /root/tools/pureftpd
cp -a /etc/pure-ftpd/pureftpd.passwd /root/tools/pureftpd/pureftpd.passwd
cp -a /etc/pure-ftpd/pureftpd.pdb /root/tools/pureftpd/pureftpd.pdb

# remove versionlocks
yum versionlock delete libc-client uw-imap-devel ImageMagick6 ImageMagick6-devel ImageMagick6-c++ ImageMagick6-c++-devel ImageMagick6-libs LibRaw

yum install -y http://repo.almalinux.org/elevate/elevate-release-latest-el$(rpm --eval %rhel).noarch.rpm
yum install -y leapp-upgrade leapp-data-almalinux
leapp preupgrade
# inspect /var/log/leapp/answerfile
cat /var/log/leapp/answerfile
leapp answer --section remove_pam_pkcs11_module_check.confirm=True
yum remove libwebp7 cmake3 bash-completion -y
leapp upgrade
echo "rebooting server..."
reboot
```

# Part 2 - Prep AlmaLinux 8

```
yum-config-manager --enable powertools
alternatives --set python /usr/bin/python3
rm -rf /etc/yum.repos.d/epel.repo
yum -y reinstall epel-release
yum -y reinstall bash-completion GeoIP GeoIP-devel perl-FindBin libc-client libc-client-devel systemd-libs open-sans-fonts libidn2-devel libpsl-devel gpgme-devel gnutls-devel virt-what acl libacl-devel attr libattr-devel lz4-devel gawk unzip libuuid-devel sqlite-devel bc wget lynx screen ca-certificates yum-utils bash mlocate subversion rsyslog dos2unix boost-program-options net-tools imake bind-utils libatomic_ops-devel time coreutils autoconf cronie crontabs cronie-anacron gcc gcc-c++ automake libtool make libXext-devel unzip patch sysstat openssh flex bison file libtool-ltdl-devel krb5-devel libXpm-devel nano gmp-devel aspell-devel numactl lsof pkgconfig gdbm-devel tk-devel bluez-libs-devel iptables* rrdtool diffutils which perl-Math-BigInt perl-Test-Simple perl-ExtUtils-Embed perl-ExtUtils-MakeMaker perl-Time-HiRes perl-libwww-perl perl-Net-SSLeay cyrus-imapd cyrus-sasl-md5 cyrus-sasl-plain strace cmake git net-snmp-libs net-snmp-utils iotop libvpx libvpx-devel t1lib t1lib-devel expect readline readline-devel libedit libedit-devel libxslt libxslt-devel openssl openssl-devel curl curl-devel openldap openldap-devel zlib zlib-devel gd gd-devel pcre pcre-devel gettext gettext-devel libidn libidn-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel e2fsprogs e2fsprogs-devel libc-client libc-client-devel cyrus-sasl cyrus-sasl-devel pam pam-devel libaio libaio-devel libevent libevent-devel recode recode-devel libtidy libtidy-devel net-snmp net-snmp-devel enchant enchant-devel lua lua-devel mailx perl-LWP-Protocol-https OpenEXR-devel OpenEXR-libs atk cups-libs fftw-libs-double fribidi gdk-pixbuf2 ghostscript-devel gl-manpages graphviz gtk2 hicolor-icon-theme ilmbase ilmbase-devel jasper-devel jasper-libs jbigkit-devel jbigkit-libs lcms2 lcms2-devel libICE-devel libSM-devel libXaw libXcomposite libXcursor libXdamage-devel libXfixes-devel libXi libXinerama libXmu libXrandr libXt-devel libXxf86vm-devel libdrm-devel libfontenc librsvg2 libtiff libtiff-devel libwebp libwebp-devel libwmf-lite mesa-libGL-devel mesa-libGLU mesa-libGLU-devel poppler-data urw-fonts xorg-x11-font-utils --skip-broken

ex -s /etc/yum.repos.d/epel.repo << EOF
:/\[epel\]/ , /gpgkey/
:a
priority=3
exclude=ImageMagick*
.
:w
:q
EOF
```

# Part 3 - Prep Centmin Mod for AlmaLinux 8

```
cd /usr/local/src
mv centminmod centminmod.old
git clone -b 130.00beta01 --depth=1 https://github.com/centminmod/centminmod centminmod
cat /etc/centminmod/custom_config.inc
echo "CENTOS_ALPHATEST='y'" > /etc/centminmod/custom_config.inc
echo "SELFSIGNEDSSL_ECDSA='y'" >> /etc/centminmod/custom_config.inc
echo "PHPFINFO='y'" >> /etc/centminmod/custom_config.inc
echo "PHP_OVERWRITECONF='n'" >> /etc/centminmod/custom_config.inc
echo "OPENSSL_SYSTEM_USE='y'" >> /etc/centminmod/custom_config.inc
echo "NGINX_GEOIPTWOLITE='y'" >> /etc/centminmod/custom_config.inc
echo "NGXDYNAMIC_GEOIPTWOLITE='y'" >> /etc/centminmod/custom_config.inc
cat /etc/centminmod/custom_config.inc
cd /usr/local/src/centminmod

wget -O centmin-option-4.sh https://github.com/centminmod/centminmod-centos7-elevate/raw/master/scripts/centmin-option-4.sh
wget -O centmin-option-5.sh https://github.com/centminmod/centminmod-centos7-elevate/raw/master/scripts/centmin-option-5.sh
wget -O centmin-option-10.sh https://github.com/centminmod/centminmod-centos7-elevate/raw/master/scripts/centmin-option-10.sh

chmod +x centmin-option-4.sh centmin-option-5.sh centmin-option-10.sh

echo "1" > /etc/centminmod/email-primary.ini
echo "2" > /etc/centminmod/email-secondary.ini
# reinstall pcre
/usr/local/src/centminmod/addons/wget.sh pcre
echo 24 | /usr/local/src/centminmod/centmin.sh

# reinstall Nginx
./centmin-option-4.sh

# create a mariadb 10.4 el8/el9 reinstall
MDB_MIRROR_BASEURL='https://mirror.rackspace.com/mariadb/yum'
MDB_REPOMD_URL="${MDB_MIRROR_BASEURL}/${MDB_REPO_DIR}/repodata/repomd.xml"
# fallback check of repomd.xml
curl -s "$MDB_REPOMD_URL" -o repomd.xml
MDB_REPOMD_URL_CHECK=$(xmllint --format repomd.xml >/dev/null 2>&1 ; echo $?)
rm -f repomd.xml
if [[ "$MDB_REPOMD_URL_CHECK" -ne '0' ]]; then
    MDB_MIRROR_BASEURL='https://yum.mariadb.org'
fi
echo "rpm --import ${MDB_MIRROR_BASEURL}/RPM-GPG-KEY-MariaDB"
rpm --import "${MDB_MIRROR_BASEURL}/RPM-GPG-KEY-MariaDB"
# https://mariadb.com/kb/en/gpg/#rpm-source-key-pre-2023
echo "rpm --import https://supplychain.mariadb.com/MariaDB-Server-GPG-KEY"
rpm --import https://supplychain.mariadb.com/MariaDB-Server-GPG-KEY

cat > "/etc/yum.repos.d/mariadb.repo" <<EOF
[mariadb]
name = MariaDB
baseurl = ${MDB_MIRROR_BASEURL}/10.4/centos8-amd64
module_hotfixes=1
gpgkey=${MDB_MIRROR_BASEURL}/RPM-GPG-KEY-MariaDB
gpgcheck=1
EOF

cat /etc/yum.repos.d/mariadb.repo

yum -y install MariaDB-client MariaDB-common MariaDB-compat MariaDB-devel MariaDB-server MariaDB-shared

cp -a /etc/my.cnf /etc/my.cnf-newold-elevated

systemctl start mariadb
systemctl status mariadb --no-pager
systemctl enable mariadb
mysql_upgrade --force

echo "------------------------------------------------"
echo "Installing MariaDB 10 plugins"
echo "------------------------------------------------"
echo "mysql -e \"INSTALL SONAME 'metadata_lock_info';\""
mysql -e "INSTALL SONAME 'metadata_lock_info';"
echo "mysql -e \"INSTALL SONAME 'query_cache_info';\""
mysql -e "INSTALL SONAME 'query_cache_info';"
echo "mysql -e \"INSTALL SONAME 'query_response_time';\""
mysql -e "INSTALL SONAME 'query_response_time';"
echo "mysql -t -e \"SELECT * FROM mysql.plugin;\""
mysql -t -e "SELECT * FROM mysql.plugin;"
echo "mysql -t -e \"SHOW PLUGINS;\""
mysql -t -e "SHOW PLUGINS;"
echo "mysql -t -e \"SHOW ENGINES;\""
mysql -t -e "SHOW ENGINES;"

# Remi YUM repo resetup
rm -rf /etc/yum.repos.d/remi*
rpm --import https://rpms.remirepo.net/RPM-GPG-KEY-remi2018
rpm --import https://rpms.remirepo.net/RPM-GPG-KEY-remi2019
rpm --import https://rpms.remirepo.net/RPM-GPG-KEY-remi2020
rpm --import https://rpms.remirepo.net/RPM-GPG-KEY-remi2021
rpm --import https://rpms.remirepo.net/RPM-GPG-KEY-remi2022
rpm --import https://rpms.remirepo.net/RPM-GPG-KEY-remi2023
yum -y install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
sed -i 's|repo_gpgcheck=1|repo_gpgcheck=0|g' /etc/yum.repos.d/remi*.repo
yum clean all
yum makecache
grep repo_gpgcheck /etc/yum.repos.d/remi*.repo
yum -y module disable composer
yum -y module reset redis
yum -y module enable redis:remi-7.2

# Percona YUM repo resetup
rm -rf /etc/yum.repos.d/percona-*
yum -y reinstall https://repo.percona.com/yum/percona-release-latest.noarch.rpm
percona-release show
percona-release disable all
percona-release enable tools

# re-install redis server
/usr/local/src/centminmod/addons/redis-server-install.sh install
redis-cli info server

# re-install php-fpm
./centmin-option-5.sh

# re-install memcached server
./centmin-option-10.sh

# create a pure-ftpd el8/el9 reinstall
yum -y reinstall pure-ftpd
cp -a /etc/pure-ftpd/pure-ftpd.conf /etc/pure-ftpd/pure-ftpd.conf-old-el7
sed -i '/UseFtpUsers/d' /etc/pure-ftpd/pure-ftpd.conf

sed -i 's/# UnixAuthentication  /UnixAuthentication  /' /etc/pure-ftpd/pure-ftpd.conf
sed -i 's/VerboseLog .*/VerboseLog yes/' /etc/pure-ftpd/pure-ftpd.conf
sed -i 's/# PureDB                       \@sysconfigdir\@\/pureftpd.pdb/PureDB                        \/etc\/pure-ftpd\/pureftpd.pdb/' /etc/pure-ftpd/pure-ftpd.conf

sed -i 's/# CreateHomeDir .*/CreateHomeDir               yes/' /etc/pure-ftpd/pure-ftpd.conf
sed -i 's/# TLS .*/TLS                      2/' /etc/pure-ftpd/pure-ftpd.conf
sed -i 's/# PassivePortRange .*/PassivePortRange    30001 50011/' /etc/pure-ftpd/pure-ftpd.conf
sed -i 's/^PassivePortRange    3000 3050/PassivePortRange    30001 50011/' /etc/pure-ftpd/pure-ftpd.conf
sed -i 's|MaxClientsNumber .*|MaxClientsNumber            1000|' /etc/pure-ftpd/pure-ftpd.conf
sed -i 's|MaxClientsPerIP .*|MaxClientsPerIP             500|' /etc/pure-ftpd/pure-ftpd.conf
sed -i 's|NoAnonymous .*|NoAnonymous                 yes|' /etc/pure-ftpd/pure-ftpd.conf

# fix default file/directory permissions
sed -i 's/Umask .*/Umask                       137:027/' /etc/pure-ftpd/pure-ftpd.conf

if [[ "$(grep 'TLSCipherSuite' /etc/pure-ftpd/pure-ftpd.conf | grep -o HIGH)" != 'HIGH' ]]; then
  # echo 'TLSCipherSuite           HIGH:MEDIUM:+TLSv1:!SSLv2:!SSLv3' >> /etc/pure-ftpd/pure-ftpd.conf
  sed -i 's|# TLSCipherSuite .*|TLSCipherSuite               HIGH|' /etc/pure-ftpd/pure-ftpd.conf
fi

mkdir -p /root/tools/pureftpd
\cp -af /root/tools/pureftpd/pureftpd.passwd /etc/pure-ftpd/pureftpd.passwd
\cp -af /root/tools/pureftpd/pureftpd.pdb /etc/pure-ftpd/pureftpd.pdb
chmod 0600 /etc/pure-ftpd/pureftpd.passwd
pure-pw mkdb

mkdir -p /etc/ssl/private
# time openssl dhparam -out /etc/ssl/private/pure-ftpd-dhparams.pem 2048
CNIP=$(curl -s -A "el8 elevated pure-ftpd" https://geoip.centminmod.com/v4 | jq -r '.ip')
openssl req -x509 -days 7300 -sha256 -nodes -subj "/C=US/ST=California/L=Los Angeles/O=Default Company Ltd/CN=$CNIP" -newkey rsa:2048 -keyout /etc/pki/pure-ftpd/pure-ftpd.pem -out /etc/pki/pure-ftpd/pure-ftpd.pem
chmod 600 /etc/pki/pure-ftpd/*.pem
openssl x509 -in /etc/pki/pure-ftpd/pure-ftpd.pem -text -noout
echo 
ls -lah /etc/pki/pure-ftpd

mkdir -p /etc/systemd/system/pure-ftpd.service.d
echo '[Service]' > /etc/systemd/system/pure-ftpd.service.d/pidfile.conf
echo 'PIDFile=/run/pure-ftpd.pid' >> /etc/systemd/system/pure-ftpd.service.d/pidfile.conf

systemctl daemon-reload
systemctl restart pure-ftpd
systemctl enable pure-ftpd
systemctl status pure-ftpd --no-pager

# remove left over el7 and elevate/leapp packages
rpm -e --nodeps $(rpm -qa | egrep 'el7|elevate|leapp' | sort |xargs)

# yum update
yum -y update --disableplugin=priorities --setopt=deltarpm=0 --enablerepo=remi
```

CentOS 7 to AlmaLinux 8 via AlmaLinux Elevate is work in progress so not 100% tested right now.

```
nginx -V
nginx version: nginx/1.27.0 (250624-162622-almalinux8-kvm-b8438a6)
built by gcc 13.2.1 20231205 (Red Hat 13.2.1-6) (GCC) 
built with OpenSSL 1.1.1k  FIPS 25 Mar 2021
TLS SNI support enabled
```
> configure arguments: --with-ld-opt='-Wl,-E -L/usr/local/zlib-cf/lib -L/usr/local/nginx-dep/lib -ljemalloc -Wl,-z,relro,-z,now -Wl,-rpath,/usr/local/zlib-cf/lib:/usr/local/nginx-dep/lib -pie -flto=2 -flto-compression-level=3 -fuse-ld=gold' --with-cc-opt='-I/usr/local/zlib-cf/include -I/usr/local/nginx-dep/include -m64 -march=native -fPIC -g -O3 -fstack-protector-strong -flto=2 -flto-compression-level=3 -fuse-ld=gold --param=ssp-buffer-size=4 -Wformat -Wno-pointer-sign -Wimplicit-fallthrough=0 -Wno-implicit-function-declaration -Wno-cast-align -Wno-builtin-declaration-mismatch -Wno-deprecated-declarations -Wno-int-conversion -Wno-unused-result -Wno-vla-parameter -Wno-maybe-uninitialized -Wno-return-local-addr -Wno-array-parameter -Wno-alloc-size-larger-than -Wno-address -Wno-array-bounds -Wno-discarded-qualifiers -Wno-stringop-overread -Wno-stringop-truncation -Wno-missing-field-initializers -Wno-unused-variable -Wno-format -Wno-error=unused-result -Wno-missing-profile -Wno-stringop-overflow -Wno-free-nonheap-object -Wno-discarded-qualifiers -Wno-bad-function-cast -Wno-dangling-pointer -Wno-array-parameter -fcode-hoisting -Wno-cast-function-type -Wno-format-extra-args -Wp,-D_FORTIFY_SOURCE=2' --prefix=/usr/local/nginx --sbin-path=/usr/local/sbin/nginx --conf-path=/usr/local/nginx/conf/nginx.conf --build=250624-162622-almalinux8-kvm-b8438a6 --with-compat --without-pcre2 --with-http_stub_status_module --with-http_secure_link_module --with-libatomic --with-http_gzip_static_module --add-dynamic-module=../ngx_http_geoip2_module --with-http_sub_module --with-http_addition_module --with-http_image_filter_module=dynamic --with-http_geoip_module --with-stream_geoip_module --with-stream_realip_module --with-stream_ssl_preread_module --with-threads --with-stream --with-stream_ssl_module --with-http_realip_module --add-dynamic-module=../ngx-fancyindex-0.4.2 --add-module=../ngx_cache_purge-2.5.1 --add-dynamic-module=../ngx_devel_kit-0.3.2 --add-dynamic-module=../set-misc-nginx-module-0.33 --add-dynamic-module=../echo-nginx-module-0.63 --add-module=../redis2-nginx-module-0.15 --add-module=../ngx_http_redis-0.4.0-cmm --add-module=../memc-nginx-module-0.20 --add-module=../srcache-nginx-module-0.33 --add-dynamic-module=../headers-more-nginx-module-0.37 --with-pcre-jit --with-zlib=../zlib-cloudflare-1.3.3 --with-zlib-opt=-fPIC --with-http_ssl_module --with-http_v2_module

```
php-config
Usage: /usr/local/bin/php-config [OPTION]
Options:
  --prefix            [/usr/local]
  --includes          [-I/usr/local/include/php -I/usr/local/include/php/main -I/usr/local/include/php/TSRM -I/usr/local/include/php/Zend -I/usr/local/include/php/ext -I/usr/local/include/php/ext/date/lib]
  --ldflags           [ -Wl,-z,relro,-z,now -pie -L/usr/local/lib64]
  --libs              [-lcrypt  -lc-client  -ltidy -largon2 -lresolv -lncurses -laspell -lpspell -lrt -lldap -llber -lstdc++ -lcrypt -lpam -lgmp -lbz2 -lutil -lrt -lm -ldl  -lsystemd -lxml2 -lgssapi_krb5 -lkrb5 -lk5crypto -lcom_err -lssl -lcrypto -lsqlite3 -lz -lcurl -lxml2 -lenchant -lgmodule-2.0 -lglib-2.0 -lffi -lssl -lcrypto -lz -lpng16 -lwebp -ljpeg -lXpm -lX11 -lfreetype -lgssapi_krb5 -lkrb5 -lk5crypto -lcom_err -lssl -lcrypto -licuio -licui18n -licuuc -licudata -lonig -lsqlite3 -ledit -lxml2 -lnetsnmp -lm -lssl -lssl -lcrypto -lxml2 -lsodium -largon2 -lxml2 -lxml2 -lxml2 -lxslt -lm -lxml2 -lexslt -lxslt -lm -lgcrypt -ldl -lgpg-error -lxml2 -lzip -lz -lssl -lcrypto -lcrypt ]
  --extension-dir     [/usr/local/lib/php/extensions/no-debug-non-zts-20200930]
  --include-dir       [/usr/local/include/php]
  --man-dir           [/usr/local/php/man]
  --php-binary        [/usr/local/bin/php]
  --php-sapis         [ cli embed fpm phpdbg cgi]
  --ini-path          [/usr/local/lib]
  --ini-dir           [/etc/centminmod/php.d]
  --configure-options [--enable-fpm --enable-opcache --enable-intl --enable-pcntl --with-mcrypt --with-snmp --enable-embed=shared --with-mhash --with-zlib --with-gettext --enable-exif --with-zip --with-libzip --with-bz2 --enable-soap --enable-sockets --enable-sysvmsg --enable-sysvsem --enable-sysvshm --with-mysql-sock=/var/lib/mysql/mysql.sock --with-curl --enable-gd --with-xmlrpc --enable-bcmath --enable-calendar --enable-ftp --enable-gd-native-ttf --with-freetype --with-jpeg --with-png-dir=/usr --with-xpm --with-webp --with-t1lib=/usr --enable-shmop --with-pear --enable-mbstring --with-openssl --with-mysql=mysqlnd --with-libdir=lib64 --with-mysqli=mysqlnd --enable-pdo --with-pdo-sqlite --with-pdo-mysql=mysqlnd --enable-inline-optimization --with-imap --with-imap-ssl --with-kerberos --with-readline --with-libedit --with-gmp --with-pspell --with-tidy --with-enchant --with-fpm-user=nginx --with-fpm-group=nginx --with-ldap --with-ldap-sasl --with-password-argon2 --with-sodium=/usr/local --with-pic --with-config-file-scan-dir=/etc/centminmod/php.d --with-fpm-systemd --with-ffi --with-xsl PKG_CONFIG_PATH=/usr/local/lib64/pkgconfig: PNG_CFLAGS=-I/usr/include PNG_LIBS=-L/usr/lib64 -lpng16 ICU_CFLAGS=-fPIC ICU_LIBS=-L/usr/lib64 -licuio -licui18n -licuuc -licudata ONIG_CFLAGS=-I/usr/include ONIG_LIBS=-L/usr/lib64 -lonig LIBSODIUM_CFLAGS=-fPIC LIBSODIUM_LIBS=-L/usr/local/lib64 -lsodium LIBZIP_CFLAGS=-fPIC LIBZIP_LIBS=-L/usr/local/lib64 -lzip]
  --version           [8.0.30]
  --vernum            [80030]
```

```
php -v
PHP 8.0.30 (cli) (built: Jun 25 2024 17:03:17) ( NTS )
Copyright (c) The PHP Group
Zend Engine v4.0.30, Copyright (c) Zend Technologies
    with Zend OPcache v8.0.30, Copyright (c), by Zend Technologies
```

```
wget -V
GNU Wget 1.21.4 built on linux-gnu.

-cares +digest +gpgme +https +ipv6 +iri +large-file +metalink +nls 
+ntlm +opie +psl +ssl/gnutls 

Wgetrc: 
    /usr/local/etc/wgetrc (system)
Locale: 
    /usr/local/share/locale 
Compile: 
    ccache gcc -DHAVE_CONFIG_H -DSYSTEM_WGETRC="/usr/local/etc/wgetrc" 
    -DLOCALEDIR="/usr/local/share/locale" -I. -I../lib -I../lib 
    -I/usr/include/libassuan2 -I/usr/local/include 
    -I/usr/include/p11-kit-1 -DHAVE_LIBGNUTLS -DNDEBUG -O2 -g -pipe 
    -Wall -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS 
    -fexceptions -fstack-protector-strong -fasynchronous-unwind-tables 
    -fstack-clash-protection -fcf-protection -grecord-gcc-switches -m64 
    -mtune=generic 
Link: 
    ccache gcc -I/usr/include/libassuan2 -I/usr/local/include 
    -I/usr/include/p11-kit-1 -DHAVE_LIBGNUTLS -DNDEBUG -O2 -g -pipe 
    -Wall -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS 
    -fexceptions -fstack-protector-strong -fasynchronous-unwind-tables 
    -fstack-clash-protection -fcf-protection -grecord-gcc-switches -m64 
    -mtune=generic -L/usr/local/lib -lmetalink -lpcre2-8 -luuid -lidn2 
    -lnettle -lgnutls -lz -lpsl -lgpgme -lgpg-error -lassuan 
    ../lib/libgnu.a 

Copyright (C) 2015 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later
<http://www.gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Originally written by Hrvoje Niksic <hniksic@xemacs.org>.
Please send bug reports and questions to <bug-wget@gnu.org>
```

```
redis-cli info server
# Server
redis_version:7.2.5
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:2da2678dc88a4deb
redis_mode:standalone
os:Linux 4.18.0-553.5.1.el8_10.x86_64 x86_64
arch_bits:64
monotonic_clock:POSIX clock_gettime
multiplexing_api:epoll
atomicvar_api:c11-builtin
gcc_version:8.5.0
process_id:1740
process_supervised:systemd
run_id:ab18105af0455964799655a051ea6fd0317b6880
tcp_port:6379
server_time_usec:1719337888729237
uptime_in_seconds:928
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:8061856
executable:/usr/bin/redis-server
config_file:/etc/redis/redis.conf
io_threads_active:0
listener0:name=tcp,bind=127.0.0.1,bind=-::1,port=6379
```

```
mysqladmin ver
mysqladmin  Ver 9.1 Distrib 10.4.34-MariaDB, for Linux on x86_64
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Server version          10.4.34-MariaDB
Protocol version        10
Connection              Localhost via UNIX socket
UNIX socket             /var/lib/mysql/mysql.sock
Uptime:                 16 min 45 sec

Threads: 3  Questions: 5  Slow queries: 0  Opens: 20  Flush tables: 1  Open tables: 13  Queries per second avg: 0.004
```

# EL7 & Elevate Left Over Packages

EL7 & Elevate packages

```
rpm -qa | egrep 'el7|elevate|leapp' | sort
centmin-libatomic_ops-7.6.12-1.el7.x86_64
centos-release-scl-2-3.el7.centos.noarch
centos-release-scl-rh-2-3.el7.centos.noarch
devtoolset-11-annobin-docs-10.38-1.el7.noarch
devtoolset-11-annobin-plugin-gcc-10.38-1.el7.x86_64
devtoolset-11-binutils-2.36.1-1.el7.2.x86_64
devtoolset-11-binutils-devel-2.36.1-1.el7.2.x86_64
devtoolset-11-dwz-0.14-2.el7.x86_64
devtoolset-11-elfutils-0.185-2.el7.x86_64
devtoolset-11-elfutils-debuginfod-client-0.185-2.el7.x86_64
devtoolset-11-elfutils-debuginfod-client-devel-0.185-2.el7.x86_64
devtoolset-11-elfutils-devel-0.185-2.el7.x86_64
devtoolset-11-elfutils-libelf-0.185-2.el7.x86_64
devtoolset-11-elfutils-libelf-devel-0.185-2.el7.x86_64
devtoolset-11-elfutils-libs-0.185-2.el7.x86_64
devtoolset-11-gcc-11.2.1-9.1.el7.x86_64
devtoolset-11-gcc-c++-11.2.1-9.1.el7.x86_64
devtoolset-11-libasan-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-libatomic-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-libgccjit-11.2.1-9.1.el7.x86_64
devtoolset-11-libgccjit-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-libgccjit-docs-11.2.1-9.1.el7.x86_64
devtoolset-11-libitm-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-liblsan-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-libquadmath-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-libstdc++-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-libtsan-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-ltrace-0.7.91-1.el7.x86_64
devtoolset-11-make-4.3-1.el7.x86_64
devtoolset-11-make-devel-4.3-1.el7.x86_64
devtoolset-11-memstomp-0.1.5-6.el7.x86_64
devtoolset-11-runtime-11.1-2.el7.x86_64
devtoolset-11-strace-5.13-3.3.el7.x86_64
devtoolset-7-binutils-2.28-11.el7.x86_64
devtoolset-7-gcc-7.3.1-5.16.el7.x86_64
devtoolset-7-gcc-c++-7.3.1-5.16.el7.x86_64
devtoolset-7-libstdc++-devel-7.3.1-5.16.el7.x86_64
devtoolset-7-runtime-7.1-4.el7.x86_64
elevate-release-1.0-2.el7.noarch
kernel-3.10.0-1160.119.1.el7.x86_64
kernel-3.10.0-1160.80.1.el7.x86_64
leapp-0.14.0-1.el7.noarch
leapp-data-almalinux-0.2-5.el7.noarch
leapp-deps-el8-5.0.8-100.202203181036Z.249925a3.master.el8.noarch
leapp-repository-deps-el8-5.0.8-100.202203181036Z.249925a3.master.el8.noarch
leapp-upgrade-el7toel8-0.16.0-6.el7.elevate.20.noarch
libicu62-62.2-1.el7.remi.x86_64
libicu73-73.2-1.el7.remi.x86_64
oniguruma5php-6.9.9-1.el7.remi.x86_64
oniguruma5php-devel-6.9.9-1.el7.remi.x86_64
python2-leapp-0.14.0-1.el7.noarch
rpmforge-release-0.5.3-1.el7.rf.x86_64
yum-plugin-priorities-1.1.31-54.el7_8.noarch
```

Remove these packages & verify their removal

```
rpm -e --nodeps $(rpm -qa | egrep 'el7|elevate|leapp' | sort |xargs)
rpm -qa | egrep 'el7|elevate|leapp' | sort
```

Elevate & Leapp only

```
rpm -qa | grep elevate
leapp-upgrade-el7toel8-0.16.0-6.el7.elevate.20.noarch
elevate-release-1.0-2.el7.noarch

rpm -qa | grep leapp
leapp-upgrade-el7toel8-0.16.0-6.el7.elevate.20.noarch
leapp-data-almalinux-0.2-5.el7.noarch
python2-leapp-0.14.0-1.el7.noarch
leapp-deps-el8-5.0.8-100.202203181036Z.249925a3.master.el8.noarch
leapp-0.14.0-1.el7.noarch
leapp-repository-deps-el8-5.0.8-100.202203181036Z.249925a3.master.el8.noarch
```

EL7 only

```
rpm -qa | egrep 'el7|elevate|leapp' | sort
centmin-libatomic_ops-7.6.12-1.el7.x86_64
centos-release-scl-2-3.el7.centos.noarch
centos-release-scl-rh-2-3.el7.centos.noarch
devtoolset-11-annobin-docs-10.38-1.el7.noarch
devtoolset-11-annobin-plugin-gcc-10.38-1.el7.x86_64
devtoolset-11-binutils-2.36.1-1.el7.2.x86_64
devtoolset-11-binutils-devel-2.36.1-1.el7.2.x86_64
devtoolset-11-dwz-0.14-2.el7.x86_64
devtoolset-11-elfutils-0.185-2.el7.x86_64
devtoolset-11-elfutils-debuginfod-client-0.185-2.el7.x86_64
devtoolset-11-elfutils-debuginfod-client-devel-0.185-2.el7.x86_64
devtoolset-11-elfutils-devel-0.185-2.el7.x86_64
devtoolset-11-elfutils-libelf-0.185-2.el7.x86_64
devtoolset-11-elfutils-libelf-devel-0.185-2.el7.x86_64
devtoolset-11-elfutils-libs-0.185-2.el7.x86_64
devtoolset-11-gcc-11.2.1-9.1.el7.x86_64
devtoolset-11-gcc-c++-11.2.1-9.1.el7.x86_64
devtoolset-11-libasan-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-libatomic-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-libgccjit-11.2.1-9.1.el7.x86_64
devtoolset-11-libgccjit-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-libgccjit-docs-11.2.1-9.1.el7.x86_64
devtoolset-11-libitm-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-liblsan-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-libquadmath-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-libstdc++-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-libtsan-devel-11.2.1-9.1.el7.x86_64
devtoolset-11-ltrace-0.7.91-1.el7.x86_64
devtoolset-11-make-4.3-1.el7.x86_64
devtoolset-11-make-devel-4.3-1.el7.x86_64
devtoolset-11-memstomp-0.1.5-6.el7.x86_64
devtoolset-11-runtime-11.1-2.el7.x86_64
devtoolset-11-strace-5.13-3.3.el7.x86_64
devtoolset-7-binutils-2.28-11.el7.x86_64
devtoolset-7-gcc-7.3.1-5.16.el7.x86_64
devtoolset-7-gcc-c++-7.3.1-5.16.el7.x86_64
devtoolset-7-libstdc++-devel-7.3.1-5.16.el7.x86_64
devtoolset-7-runtime-7.1-4.el7.x86_64
elevate-release-1.0-2.el7.noarch
kernel-3.10.0-1160.119.1.el7.x86_64
kernel-3.10.0-1160.80.1.el7.x86_64
leapp-0.14.0-1.el7.noarch
leapp-data-almalinux-0.2-5.el7.noarch
leapp-upgrade-el7toel8-0.16.0-6.el7.elevate.20.noarch
libicu62-62.2-1.el7.remi.x86_64
libicu73-73.2-1.el7.remi.x86_64
oniguruma5php-6.9.9-1.el7.remi.x86_64
oniguruma5php-devel-6.9.9-1.el7.remi.x86_64
python2-leapp-0.14.0-1.el7.noarch
rpmforge-release-0.5.3-1.el7.rf.x86_64
yum-plugin-priorities-1.1.31-54.el7_8.noarch
```