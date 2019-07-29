# Configuration
Each of RoadRunner service requires proper configuration. By default, such configuration is merged into one file which must be located in a root of your project. Each service configuration is located under the designated section. The config file must be named as `.rr.{format}` where format is `yml`, `json` and others supported by `sp13f/viper`.

Sample configuration:

```yaml
# defines environment variables for all underlying php processes
env:
  key: value

# rpc bus allows php application and external clients to talk to rr services.
rpc:
  # enable rpc server
  enable: true

  # rpc connection DSN. Supported TCP and Unix sockets.
  listen:     tcp://127.0.0.1:6001

metrics:
  # prometheus client address (path /metrics added automatically)
  address: localhost:2112
  collect:
    app_metric:
      type:    histogram
      help:    "Custom application metric"
      labels:  ["type"]
      buckets: [0.1, 0.2, 0.3, 1.0]

# http service configuration.
http:
  # http host to listen.
  address: 0.0.0.0:8080

  ssl:
    # custom https port (default 443)
    port:     443

    # force redirect to https connection
    redirect: true

    # ssl cert
    cert:     server.crt

    # ssl private key
    key:      server.key

  # HTTP service provides FastCGI as frontend
  fcgi:
    # FastCGI connection DSN. Supported TCP and Unix sockets.
    address: tcp://0.0.0.0:6920

  # HTTP service provides HTTP2 transport
  http2:
    # enable HTTP/2, only with TSL
    enabled: true
    
    # to enable H2C on TCP connections, false by default
    h2c: true
    
    # max transfer channels
    maxConcurrentStreams: 128

  # max POST request size, including file uploads in MB.
  maxRequestSize: 200

  # file upload configuration.
  uploads:
    # list of file extensions which are forbidden for uploading.
    forbid: [".php", ".exe", ".bat"]

  # cidr blocks which can set ip using X-Real-Ip or X-Forwarded-For
  trustedSubnets: ["10.0.0.0/8", "127.0.0.0/8", "172.16.0.0/12", "192.168.0.0/16", "::1/128", "fc00::/7", "fe80::/10"]

  # http worker pool configuration.
  workers:
    # php worker command.
    command:  "php psr-worker.php pipes"

    # connection method (pipes, tcp://:9000, unix://socket.unix). default "pipes"
    relay:    "pipes"

    # worker pool configuration.
    pool:
      # number of workers to be serving.
      numWorkers: 4

      # maximum jobs per worker, 0 - unlimited.
      maxJobs:  0

      # for how long worker is allowed to be bootstrapped.
      allocateTimeout: 60

      # amount of time given to worker to gracefully destruct itself.
      destroyTimeout:  60

# Additional HTTP headers and CORS control.
headers:
  # Middleware to handle CORS requests, https://www.w3.org/TR/cors/
  cors:
    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin
    allowedOrigin: "*"

    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Headers
    allowedHeaders: "*"

    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Methods
    allowedMethods: "GET,POST,PUT,DELETE"

    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials
    allowCredentials: true

    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Expose-Headers
    exposedHeaders: "Cache-Control,Content-Language,Content-Type,Expires,Last-Modified,Pragma"

    # Max allowed age in seconds
    maxAge: 600

  # Automatically add headers to every request passed to PHP.
  request:
    "Example-Request-Header": "Value"

  # Automatically add headers to every response.
  response:
    "X-Powered-By": "RoadRunner"

# monitors rr server(s)
limit:
  # check worker state each second
  interval: 1

  # custom watch configuration for each service
  services:
    # monitor http workers
    http:
      # maximum allowed memory consumption per worker (soft)
      maxMemory: 100

      # maximum time to live for the worker (soft)
      TTL: 0

      # maximum allowed amount of time worker can spend in idle before being removed (for weak db connections, soft)
      idleTTL: 0

      # max_execution_time (brutal)
      execTTL: 60

# static file serving. remove this section to disable static file serving.
static:
  # root directory for static file (http would not serve .php and .htaccess files).
  dir:   "public"

  # list of extensions for forbid for serving.
  forbid: [".php", ".htaccess"]
```

Minimal configuration:

```yaml
http:
  address:         ":8080"
  workers.command: "php psr-worker.php"
```

## Console flags
You can overwrite any of the config values using `-o` flag:

```
rr serve -v -d -o http.address=:80 -o http.workers.pool.numWorkers=1
```

## Inluding config files
You can merge multiple config files into one using `include` directive:

```yaml
include:
  - rr/http.yaml
  - rr/static.yaml
```

## Environment Variables
RoadRunner will replace some config options using reference(s) to environment variable(s):

```yaml
http:
  workers.pool.numWorkers: ${NUM_WORKERS}
```
