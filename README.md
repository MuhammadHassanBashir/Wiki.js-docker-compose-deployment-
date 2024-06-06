# Wiki.js-docker-compose-deployment

**Deployment Guide: Deploying Wiki.js with Docker Compose**

This document provides step-by-step instructions for deploying Wiki.js using Docker Compose. Additionally, it covers common issues encountered during deployment and their resolutions.

Docker Compose Configuration
Below is the Docker Compose configuration file (docker-compose.yml) for deploying Wiki.js and PostgreSQL database:


  version: "3"
  services:
  
    db:
      image: postgres:15-alpine
      environment:
        POSTGRES_DB: wiki
        POSTGRES_PASSWORD: wikijsrocks
        POSTGRES_USER: wikijs
      #logging:
      #  driver: "none"
      restart: unless-stopped
      volumes:
        - db-data:/var/lib/postgresql/data
  
    wiki:
      image: ghcr.io/requarks/wiki:2
      depends_on:
        - db
      environment:
        DB_TYPE: postgres
        DB_HOST: db
        DB_PORT: 5432
        DB_USER: wikijs
        DB_PASS: wikijsrocks
        DB_NAME: wiki
      restart: unless-stopped
      ports:
        - "8080:3000"
  
    volumes:
      db-data:

**Deployment Steps**
**Deploy Wiki.js and PostgreSQL:**
**Execute the following command to deploy Wiki.js and PostgreSQL containers:**

**sudo docker-compose up -d**

**SSL Certificate Configuration:**
**Ensure that SSL certificates are correctly configured for the domain docs.disearch.ai. Update the Nginx configuration file (/etc/nginx/sites-available/docs.disearch.ai) with the SSL certificate paths and port number 8080.**

**Example SSL configuration for Nginx:**

**nginx**

server {
    server_name docs.disearch.ai;

    location / {
        proxy_pass http://localhost:8080;  # Update this line to use the internal hostname and port
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/docs.disearch.ai/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/docs.disearch.ai/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    if ($host = docs.disearch.ai) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen 80;
    server_name docs.disearch.ai;
    return 404; # managed by Certbot
}


**Issue Resolution**

**Logging Adapter Issue:**
Initially, the logging adapter for PostgreSQL was causing issues. It was temporarily commit logging driver section in the docker-compose.yml file. And redeploy the compose file and it works fine.

**PostgreSQL Version Compatibility Issue:**
A compatibility issue was encountered due to PostgreSQL database files being initialized by a previous version (version 12) while attempting to run version 15.7. To resolve this, the Docker volume associated with the PostgreSQL data directory was deleted, and the Docker Compose file was redeployed, resulting in successful deployment.

**Conclusion**
Following these steps, Wiki.js should be successfully deployed and accessible at https://docs.disearch.ai. Ensure to monitor the deployment for any further issues and perform regular backups to prevent data loss.

This document provides a comprehensive guide for deploying Wiki.js and resolving common deployment issues. If you encounter any further issues or require additional assistance, please refer to the relevant sections in this document or reach out for support.





