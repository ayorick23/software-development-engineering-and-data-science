---
Fecha de creación: 2026-03-23T19:56:00
Materia:
  - Programación Web I (Backend)
Fecha de clase: 2026-03-23
---
[[Clase 09 - SQL Injection y Patrones de Diseño|← Clase anterior]]

# Autenticación en .NET 8
(la autenticación con Identity y JWT ya se cubrió a fondo en [[Clase 07 - Autenticación y Autorización|Clase 07]] y [[Clase 08 - Cifrado, Hashing, Salting y Arquitectura Limpia#Autenticación Segura con JWT en .NET 8|Clase 08]]; esta clase se enfoca en exponer y probar esa API localmente)

## Fundamentos de ngrok

**ngrok** es una herramienta que crea un **túnel público temporal** hacia un servidor que corre en `localhost`, asignándole una URL pública (ej. `https://a1b2c3d4.ngrok-free.app`) que reenvía el tráfico a tu máquina local.

### ¿Para qué sirve en el contexto de autenticación?

- **Probar callbacks de OAuth2** (login con Google, Microsoft, etc.): estos proveedores externos exigen una URL pública de retorno (_redirect URI_), que `localhost` no puede ofrecer directamente.
- **Probar webhooks:** servicios de pago (Stripe, PayPal) o notificaciones necesitan enviar peticiones HTTP a tu API desde internet, no desde tu propia red local.
- **Compartir un endpoint en desarrollo** con otro dispositivo (un celular real, un compañero de equipo) sin desplegar nada a producción.

### Uso Básico

```bash
# Instala ngrok, luego expón el puerto donde corre tu API .NET
ngrok http https://localhost:7071
```

Esto genera una URL pública que reenvía cada petición entrante hacia `https://localhost:7071`, permitiendo que un proveedor OAuth2 externo, o un servicio de webhooks, alcance tu API en desarrollo como si estuviera desplegada.

>[!IMPORTANT] La URL de ngrok cambia cada vez que reinicias el túnel (en el plan gratuito), por lo que hay que actualizar la configuración de redirect URI del proveedor OAuth2 cada vez.
