# Use official Go image
FROM golang:latest AS backend

# Set the working directory
WORKDIR /app/api

# Copy and install dependencies

# Copy the source code
COPY . .

# Build the application
RUN  go mod tidy &&  go build -o appdemo . && chmod +x appdemo && ./appdemo &

# Expose the port
EXPOSE 8888

# Run the application
#CMD ["./appdemo &"]
