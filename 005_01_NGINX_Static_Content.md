To configure Nginx as a reverse proxy that serves static assets when available and forwards requests to a backend server for static content when not found on the Nginx server, follow these steps:

**Nginx Configuration Example:**

```nginx
server {
    listen 80;
    server_name your_domain.com;    

    # Location block for serving static assets directly
    location ~* \.(css|js|jpe?g|png)$ {
          # Define the root directory for serving static assets
          root /var/www/static;

         expires 7d;  # Optional: Add caching headers for performance

          # Try to serve static assets from Nginx, and if not found, forward to the backend
         try_files $uri $uri/ @backend;
    }

    # Location block for proxying requests to the backend server
    location @backend {
        # Backend server's IP address and port
        proxy_pass http://backend_server;

        # Additional proxy settings if needed
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;

        # Configure caching or other proxy settings here
    }

    # Location block for other application-specific configurations
    location / {
        # Your application-specific configuration
        # ...
    }
}
```

**README Format:**

Below is a README format to explain the configuration:

# Reverse Proxy with Static Assets Using Nginx

This guide explains how to configure Nginx as a reverse proxy for your backend server. Nginx serves static assets directly when available and forwards requests to the backend server for static content when not found on the Nginx server.

## Prerequisites

- Nginx installed on your server. You can install it using your system's package manager.

## Configuration Steps

### 1. Create a Server Block

- Create a new Nginx server block configuration file or modify an existing one. For example, create a file named `your_domain.conf` in `/etc/nginx/conf.d/`.

### 2. Configure the Server Block

- Define the server name and listening port in the server block configuration. Replace `your_domain.com` and `80` with your actual domain name and port if needed.

```nginx
server {
    listen 80;
    server_name your_domain.com;
}
```

### 3. Set Up the Root Directory for Static Assets

- Define the root directory where Nginx will serve static assets. This directory should contain your static files, such as images, stylesheets, and JavaScript files. Replace `/var/www/static` with your actual static assets directory.

```nginx
    # Define the root directory for serving static assets
    root /var/www/static;
```

### 4. Configure Static Asset Handling

- Create a location block for handling static assets. This block tells Nginx to serve static files from the Nginx server if found. If not found, it forwards the request to the backend server.
- The location ~* \.(css|js|jpe?g|png)$ block uses a regular expression with the ~* modifier to make the match case-insensitive (css, CSS, jpg, JPG, etc., will all match).
- The expires directive is optional but can be used to add caching headers for improved performance.

```nginx
    # Location block for serving static assets and proxying to the backend if not found
     location ~* \.(css|js|jpe?g|png)$ {
          # Define the root directory for serving static assets
          root /var/www/static;

         expires 7d;  # Optional: Add caching headers for performance

          # Try to serve static assets from Nginx, and if not found, forward to the backend
         try_files $uri $uri/ @backend;
    }
```

### 5. Configure Reverse Proxy for Backend Server

- Create a named location block `@backend` for proxying requests to the backend server. Replace `http://backend_server` with the actual address and port of your backend server.

```nginx
    # Named location block for proxying requests to the backend server
    location @backend {
        proxy_pass http://backend_server;

        # Additional proxy settings if needed
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;

        # Configure caching or other proxy settings here
    }
}
```

### 6. Save and Test the Configuration

- Save the Nginx configuration file and test it for syntax errors.

```shell
sudo nginx -t
```

- If there are no errors, reload Nginx to apply the changes.

```shell
sudo systemctl reload nginx
```

## Conclusion

With this configuration, Nginx will serve static assets directly from the specified directory on the Nginx server when available. If a static asset is not found on the Nginx server, the request is forwarded to the backend server for handling. This setup helps optimize the delivery of static content while allowing dynamic content to be handled by the backend server.

Make sure to adjust the paths, server names, and additional proxy settings as needed to match your specific environment and backend server configuration.
