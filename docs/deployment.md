# Deployment Guide

This guide covers deploying the Payment Copilot application to various platforms.

## Overview

Payment Copilot is a Next.js application that can be deployed to multiple platforms. This guide covers the most common deployment options.

## Prerequisites

Before deploying, ensure you have:

- [ ] All environment variables configured
- [ ] Firebase project set up
- [ ] API keys for all services
- [ ] Domain name (optional)
- [ ] SSL certificate (for production)

## Deployment Options

### Vercel (Recommended)

Vercel is the recommended platform for Next.js applications.

#### 1. Connect Repository

1. Go to [Vercel Dashboard](https://vercel.com/dashboard)
2. Click "New Project"
3. Import your Git repository
4. Select the project

#### 2. Configure Environment Variables

In the Vercel dashboard:

1. Go to Project Settings > Environment Variables
2. Add all required variables:

```env
NEXT_PUBLIC_FIREBASE_API_KEY=your_api_key
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
NEXT_PUBLIC_FIREBASE_PROJECT_ID=your_project_id
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your_project.appspot.com
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
NEXT_PUBLIC_FIREBASE_APP_ID=your_app_id
OPENAI_API_KEY=your_openai_api_key
ANTHROPIC_API_KEY=your_anthropic_api_key
REPLICATE_API_TOKEN=your_replicate_token
DEEPGRAM_API_KEY=your_deepgram_api_key
```

#### 3. Deploy

1. Click "Deploy"
2. Wait for deployment to complete
3. Access your app at the provided URL

#### 4. Custom Domain (Optional)

1. Go to Project Settings > Domains
2. Add your custom domain
3. Configure DNS records as instructed

### Netlify

#### 1. Build Configuration

Create `netlify.toml` in your project root:

```toml
[build]
  command = "npm run build"
  publish = ".next"

[build.environment]
  NODE_VERSION = "18"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

#### 2. Deploy

1. Connect your repository to Netlify
2. Set build command: `npm run build`
3. Set publish directory: `.next`
4. Add environment variables
5. Deploy

### AWS Amplify

#### 1. Connect Repository

1. Go to [AWS Amplify Console](https://console.aws.amazon.com/amplify/)
2. Click "New App" > "Host web app"
3. Connect your Git repository

#### 2. Build Settings

Use these build settings:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - npm install
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: .next
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```

#### 3. Environment Variables

Add all required environment variables in the Amplify console.

### Docker

#### 1. Create Dockerfile

```dockerfile
FROM node:18-alpine AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV production
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
USER nextjs
EXPOSE 3000
ENV PORT 3000
CMD ["node", "server.js"]
```

#### 2. Build and Run

```bash
# Build image
docker build -t payment-copilot .

# Run container
docker run -p 3000:3000 --env-file .env.local payment-copilot
```

## Environment Configuration

### Production Environment Variables

Ensure all environment variables are set in your deployment platform:

```env
# Firebase (Required)
NEXT_PUBLIC_FIREBASE_API_KEY=your_api_key
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
NEXT_PUBLIC_FIREBASE_PROJECT_ID=your_project_id
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your_project.appspot.com
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
NEXT_PUBLIC_FIREBASE_APP_ID=your_app_id

# AI Services (Required)
OPENAI_API_KEY=your_openai_api_key
ANTHROPIC_API_KEY=your_anthropic_api_key
REPLICATE_API_TOKEN=your_replicate_token
DEEPGRAM_API_KEY=your_deepgram_api_key

# Optional
NEXT_PUBLIC_APP_URL=https://your-domain.com
NODE_ENV=production
```

### Environment-Specific Configuration

Create different configurations for different environments:

```typescript
// lib/config.ts
const config = {
  development: {
    apiUrl: 'http://localhost:3000',
    firebaseConfig: {
      // Development Firebase config
    }
  },
  production: {
    apiUrl: process.env.NEXT_PUBLIC_APP_URL,
    firebaseConfig: {
      // Production Firebase config
    }
  }
};

export default config[process.env.NODE_ENV || 'development'];
```

## Firebase Configuration

### Production Firebase Setup

1. **Create Production Project**
   - Create a new Firebase project for production
   - Enable Authentication
   - Configure Google Sign-In

2. **Update Authorized Domains**
   - Add your production domain to authorized domains
   - Remove localhost from production project

3. **Security Rules**
   - Update Firestore security rules for production
   - Configure Storage security rules

### Firestore Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can only access their own data
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    
    // Public data (if any)
    match /public/{document} {
      allow read: if true;
    }
  }
}
```

## Performance Optimization

### Build Optimization

1. **Enable Compression**
   ```javascript
   // next.config.js
   module.exports = {
     compress: true,
     poweredByHeader: false,
   };
   ```

2. **Image Optimization**
   ```javascript
   // next.config.js
   module.exports = {
     images: {
       domains: ['your-domain.com'],
       formats: ['image/webp', 'image/avif'],
     },
   };
   ```

3. **Bundle Analysis**
   ```bash
   npm install --save-dev @next/bundle-analyzer
   npm run analyze
   ```

### CDN Configuration

Configure CDN for static assets:

1. **Vercel**: Automatic CDN
2. **Netlify**: Automatic CDN
3. **AWS CloudFront**: Manual configuration required

## Monitoring and Analytics

### Error Tracking

1. **Sentry Integration**
   ```bash
   npm install @sentry/nextjs
   ```

2. **Vercel Analytics**
   - Built-in with Vercel
   - Enable in project settings

### Performance Monitoring

1. **Web Vitals**
   - Use Next.js built-in Web Vitals
   - Monitor Core Web Vitals

2. **Uptime Monitoring**
   - Set up uptime monitoring
   - Configure alerts

## Security

### Security Headers

Configure security headers:

```javascript
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'Referrer-Policy',
            value: 'origin-when-cross-origin',
          },
        ],
      },
    ];
  },
};
```

### API Security

1. **Rate Limiting**
   - Implement rate limiting for API routes
   - Use middleware for protection

2. **Input Validation**
   - Validate all inputs
   - Sanitize user data

## Troubleshooting

### Common Deployment Issues

1. **Build Failures**
   - Check Node.js version compatibility
   - Verify all dependencies are installed
   - Check for TypeScript errors

2. **Environment Variables**
   - Ensure all variables are set
   - Check variable names and values
   - Restart deployment after changes

3. **Firebase Issues**
   - Verify Firebase configuration
   - Check authorized domains
   - Ensure API keys are correct

### Debug Mode

Enable debug mode for troubleshooting:

```javascript
// next.config.js
module.exports = {
  env: {
    DEBUG: process.env.NODE_ENV === 'development',
  },
};
```

## Rollback Strategy

### Vercel

1. Go to Deployments tab
2. Select previous deployment
3. Click "Promote to Production"

### Other Platforms

1. Revert to previous commit
2. Trigger new deployment
3. Monitor for issues

## Maintenance

### Regular Tasks

1. **Dependency Updates**
   - Update dependencies monthly
   - Test thoroughly after updates

2. **Security Updates**
   - Monitor security advisories
   - Apply updates promptly

3. **Performance Monitoring**
   - Review performance metrics
   - Optimize as needed

### Backup Strategy

1. **Database Backups**
   - Enable Firebase automatic backups
   - Test restore procedures

2. **Code Backups**
   - Use Git for version control
   - Regular pushes to remote repository

## Support

For deployment issues:

1. Check platform-specific documentation
2. Review error logs
3. Test locally first
4. Contact platform support if needed
