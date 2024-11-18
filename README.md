# Nginx

This README provides an overview of the commands and concepts covered in the Nginx Crash Course.

## Commands

### Starting Nginx
To start Nginx, use the following command:
```
nginx
```

### Nginx Options
To view Nginx options:
```
nginx -h
```

### Reloading Nginx Configuration
To reload Nginx configuration without stopping the server:
```
nginx -s reload
```

### Stopping Nginx
To stop the Nginx server:
```
nginx -s stop
```

### Monitoring Access Logs
To monitor Nginx access logs in real-time:
```
tail -f /usr/local/var/log/nginx/access.log
```

## SSL Certificate Generation
To generate a self-signed SSL certificate:

1. Create a directory for certificates:
   ```
   mkdir ~/nginx-certs
   cd ~/nginx-certs
   ```

2. Generate the certificate:
   ```
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx-selfsigned.key -out nginx-selfsigned.crt
   ```

##  Notes

- The course covers basic Nginx configuration and usage.
- Self-signed certificates are useful for development but not recommended for production environments.
- Always refer to the official Nginx documentation for the most up-to-date information and best practices.

# Nginx Configuration Explanation

## Main Context

```nginx
worker_processes 1;

events {
    worker_connections 1024;
}
```

- `worker_processes 1;`: Defines the number of worker processes Nginx will use. In this case, it's set to 1.
- `events { ... }`: Configures connection processing.
  - `worker_connections 1024;`: Sets the maximum number of simultaneous connections that can be opened by a worker process.

## HTTP Context

```nginx
http {
    include mime.types;
    # ... (rest of the configuration)
}
```

The `http` context contains all HTTP-related directives.

- `include mime.types;`: Includes the MIME types file, which maps file extensions to content types.

### Upstream Block

```nginx
upstream nodejs_cluster {
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}
```

This defines a group of servers (Node.js instances) that Nginx will proxy requests to. It's using three local servers on different ports.

### HTTPS Server Block

```nginx
server {
    listen 443 ssl;
    server_name localhost;

    ssl_certificate /Users/harsh/nginx-selfsigned.crt;
    ssl_certificate_key /Users/harsh/nginx-selfsigned.key;

    location / {
        proxy_pass http://nodejs_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

- `listen 443 ssl;`: Listens on port 443 for HTTPS connections.
- `server_name localhost;`: Defines the server name.
- SSL certificate settings: Specifies the paths to the SSL certificate and key.
- `location / { ... }`: Configures how Nginx should handle requests.
  - `proxy_pass http://nodejs_cluster;`: Proxies requests to the upstream Node.js cluster.
  - `proxy_set_header` directives: Sets headers for the proxied request.

### HTTP to HTTPS Redirection

```nginx
server {
    listen 8080;
    server_name localhost;

    location / {
        return 301 https://$host$request_uri;
    }
}
```

This server block listens on port 8080 for HTTP connections and redirects all traffic to HTTPS.

- `listen 8080;`: Listens on port 8080 for HTTP connections.
- `return 301 https://$host$request_uri;`: Performs a permanent redirect (301) to the HTTPS version of the requested URL.

## Summary

This Nginx configuration sets up a reverse proxy that:
1. Distributes incoming HTTPS requests across a cluster of Node.js servers.
2. Uses SSL/TLS for secure connections.
3. Redirects HTTP traffic to HTTPS for enhanced security.

The configuration is suitable for a development environment (using a self-signed certificate) and demonstrates load balancing across multiple Node.js instances.
