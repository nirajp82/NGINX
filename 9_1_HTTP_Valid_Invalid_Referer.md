In Nginx, the `valid_referers` and `invalid_referer` directives are used to control access to your web server based on the referring URLs (HTTP referers) in incoming requests. These directives allow you to specify which referring URLs are allowed (valid_referers) or denied (invalid_referer) access to your server. 

Here's an explanation of each directive with an example:

### 1. `valid_referers` Directive

The `valid_referers` directive is used to specify a list of valid referring URLs that are allowed to access your server. Requests from referring URLs not in this list will be denied access. 

**Syntax:**

```nginx
valid_referers none | blocked | server_names | string ...;
```

- `none`: Disables the validation of referring URLs, allowing all requests.

- `blocked`: Blocks all requests from referring URLs.

- `server_names`: Allows requests from the same server names as your server's configuration.

- `string ...`: Specifies a list of referring URLs as strings.

**Example:**

```nginx
location /private {
    valid_referers server_names example.com www.example.com;
    if ($invalid_referer) {
        return 403;
    }
    # Your other location directives...
}
```

In this example, requests to the `/private` location are only allowed if the referring URL matches `server_names`, which includes `example.com` and `www.example.com`. If the referring URL doesn't match these server names, the server returns a 403 Forbidden error.

### 2. `invalid_referer` Directive

The `invalid_referer` directive is used to specify a list of referring URLs that are not allowed to access your server. Requests from referring URLs in this list will be denied access.

**Syntax:**

```nginx
invalid_referer string ...;
```

- `string ...`: Specifies a list of referring URLs as strings.

**Example:**

```nginx
location /downloads {
    invalid_referer blocked example.org www.example.org;
    if ($invalid_referer) {
        return 403;
    }
    # Your other location directives...
}
```

In this example, requests to the `/downloads` location are denied if the referring URL matches `example.org` or `www.example.org`. If the referring URL matches any of these invalid referers, the server returns a 403 Forbidden error.

These directives are often used for hotlink protection, which prevents other websites from directly linking to your resources like images or files, thereby reducing your server's bandwidth usage. However, they should be used with caution as they may also affect legitimate users or services that use legitimate hotlinks.


### 3. Real world example

```nginx
valid_referers none blocked example.com *.example.com;

if ($invalid_referer) {
    return 403;
}
```

Let's break it down:

1. **`valid_referers`**: This directive is set with several values:

   - `none`: This means that there are no valid referrers allowed. Essentially, this disallows access to any resources if they were directly accessed (i.e., not referred from any other website). This is often used for hotlink protection, where you want to block access to your resources unless they are accessed from specific domains.

   - `blocked`: This specifies that requests from all referring URLs are blocked by default.

   - `example.com`: Requests referred from `example.com` are considered valid.

   - `*.example.com`: Requests referred from any subdomain of `example.com` are also considered valid.

   This configuration allows access only if the request is referred from `example.com` or any subdomain of `example.com`. Requests from other domains or direct accesses will be denied.

2. **`if ($invalid_referer)`**: This `if` block checks the variable `$invalid_referer`, which is set by Nginx based on the `valid_referers` directive. If the referrer is not in the list of valid referrers (as specified earlier), `$invalid_referer` will be set to `blocked`, and the condition is met.

3. **`return 403;`**: If the condition is met (i.e., if the referrer is not in the list of valid referrers), Nginx will return a 403 Forbidden HTTP status code, effectively denying access to the requested resource.

This configuration snippet is commonly used to prevent hotlinking, where you want to restrict access to your resources (e.g., images) only to specific domains or subdomains. If someone tries to embed your resources on their website without permission, the requests will be blocked, returning a 403 error.
