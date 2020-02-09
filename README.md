sa_webhook_helper
=================

[![Build Status](https://travis-ci.com/softasap/sa_webhook_helper.svg?branch=master)](https://travis-ci.com/softasap/sa_webhook_helper)


Role to install webhook helper on unix based box

Version is controlled by  webhook_version parameter.


Example:

Simple

```YAML


     - {
         role: "sa_webhook_helper",
       }

```

```YAML


     - {
         role: "sa_webhook_helper",
         webhook_version: 2.6.11,
         webhook_port: 8999,
         webhook_listen: 0.0.0.0
       }

```





Webhook binary parameters

```
  -cert string	path to the HTTPS certificate pem file (default "cert.pem")
  -cipher-suites string   comma-separated list of supported TLS cipher suites
  -header value    response header to return, specified in format name=value, use multiple times to set multiple headers
  -hooks value    path to the json file containing defined hooks the webhook should serve, use multiple times to load from different files
  -hotreload    watch hooks file for changes and reload them automatically
  -ip string    ip the webhook should serve hooks on (default "0.0.0.0")
  -key string    path to the HTTPS certificate private key pem file (default "key.pem")
  -list-cipher-suites   list available TLS cipher suites
  -nopanic    do not panic if hooks cannot be loaded when webhook is not running in verbose mode
  -port int    port the webhook should serve hooks on (default 9000)
  -secure     use HTTPS instead of HTTP
  -template    parse hooks file as a Go template
  -tls-min-version string    minimum TLS version (1.0, 1.1, 1.2, 1.3) (default "1.2")
  -urlprefix string    url prefix to use for served hooks (protocol://yourserver:port/PREFIX/:hook-id) (default "hooks")
  -verbose    show verbose output
  -version    display webhook version and quit
```


Consuming webhooks
------------------

### Limit by ID

```json

  {
    "id": "dateip",
    "execute-command": "date",
    "command-working-directory": "/tmp",
    "include-command-output-in-response": true,
    "include-command-output-in-response-on-error": true,
    "response-message": "Executing dateip script",
    "pass-arguments-to-command":
    [
    ],
    "trigger-rule":
    {
      "match":
      {
        "type": "ip-whitelist",
        "ip-range": "192.168.88.1/24"
      }
    }
  },
```


### Limit by header

```json

  {
    "id": "dateheader",
    "execute-command": "date",
    "command-working-directory": "/tmp",
    "include-command-output-in-response": true,
    "include-command-output-in-response-on-error": true,
    "response-message": "Executing dateip script",
    "pass-arguments-to-command":
    [
    ],
    "trigger-rule":
    {
      "match":
      {
        "type": "value",
        "value": "SOMETOKEN",
        "parameter":
        {
          "source": "header",
          "name": "X-Cypress-Token"
        }
      }
    }
  }

```

```sh

curl -i -H "X-Cypress-Token: SOMETOKEN" http://192.168.88.11:9000/hooks/dateheader

HTTP/1.1 200 OK
Date: Mon, 27 Jan 2020 11:49:48 GMT
Content-Length: 29
Content-Type: text/plain; charset=utf-8

Mon Jan 27 11:49:48 UTC 2020
```

### Dummy hook with limit by get parameter

```json

  {
    "id": "dateget",
    "execute-command": "date",
    "response-message": "Executing simple webhook...",
    "include-command-output-in-response": true,
    "include-command-output-in-response-on-error": true,
    "trigger-rule":
    {
      "match":
      {
        "type": "value",
        "value": "42",
        "parameter":
        {
          "source": "url",
          "name": "token"
        }
      }
    }
  }

```

```sh
curl -i http://192.168.88.11:9000/hooks/dateget?token=42

HTTP/1.1 200 OK
Date: Mon, 27 Jan 2020 12:12:15 GMT
Content-Length: 29
Content-Type: text/plain; charset=utf-8

Mon Jan 27 12:12:15 UTC 2020

```

More notes on webhook configurations

https://github.com/adnanh/webhook/tree/master/docs


You can either expose services directly on configured port,
or wrap it to some virtual folder of your webserver, for example, nginx:

```conf

location /cypress/ {

    proxy_set_header        Host $host;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header        X-Forwarded-Proto $scheme;

    proxy_pass          http://localhost:8999;
    proxy_read_timeout  900;

    rewrite  ^/cypress/(.*)  /$1 break;
}
```


Usage with ansible galaxy workflow
----------------------------------

If you installed the sa_webhook_helper role using the command

`
   ansible-galaxy install softasap.sa_webhook_helper
`

the role will be available in the folder library/softasap.sa_webhook_helper
Please adjust the path accordingly.

```YAML

     - {
         role: "softasap.sa_webhook_helper"
       }

```



Copyright and license
---------------------

Code is dual licensed under the [BSD 3 clause] (https://opensource.org/licenses/BSD-3-Clause) and the [MIT License] (http://opensource.org/licenses/MIT). Choose the one that suits you best.

Reach us:

Subscribe for roles updates at [FB] (https://www.facebook.com/SoftAsap/)

Join gitter discussion channel at [Gitter](https://gitter.im/softasap)

Discover other roles at  http://www.softasap.com/roles/registry_generated.html

visit our blog at http://www.softasap.com/blog/archive.html

