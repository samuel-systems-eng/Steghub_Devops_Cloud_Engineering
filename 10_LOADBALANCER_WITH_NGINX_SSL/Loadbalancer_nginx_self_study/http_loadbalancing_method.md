# HTTP Load Balancing Methods and Features Supported by Nginx

## Introduction

Nginx is a high-performance HTTP server and reverse proxy renowned for its ability to handle a large number of concurrent connections efficiently. One of its core features is HTTP load balancing, which distributes incoming traffic across multiple backend servers to ensure high availability, scalability, and reliability of web applications.  
This documentation provides an overview of the HTTP load balancing methods and advanced features supported by Nginx.

## HTTP Load Balancing Methods in Nginx

### 1. Round Robin

Round Robin is the default load balancing method in Nginx. It distributes client requests evenly across the available backend servers in a cyclic order.  

**Additional Detail:** You can add a weight parameter to favor stronger servers. If one server has `weight=3` and others have `weight=1`, Nginx sends three times as many requests to the stronger server.

Configuration Example:

    nginx
    
    upstream backend {
    server backend1.example.com weight=3; # Receives 3x more traffic
    server backend2.example.com;          # Defaults to weight=1
    server backend3.example.com;          # Defaults to weight=1
    }

    server {
        listen 80;
        server_name example.com;

        location / {
            proxy_pass http://backend;
        }
    }

### 2. Least Connections

Least Connections method directs traffic to the server with the fewest active connections, helping to balance the load more evenly during uneven traffic spikes.

**Additional Detail:** It also respects server weights. This makes it ideal for environments where request processing times vary significantly.

Configuration Example:

    nginx

        upstream backend {
            least_conn;
            server backend1.example.com;
            server backend2.example.com;
            server backend3.example.com;
    }

### 3. IP HashIP

Hash method ensures that requests from the same client IP address are always sent to the same backend server, providing session persistence based on client IP.

**Additional Detail:** This algorithm uses the first three octets of an IPv4 address or the entire IPv6 address. If a server needs to be removed temporarily, mark it as down to preserve the current hashing distribution.

Configuration Example:

    nginxupstream backend {
        ip_hash;
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com down; # Safely pulled out of rotation
    }

### 4. Generic Hash

Generic Hash method allows custom key-based distribution, where a specified key (e.g., a URL parameter) determines which backend server will handle the request.

**Additional Detail:** The optional consistent parameter applies Ketama consistent hashing. This ensures that if a backend server fails, only a minimal number of keys are remapped, protecting your backend cache layers from getting overloaded.

Configuration Example:

    nginx

    upstream backend {
        hash $request_uri consistent; # Consistent hashing based on  URL
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }

### 5. Random with Two Choices

This method randomly selects two servers and then chooses the one with the fewer connections. It provides a good balance between random distribution and connection load.

**Additional Detail:** This cuts down on the overhead of centralized tracking in massive server clusters. It is highly useful in large distributed infrastructures.

Configuration Example:

nginx

    upstream backend {
        random two least_conn;
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }

### 6. Least Time

Least Time method selects the server with the lowest average response time and the least number of active connections.

**Additional Detail:** This feature requires NGINX Plus. You can measure response speed by header (time to receive first byte) or last_byte (time to receive full response).

Configuration Example:

    nginx

    upstream backend {
        zone backend_cluster 64k; # Shared memory zone required for NGINX Plus
        least_time header;        # Tracks time to receive response headers
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }

## Advanced Load Balancing Features in Nginx

### 1. Health Checks

Nginx can perform active health checks to determine the status of backend servers, ensuring that traffic is only sent to healthy servers.

**Additional Detail:** Open-source Nginx provides passive health checks via `max_fails` and `fail_timeout`. Active health checks (using the health_check directive) are exclusive to `NGINX Plus` and actively ping backends with probe requests.

Configuration Example (NGINX Plus Active Health Check):

    nginx

    upstream backend {
        zone backend_zone 64k;
        server backend1.example.com;
        server backend2.example.com;
    }

    server {
        listen 80;
        location / {
            proxy_pass http://backend;
            health_check interval=5s fails=3 passes=2 uri=/healthz;
        }
    }

### 2. Session Persistence (Sticky Sessions)

Nginx supports session persistence, allowing it to route requests from the same client to the same backend server.

**Additional Detail:** True cookie-based sticky sessions using the sticky directive require `NGINX Plus`. For open-source versions, you must rely on the ip_hash or hash methods for tracking.

Configuration Example (NGINX Plus Only):

    nginx

    upstream backend {
        zone backend_sticky 64k;
        sticky cookie srv_id expires=1h domain=.example.com path=/;
        server backend1.example.com;
        server backend2.example.com;
    }

### 3. SSL/TLS Termination

Nginx can handle SSL/TLS termination, offloading the SSL/TLS processing from backend servers.

**Additional Detail:** This decrypts traffic at the Nginx layer, protecting your backends from computing complex handshakes. For high security, ensure you disable weak TLS versions and configure strict security headers.

Configuration Example:

    nginx

    server {
        listen 443 ssl;
        server_name example.com;

        ssl_certificate /etc/nginx/ssl/certificate.crt;
        ssl_certificate_key /etc/nginx/ssl/key.key;
    
        # Modern Security Tuning
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

### 4. HTTP/2 and WebSocket Support

Nginx supports HTTP/2 and WebSocket protocols, enabling efficient handling of modern web applications.

**Additional Detail:** For modern configurations, http2 is now enabled globally inside a separate http2 on; directive rather than directly on the listen line for newer Nginx versions. WebSockets require upgrading connection headers.

Configuration Example:

    nginx

    server {
        listen 443 ssl;
        server_name example.com;
        http2 on; # Modern Nginx HTTP/2 syntax

        ssl_certificate /path/to/certificate.crt;
        ssl_certificate_key /path/to/key.key;

        location /ws/ {
            proxy_pass http://backend;
            proxy_http_version 1.1;
        
            # Necessary headers for WebSocket handshake persistence
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
    }

### 5. Load Balancing Algorithms Customization

Nginx allows customization of load balancing algorithms to suit specific needs, using variables and custom logic.

**Additional Detail:** By leveraging map blocks and internal variables like $geo, $cookie_, or $arg_, you can build sophisticated logic routes before passing traffic to specific upstreams.

Configuration Example:

    nginx

    map $cookie_user_group $selected_upstream {
        "beta"  beta_backend;
        default standard_backend;
    }

    server {
        listen 80;
        location / {
            proxy_pass http://$selected_upstream;
        }
    }

### 6. Dynamic Configuration with NGINX Plus

NGINX Plus offers advanced features such as dynamic reconfiguration, active health checks, and enhanced monitoring capabilities.
**Additional Detail:** It includes an on-the-fly HTTP API. This lets you add, remove, or modify backend upstream servers dynamically via API requests without reloading Nginx configuration files.

Example Configuration with NGINX Plus:

    nginx

    upstream backend {
        zone backend 64k; # State is saved here across worker processes
        state /var/lib/nginx/state/backend.state; # Saves upstream configuration
    
        server backend1.example.com;
        server backend2.example.com;
    }

## Configuring Load Balancing in Nginx

### 1. Basic Configuration

Setting up basic load balancing involves defining an upstream block and referencing it in server directives.

Configuration Example:

    nginx

    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
    }

    server {
        listen 80;
        server_name example.com;
    
        location / {
            proxy_pass http://backend;
        }
    }

### 2. Using Different Load Balancing Methods

Switching between load balancing methods is straightforward by adding the appropriate directive within the upstream block. Simply define your algorithm tag (`least_conn;`, `ip_hash;`) at the top of the block.

### 3. Enabling Health Checks

Health checks can be enabled with simple directives to ensure backend server reliability. For open-source Nginx setups, leverage max_fails=3 fail_timeout=10s; tags inline on your server rows.

### 4. Implementing Session Persistence

Session persistence can be implemented using sticky sessions to maintain user sessions across requests. If stuck on open-source variants, ip_hash; offers the quickest alternative setup.

### 5. SSL/TLS Termination Setup

Setting up SSL/TLS termination involves configuring the server to listen on port 443 and providing SSL certificate details. Ensure you pass proxy headers so backends read the correct client IPs.

## Best Practices

**1. Monitor with Passive Parameters:** Always include max_fails and fail_timeout parameters in open-source setups to gracefully isolate crashed backend servers.

**2. Employ SSL/TLS termination to enhance security:** Keep cipher suites updated and configure X-Forwarded-Proto so backends don't run into infinite redirect loops.

**3. Implement session persistence for user session consistency:** If your app state is local to a server, utilize ip_hash or sticky models to preserve user shopping carts or active logins.

**4. Utilize dynamic configuration capabilities for flexibility:** If using NGINX Plus, connect it to your auto-scaling service groups to scale backends seamlessly.

**5. Regularly update and monitor Nginx for performance and security:** Enable the Nginx stub status module or NGINX Plus dashboard to track connection limits.

## Conclusion

Nginx offers robust HTTP load balancing capabilities and advanced features, making it an ideal choice for managing traffic to web applications.   

By understanding and implementing the various load balancing methods and features, administrators can ensure high availability, scalability, and reliability of their services.

## Reference

NGINX Plus - HTTP Load Balancing