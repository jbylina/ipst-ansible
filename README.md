# iTESLA
http://www.itesla-project.eu/

## License
https://www.mozilla.org/en-US/MPL/2.0/

# IPST ansible scripts

This ansible playbook builds and installs IPST on a target CentOS linux. In particular:

* it installs on the target server all the OS packages, development tools and SW runtimes, needed to build and run the platform (this step can be skipped, if all the packages are already installed on the target server: ref. section Notes, below)
 - C/C++development tools (gcc, g++, make, git, wget, bzip2, zlib )
 - CMake
 - OpenMPI (sources installation + local build with flag --enable-mpi-thread-multiple)
 - Oracle JDK8 
 - Apache Maven
 - WildFly
 - DBMaria
 - MATLAB Runtime (MCR)
 - Hades ( http://www.itesla-pst.org )
 - default data in case of itesla testing
 - Nordic44 2015 dataset ( http://www.sciencedirect.com/science/article/pii/S2352340917300409 )

* it clones on the target server(s) the iPST project sources from GitHub (pull if the repository already exists), 
builds the 'platform distribution' and installs it, in place.

* integrate testing data and test the ddb-load-eurostag and security-analysis tools provide by itesla

## Requirements

- target server(s) and control machine (where these scripts are executed) running a CentOS Linux distribution (tested on v6.5, v7.x), accessible via ssh
- an user with admin rights, to be able to install the above mentioned OS packages, development tools and SW runtimes 
- ansible (ref. http://docs.ansible.com/ansible/intro_installation.html#latest-release-via-yum)



## Notes

### iPST

The default target directories for iPST are (these are all configurable, see below in the usage section): 
 - $HOME/sources/ipst  (iPST sources) 
 - $HOME/sources/powsybl-core  (powsybl-core sources)
 - $HOME/thirdparty  (thirdparty libraries: boost, hdf5, etc.)
 - $HOME/ipst (iPST binaries)

ipst, powsybl-core, provide a local logging file
 - << playbook_dir >>/install_ipst_<< target_machine >>.log
 - << playbook_dir >>/install_core_<< target_machine >>.log

SSH is used to communicate with the hosts: apparently connections do not work properly if a connection is not done once to all the servers; to fix it, run once
   ssh-keyscan server_IP or server_NAME >> ~/.ssh/known_hosts, for each target server

The installation process takes 'some time' to complete, at least for the first run when all the required SW packages
are downloaded (e.g. MATLAB MCR) and (possibly) compiled (e.g. openmpi).
iPST installation requires 2Gb of disk space;  MCR installation, if enabled, requires further 2Gb in /tmp during the installation and 2Gb in /usr


### security-analysis tool

security-analysis, ...  his usage allow check full itesla installation

the default target directories are
 - $HOME/hades2LF  (Hades binary contains)

need additional local files
 - $HOME/tmp/hades/hades2LF.tar.gz (binary file)

provide local logging file
 - << playbook_dir >>/install_hades_<< target_machine >>.log

**Remarque:** due to Hades usage, before processing, a disclaimer is prompted


### ddb-load-eurostag tool

ddb-load-eurostag, ... tool usage allow check full itesla installation

the default target directories are
 - $HOME/minimalist_DDB (datas to process)

need additional local file
 - $HOME/minimalist_DDB.zip

provide local logging file
 - << playbook_dir >>/install_ddb_<< target_machine >>.log'




   
## Usage

1. Copy the example inventory file ansible-scripts/ipst-hosts.example to ansible-scripts/ipst-hosts and edit the latter one, adding IPs / specific connection parameters of the target servers to the [ipst_hosts] section.

2. Copy the file ansible-scripts/group_vars_ipst_hosts.example to ansible-scripts/group_vars/ipst_hosts;
   Edit it to configure the installation procedure to use proxies (downloading OS packages, runtimes and IPST sources), 
     or to customize the installation parameters (e.g. setting iPST target installation directories, enable/disable requirements and MCR packages installations), 
   example:
```
# file: group_vars/ipst_hosts
---
## ipst_environment - (default is not set)
#ipst_environment:
#    http_proxy: http://proxyhttp.mydomain.com:port
#    https_proxy: https://proxyhttps.mydomain.com:port


## ipst_environment - (default is not set)
ipst_environment: {}

## install_prerequisites - (default is True)
install_prerequisites: True

## install_MCR - (Matlab Compiler Runtime, default is False)
install_MCR: False

## True force source build (default value is False)
force_build: False




##--------------------------------------------------------------------------
##-- ispt parameters

## project home - OS target project root path, will be contain all source, binary files and standalone tools
##   (default value is ansible_env.HOME)
user_project_home: "{{ ansible_env.HOME }}"

## project_temporary - OS target project temporary file path, relative to the project home
##   (default value is tmp)
user_project_temporary: "tmp"

## log_file_prefix - prefix for all locale log file
##   (default value is {{ playbook_dir }}/install)
user_log_file_prefix: "{{ playbook_dir }}/install"

## log_file_postfix - postfix for all locale log file
##   (default value is {{ inventory_hostname+'.log' }})
user_log_file_postfix: "{{ inventory_hostname+'.log' }}"

## project_bin - OS target binary directory relative to the project home
##    (default value is ipst)
user_project_bin: "ipst"

## project_branch - github branch project
##    (default value is master)
ipst_project_branch: "master"

## source_root - OS target source project directory relative to the project home
##    (default value is sources)
user_source_root: "sources"

## user_ipst_enabled if True, build ipst project
user_ipst_enabled: True

## ipst_source - OS target source ipst module directory relative to the source home
##    (default value is ipst)
user_ipst_source: "ipst"

## ipst_github - ipst github repository
##    (default value is https://github.com/itesla/ipst.git)
user_ipst_github: "https://github.com/itesla/ipst.git"

## project_branch - ipst github branch project
##    (default value is master)
ipst_project_branch: "master"

## project_branch - powsybl-core github branch project
##    (default value is master)
powsybl_core_project_branch: "master"

## core_source - OS target source core module directory relative to the source home
##    (default value is powsybl-core )
user_core_source: "powsybl-core"

## core_github - core github repository
##    (default value is https://github.com/itesla/powsybl-core.git)
user_core_github: https://github.com/powsybl/powsybl-core.git

## user_cgmes_enabled if True, build cgmes project
user_cgmes_enabled: False 

## cgmes_source - OS target source cgmes module directory relative to the source home
###    (default value is cgmes)
user_cgmes_source: "CGMES"

### cgmes_github - cgmes github repository
###    (default value is https://github.com/itesla/CGMES.git)
user_cgmes_github: https://github.com/itesla/CGMES.git

## project_branch - powsybl-cgmes github branch project
##    (default value is master)
powsybl_cgmes_project_branch: "master"

## source_thirdparty - OS target source thirdparty module directory relative to the project home
##    (default value is thirdparty)
user_source_thirdparty: "thirdparty"


##--------------------------------------------------------------------------
##-- define java user parameters

##- jdk8_url - define jdk url
user_jdk8_url: "http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u161-linux-x64.tar.gz"


##- jdk8_home - OS target jdk home, relative to the project home
##    (default value is java/jdk1.8.0_144)
user_jdk8_home: "java/jdk1.8.0_161"


##- jdk8_log - 0 no screen debug
##    (default value is 0)
user_jdk8_log: 1



##--------------------------------------------------------------------------
##-- define maven user parameters
##    ipst.maven/defaults/main.yml file define default values

## maven_version - maven install version
##    (default value is 3.5.0)
user_maven_version: "3.5.3"

## maven_dest_path - OS target maven parent install path , relative to the project home
##    (default value is maven)
user_maven_dest_path: "maven"

## maven_mirror - maven mirror prefix url
##    (default value is http://apache.mirrors.ovh.net/ftp.apache.org/dist/maven/maven-3/3.5.0/binaries)
user_maven_mirror: "http://apache.mirrors.ovh.net/ftp.apache.org/dist/maven/maven-3/3.5.3/binaries"

## maven_proxies - maven proxies
##    (default value is none)
#user_maven_proxies:
#    - {host: "proxyhttps.mydomain.com", port: "443", username: "username", password: "password", protocol: "https"}

## maven repository location, default value is $HOME/.m2/repository
#user_maven_repository: "$HOME/.m2/repository"

##--------------------------------------------------------------------------
##-- define wildfly user options
##    ipst.wildfly/defaults/main.yml file define default values

## wildfly_url - wildfly url binary download
##    (default value is http://download.jboss.org/wildfly/8.1.0.Final/wildfly-8.1.0.Final.zip)
user_wildfly_url: http://download.jboss.org/wildfly/8.1.0.Final/wildfly-8.1.0.Final.zip

## wildfly_dest_path - OS target wildfly install path
##    (default value is ansible_env.HOME)
user_wildfly_dest_path: "{{ ansible_env.HOME }}/wildfly"



##--------------------------------------------------------------------------
## ear_name - ear name
##    (default value is iidm-ddb-ear.ear)
user_ear_name: "iidm-ddb-ear.ear"

## ear_path - ear source path, relative to the source install path
##    (default value is iidm-ddb/iidm-ddb-ear/target)
user_ear_path: "iidm-ddb/iidm-ddb-ear/target"


##--------------------------------------------------------------------------
## ddb-load-eurostag Part

## ddb_process - (provide database with data and test the integration, default is False)
user_ddb_process: False

## ddb_archive_name - data archive to integrate
##    (default value is minimalist_DDB.zip)
user_ddb_archive_name: "minimalist_DDB.zip"

## ddb_locale_path - locale path of the data archive
##    (default value is ~/)
user_ddb_locale_path: "~/"

## ddb_remote_path - OS target path for the data relative to the home project
##    (default value is minimalist_DDB)
user_ddb_remote_path: "minimalist_DDB"

## ddb_eurostag_version - define the eurostag version
##     (default value is 5.1.1)
user_ddb_eurostag_version: "5.1.1"


##--------------------------------------------------------------------------
##- security-analysis Part

## hades_process - process_hades - (install and process hades test, default is False)
user_hades_process: True
user_hades_tests: False

## hades_archive_name - hades binary archive name
##    (default value is hades2LF.tar.gz)
user_hades_archive_name: hades2LF.tar.gz

## hades_src - local hades binary path archive
##    (default value is ~/tmp/hades)
user_hades_src: "~/Downloads"

## hades_dst - OS target hades binary prefix destination path, relative to the project home
##    (default value is empty)
user_hades_dst: ""

## hades_create - OS target hades binary destination path relative to user_hades_dst
##    (default value is hades2LF)
user_hades_create: "hades2LF"

## hades_situation_test - this is the situation to integrate to hades
##    (default value is 20160101_0030_FO5_FR0.xiidm.gz)
user_hades_situation_test: "20160101_0030_FO5_FR0.xiidm.gz"

## hades_situation_prefix - hades data prefix path, relative to user_hades_situations_prefix_src on local machine, relative to user_hades_situations_home on OS target
##    (default value is /DATA/IIDM/FO/2016/01/01)
user_hades_situation_prefix: "/DATA/IIDM/FO/2016/01/01"

## hades_situations_prefix_src - local hades data path prefix
##    (default value is ~/tmp/situations/grovslb1/local)
user_hades_situations_prefix_src: "~/tmp/situations/grovslb1/local"

## OS target situation path, relative to the project home
##    (default value is empty)
user_hades_situations_home: ""

## OS target hades processing result, relative to the project home
##    (default value is security-analysis-result.txt)
user_hades_result_file: "security-analysis-result.txt"

## activate a disclaimer for hades usage, if activate, process is stopped meanwhile user press [Enter] key
##    (default value is false)
user_hades_isDisclaimerPrompt: False

## the disclaimer
##    (default value is 'no disclaimer')
user_hades_disclaimer: 'Disclaimer
 Please confirm you want to ...
 Press return to continue.
 Press Ctrl+c and then "a" to abort'

```

Notes: 
- when variable install_prerequisites is set to False, installations of packages requiring admin rights are skipped (to be used when those packages are already installed on the target machines)


3. Run: ansible-playbook -i ipst-hosts ./ipst.yml -u USERNAME -k

  ansible connects to the remote servers (listed in ipst-hosts file) as USERNAME (asking interactively for USERNAME's password, when needed - parameter k)
  and starts the build+installation process.

  Additional command line parameters are needed to execute tasks on the target servers that require admin rights, e.g. to install system packages;
  Ansible 
  
  - --become-method=su (or sudo) , to specify what privilege escalation tool to use (default is: sudo) 
  - --become-user=ADMINUSER  , to specify what admin user to use (default is root)
  - --ask-become-pass , interactively asks for ADMINUSER password
  
  These parameters could also be set, per host, in the inventory file ipst-hosts (examples in ipst-hosts.example); note that command line parameter names and inventory file names are different
  (e.g. ansible_become_user instead of become-user)
   
4. To install the Nordic44 dataset, run:  ansible-playbook -i ipst-hosts ./ipst.yml -e "installNordic44=true" -t Nordic44


  More details on ansible-playbook command and parameters, here:  http://docs.ansible.com/ansible/playbooks_variables.html 
  Privileges escalation related configurations are explained here: http://docs.ansible.com/ansible/become.html
