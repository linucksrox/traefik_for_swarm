# What is Traefik
"Traefik (pronounced traffic) is a modern HTTP reverse proxy and load balancer that makes deploying microservices easy."

This is a reverse proxy that automatically generates the necessary configurations on the fly by watching any services added to docker.
For example, you can start up a Gitlab container, specifying the hostname gitlab.example.local, and this automatically routes to it without having to do any manual configuration.

## Traefik Version Configuration
This project uses Traefik v2
* Update the Traefik Docker image version by changing the image version number in stack.yml (version numbers are referred to as Tags, and you can [find available tags on Docker Hub](hub.docker.com/_/traefik?tab=description)). **Do not use "latest" as it can cause breaking changes**

## Traefik Dashboard
This example uses a basic HTTP authentication middleware for connecting to the dashboard. You can disable authentication, otherwise you'll need to use htpasswd to hash your password and update it in stack.yml.

## HTTPS Encryption
You can provide your own certificates, or use Let's Encrypt to automate the process. This project contains configuration samples for self provided, or Let's Encrypt wildcard certificate using DNS challenge with Cloudflare.

### Providing Your Own Certificates
* In stack.yml, make sure traefik_certificates.yml is enabled (not commented out) in the volumes section
* Copy SSL certificate files into the ssl folder, and change permissions.
```
sudo chown root:root ./ssl/*
sudo chmod 440 ./ssl/*
```
* Update the certificate and key path in traefik_certificates.yml

### Let's Encrypt Wildcard Certificates Using DNS Challenge and Cloudflare
* In stack.yml, uncomment the lines relating to secrets and environment variables
* In stack.yml, comment out or delete traefik_certificates.yml volumes section
* In stack.yml, make sure the letsencrypt folder is mapped in the volumes section for use with the auto generated acme.json file (it will be created when traefik attemps to get certificates from Let's Encrypt)
* Add a Docker Swarm secret called "traefik_cf-api-token" with the token value for your Cloudflare DNS management (optionally bypass and enter your token directly in as an ENVIRONMENT variable in stack.yml, but that is not recommended for security purposes).
* In traefik.yml, make any necessary adjustments in the certificateResolvers section. Replace all occurrences of "exampledomain" with your domain name (use whatever email you want), and optionally comment out or uncomment the DNS resolver option if you are using internal DNS and can't access public DNS servers from the host that Traefik is running on. As an aside, this allows Traefik to generate certificates without the need for any external ports being forwarded to it, so you can use Traefik for internal only systems, but generate legitimate certificates. This is also handy for testing before going live.

## Start the service
```
docker stack deploy traefik -c stack.yml
```
* Check that you can access the Traefik dashboard at http://traefik.example.com:8080/dashboard/#/

## Notes on redirection
Redirection from http to https is done through a router + middleware. This is currently defined on the traefik container right in stack.yml, so **any** traffic coming in on http will be permanently redirected to https (HTTP 302). If necessary, you could place individual router + middlware redirects on each container if you want to do this per service (and remove the global redirect on the traefik container). Another option is to have alternative entry points on different ports (like the birtreports entrypoint) so you can access services without redirecting to https.

## Notes on session stickiness
If you are running services that maintain a session, you can enable session stickiness in Traefik using labels in the deploy section of stack.yml. See the whoami.yml sample for a demonstration.
```
  - "traefik.http.services.tomcat-service.loadbalancer.sticky=true"
  - "traefik.http.services.tomcat-service.loadbalancer.sticky.cookie.httponly=true"
  - "traefik.http.services.tomcat-service.loadbalancer.sticky.cookie.secure=true"
  # - "traefik.http.services.tomcat-service.loadbalancer.sticky.cookie.name=BirtReports"
```