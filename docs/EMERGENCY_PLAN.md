# 🚨 Protocolo de Emergencia: Conexión Fallida (joinQueue)

Este documento es una lista de verificación paso a paso para identificar POR QUÉ el mensaje del cliente no llega al servidor. **No saltarse ningún paso.**

## 🕵️‍♂️ Fase 1: El Cliente (Frontend)
El objetivo es confirmar que el mensaje sale del navegador.

- [ ] **1. Abrir Consola del Navegador:** (F12 -> Console).
- [ ] **2. Verificar Envío:** Al hacer clic en "Unirse", ¿aparece algún error rojo en la consola?
- [ ] **3. Verificar Network:** Ve a la pestaña **Network** -> Filtra por **WS** (WebSockets).
    - [ ] Selecciona la conexión activa (suele decir `prod` o similar).
    - [ ] Ve a la pestaña **Messages** (o Frames).
    - [ ] Dale clic a "Unirse". ¿Ves una flecha verde ⬆️ saliendo con `{"action":"joinQueue"}`?
    - *Si esto no ocurre, el JS del frontend está roto. Si ocurre, el problema es AWS.*

## 🛣️ Fase 2: El Puente (API Gateway) -> **SOSPECHOSO PRINCIPAL**
El objetivo es confirmar que la ruta existe y está desplegada.

- [ ] **1. Ruta `joinQueue`:**
    - [ ] Ir a API Gateway -> Routes.
    - [ ] ¿Existe `joinQueue`? (Debe ser exacta, case-sensitive).
    - [ ] ¿Tiene una "Integration Request"?
    - [ ] ¿La integración apunta a `gameConnectionHandler`?
- [ ] **2. Ruta `$default` (El comodín):**
    - [ ] Si `joinQueue` falla, debería caer aquí. ¿A dónde apunta `$default`?
- [ ] **3. EL DESPLIEGUE (DEPLOY):**
    - [ ] ¿Hiciste clic en **Actions -> Deploy API** DESPUÉS de crear la ruta?
    - [ ] ¿Elegiste el Stage `prod`?
    - *Sin esto, los cambios NO existen para el mundo real.*

## 🧠 Fase 3: El Cerebro (Lambda)
El objetivo es confirmar que el código nuevo está activo.

- [ ] **1. Código Desplegado:**
    - [ ] Ir a Lambda (`gameConnectionHandler`).
    - [ ] Ver pestaña **Code**.
    - [ ] ¿Ves la línea `print(f"Endpoint recibido: {route_key}...")` al principio?
    - [ ] Si la ves, ¿está el botón "Deploy" gris (inhabilitado) o naranja/azul? (Debe estar presionado).
- [ ] **2. Logs (CloudWatch):**
    - [ ] Asegurarse de estar viendo el "Log Stream" más reciente (hora actual).
    - [ ] Buscar la cadena `Endpoint recibido`.

## 🛠️ Acción Correctiva Inmediata (Plan B)

Si todo lo anterior parece estar bien y sigue fallando:

1.  **Borrar la ruta `joinQueue`**.
2.  **Volver a crearla** desde cero.
3.  Asociarla de nuevo a la Lambda.
4.  **Hacer Deploy a un NUEVO Stage** (ej: `v2`).
5.  Cambiar la URL en el Frontend (`index.html`) para apuntar a `v2`.
    *   *Esto descarta problemas de caché o "pegado" en el stage `prod`.*

---
*Seguir este orden exacto.*
