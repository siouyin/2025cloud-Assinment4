# 自動產生 Docker Image 與 Tag 流程說明

專案透過 GitHub Actions 實現自動化流程，能夠在每次程式碼變更時自動完成：
- Docker Image 的打包
- 推送至 Docker Hub 並自動上傳對應的 tag

---

### 自動化流程邏輯（含產生 Image 條件）

1. 當 `main` 分支發生以下任一情況時，會觸發 GitHub Action：
   - `push`
   - 建立 Pull Request

2. Action 自動執行下列流程：
   - 讀取 `.github/workflows/docker-build.yml`
   - 分別執行以下步驟：
     - `docker build -t app1 ./app1`
     - `docker build -t app2 ./app2`
     - 登入 Docker Hub（透過 Secret 儲存帳號密碼）
     - 將建好的 Image Tag 並 Push：
       - `docker tag app1 siouyin/2025cloud:app1`
       - `docker tag app2 siouyin/2025cloud:app2`
       - `docker push ...`

---

### Tag 設計邏輯

- 為了區分不同應用程式，Image 的 tag 採用 `app1`、`app2` 命名
- 每一個子目錄（如：`app1/`, `app2/`）都會對應一個 Image
- 未來若有版本需求，可進一步採用 `app1-v1`, `app2-v2` 的方式管理

---

### 錯誤處理與測試設計

- 除了正常流程，也設計了一個「故意失敗的 Pull Request」
- 將 `Dockerfile` 中的 `FROM` 改為錯誤來源，驗證 CI/CD 能正確中斷
- 失敗紀錄會顯示於 PR 中，作為測試與回歸驗證的一環

# 專案詳細步驟及畫面

## 第一部分：Docker Hub

### 1-1 建立一個 Docker Hub Repo，命名為 `2025cloud`

### **操作步驟：**

- 打開瀏覽器前往 https://hub.docker.com
- 登入 Docker Hub 帳號

![Image](https://github.com/user-attachments/assets/ac2dfb1e-e603-4664-b959-69286a2f1d69)

Docker Hub登入畫面

- 點選右上角的 ➕ → `Create Repository`
- 填寫：
    - **Repository Name**: `2025cloud`
    - **Visibility**: **Public**
    - **Description**: `For assignment`
    
    ![Image](https://github.com/user-attachments/assets/92b6ded6-07af-4791-8546-31b80b351a51)
    
    Create Repository 畫面
    

完成後會看到網址為 [`https://hub.docker.com/repository/docker/siouyin(我的帳號)/2025cloud`](https://hub.docker.com/repository/docker/siouyin/2025cloud/general)

---

### 1-2 專案內有兩個以上的 Container Image

### 檔案架構設計

- 建立資料夾與檔案：

```
2025cloud-project/── app1/
│   ├── Dockerfile
│   └── app.py
├── app2/
│   ├── Dockerfile
│   └── main.js
```

---

### App1 - Python（`app1/Dockerfile`）

- 1. 建立 Python App（`app.py`）：

```python
# app1/app.py
print("Hello from App 1")
```

- 2. 建立 Dockerfile（`Dockerfile`）：

```
# app1/Dockerfile
FROM python:3.10-slim
COPY app.py .
CMD ["python", "app.py"]
```

---

### App2 - Node.js（`app2/Dockerfile`）

- 1. 建立 JS App（`main.js`）：

```
// app2/main.js
console.log("Hello from App 2")
```

- 2. 建立 Dockerfile：

```
# app2/Dockerfile
FROM node:18-slim
COPY main.js .
CMD ["node", "main.js"]
```

---

### Build Image 並 Push 到 Docker Hub

```bash
# 登入 Docker Hub
docker login

# Build App1 的 image
docker build -t siouyin/2025cloud:app1 ./app1
# 推上去
docker push siouyin/2025cloud:app1

# Build App2 的 image
docker build -t siouyin/2025cloud:app2 ./app2
# 推上去
docker push siouyin/2025cloud:app2
```

app2同上

---

### 完成確認

- Docker Hub 的 `siouyin/2025cloud` 頁面，會看到兩個 Image：`app1,app2`

![Image](https://github.com/user-attachments/assets/d9a368a2-b2de-4fcc-8103-87a783082e78)

到Docker Desktop也可以看到創建結果

![Image](https://github.com/user-attachments/assets/7d544ece-5a0e-4653-9779-f474f5feeee7)

---

## 第二部分：Git Repo + README + Dockerfile

### 2-1 GitHub 專案有 Dockerfile ( LINK: https://github.com/siouyin/2025cloud-Assinment4)

- **操作步驟：**
1. 到 GitHub 建立一個 Repo，命名為 `2025cloud-Assignment4`
2. 在先前創建的專案資料夾 `2025cloud-project` （有Dockerfile)的Terminal執行指令，將他push到GitHub上：

```bash
# 初始化 Git
git init

# 新增所有檔案到 Git 暫存區
git add .

# 做第一次 Commit
git commit -m "Initial commit with Dockerfiles, apps and README"

# 設定 GitHub 上的遠端倉庫
git remote add origin https://github.com/siouyin/2025cloud-Assignment4.git

# 將本地程式碼推送上 GitHub
git branch -M main
git push -u origin main
```

Terminal 結果：

```jsx
Enumerating objects: 11, done.
Counting objects: 100% (11/11), done.
Delta compression using up to 8 threads
Compressing objects: 100% (9/9), done.
Writing objects: 100% (11/11), 1.22 KiB | 1.22 MiB/s, done.
Total 11 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), done.
To https://github.com/siouyin/2025cloud-Assinment4.git
```

1. 確保 commit 完後 GitHub 上有 Dockerfile 

![Image](https://github.com/user-attachments/assets/d188f2bc-66dc-4ee5-a6b0-e931e7938a87)

---

### 2-2 README 清楚說明 `docker build` 的流程

在 `README.md` 中加入：

```markdown
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
### 執行時會分別輸出：
App1: Hello from App 1

App2: Hello from App 2


### 2-3：在 GitHub 上建立一個 Issue

1. `New Issue`
2. 填寫標題與內容：
    - **Title**: `docker build 操作說明`
    - **Body**:
        
        ```
        我已經在 README.md 裡補充了如何使用 docker build 打包 app1 與 app2。
        ```
        

確保該 issue 是「open」狀態

![Image](https://github.com/user-attachments/assets/0ace8186-98e4-4759-b8f3-2f681b9496bc)

---

## 第三部分：GitHub Action 整合

### 3-1 自動執行 Docker Build

1. 在 GitHub Repo 中新增 `.github/workflows/docker-build.yml`
2. 內容：

```yaml
name: Docker Build

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build App1
        run: docker build -t app1 ./app1
      - name: Build App2
        run: docker build -t app2 ./app2
```

- GitHub Action 自動 Build 成功

GH Action Link: https://github.com/siouyin/2025cloud-Assinment4/actions/runs/14882362596

### 3-2 自動推送至 DockerHub/2025cloud

### 第一步：在 GitHub 專案設定 Secrets

- `DOCKER_USERNAME` → Docker Hub 帳號
- `DOCKER_PASSWORD` → Access Token

### 第二步：修改 GitHub Action：

```yaml
      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin

      - name: Push App1 Image
        run: |
          docker tag app1 ${{ secrets.DOCKER_USERNAME }}/2025cloud:app1
          docker push ${{ secrets.DOCKER_USERNAME }}/2025cloud:app1

      - name: Push App2 Image
        run: |
          docker tag app2 ${{ secrets.DOCKER_USERNAME }}/2025cloud:app2
          docker push ${{ secrets.DOCKER_USERNAME }}/2025cloud:app2
```

成功自動執行 Docker Push 將產生好的 Image 推到前述的 2025cloud repo

https://github.com/siouyin/2025cloud-Assinment4/actions/runs/14882860220/job/41794860289

在dockerhub出現兩個新push的tags: app1, app2

![Image](https://github.com/user-attachments/assets/083c736f-7d5b-49dc-a6de-80b582d5e504)

https://hub.docker.com/repository/docker/siouyin/2025cloud/tags

---

### 3-3 故意讓 GitHub Action 失敗的 PR

1. 開一個 branch（`bug-dockerfile`）

```bash
git checkout -b bug-dockerfile
```

1. 修改 `app1/Dockerfile`：

```
FROM unknowbaseimage
```

完成畫面：

![Image](https://github.com/user-attachments/assets/033f2fbb-f3af-43c6-952b-8d14f5406ecd)
