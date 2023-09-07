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
