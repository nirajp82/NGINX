---

## Setting Up an IP Whitelist in Nginx

In this example, we will allow access to the "/downloads" path based on a predefined whitelist of IP addresses.

### Table of Contents

1. **Introduction**
2. **Configuration Steps**
3. **Testing the Whitelist**

---

## 1. Introduction

An IP whitelist, also known as an access control list, is a security measure used to allow access only to specific IP addresses or IP ranges while blocking all other traffic. In Nginx, this is achieved through configuration directives like `allow` and `deny`.

In this example, we will configure Nginx to allow access to the "/downloads" path only for IP addresses listed in a whitelist file.

## 2. Configuration Steps

1. **Edit Nginx Configuration:**

   Open your Nginx configuration file for editing. This file is commonly located at `/etc/nginx/nginx.conf` or `/etc/nginx/sites-available/default`.

2. **Create a Location Block:**

   Inside the server block of your Nginx configuration, create a location block for the path you want to protect. In this example, we are using "/downloads."

   ```nginx
   server {
       listen 80;
       server_name localhost;

       location /downloads {
           # Whitelist configuration goes here
       }
   }
   ```

3. **Include the Whitelist:**

   Inside the location block, include the whitelist file that contains the allowed IP addresses.

   ```nginx
   location /downloads {
       include /etc/nginx/conf.d/whitelist_ipaddress;
       deny all;
   }
   ```

4. **Create the Whitelist File:**

   Create a whitelist file at the specified path (`/etc/nginx/conf.d/whitelist_ipaddress` in this example) and add the IP addresses you want to whitelist.

   Example whitelist file content:

   ```nginx
   allow 127.0.0.1;
   allow 127.0.0.3;
   ```

5. **Reload Nginx:**

   After making changes to your Nginx configuration, reload Nginx to apply the new settings.

   ```bash
   sudo systemctl reload nginx
   ```

## 3. Testing the Whitelist

To test the whitelist, try accessing the "/downloads" path from a client with an IP address that is either in the whitelist or not. The client with an allowed IP address should be granted access, while others will be denied.


## 4. whitelist.conf
```nginx
server {
        listen 80;
        server_name localhost;

        location /downloads {
                # allow 127.0.0.1;
                include /etc/nginx/conf.d/whitelist_ipaddress;
                deny all;
        }
}
```

## 5. whitelist_ipaddress
```nginx
allow 127.0.0.1;
allow 127.0.0.3;
```
