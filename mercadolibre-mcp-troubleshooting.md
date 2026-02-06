# Guía de Configuración y Solución de Problemas: MercadoLibre MCP

Esta guía documenta los aprendizajes y pasos necesarios para configurar con éxito el servidor MCP de MercadoLibre en Antigravity/Cursor, evitando conflictos de puertos y errores de autenticación.

---

## 1. Configuración de `mcp_config.json`

Para que la herramienta funcione con MercadoLibre, **no** se debe usar un token estático (Bearer) si se desea usar el flujo de renovación automática (OAuth).

### Estructura Correcta
La clave es agregar el puerto **18999** como un argumento adicional después de la URL del servidor. Esto obliga a la herramienta a usar el puerto que MercadoLibre tiene autorizado por defecto.

```json
"mercadolibre": {
  "command": "npx",
  "args": [
    "-y",
    "mcp-remote",
    "https://mcp.mercadolibre.com/mcp",
    "18999" 
  ],
  "disabled": false
}
```

> [!IMPORTANT]
> **NO incluya la cabecera `--header "Authorization: Bearer ..."`** en esta configuración si planea usar el flujo de autorización por navegador. Tener ambos causa conflictos que rompen el intercambio del código de acceso.

---

## 2. El Puerto Crítico: 18999

MercadoLibre tiene pre-autorizada la URI de redirección `http://localhost:18999/oauth/callback` para su cliente oficial de MCP. Si el puerto 18999 está ocupado, la herramienta intentará usar otro puerto aleatorio (ej. 17135), lo cual provocará el error:
> *"La aplicación no está preparada para conectarse a Mercado Libre"*

### Cómo liberar el puerto (Windows)
Si recibes un error de `EADDRINUSE: 18999`, ejecuta estos comandos en la terminal:

1. **Identificar el proceso:**
   ```powershell
   netstat -ano | findstr :18999
   ```
2. **Matar el proceso (usando el PID del comando anterior):**
   ```powershell
   taskkill /PID <PID_AQUÍ> /F
   ```

---

## 3. Limpieza de procesos y caché

Si la autenticación falla repetidamente con errores de "Existing OAuth client information is required", es necesario limpiar los estados corruptos:

1. **Cerrar todos los procesos de Node:**
   ```powershell
   taskkill /F /IM node.exe
   ```
2. **Borrar carpetas de configuración (si existen):**
   - `%APPDATA%\.mcp-auth`
   - `%LOCALAPPDATA%\npm-cache\_npx`

---

## 4. Flujo de Autenticación y Navegadores

- **Navegador**: Aunque Firefox puede dar problemas con el retorno del callback en algunas configuraciones, **sí funciona** si el puerto 18999 está libre. Si falla, Chrome o Edge son alternativas más directas.
- **Rapidez**: Cursor tiene un *timeout* (tiempo límite). Una vez que se abra el navegador, realiza la autorización rápidamente para evitar el error `context deadline exceeded`.
- **Ngrok**: **NO es necesario** para el servidor MCP. La herramienta usa una App oficial de MeLi que ya confía en `localhost`. El túnel solo es necesario para tu aplicación propia (HITO).

---

## Resumen de la Solución "Atómica" para fallos técnicos:
1. Cerrar Cursor.
2. Ejecutar `taskkill /F /IM node.exe`.
3. Borrar la carpeta `C:\Users\<Usuario>\.mcp-auth`.
4. Asegurarse de que el JSON no tenga Headers de Authorization.
5. Reiniciar y Autorizar rápido en el navegador.
