## Without SSl
```
# /etc/nginx/sites-available/doc-free2space-http
server {
    listen 80;
    listen [::]:80;
    server_name doc-free2space.syed-kamruzzaman.com;

    # Static site root
    root /var/www/doc-free2space;
    index index.html;

    # Let’s Encrypt challenge (ভবিষ্যতে SSL নিলে কাজ লাগবে)
    location /.well-known/acme-challenge/ { root /var/www/doc-free2space; }

    # Serve files (SPA হলে index.html ফfallback)
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location ~* \.(?:css|js|mjs|json|png|jpg|jpeg|gif|svg|ico|webp|avif|woff2?)$ {
        expires 7d;
        add_header Cache-Control "public, max-age=604800, immutable";
        try_files $uri =404;
    }

    # Block sensitive/hidden files
    location ^~ /.git { deny all; }
    location = /.env  { deny all; }
    location ~* /\.(?!well-known) { deny all; access_log off; log_not_found off; }

    # Security headers
    add_header X-Content-Type-Options nosniff;
    add_header Referrer-Policy no-referrer-when-downgrade;
    add_header X-Frame-Options SAMEORIGIN;

    access_log /var/log/nginx/doc-free2space.access.log;
    error_log  /var/log/nginx/doc-free2space.error.log;
}
```

```
sudo ln -sf /etc/nginx/sites-available/doc-free2space-http /etc/nginx/sites-enabled/doc-free2space
sudo nginx -t
sudo systemctl reload nginx
```


## With SSl
```
# HTTP -> HTTPS (ACME allow + redirect)
server {
    listen 80;
    listen [::]:80;
    server_name doc-free2space.syed-kamruzzaman.com;

    # Let's Encrypt challenge allow (renewal এর সময় কাজ লাগবে)
    location /.well-known/acme-challenge/ { root /var/www/doc-free2space; }

    return 301 https://$host$request_uri;
}

# HTTPS: serve static docs
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name doc-free2space.syed-kamruzzaman.com;

    # --- SSL (Let's Encrypt) ---
    ssl_certificate     /etc/letsencrypt/live/doc-free2space.syed-kamruzzaman.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/doc-free2space.syed-kamruzzaman.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # --- Static site root ---
    root /var/www/doc-free2space;
    index index.html;

    # ACME challenge (backup — rare case)
    location /.well-known/acme-challenge/ { root /var/www/doc-free2space; }

    # SPA-friendly fallback
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location ~* \.(?:css|js|mjs|json|png|jpg|jpeg|gif|svg|ico|webp|avif|woff2?)$ {
        expires 7d;
        add_header Cache-Control "public, max-age=604800, immutable";
        try_files $uri =404;
    }

    # 🔒 Block sensitive/hidden files
    location ^~ /.git { deny all; }
    location = /.env  { deny all; }
    location ~* /\.(?!well-known) { deny all; access_log off; log_not_found off; }

    # Security headers
    add_header X-Content-Type-Options nosniff;
    add_header Referrer-Policy no-referrer-when-downgrade;
    add_header X-Frame-Options SAMEORIGIN;

    # HSTS (সাইট HTTPS ঠিকমতো চলার পর অন করলে ভালো)
    # add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    access_log /var/log/nginx/doc-free2space.access.log;
    error_log  /var/log/nginx/doc-free2space.error.log;
}

```

## Certbot
```
sudo certbot --nginx -d doc-free2space.syed-kamruzzaman.com

sudo nginx -t
sudo systemctl reload nginx
```