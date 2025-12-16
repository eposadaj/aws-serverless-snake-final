# 🛠️ Guía de Configuración Extendida (AWS Comprehend + SNS)

Esta guía detalla los pasos manuales "extras" para activar los servicios de Inteligencia Artificial y Notificaciones.

**IMPORTANTE:** El código del juego YA incluye la lógica. Solo falta dar los permisos en AWS para que no dé errores en el log.

---

## 🚀 Paso 1: Activar Amazon SNS (Notificaciones)

Este servicio enviará alertas cuando termine una partida.

1.  Ve a la **Consola de AWS** y busca el servicio **Simple Notification Service (SNS)**.
2.  En el menú lateral, clic en **Temas** (Topics) -> **Crear tema**.
3.  Selecciona tipo **Estándar**.
4.  Nombre: `GameEvents` (Exactamente así).
5.  Dale a **Crear tema**.
6.  Copia el **ARN** del tema que acabas de crear (empieza por `arn:aws:sns:us-east-1...`).
    *   *Nota: Si es diferente a `arn:aws:sns:us-east-1:420660210592:GameEvents`, avísanos para ajustar el código, pero usualmente solo cambia el ID de cuenta.*

### Suscribirse al Tema (Para ver que funciona)
1.  Dentro del tema `GameEvents`, clic en **Crear suscripción**.
2.  Protocolo: **Email**.
3.  Punto de enlace: Tu correo electrónico universitario.
4.  Clic en **Crear suscripción**.
5.  ⚠️ **¡IMPORTANTE!** Ve a tu bandeja de entrada y confirma el correo de AWS. Si no confirmas, no llegará nada.

---

## 🧠 Paso 2: Permisos de IAM (La Llave Maestra)

Las Lambdas necesitan permiso para usar estos nuevos juguetes.

1.  Ve a la consola **IAM** -> **Roles**.
2.  Busca el rol de tu Lambda. Probablemente se llame `GameLoopLambdaRole` o `GameMatchmakerRole`.
    *   *Truco: Si usaste el mismo rol para todas, solo tienes que hacer esto una vez.*
3.  Clic en el Rol -> Botón **Agregar permisos** -> **Asociar políticas**.
4.  Busca y selecciona estas dos políticas (casilla a la izquierda):
    *   `ComprehendFullAccess` (Para leer sentimientos de nicks).
    *   `AmazonSNSFullAccess` (Para enviar alertas de Game Over).
5.  Clic en **Agregar permisos**.

---

## ✅ Verificación

1.  Juega una partida de Snake.
2.  Al terminar (Choque de ambos jugadores o Game Over):
    *   Deberías recibir un **Correo Electrónico** con el asunto "AWS Snake - Game Over Report".
3.  En los Logs de CloudWatch (`gameConnectionHandler`):
    *   Deberías ver una línea que dice: `Analizando jugador Player_XXXX: Sentimiento NEUTRAL`.

¡Listo! Con esto tienes **7 Servicios AWS** integrados y funcionales. 🌟
