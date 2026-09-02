# ------- build stage --------
FROM node:20-alpine AS build

WORKDIR /app

COPY package.json package-lock.json* ./

RUN npm ci && npm cache clean --force

COPY . .

RUN npm run build

# ----- final stage ---------
FROM nginx:1.27-alpine AS production

# Remove default static files and configurations
RUN rm -rf /etc/nginx/conf.d/default.conf /usr/share/nginx/html/*

# Copy your custom Nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy build artifacts from the build stage
COPY --from=build /app/dist /usr/share/nginx/html

# Fix permissions for the non-root 'nginx' user (including temp folders)
RUN chown -R nginx:nginx /usr/share/nginx/html && \
    chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /var/log/nginx && \
    chown -R nginx:nginx /etc/nginx/conf.d && \
    touch /var/run/nginx.pid && \
    chown -R nginx:nginx /var/run/nginx.pid

# Switch to non-root user
USER nginx

# Expose the unprivileged port
EXPOSE 8080

CMD [ "nginx", "-g", "daemon off;" ]
