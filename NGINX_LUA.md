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


## How to return a list of CNAME records for a given domain name using Lua script inside NGINX+

### Lua script

```lua
local function get_cname_records(domain)
  -- Open a TCP socket to the DNS server
  local socket = net.socket(net.AF_INET, net.SOCK_STREAM)
  socket:connect("8.8.8.8", 53)

  -- Create a DNS query packet
  local query = ngx.encode_query(domain, "CNAME", 1)

  -- Send the query packet to the DNS server
  socket:send(query)

  -- Receive the response packet from the DNS server
  local response = socket:receive()

  -- Decode the response packet
  local answers, errcode, errmsg = ngx.decode_response(response)

  -- Close the TCP socket
  socket:close()

  -- If the query was successful, return the list of CNAME records
  if errcode == 0 then
    local cname_records = {}
    for _, answer in ipairs(answers) do
      table.insert(cname_records, answer.cname)
    end
    return cname_records
  end

  -- Otherwise, return nil
  return nil
end

-- Get the list of CNAME records for the given domain name
local domain = "google.com"
local cname_records = get_cname_records(domain)

-- Print the list of CNAME records to the NGINX error log
if cname_records then
  for _, cname_record in ipairs(cname_records) do
    ngx.log(ngx.ERR, "CNAME record: " .. cname_record)
  end
else
  ngx.log(ngx.ERR, "No CNAME records found for domain " .. domain)
end


### Explanation

The Lua script first opens a TCP socket to the DNS server at the address `8.8.8.8`. The DNS server is a computer that knows how to translate domain names into IP addresses.

The script then creates a DNS query packet for the given domain name and sends it to the DNS server. The DNS query packet contains a request for the CNAME records of the domain name.

The script then receives the response packet from the DNS server and decodes it. The response packet contains the answer to the DNS query.

If the query was successful, the script returns a table of CNAME records for the given domain name. Otherwise, the script returns nil.

The script then prints the list of CNAME records to the NGINX error log.

### Usage

To use the script, you need to save it as a `.lua` file and load it into your NGINX+ configuration file using the following directive:


lua_package_path "/path/to/cname-records.lua";


Once the script is loaded, you can call the `get_cname_records()` function to get the list of CNAME records for any domain name. The function returns a table of CNAME records, or nil if no CNAME records are found.

For example, the following code would print the list of CNAME records for the domain name `google.com` to the NGINX error log:

lua
local cname_records = get_cname_records("google.com")

if cname_records then
  for _, cname_record in ipairs(cname_records) do
    ngx.log(ngx.ERR, "CNAME record: " .. cname_record)
  end
else
  ngx.log(ngx.ERR, "No CNAME records found for domain google.com")
end


You can use this script in your NGINX+ configuration files or in your Lua scripts to implement a variety of tasks, such as:

* Resolving the CNAME records of upstream servers.
* Implementing load balancing algorithms.
* Implementing rate limiting algorithms.
Use code with caution. Learn more



Bard may display inaccurate or offensive information that doesn’t represent Google’s views. Bard Privacy Notice

