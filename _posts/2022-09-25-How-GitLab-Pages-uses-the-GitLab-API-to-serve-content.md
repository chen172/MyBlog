---
layout: post
title: "How GitLab Pages uses the GitLab API to serve content"
---

### How it generates routes
* Before [Gitlab 12.10](https://about.gitlab.com/releases/2020/04/22/gitlab-12-10-released/) the Pages daemon would rely on a file named ```config.json``` located in your 
project's directory, that is ```/path/to/shared/pages/myproject/myproject.gitlab.io/config.json.``` 

* Now it use GitLab API-based configuration. the Pages daemon sources the domain configuration via an internal API endpoint ```/api/v4/internal/pages?host=myproject.gitlab.io```.
The response from the API is very similar to the contents of the ```config.json``` file:
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
### How it serves content
1. ```gitlab-pages``` command option ```pages-root``` specific Directory where the pages are stored, for example: ```-pages-root="shared/pages"```
2. Gitlab domain source configuration specific the ```root path```, for example: ```"path": "myproject/myproject.gitlab.io/public/"```
3. Use [reader.resolvePath](https://gitlab.com/gitlab-org/gitlab-pages/-/blob/master/internal/serving/disk/reader.go#L160) to parse ```URL.subpath```
4. Finally we get the file path ```shared/pages/myproject/myproject.gitlab.io/public/URL.subpath``` to serve

#### reference
1. https://gitlab.com/gitlab-org/gitlab-pages/-/tree/master/
2. https://about.gitlab.com/blog/2020/08/03/how-gitlab-pages-uses-the-gitlab-api-to-serve-content/
