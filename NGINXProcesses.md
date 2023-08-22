# NGINX Processes

## Master and Worker Processes
  NGINX has one master process and one or more worker processes. If caching is enabled, the cache loader and cache manager processes also run at startup.
  
* The master process performs the privileged operations such as reading configuration and binding to ports, and then creates and maintain a small number of child processes (the next three types).
* The worker processes do the actual processing of requests. They handle network connections, read and write content to disk, and communicate with upstream servers. NGINX relies on OS-dependent mechanisms to efficiently distribute requests among worker processes. The number of worker processes is defined by the worker_processes directive in the nginx.conf configuration file and can either be set to a fixed number or configured to adjust automatically to the number of available CPU cores.
* The cache loader process runs at startup to load the disk‑based cache into memory, and then exits. It is scheduled conservatively, so its resource demands are low.
* The cache manager process runs periodically and prunes entries from the disk caches to keep them within the configured sizes.






References: 
* https://docs.nginx.com/nginx/admin-guide/basic-functionality/runtime-control/
* https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/
