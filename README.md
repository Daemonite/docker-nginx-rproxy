# Simple NGINX reverse proxy in a Docker container

This is a standard NGINX Docker container configured to act as a reverse proxy on ports 80 (HTTP) and 443 (HTTPS) for a backend Web application container.

## Why does this exist?

Production Docker environments almost always place reverse proxies in front of Web application containers. This allows the environment to deploy an application's containers to different hosts without changing the endpoint used to access them.

While local development environments usually run all containers on a single host, it may be important for some applications to reproduce a similar reverse proxy setup in development. For example, an application may try to detect the IP address of the client or whether the client issued the requiest via SSL. The logic required to do this changes when a proxy sits between the application container and client. To implement such features correctly a reverse proxy is required in the development environment.

## How to Use

Pull the reverse proxy image from Docker Hub:

    docker pull daemonite/nginx-rproxy

Run your application container

    docker run --name myapp <...>

Run the reverse proxy container, publishing ports 80 (and 443 if needed) and linking to your application container using the alias `backend`:

    docker run -d -p 80:80 -p 443:443 --link myapp:backend daemonite/nginx-rproxy

Requests to the published ports on the Docker host will then be forwarded to the application.

## Proxy HTTP Headers

The reverse proxy sets the following HTTP headers on forwarded requests:

- `Host` is set to the Host header sent by the client (which may not be the hostname used by the proxy)
- `X-Real-IP` is set to the client IP address
- `X-Forwarded-For` is set to the list of IP addresses the request has been forwarded for. This includes the client and any proxies preceding the current proxy
- `X-Forwarded-Proto` is set to the URL protocol of the original request (usually either `http` or `https`)

## Limitations and Issues

The bundled configuration assumes the backend application listens on port 80 for HTTP and port 443 for HTTPS.

The dummy SSL self-signed certificate is generated every time the image is built. This will cause your browser to issue a warning after every update of the proxy's Docker image.

## Further Reading

For more information on configuring NGINX as a reverse proxy, read the [NGINX Reverse Proxy Admin Guide](https://www.nginx.com/resources/admin-guide/reverse-proxy/).

## License

This project is provided compliments of [Daemon](http://www.daemon.com.au/) and made available under the [MIT license](LICENSE.txt).