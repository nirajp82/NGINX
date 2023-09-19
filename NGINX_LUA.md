```nginx
server {
        # Listen on port 80 for HTTP traffic
        listen 80;	
        location / {
	                  set $upstream_svr_addr '';   
	                  access_by_lua_block {			
			                    ngx.var.upstream_addr = "https://mydomain.com"
	                  }  	   
	       proxy_pass $upstream_svr_addr;
}
```
