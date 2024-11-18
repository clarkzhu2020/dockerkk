# Use official Go image
FROM golang:latest

# Set the working directory
WORKDIR /app

# Copy and install dependencies
COPY go.mod go.sum ./
RUN go mod download

# Copy the source code
COPY . .

# Build the application
RUN go build -o hertz-app main.go

# Expose the port
EXPOSE 8888

# Run the application
CMD ["./hertz-app"]
