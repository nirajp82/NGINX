```sh
# Installation
sudo apt-get install nginx-plus-module-lua
sudo apt-get install luarocks
sudo luarocks install lua-cjson
sudo luarocks install lua-resty-http
```

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
			           ngx.var.upstream_svr_addr = upstream_util.find_upstream_server()
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

```lua
function get_module_info()
      -- local http = require("resty.http")
      -- Iterate over the package.loaded table to list loaded Lua modules
      for moduleName, _ in pairs(package.loaded) do
            local modulePath = package.searchpath(moduleName, package.path)
            ngx.log(ngx.DEBUG, "Module: " ..  moduleName .. " Location: " ..  (modulePath or ""))
      end
end
```
