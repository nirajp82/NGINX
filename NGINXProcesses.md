# NGINX Processes

## Master and Worker Processes
  NGINX has one master process and one or more worker processes. If caching is enabled, the cache loader and cache manager processes also run at startup.
  
* The master process performs the privileged operations such as reading configuration and binding to ports, and then creates and maintain a small number of child processes (the next three types).
* The worker processes do the actual processing of requests. They handle network connections, read and write content to disk, and communicate with upstream servers. NGINX relies on OS-dependent mechanisms to efficiently distribute requests among worker processes. The number of worker processes is defined by the worker_processes directive in the nginx.conf configuration file and can either be set to a fixed number or configured to adjust automatically to the number of available CPU cores.

    The NGINX configuration recommended in most cases – running one worker process per CPU core – makes the most efficient use of hardware resources. You configure it by setting the auto parameter on the worker_processes directive:

`worker_processes auto;`
When an NGINX server is active, only the worker processes are busy. Each worker process handles multiple connections in a nonblocking fashion, reducing the number of context switches.

Each worker process is single‑threaded and runs independently, grabbing new connections and processing them. The processes can communicate using shared memory for shared cache data, session persistence data, and other shared resources.

* The cache loader process runs at startup to load the disk‑based cache into memory, and then exits. It is scheduled conservatively, so its resource demands are low.
* The cache manager process runs periodically and prunes entries from the disk caches to keep them within the configured sizes.



## Controlling NGINX
To reload your configuration, you can stop or restart NGINX, or send signals to the master process. A signal can be sent by running the nginx command (invoking the NGINX executable) with the -s argument.

`nginx -s <SIGNAL>`
where <SIGNAL> can be one of the following:

* quit – Shut down gracefully (the SIGQUIT signal)
* reload – Reload the configuration file (the SIGHUP signal)
* reopen – Reopen log files (the SIGUSR1 signal)
* stop – Shut down immediately (or fast shutdown, the SIGTERM singal)

  The kill utility can also be used to send a signal directly to the master process. The process ID of the master process is written, by default, to the nginx.pid file, which is located in the /usr/local/nginx/logs or /var/run directory.




References: 
* https://docs.nginx.com/nginx/admin-guide/basic-functionality/runtime-control/
* https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/
