# TxBot Chatbot UI - Deployment Guidelines

A comprehensive guide for deploying the TxBot chatbot UI across different environments.

## Table of Contents

1. [Local Development](#local-development)
2. [Self-Hosted](#self-hosted)
3. [Cloud Deployment](#cloud-deployment)
4. [Docker Containerization](#docker-containerization)
5. [CI/CD Pipeline](#cicd-pipeline)
6. [Production Checklist](#production-checklist)

---

## Local Development

### Prerequisites
- Modern web browser (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- Text editor (VS Code, Sublime, etc.)
- Python 3.8+ (for backend integration)

### Setup Steps

1. **Clone the repository:**
```bash
git clone https://github.com/KENWELL-TX-ORG/Team---Name-file-C-Users-VST-Downloads-kenwell-publisher.htm.git
cd Team---Name-file-C-Users-VST-Downloads-kenwell-publisher.htm/chatbot.ui
```

2. **Start a local server:**
```bash
# Using Python 3
python -m http.server 8000

# Or using Node.js
npx http-server
```

3. **Open in browser:**
```
http://localhost:8000
```

4. **Configure backend (optional):**
Edit `script.js` and update the API endpoint:
```javascript
const API_URL = 'http://localhost:5000/api/chat';
```

---

## Self-Hosted

### Requirements
- Web server (Apache, Nginx, or similar)
- Static file serving capability
- HTTPS certificate (recommended)

### Apache Configuration

Create `.htaccess` file in `chatbot.ui/` directory:
```apache
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /chatbot.ui/
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^(.*)$ index.html [L]
</IfModule>

# Enable CORS if needed
<IfModule mod_headers.c>
    Header set Access-Control-Allow-Origin "*"
</IfModule>
```

### Nginx Configuration

Add to your nginx config:
```nginx
server {
    listen 80;
    server_name yourdomain.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.com;
    
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;
    
    root /var/www/txbot/chatbot.ui;
    index index.html;
    
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
    
    # Disable caching for index.html
    location = /index.html {
        add_header Cache-Control "no-cache, no-store, must-revalidate";
    }
}
```

### Deployment Steps

1. **Copy files to server:**
```bash
scp -r chatbot.ui/* user@server:/var/www/txbot/chatbot.ui/
```

2. **Set permissions:**
```bash
chmod 755 /var/www/txbot/chatbot.ui
chmod 644 /var/www/txbot/chatbot.ui/*
```

3. **Verify SSL:**
```bash
ssl-test yourdomain.com
```

---

## Cloud Deployment

### Vercel (Recommended for Next.js/Static Sites)

1. **Install Vercel CLI:**
```bash
npm install -g vercel
```

2. **Deploy:**
```bash
cd chatbot.ui
vercel
```

3. **Configure environment variables** in Vercel dashboard if needed.

### GitHub Pages

1. **Enable in repository settings:**
   - Go to Settings → Pages
   - Select `main` branch, `/chatbot.ui` folder
   - Save

2. **Access at:**
```
https://KENWELL-TX-ORG.github.io/Team---Name-file-C-Users-VST-Downloads-kenwell-publisher.htm/chatbot.ui/
```

### Netlify

1. **Connect repository:**
   - Sign in to Netlify
   - Click "New site from Git"
   - Select your repository
   - Set build folder to `chatbot.ui`

2. **Deploy:**
   - Netlify auto-deploys on push to main branch

### AWS S3 + CloudFront

1. **Create S3 bucket:**
```bash
aws s3 mb s3://txbot-chatbot-ui
```

2. **Upload files:**
```bash
aws s3 sync . s3://txbot-chatbot-ui --exclude ".git/*"
```

3. **Create CloudFront distribution:**
   - Set origin to S3 bucket
   - Enable HTTPS
   - Set default root object to `index.html`

4. **Access via CloudFront URL**

---

## Docker Containerization

### Dockerfile

Create `Dockerfile` in the root directory:

```dockerfile
FROM nginx:alpine

# Copy chatbot files
COPY chatbot.ui/ /usr/share/nginx/html/

# Copy nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port
EXPOSE 80

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1
```

### nginx.conf

```nginx
server {
    listen 80;
    
    root /usr/share/nginx/html;
    index index.html;
    
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # Cache control
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 30d;
    }
    
    location = /index.html {
        add_header Cache-Control "no-cache, must-revalidate";
    }
}
```

### Docker Compose

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  chatbot-ui:
    build: .
    ports:
      - "80:80"
    environment:
      - API_URL=http://backend:5000
    depends_on:
      - backend
    restart: unless-stopped

  backend:
    build: ../
    ports:
      - "5000:5000"
    environment:
      - FLASK_APP=Kenwell.py
    restart: unless-stopped
```

### Build and Run

```bash
# Build image
docker build -t txbot-chatbot-ui .

# Run container
docker run -p 80:80 txbot-chatbot-ui

# Or use Docker Compose
docker-compose up -d
```

---

## CI/CD Pipeline

### GitHub Actions Workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy TxBot Chatbot UI

on:
  push:
    branches: [main]
    paths:
      - 'chatbot.ui/**'
      - '.github/workflows/deploy.yml'

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Validate HTML/CSS/JS
        run: |
          npm install -g htmlhint stylelint
          htmlhint chatbot.ui/index.html
          stylelint chatbot.ui/styles.css
      
      - name: Build Docker Image
        run: docker build -t txbot-chatbot-ui:${{ github.sha }} .
      
      - name: Push to Registry
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker tag txbot-chatbot-ui:${{ github.sha }} txbot-chatbot-ui:latest
          docker push txbot-chatbot-ui:latest
      
      - name: Deploy to Production
        run: |
          ssh -i ${{ secrets.DEPLOY_KEY }} user@server 'cd /var/www/txbot && docker pull txbot-chatbot-ui:latest && docker-compose up -d'
```

---

## Production Checklist

Before deploying to production:

### Security
- [ ] Enable HTTPS/SSL
- [ ] Set CORS headers properly
- [ ] Validate all user inputs
- [ ] Sanitize API responses
- [ ] Use Content Security Policy (CSP) headers
- [ ] Keep dependencies updated

### Performance
- [ ] Minimize CSS/JavaScript files
- [ ] Enable gzip compression
- [ ] Set cache headers appropriately
- [ ] Use CDN for static assets
- [ ] Monitor page load times

### Monitoring
- [ ] Set up error logging (Sentry, LogRocket)
- [ ] Monitor API availability
- [ ] Track user interactions
- [ ] Set up alerts for failures
- [ ] Monitor resource usage

### Compliance
- [ ] Review privacy policy
- [ ] Ensure GDPR compliance
- [ ] Document data handling
- [ ] Add terms of service
- [ ] Implement cookie consent

### Backup & Recovery
- [ ] Backup configuration files
- [ ] Document deployment process
- [ ] Create rollback procedure
- [ ] Test disaster recovery
- [ ] Maintain version history

---

## Troubleshooting Deployment

### Issue: CORS Errors
**Solution:** Configure backend to send proper CORS headers:
```python
from flask_cors import CORS
CORS(app)
```

### Issue: 404 on Page Reload
**Solution:** Configure server to serve `index.html` for all routes.

### Issue: Slow Loading
**Solution:**
- Enable compression: `gzip on;`
- Minify assets
- Use CDN
- Enable browser caching

### Issue: SSL Certificate Errors
**Solution:** Use Let's Encrypt for free certificates:
```bash
certbot certonly --webroot -w /var/www/txbot/chatbot.ui -d yourdomain.com
```

---

## Performance Optimization

### Image Optimization
```bash
# Convert images to WebP
cwebp input.png -o output.webp

# Compress images
imagemin chatbot.ui/images/* --out-dir=chatbot.ui/images
```

### Asset Bundling
```bash
# Minify JavaScript
uglifyjs script.js -c -m -o script.min.js

# Minify CSS
cleancss styles.css -o styles.min.css
```

### HTTP/2 Server Push
```nginx
location = /index.html {
    add_header Link "</styles.css>; rel=preload; as=style" always;
    add_header Link "</script.js>; rel=preload; as=script" always;
}
```

---

## Monitoring & Logging

### Application Monitoring
- Datadog
- New Relic
- SignalFx

### Error Tracking
- Sentry
- Rollbar
- Airbrake

### Analytics
- Google Analytics
- Mixpanel
- Amplitude

---

## Support & Documentation

- 📖 Main README: [chatbot.ui/README.md](README.md)
- 🐛 Issue Tracker: [GitHub Issues](../../issues)
- 💬 Discussions: [GitHub Discussions](../../discussions)

---

**TxBot Chatbot UI** - Ready for Production Deployment 🚀

Powered by KENWELL-TX-ORG | MIT License
