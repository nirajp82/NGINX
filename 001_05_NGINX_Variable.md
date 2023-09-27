In NGINX, variables are used to store and manipulate values that can be used in various parts of your configuration, such as in location blocks, if statements, and rewrite rules. Variables can hold values like strings, numbers, or even more complex data structures.

### Setting Variables:

1. **Simple Variable Assignment**:

   To set a variable in NGINX, you can use the `set` directive within your configuration. Here's a basic example of setting a custom variable:

   ```nginx
   set $my_variable "Hello, NGINX!";
   ```

2. **Using Built-in Variables**:

   NGINX also comes with a set of built-in variables that capture information about the request, server, and environment. You can use these variables directly in your configuration. For example:

   ```nginx
   location / {
       add_header X-Server-Name $host;
   }
   ```

### Built-in Variables:

Here is a list of some common built-in variables in NGINX:

- `$host`: The host name from the request. Represents the hostname specified in the request made to the server.
- `$request_uri`: Contains the full original request URI, including the query string (e.g., /page?param=value).
- `$remote_addr`: The IP address of the client.
- `$remote_port`: Contains the port number of the client making the request.
- `$server_name`:  Contains the server name that was used to match the current server block.
- `$uri`: The current URI after any rewrite rules. Contains the request URI without the query string (e.g., /page).
- `$args`: Contains the query string part of the URL, excluding the question mark (e.g., param=value).
- `$query_string`: Contains the full query string, including the question mark (e.g., ?param=value).
- `$http_user_agent`: The user agent string sent by the client's browser.
- `$http_referer`: The referring URL. if provided by the client.
- `$request_method`: The HTTP request method (e.g., GET, POST).
- `$http_cookie`: The value of the Cookie header sent by the client.
- `$server_addr`: Contains the IP address of the server handling the request.
- `$server_port`: Contains the port number on which the server is listening.
- `$scheme`: Represents the scheme (HTTP or HTTPS) of the incoming request.
- `$http_user_agent`: Stores the User-Agent string sent by the client's browser or user agent.
- `$document_root`: Represents the root directory for the current server block.
- `$request_filename`: Contains the path to the requested file on the server's filesystem.
- `$is_args`: Will be empty unless a query string is present, in which case it will contain ?.

### Using Variables in NGINX Config:

Once you've set or used a variable, you can use it in various parts of your configuration. Here's an example of how to use a custom variable and a built-in variable within a location block:

```nginx
location /var {
    set $my_variable "Hello, NGINX!";
    add_header Custom-Var $my_variable;
    add_header Request-URI $request_uri;
}
```

In this example, when a user makes a request to the URL path `/var`, NGINX sets the custom variable `$my_variable` to "Hello, NGINX!" and adds two headers to the response: "Custom-Var" with the value of `$my_variable` and "Request-URI" with the value of the built-in variable `$request_uri`.

So, if a user makes a request to `/var`, the response headers would include these two headers, like so:

```
Custom-Var: Hello, NGINX!
Request-URI: /var
```

You can use variables to customize the behavior of your NGINX server, make decisions based on request data, and generate dynamic responses.

```nginx
location /var {
    set $my_variable "Hello, NGINX!";
    rewrite ^ /new-path last; # Rewrite the URL
    add_header Custom-Var $my_variable; # Add a header with the custom variable
    return 200 "Response Body: $my_variable"; # Return a response with the custom variable in the body
}

location /new-path {
    return 200 "This is the new path."; # Return a response for the rewritten URL
}
```
In this example, when a user makes a request to /var, NGINX does the following:

Sets a custom variable $my_variable to "Hello, NGINX!".
Uses the rewrite directive to rewrite the URL to /new-path.
Adds a response header "Custom-Var" with the value of $my_variable.
Uses the return directive to send a response with a 200 status code and includes the custom variable in the response body.
So, if a user makes a request to /var, the response would look like this:

```yaml
HTTP/1.1 200 OK
Server: nginx
Date: [Date]
Content-Type: text/plain
Content-Length: [Length]
Response Body: Hello, NGINX!
```
Please replace [Date] and [Length] with actual values in your response header.
