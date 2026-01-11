# Análisis de Paneles de Gestión para Servidor Minecraft con Docker

> Fecha: 2026-01-11
> Decisión: **Crafty Controller 4**

## Resumen Ejecutivo

Este documento analiza las opciones disponibles para levantar un servidor de Minecraft con interfaz de gestión (UI) usando Docker/Docker Compose, orientado al despliegue en Dokploy.

---

## Opciones Analizadas

### 1. Crafty Controller 4 ⭐ (SELECCIONADO)

**Descripción:** Panel de control gratuito y open-source para servidores Minecraft con interfaz web moderna.

**Pros:**
- UI moderna y fácil de usar
- Un solo contenedor (simple de desplegar)
- Backups automáticos integrados
- Soporte multi-servidor desde un solo panel
- Soporte Java y Bedrock Edition
- Imagen oficial soporta arm64 y amd64

**Contras:**
- Limitado solo a Minecraft (no otros juegos)
- Advertencia: No usar en WSL/WSL2 con Docker Desktop

**Docker Compose:**
```yaml
services:
  crafty:
    container_name: crafty
    image: registry.gitlab.com/crafty-controller/crafty-4:latest
    restart: always
    environment:
      - TZ=America/Mexico_City
    ports:
      - "8443:8443"       # Panel HTTPS
      - "25565:25565"     # Minecraft Java
      - "19132:19132/udp" # Minecraft Bedrock
    volumes:
      - ./backups:/crafty/backups
      - ./logs:/crafty/logs
      - ./servers:/crafty/servers
      - ./config:/crafty/app/config
```

**Credenciales por defecto:** Se generan automáticamente en `/config/default-creds.txt`

**Documentación oficial:** https://docs.craftycontrol.com/pages/getting-started/installation/docker/

---

### 2. MCSManager

**Descripción:** Panel distribuido, multi-usuario y moderno para servidores de Minecraft y Steam.

**Pros:**
- Arquitectura distribuida (daemon + web)
- Multi-usuario con permisos granulares
- Soporte Docker nativo para servidores
- Ligero, solo requiere Node.js
- Interfaz disponible en español
- Soporta también servidores de Steam

**Contras:**
- Arquitectura más compleja (daemon separado)
- Menos documentación en español

**Docker Compose:**
```yaml
services:
  mcsm-web:
    image: mcsmanager/mcsmanager:latest
    container_name: mcsmanager
    restart: always
    ports:
      - "23333:23333"  # Panel Web
      - "24444:24444"  # Daemon
      - "25565:25565"  # Minecraft
    volumes:
      - ./data:/opt/mcsmanager/data
```

**Documentación oficial:** https://www.mcsmanager.com/

---

### 3. Pterodactyl

**Descripción:** Panel profesional de gestión de servidores de juegos, usado por empresas de hosting.

**Pros:**
- El más completo y profesional
- Aislamiento total con Docker por servidor
- Ideal para hosting comercial
- "Eggs" pre-configurados para todo tipo de servidores
- Gran comunidad y soporte

**Contras:**
- Requiere 2 componentes: Panel + Wings
- Configuración más compleja
- Necesita Redis, MySQL/MariaDB
- Overhead mayor de recursos

**Docker Compose (simplificado):**
```yaml
services:
  panel:
    image: ghcr.io/pterodactyl/panel:latest
    restart: always
    ports:
      - "80:80"
      - "443:443"
    environment:
      - APP_URL=https://tu-dominio.com
      - DB_HOST=database
    depends_on:
      - database
      - cache

  database:
    image: mariadb:10.11
    environment:
      - MYSQL_ROOT_PASSWORD=secret
      - MYSQL_DATABASE=panel

  cache:
    image: redis:alpine

  wings:
    image: ghcr.io/pterodactyl/wings:latest
    ports:
      - "25565:25565"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

**Documentación oficial:** https://pterodactyl.io/

---

### 4. PufferPanel

**Descripción:** Panel de gestión de servidores de juegos open-source, balance entre simplicidad y funcionalidad.

**Pros:**
- Open source y completamente gratuito
- Soporta múltiples juegos (Minecraft, Source, etc.)
- Más simple que Pterodactyl
- Buena documentación

**Contras:**
- Menos features que Pterodactyl
- Comunidad más pequeña

**Docker Compose:**
```yaml
services:
  pufferpanel:
    image: pufferpanel/pufferpanel:latest
    restart: always
    ports:
      - "8080:8080"
      - "5657:5657"
      - "25565:25565"
    volumes:
      - ./config:/etc/pufferpanel
      - ./servers:/var/lib/pufferpanel/servers
```

**Documentación oficial:** https://docs.pufferpanel.com/

---

## Tabla Comparativa

| Panel | Complejidad | Contenedores | Multi-juego | Recomendación |
|-------|-------------|--------------|-------------|---------------|
| **Crafty 4** | Baja | 1 | No | Mejor para empezar |
| **MCSManager** | Media | 1-2 | Parcial (Steam) | Multi-usuario |
| **PufferPanel** | Media | 1 | Sí | Varios juegos |
| **Pterodactyl** | Alta | 4+ | Sí | Escala comercial |

---

## Decisión Final

**Seleccionado: Crafty Controller 4**

**Justificación:**
1. Simplicidad de despliegue (1 solo contenedor)
2. Ideal para Dokploy
3. UI moderna e intuitiva
4. Todas las funciones necesarias para gestionar servidores Minecraft
5. Backups automáticos incluidos
6. Bajo consumo de recursos

---

## Referencias

- [Crafty Documentation - Docker](https://docs.craftycontrol.com/pages/getting-started/installation/docker/)
- [Crafty Wiki](https://wiki.craftycontrol.com/en/4/Docker%20Setup%20Documentation)
- [MCSManager GitHub](https://github.com/MCSManager/MCSManager)
- [Pterodactyl Official](https://pterodactyl.io/)
- [PufferPanel SpigotMC](https://www.spigotmc.org/resources/pufferpanel.40158/)
- [Awesome Self-Hosted - Game Panels](https://awesome-selfhosted.net/tags/games---administrative-utilities--control-panels.html)
