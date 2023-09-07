## Nginx `access_log` Directive

The `access_log` directive in Nginx is used to configure access logs for HTTP server requests. Access logs record information about client requests, including details like IP addresses, requested URLs, HTTP status codes, and more.

### Table of Contents

1. **Basic Usage**
2. **Custom Log Formats**
3. **Specifying Different Log File Names**
4. **Built in variables**
5. **Important Considerations**
6. **Log Rotation**

---

## 1. Basic Usage

The basic syntax of the `access_log` directive is as follows:

```nginx
access_log path [format];
```

- `path`: Specifies the path to the access log file.
- `format`: (Optional) Specifies the log format. If omitted, the default format is used.

Example:

```nginx
http {
    server {
        listen 80;
        server_name example.com;
        access_log /var/log/nginx/access.log;
        ...
    }
}
```

In this example, the `access_log` directive logs access information to the file `/var/log/nginx/access.log`.

## 2. Custom Log Formats

Nginx allows you to define custom log formats using the `log_format` directive. Custom log formats can include specific variables to log various details about requests.

Example:

```nginx
http {
    log_format my_custom_format '$remote_addr - $remote_user [$time_local] '
                               '"$request" $status $body_bytes_sent '
                               '"$http_referer" "$http_user_agent"';
    
    server {
        listen 80;
        server_name example.com;
        access_log /var/log/nginx/custom_access.log my_custom_format;
        ...
    }
}
```

Here, we've defined a custom log format named `my_custom_format` that logs the client's IP address, user, request time, request details, HTTP status, response body size, referer, and user agent.

## 3. Specifying Different Log File Names

You can specify different log file names for different parts of your configuration using the `access_log` directive. This can help you organize logs for multiple virtual hosts or applications.

Example:

```nginx
http {
    server {
        listen 80;
        server_name example.com;
        access_log /var/log/nginx/example_access.log;
        ...
    }
    
    server {
        listen 80;
        server_name another-example.com;
        access_log /var/log/nginx/another_example_access.log;
        ...
    }
}
```

In this example, each server block specifies a different log file for its access logs.

## 5. Built in variables
Certainly! Nginx provides a wide range of built-in variables that you can use in your log formats to include various request and server-related information in your access and error logs. Here is a list of some commonly used built-in variables in Nginx:

1. **$remote_addr**: The client's IP address.

2. **$remote_user**: The authenticated user, if applicable.

3. **$time_local**: The local time in the Common Log Format (CLF), which includes the date and time of the request.

4. **$request**: The first line of the request, including the HTTP method, URL, and HTTP version.

5. **$status**: The HTTP status code returned to the client.

6. **$body_bytes_sent**: The number of bytes sent to the client as part of the response body.

7. **$http_referer**: The URL of the referring page, if available.

8. **$http_user_agent**: The user agent string of the client's browser or user agent.

9. **$host**: The value of the Host header in the client request.

10. **$request_time**: The time it took to process the request in seconds with millisecond resolution.

11. **$upstream_addr**: The IP address and port of the upstream server that processed the request in the case of proxying.

12. **$upstream_response_time**: The time it took for the upstream server to respond in seconds with millisecond resolution.

13. **$ssl_protocol**: The SSL/TLS protocol version used in the request.

14. **$ssl_cipher**: The SSL/TLS cipher suite used in the request.

15. **$server_name**: The server name that matched the request.

16. **$request_length**: The length of the request in bytes, including the request line, headers, and request body.

17. **$bytes_sent**: The total number of bytes sent to the client in response to the request.

18. **$connection**: The connection serial number.

19. **$connection_requests**: The number of requests made in the current connection.

20. **$scheme**: The request scheme (http or https).

These variables can be used in the `log_format` directive to create custom log formats or directly in the `access_log` and `error_log` directives to specify what information should be included in your log files.

For example, you can define a custom log format like this:

```nginx
log_format my_custom_format '$remote_addr - $remote_user [$time_local] '
                           '"$request" $status $body_bytes_sent '
                           '"$http_referer" "$http_user_agent"';
```

And then use it in your `access_log` directive:

```nginx
access_log /var/log/nginx/access.log my_custom_format;
```

## 5. Important Considerations

- **Security**: Ensure that log files are stored securely, and access to log directories is restricted to authorized users.

- **Log Rotation**: Implement log rotation mechanisms to manage log file sizes and prevent them from filling up your disk.

- **Privacy**: Be mindful of logging sensitive user data. Ensure compliance with privacy regulations.

- **Performance Impact**: Logging can impact server performance, especially with high traffic. Use logging judiciously and consider asynchronous logging options.

## 6. Log Rotation

Log rotation is crucial for managing log files efficiently. You can use tools like `logrotate` to automatically rotate and archive log files, preventing them from consuming excessive disk space.

Example `/etc/logrotate.d/nginx` configuration:

```ini
/var/log/nginx/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        [ ! -f /var/run/nginx.pid ] || kill -USR1 `cat /var/run/nginx.pid`
    endscript
}
```

This `logrotate` configuration specifies daily rotation, compression, and retention of 14 rotated log files.

---

The `access_log` directive in Nginx is a powerful tool for monitoring and troubleshooting web server requests. Custom log formats, different log file names, and proper log rotation are essential aspects of maintaining an effective logging system.
