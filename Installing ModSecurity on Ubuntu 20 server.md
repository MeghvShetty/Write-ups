# Installing ModSecurity on Ubuntu 20


Source : [NGINX Offical Docs](https://www.nginx.com/blog/compiling-and-installing-modsecurity-for-open-source-nginx/) [Document](https://www.nginx.com/blog/compiling-and-installing-modsecurity-for-open-source-nginx/) 
 
### 1 ) Install NGINX 
Install NGINX, note for this documented steps you would NGINX 1.11.5 or later is required
### 2 ) Install Prerequisite Package
Install the following packages to complete the remaining setps in this document.
```
$ sudo apt-get install -y apt-utils autoconf automake build-essential git libcurl4-openssl-dev libgeoip-dev liblmdb-dev libpcre++-dev libtool libxml2-dev libyajl-dev pkgconf wget zlib1g-dev
```

### 3 ) Download and compile the ModSecurity 3.0 source code
In ModSecurity 3.0’s new modular architecture, libmodsecurity is the core component which includes all rules and functionality. The second main component in the architecture is a connector that links libmodsecurity to the web server it is running with. There are separate connectors for NGINX, Apache HTTP Server, and IIS.

*To Compile* **Libmodsecurity**

1. clone the GitHub repository
```
$ git clone --depth 1 -b v3/master --single-branch https://github.com/SpiderLabs/ModSecurity
``` 
2. cd into ModSecurity and compile the source code:

 ```
$ cd ModSecurity
$ git submodule init
$ git submodule update
$ ./build.sh
$ ./configure
$ make
$ sudo make install
$ cd ..
```
```
export MODSECURITY_LIB="/usr/local/modsecurity/lib/"
export MODSECURITY_INC="/usr/local/modsecurity/include/"

 ```
### 4 ) Download the NGINX connector for ModSecurity and compile it as a Dynamic Module
Compile the ModSecurity connector for NGINX as a dynamic module for NGINX.
1. Clone the Github repository
```
$ git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git
```
2. Determine which version of NGINX is running on the host where the MOdSecurity module will be loaded.
```
$ nginx -v
 nginx version: nginx/1.18.0 (Ubuntu)
```
3. Download the source code corresponding to the installed version of NGINX (*the complete sources are required even though only the dynamic module is being compiled*):
```
$ wget http://nginx.org/download/nginx-1.18.0.tar.gz
$ tar zxvf nginx-1.18.0.tar.gz
```
4. Compile the dynamic module and cop it to the standard directory for modules:
```
$ cd nginx-1.18.0
$ ./configure --with-compat --add-dynamic-module=../ModSecurity-nginx
$ make modules
$ cp objs/ngx_http_modsecurity_module.so /etc/nginx/modules
$ cd ..
```

### 5 ) Load the NGINX ModSecurity Connector Dynamic Module
Add the following load_module directive to the main (top‑level) context in /etc/nginx/nginx.conf. It instructs NGINX to load the ModSecurity dynamic module when it processes the configuration:

```
load_module modules/ngx_http_modsecurity_module.so;
```

### 6 ) Configure, Enable, and Test ModSecurity
1. Set up the appropriate ModSecurity configuration file. Here we’re using the recommended ModSecurity configuration provided by TrustWave Spiderlabs, the corporate sponsors of ModSecurity
```
$ mkdir /etc/nginx/modsec
$ wget -P /etc/nginx/modsec/ https://raw.githubusercontent.com/SpiderLabs/ModSecurity/v3/master/modsecurity.conf-recommended
$ mv /etc/nginx/modsec/modsecurity.conf-recommended /etc/nginx/modsec/modsecurity.conf
```
2. To guarantee that ModSecurity can find the unicode.mapping file (distributed in the top‑level ModSecurity directory of the GitHub repo), copy it to /etc/nginx/modsec.

```
cp ModSecurity/unicode.mapping /etc/nginx/modsec
```
3. Change the SecRuleEngine directive in the configuration to change from the default “detection only” mode to actively dropping malicious traffic.
 ```
  sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /etc/nginx/modsec/modsecurity.conf
 ```

