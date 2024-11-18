# Use Node.js to build the Vue project
FROM node:latest AS builder

# Set the working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package.json package-lock.json ./
RUN npm install

# Copy the source code and build
COPY . .
RUN npm run build

# Use Nginx to serve the built files
FROM nginx:latest

# Copy the built files to Nginx
COPY --from=builder /app/dist /usr/share/nginx/html

# Expose the port
EXPOSE 80
