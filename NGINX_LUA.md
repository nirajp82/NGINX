```nginx
http {
    lua_package_path "/etc/nginx/x_lua_modules/?.lua;;";
    server {
        # Listen on port 80 for HTTP traffic
        listen 80;	
        location / {
	                  set $upstream_svr_addr '';   
	                  access_by_lua_block {
                                   local upstream_util = require("upstream_util")		
			           ngx.var.upstream_addr = upstream_util.find_upstream_server()
	                  }  	   
	       proxy_pass $upstream_svr_addr;
   }
}
```

```lua
local upstream_util = {}
function upstream_util.find_upstream_server(pod, tenant, api)
       return "https://mydomain.com"
end
return upstream_util
```
