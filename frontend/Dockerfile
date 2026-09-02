# ---- Build stage ----
# Optimization 1: Hardening the Node base image to use a specific patch version
FROM node:20.17.0-alpine AS build
WORKDIR /app
COPY package.json package-lock.json* ./

# FIXED: Fallback to standard install if package-lock.json doesn't exist
RUN if [ -f package-lock.json ]; then \
      npm ci; \
    else \
      npm install; \
    fi && npm cache clean --force

COPY . .
RUN npm run build

# ---- Production stage ----
# Optimization 2: Explicitly tag Nginx version to clear out base layer CVEs 
FROM nginx:1.27.1-alpine AS production

# Security: remove default nginx config
RUN rm -rf /etc/nginx/conf.d/default.conf /usr/share/nginx/html/*

# Copy custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy built frontend from build stage
COPY --from=build /app/dist /usr/share/nginx/html

# Security: run nginx as non-root
RUN chown -R nginx:nginx /usr/share/nginx/html && \
    chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /var/log/nginx && \
    touch /var/run/nginx.pid && \
    chown -R nginx:nginx /var/run/nginx.pid

USER nginx

# Pitfall Fix: Switched from port 80 to port 8080
EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
