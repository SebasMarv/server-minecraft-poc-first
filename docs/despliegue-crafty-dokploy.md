# Despliegue de Crafty Controller 4 en Dokploy

## Requisitos Previos

- VPS con Dokploy instalado
- Traefik configurado (viene con Dokploy)
- Dominio apuntando a la VPS
- Puertos abiertos: 25565 (TCP), 19132 (UDP)

## Estructura de Archivos

```
.
├── docker-compose.yml
├── .env
├── traefik/
│   └── dynamic/
│       └── crafty.yml
└── crafty/
    ├── backups/
    ├── logs/
    ├── servers/
    ├── config/
    └── import/
```

## Pasos de Instalación

### 1. Configurar Variables de Entorno

Copia el archivo de ejemplo y configura tu dominio:

```bash
cp .env.example .env
nano .env
```

Edita `CRAFTY_DOMAIN` con tu dominio real:
```
CRAFTY_DOMAIN=minecraft.tudominio.com
```

### 2. Configurar Traefik para Crafty

Crafty usa HTTPS internamente con certificado auto-firmado. Necesitas agregar la configuración de `serversTransport` a Traefik.

**Opción A: Si usas Traefik de Dokploy**

Agrega esto a la configuración dinámica de Traefik en Dokploy:

```yaml
http:
  serversTransports:
    crafty-transport:
      insecureSkipVerify: true
```

**Opción B: Si tienes tu propio Traefik**

Copia el archivo `traefik/dynamic/crafty.yml` a tu directorio de configuración dinámica de Traefik.

### 3. Crear la Red de Traefik (si no existe)

```bash
docker network create traefik-public
```

### 4. Desplegar

```bash
docker compose up -d
```

### 5. Obtener Credenciales Iniciales

Las credenciales se generan automáticamente en el primer inicio:

```bash
docker exec crafty cat /crafty/app/config/default-creds.txt
```

O accede al volumen:
```bash
cat ./crafty/config/default-creds.txt
```

## Acceso

- **Panel Web:** `https://minecraft.tudominio.com`
- **Servidor Minecraft Java:** `tudominio.com:25565`
- **Servidor Minecraft Bedrock:** `tudominio.com:19132`

## Puertos Necesarios en Firewall

| Puerto | Protocolo | Uso |
|--------|-----------|-----|
| 443 | TCP | Panel Web (via Traefik) |
| 25565 | TCP | Minecraft Java Edition |
| 19132 | UDP | Minecraft Bedrock Edition |

## Comandos Útiles

```bash
# Ver logs
docker compose logs -f crafty

# Reiniciar
docker compose restart crafty

# Actualizar imagen
docker compose pull && docker compose up -d

# Backup manual
docker exec crafty /crafty/backup.sh
```

## Notas Importantes

1. **Primera vez:** Después del primer inicio, accede al panel y cambia la contraseña por defecto.

2. **Servidores adicionales:** Si necesitas más puertos para servidores adicionales, agrégalos al `docker-compose.yml`:
   ```yaml
   ports:
     - "25566:25566"  # Segundo servidor
     - "25567:25567"  # Tercer servidor
   ```

3. **Backups:** Los backups automáticos se guardan en `./crafty/backups/`. Configura backups externos para mayor seguridad.

4. **Recursos:** Ajusta los límites de memoria según tu VPS:
   ```yaml
   deploy:
     resources:
       limits:
         memory: 4G
   ```
