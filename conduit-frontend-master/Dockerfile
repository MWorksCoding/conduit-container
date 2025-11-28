# Frontend Dockerfile (Angular) - multi-stage: build -> nginx

# --- builder stage: build Angular assets
FROM node:18.19 AS builder
WORKDIR /app

# copy package files and install first to leverage caching
COPY conduit-frontend-master/package*.json ./
RUN npm install --force

# copy source and build
COPY conduit-frontend-master/ .

# adjust build command to your angular.json project name if needed
RUN npm run build

# --- final stage: nginx to serve built static files
FROM nginx:alpine
# remove default nginx content
RUN rm -rf /usr/share/nginx/html/*

# copy built files from builder
COPY --from=builder /app/dist/angular-conduit/ /usr/share/nginx/html/

# optional: copy a small nginx conf (if you want custom headers / routing)
# COPY conduit-frontend-master/nginx.conf /etc/nginx/conf.d/default.conf

RUN chmod -R 755 /usr/share/nginx/html

EXPOSE 80

# Start nginx (default CMD in nginx image)
CMD ["nginx", "-g", "daemon off;"]