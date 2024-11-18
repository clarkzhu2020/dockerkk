# Use Node.js to build the Vue project
FROM node:latest AS frontend

# Set the working directory
WORKDIR /app

# Copy package.json and install dependencies

# Copy the source code and build
COPY . .
RUN yarn && yarn build && yarn dev &

# Use Nginx to serve the built files
#FROM nginx:latest

# Copy the built files to Nginx
#COPY --from=builder /app/dist /www/dist

# Expose the port
EXPOSE 80
