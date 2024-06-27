Centmin Mod based CentOS 7 in-place migration upgrade from [AlmaLinux 8 then to AlmaLinux 9 via AlmaLinux Elevate](https://wiki.almalinux.org/elevate/ELevating-CentOS7-to-AlmaLinux-9.html#migrating-almalinux-8-to-almalinux-9). Do not use on production live servers. Test on a test server first. This is just alpha testing. 

**Notes:**

1. EL9+ memory requirements are higher for operating system itself. So paired with Centmin Mod LEMP stack usage, recommended 4GB memory + 4GB swap disk for EL8+. So do not attempt CentOS 7 in-place migration upgrade from AlmaLinux 8 then to AlmaLinux 9 if you have less than 4GB memory + 4GB swap disk.
   ```
    free -mlt
                  total        used        free      shared  buff/cache   available
    Mem:           3664         766         977           2        1919        2615
    Low:           3664        2686         977
    High:             0           0           0
    Swap:          4095           4        4091
    Total:         7760         770        5069
   ```
2. There are 3 parts to the CentOS 7 in-place migration upgrade from AlmaLinux 8 then to AlmaLinux 9 outlined below. Please do a full data backup for your Nginx vhost sites + MariaDB MySQL databases before attempting the below in-place migration steps from AlmaLinux 8 then to AlmaLinux 9.

# Part 1 - Install AlmaLinux Elevate & Reboot Server

On Centmin Mod AlmaLinux 8 that was [just migrated to from CentOS 7 installed server](https://github.com/centminmod/centminmod-centos7-elevate) run the following commands.


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
mkdir -p /root/tools/pureftpd-el8
cp -a /etc/pure-ftpd/pureftpd.passwd /root/tools/pureftpd-el8/pureftpd.passwd
cp -a /etc/pure-ftpd/pureftpd.pdb /root/tools/pureftpd-el8/pureftpd.pdb

# remove versionlocks
yum versionlock delete ImageMagick ImageMagick-devel ImageMagick-c++ ImageMagick-c++-devel ImageMagick-libs LibRaw ImageMagick-djvu

yum install -y http://repo.almalinux.org/elevate/elevate-release-latest-el$(rpm --eval %rhel).noarch.rpm
sed -i '/exclude=/d' /etc/yum.conf
sed -i '/exclude=/d' /etc/dnf/dnf.conf
yum install -y leapp-upgrade leapp-data-almalinux
rpm -e percona-release
leapp preupgrade
# inspect /var/log/leapp/answerfile
cat /var/log/leapp/answerfile
yum -y remove make-devel
leapp upgrade
echo "rebooting server..."
reboot
```

An example of the output from `leapp upgrade` and reboot process console output can be seen [here](https://github.com/centminmod/centminmod-centos7-elevate/blob/master/el8-el9-leapp-upgrade-console.md)

# Part 2 - Prep AlmaLinux 8

```
yum-config-manager --enable crb
alternatives --set python /usr/bin/python3
rm -rf /etc/yum.repos.d/epel.repo
rm -rf /etc/yum.repos.d/epel-testing.repo
yum -y reinstall epel-release
yum -y reinstall bash-completion perl-FindBin perl-diagnostics libc-client libc-client-devel systemd-libs open-sans-fonts libidn2-devel libpsl-devel gpgme-devel gnutls-devel virt-what acl libacl-devel attr libattr-devel lz4-devel gawk unzip libuuid-devel sqlite-devel bc wget lynx screen ca-certificates yum-utils bash mlocate subversion rsyslog dos2unix boost-program-options net-tools imake bind-utils libatomic_ops-devel time coreutils autoconf cronie crontabs cronie-anacron gcc gcc-c++ automake libtool make libXext-devel unzip patch sysstat openssh flex bison file libtool-ltdl-devel krb5-devel libXpm-devel nano gmp-devel aspell-devel numactl lsof pkgconfig gdbm-devel tk-devel bluez-libs-devel iptables* rrdtool diffutils which perl-Math-BigInt perl-Test-Simple perl-ExtUtils-Embed perl-ExtUtils-MakeMaker perl-Time-HiRes perl-libwww-perl perl-Net-SSLeay cyrus-imapd cyrus-sasl-md5 cyrus-sasl-plain strace cmake git net-snmp-libs net-snmp-utils iotop libvpx libvpx-devel t1lib t1lib-devel expect readline readline-devel libedit libedit-devel libxslt libxslt-devel openssl openssl-devel curl curl-devel openldap openldap-devel zlib zlib-devel gd gd-devel pcre pcre-devel gettext gettext-devel libidn libidn-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel e2fsprogs e2fsprogs-devel libc-client libc-client-devel cyrus-sasl cyrus-sasl-devel pam pam-devel libaio libaio-devel libevent libevent-devel recode recode-devel libtidy libtidy-devel net-snmp net-snmp-devel enchant enchant-devel lua lua-devel mailx perl-LWP-Protocol-https OpenEXR-devel OpenEXR-libs atk cups-libs fftw-libs-double fribidi gdk-pixbuf2 ghostscript-devel gl-manpages graphviz gtk2 hicolor-icon-theme ilmbase ilmbase-devel jasper-devel jasper-libs jbigkit-devel jbigkit-libs lcms2 lcms2-devel libICE-devel libSM-devel libXaw libXcomposite libXcursor libXdamage-devel libXfixes-devel libXi libXinerama libXmu libXrandr libXt-devel libXxf86vm-devel libdrm-devel libfontenc librsvg2 libtiff libtiff-devel libwebp libwebp-devel libwmf-lite mesa-libGL-devel mesa-libGLU mesa-libGLU-devel poppler-data urw-fonts xorg-x11-font-utils --skip-broken
yum -y install checksec systemd-libs xxhash-devel libzstd xxhash libzstd-devel datamash qrencode jq clang clang-devel jemalloc jemalloc-devel zstd python2-pip libmcrypt libmcrypt-devel libraqm oniguruma5php oniguruma5php-devel figlet moreutils nghttp2 libnghttp2 libnghttp2-devel pngquant optipng jpegoptim pwgen pigz pbzip2 xz pxz lz4 bash-completion mlocate re2c kernel-headers kernel-devel --skip-broken
yum -y module disable nginx mariadb php
systemctl restart NetworkManager
systemctl status NetworkManager --no-pager

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
CCACHE_VER="3.7.12"
DIR_TMP=/svr-setup
CCCACHELINK="https://centminmod.com/centminmodparts/ccache/ccache-${CCACHE_VER}.tar.gz"
wget -O "${DIR_TMP}/ccache-${CCACHE_VER}.tar.gz" "$CCCACHELINK"
cd ${DIR_TMP}
tar -xzf "ccache-${CCACHE_VER}.tar.gz"
cd ccache-${CCACHE_VER}
make clean -s
./configure
make -j$(nproc) -s
make install -s
ccache -V

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

echo "1" > /etc/centminmod/email-primary.ini
echo "2" > /etc/centminmod/email-secondary.ini
# reinstall pcre
/usr/local/src/centminmod/addons/wget.sh install
# reinstall custom wget and any centmin mod configured auto applied and installed settings/packages
echo 24 | /usr/local/src/centminmod/centmin.sh

wget -O centmin-option-4.sh https://github.com/centminmod/centminmod-centos7-elevate/raw/master/scripts/centmin-option-4.sh
wget -O centmin-option-5.sh https://github.com/centminmod/centminmod-centos7-elevate/raw/master/scripts/centmin-option-5.sh
wget -O centmin-option-10.sh https://github.com/centminmod/centminmod-centos7-elevate/raw/master/scripts/centmin-option-10.sh
wget -O centmin-option-15.sh https://github.com/centminmod/centminmod-centos7-elevate/raw/master/scripts/centmin-option-15.sh

chmod +x centmin-option-4.sh centmin-option-5.sh centmin-option-10.sh centmin-option-15.sh

# reinstall Nginx
yum -y install GeoIP GeoIP-devel bash-completion jemalloc-devel --enablerepo=remi
/usr/local/src/centminmod/tools/geoipdb-update.sh
/usr/local/src/centminmod/tools/geoip2db-update.sh
cmupdate
./centmin-option-4.sh

# create a mariadb 10.6 el8/el9 reinstall
# install mariadb 10.6 version to centos 7 for compatibility first
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
baseurl = ${MDB_MIRROR_BASEURL}/10.6/rhel9-amd64
module_hotfixes=1
gpgkey=${MDB_MIRROR_BASEURL}/RPM-GPG-KEY-MariaDB
gpgcheck=1
EOF

cat /etc/yum.repos.d/mariadb.repo

yum -y install MariaDB-client MariaDB-common MariaDB-compat MariaDB-devel MariaDB-server MariaDB-shared

cp -a /etc/my.cnf /etc/my.cnf-newold-elevated-el9

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
yum -y install https://rpms.remirepo.net/enterprise/remi-release-9.rpm
sed -i 's|repo_gpgcheck=1|repo_gpgcheck=0|g' /etc/yum.repos.d/remi*.repo
yum clean all
yum makecache
grep repo_gpgcheck /etc/yum.repos.d/remi*.repo
yum -y module disable composer
yum -y module reset redis
yum -y module enable redis:remi-7.2
yum -y install libc-client uw-imap-devel --enablerepo=remi,remi-safe

# Percona YUM repo resetup
rm -rf /etc/yum.repos.d/percona-*
yum -y install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
percona-release show
percona-release disable all
percona-release enable tools

# re-install redis server
/usr/local/src/centminmod/addons/redis-server-install.sh install
redis-cli info server

# re-install php-fpm
yum -y install libtidy-devel libraqm-devel
yum -y reinstall ImageMagick ImageMagick-devel ImageMagick-c++ ImageMagick-c++-devel ImageMagick-libs LibRaw --enablerepo=remi --disableplugin=priorities,versionlock -x ImageMagick7* --disablerepo=base
./centmin-option-15.sh
./centmin-option-5.sh

# re-install memcached server
./centmin-option-10.sh
systemctl restart memcached
chkconfig memcached on

# create a pure-ftpd el8/el9 reinstall
yum -y reinstall pure-ftpd
cp -a /etc/pure-ftpd/pure-ftpd.conf /etc/pure-ftpd/pure-ftpd.conf-old-el7
\cp -af /root/tools/pureftpd-el8/pureftpd.passwd /etc/pure-ftpd/pureftpd.passwd
\cp -af /root/tools/pureftpd-el8/pureftpd.pdb /etc/pure-ftpd/pureftpd.pdb
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

# haveged
yum -y install haveged rng-tools
mkdir -p /etc/systemd/system/haveged.service.d
cat > "/etc/systemd/system/haveged.service.d/haveged.conf" <<EFF
[Service]
ExecStart=
ExecStart=/usr/sbin/haveged -w 4067 -v 1 --Foreground
EFF
systemctl daemon-reload
systemctl enable haveged
systemctl restart haveged

# remove left over el7 and elevate/leapp packages
rpm -e --nodeps $(rpm -qa | egrep 'el8|elevate|leapp' | sort |xargs)
sudo rm -rf /root/tmp_leapp_py3
sed -i '/exclude=/d' /etc/yum.conf
sed -i '/exclude=/d' /etc/dnf/dnf.conf

# yum update
yum -y update --disableplugin=priorities --setopt=deltarpm=0 --enablerepo=remi
```

AlmaLinux 8 to AlmaLinux 9 via AlmaLinux Elevate is work in progress so not 100% tested right now.

# Centmin Mod LEMP Stack On AlmaLinux 9 Elevated

```
cat /etc/redhat-release 
AlmaLinux release 9.4 (Seafoam Ocelot)
```
```
cat /etc/os-release
NAME="AlmaLinux"
VERSION="9.4 (Seafoam Ocelot)"
ID="almalinux"
ID_LIKE="rhel centos fedora"
VERSION_ID="9.4"
PLATFORM_ID="platform:el9"
PRETTY_NAME="AlmaLinux 9.4 (Seafoam Ocelot)"
ANSI_COLOR="0;34"
LOGO="fedora-logo-icon"
CPE_NAME="cpe:/o:almalinux:almalinux:9::baseos"
HOME_URL="https://almalinux.org/"
DOCUMENTATION_URL="https://wiki.almalinux.org/"
BUG_REPORT_URL="https://bugs.almalinux.org/"

ALMALINUX_MANTISBT_PROJECT="AlmaLinux-9"
ALMALINUX_MANTISBT_PROJECT_VERSION="9.4"
REDHAT_SUPPORT_PRODUCT="AlmaLinux"
REDHAT_SUPPORT_PRODUCT_VERSION="9.4"
SUPPORT_END=2032-06-01
```

```
yum repolist all
repo id                                     repo name                                                                                    status
appstream                                   AlmaLinux 9 - AppStream                                                                      enabled
appstream-debuginfo                         AlmaLinux 9 - AppStream - Debug                                                              disabled
appstream-source                            AlmaLinux 9 - AppStream - Source                                                             disabled
backports-rsync                             AlmaLinux 9 Backports - rsync                                                                enabled
baseos                                      AlmaLinux 9 - BaseOS                                                                         enabled
baseos-debuginfo                            AlmaLinux 9 - BaseOS - Debug                                                                 disabled
baseos-source                               AlmaLinux 9 - BaseOS - Source                                                                disabled
crb                                         AlmaLinux 9 - CRB                                                                            enabled
crb-debuginfo                               AlmaLinux 9 - CRB - Debug                                                                    disabled
crb-source                                  AlmaLinux 9 - CRB - Source                                                                   disabled
epel                                        Extra Packages for Enterprise Linux 9 - x86_64                                               enabled
epel-cisco-openh264                         Extra Packages for Enterprise Linux 9 openh264 (From Cisco) - x86_64                         enabled
epel-cisco-openh264-debuginfo               Extra Packages for Enterprise Linux 9 openh264 (From Cisco) - x86_64 - Debug                 disabled
epel-cisco-openh264-source                  Extra Packages for Enterprise Linux 9 openh264 (From Cisco) - x86_64 - Source                disabled
epel-debuginfo                              Extra Packages for Enterprise Linux 9 - x86_64 - Debug                                       disabled
epel-source                                 Extra Packages for Enterprise Linux 9 - x86_64 - Source                                      disabled
epel-testing                                Extra Packages for Enterprise Linux 9 - Testing - x86_64                                     disabled
epel-testing-debuginfo                      Extra Packages for Enterprise Linux 9 - Testing - x86_64 - Debug                             disabled
epel-testing-source                         Extra Packages for Enterprise Linux 9 - Testing - x86_64 - Source                            disabled
extras                                      AlmaLinux 9 - Extras                                                                         enabled
extras-debuginfo                            AlmaLinux 9 - Extras - Debug                                                                 disabled
extras-source                               AlmaLinux 9 - Extras - Source                                                                disabled
highavailability                            AlmaLinux 9 - HighAvailability                                                               disabled
highavailability-debuginfo                  AlmaLinux 9 - HighAvailability - Debug                                                       disabled
highavailability-source                     AlmaLinux 9 - HighAvailability - Source                                                      disabled
mariadb                                     MariaDB                                                                                      enabled
nfv                                         AlmaLinux 9 - NFV                                                                            disabled
nfv-debuginfo                               AlmaLinux 9 - NFV - Debug                                                                    disabled
nfv-source                                  AlmaLinux 9 - NFV - Source                                                                   disabled
plus                                        AlmaLinux 9 - Plus                                                                           disabled
plus-debuginfo                              AlmaLinux 9 - Plus - Debug                                                                   disabled
plus-source                                 AlmaLinux 9 - Plus - Source                                                                  disabled
prel-release-noarch                         Percona Release release/noarch YUM repository                                                enabled
remi                                        Remi's RPM repository for Enterprise Linux 9 - x86_64                                        disabled
remi-debuginfo                              Remi's RPM repository for Enterprise Linux 9 - x86_64 - debuginfo                            disabled
remi-modular                                Remi's Modular repository for Enterprise Linux 9 - x86_64                                    enabled
remi-modular-debuginfo                      Remi's Modular repository for Enterprise Linux 9 - x86_64 - debuginfo                        disabled
remi-modular-test                           Remi's Modular testing repository for Enterprise Linux 9 - x86_64                            disabled
remi-modular-test-debuginfo                 Remi's Modular testing repository for Enterprise Linux 9 - x86_64 - debuginfo                disabled
remi-safe                                   Safe Remi's RPM repository for Enterprise Linux 9 - x86_64                                   disabled
remi-safe-debuginfo                         Remi's RPM repository for Enterprise Linux 9 - x86_64 - debuginfo                            disabled
remi-test                                   Remi's test RPM repository for Enterprise Linux 9 - x86_64                                   disabled
remi-test-debuginfo                         Remi's test RPM repository for Enterprise Linux 9 - x86_64 - debuginfo                       disabled
resilientstorage                            AlmaLinux 9 - ResilientStorage                                                               disabled
resilientstorage-debuginfo                  AlmaLinux 9 - ResilientStorage - Debug                                                       disabled
resilientstorage-source                     AlmaLinux 9 - ResilientStorage - Source                                                      disabled
rt                                          AlmaLinux 9 - RT                                                                             disabled
rt-debuginfo                                AlmaLinux 9 - RT - Debug                                                                     disabled
rt-source                                   AlmaLinux 9 - RT - Source                                                                    disabled
sap                                         AlmaLinux 9 - SAP                                                                            disabled
sap-debuginfo                               AlmaLinux 9 - SAP - Debug                                                                    disabled
sap-source                                  AlmaLinux 9 - SAP - Source                                                                   disabled
saphana                                     AlmaLinux 9 - SAPHANA                                                                        disabled
saphana-debuginfo                           AlmaLinux 9 - SAPHANA - Debug                                                                disabled
saphana-source                              AlmaLinux 9 - SAPHANA - Source                                                               disabled
tools-release-sources                       Percona Tools release/sources YUM repository                                                 disabled
tools-release-x86_64                        Percona Tools release/x86_64 YUM repository                                                  enabled
```

```
nginx -V
nginx version: nginx/1.27.0 (260624-063334-almalinux9-kvm-ed365ca)
built by gcc 13.2.1 20231205 (Red Hat 13.2.1-6) (GCC) 
built with OpenSSL 3.0.7 1 Nov 2022
TLS SNI support enabled
```
> configure arguments: --with-ld-opt='-Wl,-E -L/usr/local/zlib-cf/lib -L/usr/local/nginx-dep/lib -ljemalloc -Wl,-z,relro,-z,now -Wl,-rpath,/usr/local/zlib-cf/lib:/usr/local/nginx-dep/lib -pie -flto=2 -flto-compression-level=3 -fuse-ld=gold' --with-cc-opt='-I/usr/local/zlib-cf/include -I/usr/local/nginx-dep/include -m64 -march=native -fPIC -g -O3 -fstack-protector-strong -flto=2 -flto-compression-level=3 -fuse-ld=gold --param=ssp-buffer-size=4 -Wformat -Wno-pointer-sign -Wimplicit-fallthrough=0 -Wno-implicit-function-declaration -Wno-cast-align -Wno-builtin-declaration-mismatch -Wno-deprecated-declarations -Wno-int-conversion -Wno-unused-result -Wno-vla-parameter -Wno-maybe-uninitialized -Wno-return-local-addr -Wno-array-parameter -Wno-alloc-size-larger-than -Wno-address -Wno-array-bounds -Wno-discarded-qualifiers -Wno-stringop-overread -Wno-stringop-truncation -Wno-missing-field-initializers -Wno-unused-variable -Wno-format -Wno-error=unused-result -Wno-missing-profile -Wno-stringop-overflow -Wno-free-nonheap-object -Wno-discarded-qualifiers -Wno-bad-function-cast -Wno-dangling-pointer -Wno-array-parameter -fcode-hoisting -Wno-cast-function-type -Wno-format-extra-args -Wp,-D_FORTIFY_SOURCE=2' --prefix=/usr/local/nginx --sbin-path=/usr/local/sbin/nginx --conf-path=/usr/local/nginx/conf/nginx.conf --build=260624-063334-almalinux9-kvm-ed365ca --with-compat --without-pcre2 --with-http_stub_status_module --with-http_secure_link_module --with-libatomic --with-http_gzip_static_module --add-dynamic-module=../ngx_http_geoip2_module --with-http_sub_module --with-http_addition_module --with-http_image_filter_module=dynamic --with-http_geoip_module --with-stream_geoip_module --with-stream_realip_module --with-stream_ssl_preread_module --with-threads --with-stream --with-stream_ssl_module --with-http_realip_module --add-dynamic-module=../ngx-fancyindex-0.4.2 --add-module=../ngx_cache_purge-2.5.1 --add-dynamic-module=../ngx_devel_kit-0.3.2 --add-dynamic-module=../set-misc-nginx-module-0.33 --add-dynamic-module=../echo-nginx-module-0.63 --add-module=../redis2-nginx-module-0.15 --add-module=../ngx_http_redis-0.4.0-cmm --add-module=../memc-nginx-module-0.20 --add-module=../srcache-nginx-module-0.33 --add-dynamic-module=../headers-more-nginx-module-0.37 --with-pcre-jit --with-zlib=../zlib-cloudflare-1.3.3 --with-zlib-opt=-fPIC --with-http_ssl_module --with-http_v2_module

```
php-config
Usage: /usr/local/bin/php-config [OPTION]
Options:
  --prefix            [/usr/local]
  --includes          [-I/usr/local/include/php -I/usr/local/include/php/main -I/usr/local/include/php/TSRM -I/usr/local/include/php/Zend -I/usr/local/include/php/ext -I/usr/local/include/php/ext/date/lib]
  --ldflags           [ -Wl,-z,relro,-z,now -pie -L/opt/openssl/lib -L/usr/lib64/../lib64 -L/usr/local/lib64]
  --libs              [-lcrypt  -lc-client  -ltidy -largon2 -lncurses -laspell -lpspell -lrt -lldap -llber -lstdc++ -lcrypt -lpam -lgmp -lbz2 -lrt -lm  -lsystemd -lxml2 -lgssapi_krb5 -lkrb5 -lk5crypto -lcom_err -lssl -lcrypto -ldl -lpthread -lsqlite3 -lz -lcurl -lssl -lcrypto -ldl -lpthread -lxml2 -lenchant -lgmodule-2.0 -lglib-2.0 -lffi -lssl -lcrypto -ldl -lpthread -lz -lpng16 -lwebp -ljpeg -lXpm -lX11 -lfreetype -lgssapi_krb5 -lkrb5 -lk5crypto -lcom_err -lssl -lcrypto -ldl -lpthread -licuio -licui18n -licuuc -licudata -lonig -lsqlite3 -ledit -lxml2 -lnetsnmp -lm -lm -lssl -lssl -lcrypto -lxml2 -lsodium -largon2 -lxml2 -lxml2 -lxml2 -lxslt -lxml2 -lexslt -lxslt -lxml2 -lzip -lz -lssl -lcrypto -ldl -lpthread -lcrypt ]
  --extension-dir     [/usr/local/lib/php/extensions/no-debug-non-zts-20200930]
  --include-dir       [/usr/local/include/php]
  --man-dir           [/usr/local/php/man]
  --php-binary        [/usr/local/bin/php]
  --php-sapis         [ cli embed fpm phpdbg cgi]
  --ini-path          [/usr/local/lib]
  --ini-dir           [/etc/centminmod/php.d]
  --configure-options [--enable-fpm --enable-opcache --enable-intl --enable-pcntl --with-mcrypt --with-snmp --enable-embed=shared --with-mhash --with-zlib --with-gettext --enable-exif --with-zip --with-libzip --with-bz2 --enable-soap --enable-sockets --enable-sysvmsg --enable-sysvsem --enable-sysvshm --with-mysql-sock=/var/lib/mysql/mysql.sock --with-curl --enable-gd --with-xmlrpc --enable-bcmath --enable-calendar --enable-ftp --enable-gd-native-ttf --with-freetype --with-jpeg --with-png-dir=/usr --with-xpm --with-webp --with-t1lib=/usr --enable-shmop --with-pear --enable-mbstring --with-openssl=/opt/openssl --with-mysql=mysqlnd --with-libdir=lib64 --with-mysqli=mysqlnd --enable-pdo --with-pdo-sqlite --with-pdo-mysql=mysqlnd --enable-inline-optimization --with-imap --with-imap-ssl --with-kerberos --with-readline --with-libedit --with-gmp --with-pspell --with-tidy --with-enchant --with-fpm-user=nginx --with-fpm-group=nginx --with-ldap --with-ldap-sasl --with-password-argon2 --with-sodium=/usr/local --with-pic --with-config-file-scan-dir=/etc/centminmod/php.d --with-fpm-systemd --with-ffi --with-xsl PKG_CONFIG_PATH=/usr/local/lib64/pkgconfig:/opt/openssl/lib/pkgconfig OPENSSL_CFLAGS=-I/opt/openssl/include OPENSSL_LIBS=-L/opt/openssl/lib -lssl -lcrypto -ldl -lpthread PNG_CFLAGS=-I/usr/include PNG_LIBS=-L/usr/lib64 -lpng16 ICU_CFLAGS=-fPIC ICU_LIBS=-L/usr/lib64 -licuio -licui18n -licuuc -licudata ONIG_CFLAGS=-I/usr/include ONIG_LIBS=-L/usr/lib64 -lonig LIBSODIUM_CFLAGS=-fPIC LIBSODIUM_LIBS=-L/usr/local/lib64 -lsodium LIBZIP_CFLAGS=-fPIC LIBZIP_LIBS=-L/usr/local/lib64 -lzip]
  --version           [8.0.30]
  --vernum            [80030]
```

```
php -v
PHP 8.0.30 (cli) (built: Jun 26 2024 06:46:25) ( NTS )
Copyright (c) The PHP Group
Zend Engine v4.0.30, Copyright (c) Zend Technologies
    with Zend OPcache v8.0.30, Copyright (c), by Zend Technologies
```

```
wget -V
GNU Wget 1.21.4 built on linux-gnu.

-cares +digest -gpgme +https +ipv6 +iri +large-file -metalink +nls 
+ntlm +opie +psl +ssl/openssl 

Wgetrc: 
    /root/.wgetrc (user)
    /usr/local/etc/wgetrc (system)
Locale: 
    /usr/local/share/locale 
Compile: 
    ccache gcc -DHAVE_CONFIG_H -DSYSTEM_WGETRC="/usr/local/etc/wgetrc" 
    -DLOCALEDIR="/usr/local/share/locale" -I. -I../lib -I../lib 
    -DHAVE_LIBSSL -DNDEBUG -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 
    -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 
    -grecord-gcc-switches -mtune=generic 
Link: 
    ccache gcc -DHAVE_LIBSSL -DNDEBUG -O2 -g -pipe -Wall 
    -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong 
    --param=ssp-buffer-size=4 -grecord-gcc-switches -mtune=generic 
    -lpcre2-8 -luuid -lidn2 -lssl -lcrypto -lz -lpsl ../lib/libgnu.a 

Copyright (C) 2015 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later
<http://www.gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Originally written by Hrvoje Niksic <hniksic@xemacs.org>.
Please send bug reports and questions to <bug-wget@gnu.org>.
```

```
memcached -V
memcached 1.6.22
```

```
redis-cli info server
# Server
redis_version:7.2.5
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:ce975a21d248780e
redis_mode:standalone
os:Linux 5.14.0-362.24.2.el9_3.x86_64 x86_64
arch_bits:64
monotonic_clock:POSIX clock_gettime
multiplexing_api:epoll
atomicvar_api:c11-builtin
gcc_version:11.4.1
process_id:61019
process_supervised:systemd
run_id:74af066a89be726498b84e750fc4b9b5f9153df0
tcp_port:6379
server_time_usec:1719384924390619
uptime_in_seconds:2611
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:8108892
executable:/usr/bin/redis-server
config_file:/etc/redis/redis.conf
io_threads_active:0
listener0:name=tcp,bind=127.0.0.1,bind=-::1,port=6379
```

```
mysqladmin ver
mysqladmin  Ver 10.0 Distrib 10.6.18-MariaDB, for Linux on x86_64
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Server version          10.6.18-MariaDB
Protocol version        10
Connection              Localhost via UNIX socket
UNIX socket             /var/lib/mysql/mysql.sock
Uptime:                 50 min 36 sec

Threads: 1  Questions: 786  Slow queries: 0  Opens: 219  Open tables: 60  Queries per second avg: 0.258
```

# EL8 & Elevate Left Over Packages

EL8 & Elevate packages

```
rpm -qa | egrep 'el8|elevate|leapp' | sort
```

Remove these packages & verify their removal

```
rpm -e --nodeps $(rpm -qa | egrep 'el8|elevate|leapp' | sort |xargs)
rpm -qa | egrep 'el8|elevate|leapp' | sort
```
