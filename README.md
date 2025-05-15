# 🚀 Despliegue Automático de FastAPI con GitHub Actions y Runner Self-Hosted

Este repositorio contiene un ejemplo funcional para probar el despliegue continuo de una API FastAPI con Docker Compose en un servidor local, utilizando un runner self-hosted y GitHub Actions.

---

## 📦 Estructura del Proyecto

```
test_actions/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── app/
│   └── main.py
├── Dockerfile
├── docker-compose.yml
└── README.md
```

---

## ✅ Paso 1: Crear la API FastAPI

### `app/main.py`

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello World"}
```

> Luego puedes modificar este mensaje por "Hello Aplicaciones" para testear el flujo completo.

---

## 🐳 Paso 2: Crear el Dockerfile

### `Dockerfile`

```Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY app/ /app/
RUN pip install fastapi uvicorn

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## ⚙️ Paso 3: Crear `docker-compose.yml`

```yaml
version: '3.8'

services:
  fastapi:
    build: .
    container_name: fastapi_app
    ports:
      - "8000:8000"
```

---

## 🤖 Paso 4: Configurar GitHub Actions

### `.github/workflows/deploy.yml`

```yaml
name: Deploy FastAPI App on Merge to Develop

on:
  push:
    branches:
      - develop

jobs:
  deploy:
    name: Deploy on local server
    runs-on: self-hosted

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Stop existing container
      run: docker compose down || true

    - name: Build and run new container
      run: docker compose up -d --build

    - name: Notify on Discord
      uses: Ilshidur/action-discord@master
      with:
        webhook: ${{ secrets.DISCORD_WEBHOOK }}
        message: "🚀 FastAPI desplegado en servidor local tras merge en develop."
```

---

## 🧑‍💻 Paso 5: Configurar Runner Self-Hosted

### En tu servidor local ejecuta:

```bash
mkdir -p ~/actions-runner && cd ~/actions-runner
curl -o actions-runner-linux-x64-2.324.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.324.0/actions-runner-linux-x64-2.324.0.tar.gz
echo "e8e24a3477da17040b4d6fa6d34c6ecb9a2879e800aa532518ec21e49e21d7b4  actions-runner-linux-x64-2.324.0.tar.gz" | shasum -a 256 -c
tar xzf actions-runner-linux-x64-2.324.0.tar.gz
./config.sh --url https://github.com/danunziata/test_actions --token TU_TOKEN

# Instalar como servicio
sudo ./svc.sh install
sudo ./svc.sh start
```

> Podés obtener TU\_TOKEN desde: GitHub > Settings > Actions > Runners > Add runner

---

## 🔐 Paso 6: Configurar Secrets en el repositorio

Ir a GitHub > Settings > Secrets and variables > Actions y agregar:

* `DISCORD_WEBHOOK`: URL de webhook de Discord.
* `TELEGRAM_TOKEN`: Token de bot de Telegram.
* `TELEGRAM_TO`: ID del chat o usuario destino.

---

## 🚀 Paso 7: Probar el flujo

1. Cambiá `main.py` para que diga:

   ```python
   return {"message": "Hello Aplicaciones"}
   ```

2. Hacé `git checkout -b test-change`, `git push` y creá un **Pull Request** a `develop`.

3. Una vez mergeado, el runner desplegará la app en tu servidor.

4. Verificá en tu navegador:

   ```
   http://IP_DEL_SERVIDOR:8000
   ```

---

## ✅ Resultado esperado

```json
{"message": "Hello Aplicaciones"}
```

---

Listo. Este proyecto ahora despliega FastAPI automáticamente en el servidor al hacer merge en `develop`, y notifica por Discord y Telegram.
