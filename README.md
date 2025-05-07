# 2025cloud Assignment 4

本專案包含兩個應用程式，分別以 Python 與 Node.js 製作，並透過 Docker 進行打包與運行。

---

## 🐳 Docker Build 指令

### App1 (Python)
```bash
docker build -t siouyin/2025cloud:app1 ./app1
```
### App2 (Node.js)
```bash
docker build -t siouyin/2025cloud:app2 ./app2
```

## ▶️ Docker Run 指令

### 執行 App1
```bash
docker run --rm siouyin/2025cloud:app1
```
### 執行 App2
```bash
docker run --rm siouyin/2025cloud:app2
```
💡 說明:

打包完成後的 Docker image 可透過上述指令運行，並分別輸出「Hello from App1」與「Hello from App2」。
