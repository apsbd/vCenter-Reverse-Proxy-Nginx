# vCenter Reverse Proxy Nginx

1. Create a VPS with Ubuntu 23
2. Install Nginx
```
sudo apt update
sudo apt install nginx
```
3. Point the domain to the VPS ip address
4. Create a SSL certificate for the Domain
5. Create a Nginx Config file
```
sudo nano /etc/nginx/conf.d/vcsa.conf
```

```
server { 
   listen 443 ssl http2; 
   server_name my_internet_vcenter_fqdn; 
   ssl_certificate /etc/letsencrypt/live/my_letsencrypt_domain/fullchain.pem; 
   ssl_certificate_key /etc/letsencrypt/live/my_letsencrypt_domain/privkey.pem; 
   include /etc/letsencrypt/options-ssl-nginx.conf; 

   location / { 
      proxy_set_header Host "your_vCenter_fqdn"; 
      proxy_set_header Origin "your_vCenter_fqdn";
      proxy_set_header X-Real-IP $remote_addr; 
      proxy_ssl_verify off; 
      proxy_pass https://your_vCenter_fqdn; 
      proxy_http_version 1.1; 
      proxy_set_header Upgrade $http_upgrade; 
      proxy_set_header Connection "upgrade"; 
      proxy_buffering off; 
      client_max_body_size 0; 
      proxy_read_timeout 36000s; 
      proxy_redirect https://your_vCenter_fqdn/ https://my_internet_vcenter_fqdn/; 
   } 

   location /websso/SAML2 { 
      sub_filter "your_vCenter_fqdn" "my_internet_vcenter_fqdn"; 
      proxy_set_header Host your_vCenter_fqdn; 
      proxy_set_header X-Real-IP $remote_addr; 
      proxy_ssl_verify off; 
      proxy_pass https://your_vCenter_fqdn; 
      proxy_http_version 1.1; 
      proxy_set_header Upgrade $http_upgrade; 
      proxy_set_header Connection "upgrade"; 
      proxy_buffering off; 
      client_max_body_size 0; 
      proxy_read_timeout 36000s; 
      proxy_ssl_session_reuse on; 
      proxy_redirect https://your_vCenter_fqdn/ https://my_internet_vcenter_fqdn/; 
   } 
}
```

```
sudo service nginx restart
```
