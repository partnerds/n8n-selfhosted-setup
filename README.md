# n8n-selfhosted-setup con Docker
repositorio de n8n selfhosted en linux

Guía real para instalar **n8n** en un servidor Linux y acceder de forma segura desde otro equipo usando **Tailscale**.

Este documento incluye **errores reales** que aparecieron durante la instalación y cómo se solucionaron.

---

## 🧠 Arquitectura

Windows (navegador)  
⬇ vía Tailscale  
Servidor Linux (Docker)  
⬇  
Contenedor n8n (puerto 5678)

---

## 📦 Requisitos

- Servidor Linux (Ubuntu/Debian recomendado)
- Docker y Docker Compose
- Tailscale instalado y conectado
- Puerto interno 5678 disponible

---

## 🐳 Instalación de n8n con Docker

Crear carpeta del proyecto:

```bash
mkdir -p ~/n8n
cd ~/n8n

Crear archivo docker-compose.yml (ver más abajo).

## Levantar contenedor:
docker compose up -d
docker ps
