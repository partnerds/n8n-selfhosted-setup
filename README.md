# 🚀n8n-selfhosted-setup con Docker
repositorio de n8n selfhosted en linux

Guía completa para instalar **n8n** en un servidor Linux usando Docker y acceder de forma segura desde otro equipo a través de Tailscale.

Incluye problemas reales encontrados durante la instalación y sus soluciones.

---

## 🧠 Arquitectura

Windows / Laptop  
⬇ (Red privada Tailscale)  
Servidor Linux  
⬇  
Docker  
⬇  
Contenedor n8n (puerto 5678)

n8n **NO se expone a internet pública**, solo funciona dentro de la red privada de Tailscale.

---

## 📦 Requisitos

- Servidor Linux (Ubuntu/Debian recomendado)
- Acceso SSH al servidor
- Docker y Docker Compose
- Tailscale instalado y conectado en el servidor
- Puerto 5678 libre

---

## 🐳 Paso 1 — Instalar Docker

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Opcional (para no usar sudo siempre):
```bash
sudo usermod -aG docker $USER
newgrp docker
```
## 📁 Paso 2 — Crear carpeta del proyecto

```bash
mkdir -p ~/n8n
cd ~/n8n
```

## 🧾 Paso 3 — Crear docker-compose.yml
```bash
nano docker-compose.yml
```
Pegar el siguiente contenido:

```yaml
services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=0.0.0.0
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - WEBHOOK_URL=http://IP_TAILSCALE:5678/
      - GENERIC_TIMEZONE=America/Santiago
      - N8N_SECURE_COOKIE=false #no necesario si no te sale el error que me aparecio a mi en el video
      - N8N_DIAGNOSTICS_ENABLED=false #no necesario si no te sale el error que me aparecio a mi en el video
      - N8N_VERSION_NOTIFICATIONS_ENABLED=false #no necesario si no te sale el error que me aparecio a mi en el video
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

## Paso 4: Levantar contenedor de n8n:
```bash
docker compose up -d
docker ps
```
## 🌐 Paso 5: Acceso remoto por Tailscale
Obtener IP Tailscale del servidor:

```bash
tailscale ip -4
```

Ejemplo de resultado:
```bash
100.87.240.47
```
Editar el docker-compose.yml y reemplazar:
```yaml
WEBHOOK_URL=http://100.87.240.47:5678/
```

reinicia y sube el contenedor:
```bash
docker compose down
docker compose up -d
```

## 💻 Paso 6 — Acceder desde Windows u otro equipo a través de Tailscale

Abrir en navegador:
```bash
http://IP_TAILSCALE:5678
```


## 🔐 Paso 7 — Primer inicio

n8n pedirá:

- Email
- Nombre
- Apellido
- Contraseña

Esto crea el usuario administrador local, NO es suscripción de pago.

# 🧩 Problemas comunes y soluciones

## ❌ Error: services.n8n.volumes must be an array

Causa: YAML mal indentado

Solución:
```yaml
volumes:
  - n8n_data:/home/node/.n8n
```

## ❌ Error: Setup TLS / Secure Cookie
Causa: Acceso por IP Tailscale (no localhost)

Solución:
```yaml
- N8N_SECURE_COOKIE=false
```

## ❌ Error: Failed to register community edition (429)
Causa: Demasiados intentos de registro

Solución:

- Esperar varias horas
- Desactivar llamadas automáticas:

```yaml
- N8N_DIAGNOSTICS_ENABLED=false
- N8N_VERSION_NOTIFICATIONS_ENABLED=false
```
n8n funciona igual sin esa licencia.

## ❌ Logs parecen “pegados”
Comando usado:
```bash
docker compose logs -f
```
No está congelado, está siguiendo logs en vivo.

Salir con Ctrl + C.

o Salir con Ctrl + Z.

## 🚀 Resultado Final

- ✔ n8n corriendo en Docker
- ✔ Datos persistentes
- ✔ Acceso remoto por Tailscale
- ✔ Sin necesidad de dominio ni HTTPS público
- ✔ Login seguro activado

## 📚 Proyecto oficial

Repositorio oficial: https://github.com/n8n-io/n8n