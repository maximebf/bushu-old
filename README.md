# Bushu

Bushu is a very simple command line deployment tool. It only needs a bash shell and ssh to work.

## Using Bushu

Bushu deploys roles onto servers. A role is composed of tasks and optionnaly shared libs.
Multiple roles can be deployed to one server. Common roles can also be defined for all servers.

When bushu deploys roles to a server, it will first copy and install itself onto this one.
Thus, two configuration environments exist: the local and remote ones. This method allows 
recursive deployments. For example, you can contact a first server that acts as an access point
for multiple servers.

Bushu can be configured in interactive mode or using a configuration file.

### Command line arguments

    bushu [/path/to/configuration/file] [command [args...]]
    
If no commands are specified, Bushu will start in interactive mode.
You can print available commands using _help_ (also available in interactive mode).

    bushu help

### Configuration file

The configuration is a bash file where all Bushu commands are available. Bushu will try to load
the file situated at /etc/bushu.conf when it starts. If a configuration file is speicified as an 
argument, it will be loaded next. Other files can be loaded using the *load_config* command.

Bushu uses env vars to configure itself. You can modify the local configuration by setting env vars 
traditionnaly:

    ROLES_DIR=~/roles
    
You can set env vars for both the local and remote environments using _setconf_:

    setconf ROLES_DIR ~/roles

### Interactive mode

In interactive mode, Bushu provides a shell where you can type commands to configure, deploy
or monitor your infrastructure.
    
### Deploying

Simply execute the _deploy_ command either from the interactive mode or from the command line as an argument 
(in which case the configuration file must be specified).

    bushu /path/to/conf deploy
    
The deploy command won't deploy a role already deployed on a server.
    
### Infrastructure status and update

You can check if all roles are deployed according to the current configuration using the _status_ command.

    bushu /path/to/conf status

If the configuration file changes, if you have removed or added roles to a server for example, you can run
the deploy command again to update your infrastructure.


## Roles and servers

### Roles

A role is a directory containing bash files with the .task extension representing tasks.
Tasks files can be ordered by prefixing them with numbers, like 00_something.task, 01_something.task...

Additionaly, roles can contain bash files with the .lib extension. These files will be
loaded when Bushu starts. Lib files from all available roles will be loaded. Thus, you can define
reusable functions between roles.

Example:
	/myrole
	    01_download.task
        02_configure.task
        03_make.task
	    common.lib

You can also create a special uninstall.task which will be executed when the role is uninstalled.

Roles can also have sub-roles. Imagine we have a _mysql_ role, we could have two subroles: _master_ 
and _slave_. The first one would install and configure a master sql server, the second one a slave.

Subroles are subfolders inside the role folder. Subroles name must be prefixed with their parent's name
separated with a slash (eg. mysql/slave).

Example:
    /mysql
        /slave
            install.task
        /master
            install.task
        mysql.lib
        
### Servers

All servers must be accessible using ssh and without prompting for a password (ie. you must use keys).
You can add servers using the *add_server* command.

    bushu> add_server HOST ROLE1 [ROLE2 [ROLE3...]]
    
Example:

    bushu> add_server example.com database web
    
You can add additiona roles using *add_role*:

    bushu> add_role example.com load_balancer
    
## Example

Let's create a roles directories in ~ and add two roles: mysql and apache.

    mkdir -p ~/roles/{mysql,apache}
    echo "aptitude install mysql-server" > ~/roles/mysql/install.task
    echo "aptitude install apache2" > ~/roles/apache/install.task
    echo "
        ROLES_DIR=~/roles
        add_server database.example.com mysql
        add_server example.com apache" > ~/bushu.conf
    bushu ~/bushu.conf
        > deploy
        > status
        
## Tips

*   You can setup your servers and roles from the interactive mode and save your configuration using
    *save_config*
*   It is adviced to keep a configuration file to avoid setuping the environment each times.
    
