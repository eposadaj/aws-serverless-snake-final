# 🕵️‍♂️ GUÍA FASE 1: El Detector de Mentiras (Paso a Paso)

Si el juego no conecta, no sabemos si es culpa del Navegador (Chrome) o de AWS.
Esta prueba elimina al navegador de la ecuación. Si esto funciona, tu navegador es el culpable. Si esto falla, AWS es el culpable.

---

## 🛠️ Paso 1: Preparar la Herramienta

Necesitamos instalar una pequeña librería de Python para que tu computador pueda hablar "Idioma WebSocket".

1.  Abre tu **Terminal** (Consola de comandos).
2.  Copia y pega este comando y dale Enter:
    ```bash
    pip install websocket-client
    ```
    *(Si te sale error de permisos, prueba: `pip3 install websocket-client` o `sudo pip3 install websocket-client`).*

---

## 🚀 Paso 2: Ejecutar el Test

Yo ya dejé el script listo en tu carpeta. Se llama `debug_client.py`.

1.  En la misma terminal, asegúrate de estar en la carpeta del proyecto.
2.  Ejecuta este comando exacto:
    ```bash
    python3 /home/edwin/.gemini/antigravity/playground/quantum-astro/debug_client.py
    ```

---

## 📝 Paso 3: Interpretar el Resultado

Mira lo que sale en la pantalla negra de la terminal.

### 🔴 ESCENARIO A: Falla Total
Si ves mensajes en rojo, `Error`, `Connection Refused` o `403 Forbidden`.
*   **Significado:** La puerta está cerrada con llave.
*   **Culpable:** API Gateway.
*   **Acción:** Revisa que hayas hecho el **Deploy API** en la etapa `prod`.

### 🟢 ESCENARIO B: Éxito
Si ves algo como:
```text
[CONECTADO] WebSocket abierto con éxito.
[CLIENTE ENVIANDO]: {"action": "joinQueue"}
[SERVIDOR DICE]: Joined Queue
```
*   **Significado:** ¡La tubería de AWS funciona perfecta!
*   **Culpable:** Tu archivo `index.html` o el navegador.
*   **Acción:** El problema está en el Javascript del cliente (quizás la URL está mal copiada o hay caché).

---

**¡Haz la prueba y dime si te salió A o B!**
