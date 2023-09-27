In Nginx, the `map` directive is used to create key-value mappings, allowing you to map one value to another. It's a versatile directive that can be used for a variety of purposes, such as URL rewriting, dynamic configurations, and request routing. Here's how it works:

The basic syntax of the `map` directive is as follows:

```nginx
map $variable $new_variable {
    key value;
    ...
    default default_value;
}
```

- `$variable`: The variable whose value you want to map.

- `$new_variable`: The variable where the mapped value will be stored.

- `key`: The original value you want to match.

- `value`: The corresponding value to assign to `$new_variable` when `$variable` matches `key`.

- `default_value`: The default value to assign to `$new_variable` if no match is found.

### Example 1
Here's an example that demonstrates how to use the `map` directive:

```nginx
http {
    map $request_method $request_method_lowercase {
        GET     "get";
        POST    "post";
        PUT     "put";
        DELETE  "delete";
        default "unknown";
    }

    server {
        listen 80;
        server_name example.com;

        location / {
            return 200 "Request Method: $request_method_lowercase\n";
        }
    }
}
```

In this example:

- We create a mapping between different HTTP request methods (`$request_method`) and their lowercase equivalents using the `map` directive. If the `$request_method` matches a key, it is replaced with the corresponding value in `$request_method_lowercase`. If there's no match, it is replaced with "unknown."

- Inside the `server` block, we listen on port 80 for requests with the `Host` header set to `example.com`.

- In the `location /` block, we use the mapped variable `$request_method_lowercase` to display the lowercase request method in the response.

When you make an HTTP request to this Nginx server, the response will include the lowercase request method:

```
Request Method: get
```

### Example 2
In Nginx, the `map` directive can also be used with regular expressions to extract values from a request URL and map them to other variables. This can be useful for dynamic routing or rewriting of URLs based on specific patterns within the request URL. Here's how it works with an example:

Let's say you want to extract a specific value from a URL and use it as a variable. For example, suppose you have URLs like `/product/12345` where `12345` is a product ID, and you want to extract this product ID and use it in your configuration.

Here's how you can achieve this using the `map` directive and regular expressions:

```nginx
http {
    map $request_uri $product_id {
        ~^/product/(\d+)$  $1;
        default            "";
    }

    server {
        listen 80;
        server_name example.com;

        location / {
            if ($product_id != "") {
                return 200 "Product ID: $product_id\n";
            }
            return 404 "Product ID not found\n";
        }
    }
}
```

In this example:

1. We define an HTTP block and create a mapping using the `map` directive. The key is `$request_uri`, which represents the full request URI, and the `$product_id` variable will store the extracted product ID.

2. Within the `map` block, we use a regular expression pattern (`~^/product/(\d+)$`) to match URLs that start with `/product/` followed by one or more digits (`\d+`). The `(\d+)` part captures the numeric portion of the URL and assigns it to the variable `$1`. The `$1` variable represents the first capturing group in the regular expression.

3. We provide a `default` value of `""` to `$product_id`, which means that if the regular expression pattern doesn't match (i.e., the URL doesn't have the expected format), `$product_id` will be an empty string.

4. In the `server` block, we listen on port 80 for requests with the `Host` header set to `example.com`.

5. Inside the `location /` block, we check if `$product_id` is not an empty string. If it's not empty, we return a response that includes the extracted product ID. Otherwise, we return a 404 response.

Now, when you make a request to URLs like `/product/12345`, Nginx will extract the product ID using the regular expression and display it in the response:

```
Product ID: 12345
```

If the URL doesn't match the expected format, you will receive a 404 response:

```
Product ID not found
```

This demonstrates how the `map` directive in combination with regular expressions can be used to extract specific values from request URLs and use them in your Nginx configuration.

### Example 3
Check if variable is null or empty string. 

The `map` block in Nginx is primarily used for creating key-value mappings based on variables, and it's not well-suited for directly checking if a variable is null or empty. However, you can achieve this by combining the `map` block with other directives like `if`. Here's an example of how to use the `map` block to check if a variable is null or empty:

```nginx
http {
    map $lua_variable $is_empty {
        "~^$"      1;  # Set 1 if $lua_variable is empty or null
        default    0;  # Set 0 otherwise
    }

    server {
        listen 80;
        server_name example.com;

        location / {
             access_by_lua_block {
                # Lua script to set $lua_variable based on your logic
                if some_condition then
                    ngx.var.lua_variable = "not empty"
                else
                    ngx.var.lua_variable = ""
                end
            }

            if ($is_empty = 1) {
                return 404 "The Lua variable is empty\n";
            }

            return 200 "The Lua variable is not empty: $lua_variable\n";
        }
    }
}
```

In this configuration:

- We create an http block with a map block. The $lua_variable (which will be set by your Lua script) is mapped to the $is_empty variable. If $lua_variable is empty (null or an empty string), $is_empty will be set to 1; otherwise, it will be set to 0.

- Inside the `location /` block, the Lua script sets the $lua_variable based on your logic. In this example, it sets $lua_variable to "not empty" if some_condition is true; otherwise, it sets it to an empty string.

- We use an `if` directive to check the value of `$is_empty`. If it's equal to 1, we return a 404 response indicating that the Lua variable is empty or null. Otherwise, we proceed with further processing, and you can access the value of `$lua_variable` as needed.

This configuration allows you to use the `map` block to check if an Nginx variable is null or empty and take appropriate actions based on that condition within your Nginx server block.



You can use the `map` directive for more complex mappings and configurations, such as mapping URL paths to different backends, creating custom headers, or dynamically configuring Nginx based on variables like IP addresses or user agents. It's a powerful tool for customizing the behavior of your Nginx server based on various conditions.
