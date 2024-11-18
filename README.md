# dockerkk
以下是为您的大并发应用设计的完整 `docker-compose.yml` 和各服务的 `Dockerfile` 文件。此配置包括 **Nginx**（反向代理）、**Hertz**（API服务）、**Vue3**（前端）、**MySQL**（关系型数据库）和 **Redis**（缓存）。

---

### **1. `docker-compose.yml`**
```yaml
version: "3.9"

services:
  nginx:
    image: nginx:latest
    container_name: nginx-gateway
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./vue/dist:/usr/share/nginx/html:ro
    depends_on:
      - hertz
    networks:
      - app-network

  hertz:
    build:
      context: ./hertz
      dockerfile: Dockerfile
    container_name: hertz-api
    ports:
      - "8080:8080"
    networks:
      - app-network

  mysql:
    image: mysql:8.0
    container_name: mysql-db
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: appdb
      MYSQL_USER: appuser
      MYSQL_PASSWORD: apppassword
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - app-network

  redis:
    image: redis:latest
    container_name: redis-cache
    ports:
      - "6379:6379"
    networks:
      - app-network

  vue:
    build:
      context: ./vue
      dockerfile: Dockerfile
    container_name: vue-frontend
    networks:
      - app-network

volumes:
  mysql-data:

networks:
  app-network:
    driver: bridge
```

---

### **2. Nginx 配置 (`nginx/nginx.conf`)**
```nginx
events {}

http {
    upstream hertz_api {
        server hertz-api:8080;
    }

    server {
        listen 80;

        # Serve Vue frontend
        location / {
            root /usr/share/nginx/html;
            index index.html;
            try_files $uri /index.html;
        }

        # Proxy to Hertz API
        location /api/ {
            proxy_pass http://hertz_api/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

---

### **3. Hertz Dockerfile (`hertz/Dockerfile`)**
```dockerfile
# Use official Go image
FROM golang:1.21

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
EXPOSE 8080

# Run the application
CMD ["./hertz-app"]
```

---

### **4. Vue Dockerfile (`vue/Dockerfile`)**
```dockerfile
# Use Node.js to build the Vue project
FROM node:18 AS builder

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
```

---

### **5. 配置 MySQL 和 Redis**
无需额外配置，它们的官方镜像已经预设了常用环境。

---

### **启动环境**
1. 将以上文件存放到合适的目录结构中，例如：
   ```
   ├── docker-compose.yml
   ├── nginx/
   │   └── nginx.conf
   ├── hertz/
   │   ├── Dockerfile
   │   ├── go.mod
   │   ├── go.sum
   │   └── main.go
   ├── vue/
   │   ├── Dockerfile
   │   ├── package.json
   │   ├── package-lock.json
   │   └── src/
   └── mysql-data/
   ```

2. 使用 `docker-compose up --build` 启动所有服务。

---

### **运行结果**
- **Nginx** 监听在 `http://localhost`，处理前端页面和 API 代理请求。
- **Hertz** API 可通过 `http://localhost/api` 访问。
- **Vue3** 前端通过 Nginx 提供服务。
- **MySQL** 数据库可通过 `localhost:3306` 连接。
- **Redis** 缓存服务可通过 `localhost:6379` 连接。

此架构设计优化了大并发访问和动态服务效率，同时使用了 Docker 容器化便于部署和管理。