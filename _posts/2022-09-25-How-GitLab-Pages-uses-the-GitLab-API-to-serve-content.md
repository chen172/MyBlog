---

layout: post
title: "How GitLab Pages Uses the GitLab API to Serve Content"

---

### How GitLab Pages Generates Routes

Prior to GitLab **12.10**, the Pages daemon relied on a configuration file named `config.json` within the project directory to generate routes for serving pages. This file would typically be located at:

```
/path/to/shared/pages/myproject/myproject.gitlab.io/config.json
```

However, starting from **GitLab 12.10**, the Pages daemon no longer depends on this `config.json` file. Instead, it sources its configuration from the **GitLab API** via an internal API endpoint:

```
/api/v4/internal/pages?host=myproject.gitlab.io
```

The response from this API call closely mirrors the structure of the old `config.json` file but is now dynamically fetched. Here’s an example of the API response:

```json
{
    "certificate": "--cert-contents--",
    "key": "--key-contents--",
    "lookup_paths": [
        {
            "access_control": true,
            "https_only": true,
            "prefix": "/",
            "project_id": 123,
            "source": {
                "path": "myproject/myproject.gitlab.io/public/",
                "type": "file"
            }
        }
    ]
}
```

This configuration contains:

* **certificate**: The SSL certificate contents for secure HTTPS communication.
* **key**: The SSL private key.
* **lookup\_paths**: An array of paths that define how the Pages daemon should look up the content and serve it.

Each entry under `lookup_paths` defines:

* **access\_control**: A flag indicating if access control is enabled.
* **https\_only**: A flag indicating whether the content should only be served over HTTPS.
* **prefix**: The base path for serving content.
* **project\_id**: The ID of the GitLab project that hosts the Pages.
* **source**: Defines the source of the content, including the path to the project’s public files.

### How GitLab Pages Serves Content

Once the configuration is fetched and the routes are generated, the Pages daemon starts serving the content. Here’s how the process works:

1. **The `gitlab-pages` Command Option**:
   The `gitlab-pages` command is started with the option `pages-root`, which specifies the directory where the Pages content is stored. For example:

   ```bash
   gitlab-pages -pages-root="shared/pages"
   ```

2. **Domain Source Configuration**:
   The API response contains the `source` configuration, which specifies the root path for serving content. In our example, this path is:

   ```json
   "path": "myproject/myproject.gitlab.io/public/"
   ```

3. **Resolving the Path**:
   The Pages daemon uses the `reader.resolvePath` function to parse the requested `URL.subpath` from the incoming request. This function determines the exact location of the requested content.

   You can check out the implementation of `reader.resolvePath` in the [GitLab Pages source code](https://gitlab.com/gitlab-org/gitlab-pages/-/blob/master/internal/serving/disk/reader.go#L160).

4. **Serving the Content**:
   Finally, the resolved file path is constructed, and the Pages daemon serves the requested content. For example, the final path might be:

   ```
   shared/pages/myproject/myproject.gitlab.io/public/URL.subpath
   ```

   The content at this path is then served to the user.

---

### References

1. [GitLab Pages Repository on GitLab](https://gitlab.com/gitlab-org/gitlab-pages/-/tree/master/)
2. [How GitLab Pages Uses the GitLab API to Serve Content (GitLab Blog)](https://about.gitlab.com/blog/2020/08/03/how-gitlab-pages-uses-the-gitlab-api-to-serve-content/)
