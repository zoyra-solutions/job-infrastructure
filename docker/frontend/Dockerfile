# Frontend Dockerfile (Multi-stage build)
FROM node:18-alpine AS build

# Set working directory
WORKDIR /app

# Copy package files
COPY frontend/web/package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY frontend/web/ .

# Build the application
RUN npm run build

# Production stage with Nginx
FROM nginx:alpine

# Copy built app to nginx
COPY --from=build /app/build /usr/share/nginx/html

# Copy nginx configuration
COPY infrastructure/docker/frontend/nginx.conf /etc/nginx/conf.d/default.conf

# Copy nginx main configuration
COPY infrastructure/docker/frontend/nginx.main.conf /etc/nginx/nginx.conf

# Create non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nextjs -u 1001

# Change ownership of nginx directories
RUN chown -R nextjs:nodejs /var/cache/nginx /var/run /var/log/nginx /usr/share/nginx/html

# Switch to non-root user
USER nextjs

# Expose port
EXPOSE 80

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:80 || exit 1

# Start nginx
CMD ["nginx", "-g", "daemon off;"]