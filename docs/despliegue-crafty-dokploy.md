# Despliegue de Crafty Controller 4 en Dokploy

## Requisitos Previos

- VPS con Dokploy instalado
- Dominio apuntando a la VPS
- Puertos abiertos en firewall: 25565 (TCP), 19132 (UDP)

## Paso 1: Configurar Traefik en Dokploy

Crafty usa HTTPS interno con certificado auto-firmado. Debes configurar Traefik para aceptarlo.

1. Ve a **Settings → Server → Traefik**
2. En **Traefik Config** agrega esto al final:

```yaml
http:
  serversTransports:
    crafty-transport:
      insecureSkipVerify: true
```

3. Guarda los cambios

## Paso 2: Crear el Compose en Dokploy

1. Crea un nuevo **Compose** en Dokploy
2. Conecta el repositorio de GitHub
3. Despliega

## Paso 3: Configurar Dominio

En la sección **Domains** del compose:

| Campo | Valor |
|-------|-------|
| Service Name | `crafty` |
| Host | `tu-dominio.com` |
| Path | `/` |
| Container Port | `8443` |
| HTTPS | ✅ Activado |

**Importante:** En configuración avanzada del dominio, agrega:

```
serversTransport: crafty-transport
```

O configura manualmente en Traefik Config:

```yaml
http:
  routers:
    crafty-secure:
      rule: Host(`tu-dominio.com`)
      service: crafty-service
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt

  services:
    crafty-service:
      loadBalancer:
        servers:
          - url: https://crafty:8443
        serversTransport: crafty-transport
```

## Paso 4: Obtener Credenciales

Accede por SSH a la VPS y ejecuta:

```bash
docker exec crafty cat /crafty/app/config/default-creds.txt
```

## Acceso

- **Panel Web:** `https://tu-dominio.com`
- **Minecraft Java:** `tu-dominio.com:25565`
- **Minecraft Bedrock:** `tu-dominio.com:19132`

## Puertos en Firewall

| Puerto | Protocolo | Uso |
|--------|-----------|-----|
| 443 | TCP | Panel Web (via Traefik) |
| 25565 | TCP | Minecraft Java Edition |
| 19132 | UDP | Minecraft Bedrock Edition |

## Comandos Útiles

```bash
# Ver logs
docker logs -f crafty

# Ver credenciales
docker exec crafty cat /crafty/app/config/default-creds.txt

# Reiniciar
docker restart crafty
```

## Agregar Más Servidores

Si necesitas más puertos para servidores adicionales, edita el `docker-compose.yml`:

```yaml
ports:
  - "25565:25565"
  - "25566:25566"  # Segundo servidor
  - "25567:25567"  # Tercer servidor
  - "19132:19132/udp"
```
