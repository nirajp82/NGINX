The provided command sequence is used to compile and install the Nginx web server on a Linux system. 
 
1. `yum -y install gcc make zlib-devel pcre-devel openssl-devel wget nano`: This command uses the `yum` package manager to install various development and utility packages that are required (prerequisite) for compiling Nginx. The `-y` flag is used to automatically answer "yes" to any prompts during the installation.
2. `sudo su`
   
3. `cd ~`: Go to the root directory.

4. `wget http://nginx.org/download/nginx-1.24.0.tar.gz`: This command uses the `wget` utility to download the Nginx source code archive from the specified URL ([http://nginx.org/download/nginx-1.24.0.tar.gz](http://nginx.org/download/nginx-1.24.0.tar.gz)) and save it to the current directory.

5. `tar -xzvf nginx-1.24.0.tar.gz`: This command extracts the contents of the downloaded archive using `tar`. The options `-xzvf` stand for:
   - `-x`: Extract files from the archive.
   - `-z`: Decompress the archive if it's compressed with gzip.
   - `-v`: Verbose mode to display progress.
   - `-f`: Specifies the input file (nginx-1.24.0.tar.gz in this case).

6. `useradd nginx`: This command creates a system user named "nginx." This user will likely be used to run the Nginx server process.

7. `mkdir -p /var/lib/nginx/tmp/`: This command creates the `/var/lib/nginx/tmp/` directory, which was previously specified as the temporary directory for storing client request bodies in the configure step.

8. `chown -R nginx.nginx /var/lib/nginx/tmp/`: This command changes the ownership of the `/var/lib/nginx/tmp/` directory and its contents to the "nginx" user and group. This is necessary to ensure that Nginx can write to this directory for temporary storage.
   
9. `cd nginx-1.24.0/` 
   
10. `./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-http_mp4_module`: This command configures the Nginx build process with various options. Let's break down the important options:
   - `--prefix=/usr/share/nginx`: Specifies the installation directory prefix.
   - `--sbin-path=/usr/sbin/nginx`: Sets the path for the Nginx binary executable.
   - `--modules-path=/usr/lib64/nginx/modules`: Defines the directory for Nginx modules.
   - `--conf-path=/etc/nginx/nginx.conf`: Specifies the main configuration file path.
   - `--error-log-path=/var/log/nginx/error.log`: Sets the error log file path.
   - `--http-log-path=/var/log/nginx/access.log`: Sets the HTTP access log file path.
   - `--http-client-body-temp-path=/var/lib/nginx/tmp/client_body`: Specifies the temporary directory for storing client request bodies.
   - `--pid-path=/var/run/nginx.pid`: Defines the PID file path.
   - `--lock-path=/var/lock/subsys/nginx`: Sets the path for the lock file.
   - `--user=nginx` and `--group=nginx`: Specifies the user and group under which Nginx worker process will run. We will create this user later
   - `--with-http_mp4_module`: Enables the MP4 module for serving MP4 video files.
  
11. `make`: Compile and build the Nginx software from its source code.
    
12. `make install`: Install the compiled software onto your Linux system, making it ready for use as a web server. These commands are essential for the process of building and deploying software on a Linux system.

13. Manage the Nginx web server as a system service.

    https://www.nginx.com/resources/wiki/start/topics/examples/systemd/
    
    `vim /lib/systemd/system/nginx.service`

    The file `/lib/systemd/system/nginx.service` is a systemd service unit file for the Nginx web server. This file is used by the systemd init system to manage the Nginx service, including starting, stopping, restarting, and managing its behavior.

Here's an overview of the purpose and contents of the `nginx.service` file:
   - The `nginx.service` file contains configuration directives that specify how systemd should interact with the Nginx service. Below is a simplified example of what you might find in this file:

   ```ini
   [Unit]
    Description=The NGINX HTTP and reverse proxy server
    After=syslog.target network-online.target remote-fs.target nss-lookup.target
    Wants=network-online.target
    
    [Service]
    Type=forking
    PIDFile=/run/nginx.pid
    ExecStartPre=/usr/sbin/nginx -t
    ExecStart=/usr/sbin/nginx
    ExecReload=/usr/sbin/nginx -s reload
    ExecStop=/bin/kill -s QUIT $MAINPID
    PrivateTmp=true
    
    [Install]
    WantedBy=multi-user.target
   ```

   Here's what some of the key directives mean:
   - `Description`: A human-readable description of the service.
   - `Type`: Defines the process startup type. "forking" is used when the service forks itself into the background (common for Nginx).
   - `ExecStartPre` and `ExecStart`: Commands to start the Nginx service and verify its configuration.
   - `ExecReload`: Command to reload the Nginx configuration.
   - `KillSignal`: The signal used to stop the Nginx service (default is SIGQUIT).
   - `TimeoutStopSec`: Maximum time to wait for the service to stop gracefully.
   - `KillMode`: How to kill processes when stopping the service.
   - `PrivateTmp`: Creates a private /tmp directory for the service to isolate it from other processes.

   - systemd uses this file to control the Nginx service. You can use standard systemd commands like `systemctl start nginx`, `systemctl stop nginx`, `systemctl restart nginx`, and `systemctl status nginx` to manage the Nginx service based on the configuration in this unit file.

In summary, the `/lib/systemd/system/nginx.service` file is essential for systemd to manage the Nginx web server as a system service. It specifies how Nginx should be started, stopped, and reloaded, among other service management tasks.
    



After executing these commands, you should have Nginx compiled and configured on your system, ready to be started with the `nginx` command.
