# 🐍 AWS Serverless Snake: Real-Time Multiplayer Architecture

![AWS](https://img.shields.io/badge/AWS-Serverless-orange?style=for-the-badge&logo=amazon-aws)
![Python](https://img.shields.io/badge/Backend-Python-blue?style=for-the-badge&logo=python)
![DynamoDB](https://img.shields.io/badge/Database-DynamoDB-4053D6?style=for-the-badge&logo=amazon-dynamodb)
![Status](https://img.shields.io/badge/Status-Operational-green?style=for-the-badge)

## 📖 Descripción General
**AWS Serverless Snake** es un videojuego multijugador masivo en tiempo real (MMO-lite) construido **100% sobre arquitectura Serverless**. 
A diferencia de los juegos tradicionales que requieren servidores dedicados (EC2) corriendo las 24 horas, este proyecto utiliza un enfoque **Event-Driven** donde la infraestructura solo existe cuando hay jugadores activos, reduciendo los costos de operación a casi cero cuando no hay tráfico.

### 🌟 Características Principales
*   **Multijugador Sincronizado:** 2 jugadores compiten en el mismo tablero en tiempo real.
*   **Matchmaking Dinámico:** Sistema de Lobby inteligente que agrupa jugadores automáticamente.
*   **Cost-Efficient:** Arquitectura "Scale-to-Zero". No pagas si nadie juega.
*   **Game Loop Persistente:** Motor de juego capaz de mantener estado continuo sin servidores fijos.
*   **Mecánicas Competitivas:** Colisiones, Puntuación, y Detección de Abandono.

---

## 🏗️ Arquitectura Técnica

El sistema utiliza un patrón **API Gateway WebSocket + Lambda Loop**.

```mermaid
graph TD
    User((Jugador)) -->|WebSocket Connect| APIG[API Gateway (WSS)]
    
    subgraph "Event Layer"
        APIG -->|Connect/Disconnect| ConnectionHandler[λ ConnectionHandler]
        APIG -->|Input Action| DynamoDB
    end
    
    subgraph "Matchmaking Layer"
        ConnectionHandler -->|Join Queue| SQS[Amazon SQS]
        SQS -->|Trigger| Matchmaker[λ Matchmaker]
        Matchmaker -->|Create Match| DynamoMatches[(DynamoDB Matches)]
    end
    
    subgraph "Game Engine Layer"
        Matchmaker -->|Invoke Async| GameLoop[λ GameLoop Handler]
        GameLoop <-->|Read Inputs / Write State| DynamoMatches
        GameLoop -->|Broadcast State (200ms)| APIG
        APIG -->|Update UI| User
    end
```

### COMPONENTES

| Componente | Rol en el Sistema |
| :--- | :--- |
| **API Gateway (WebSocket)** | Mantiene la conexión persistente full-duplex con los navegadores. |
| **λ ConnectionHandler** | Recibe conexiones y solicitudes de entrada al Lobby. Gatilla el Matchmaker. |
| **λ Matchmaker** | Escanea la "Sala de Espera", agrupa jugadores en pares y lanza la partida. |
| **λ GameLoopHandler** | **El Corazón del Juego.** Ejecuta un bucle continuo (600s) que procesa física, colisiones y estado. |
| **DynamoDB** | Base de datos de ultra-baja latencia para almacenar el Estado del Juego y Inputs. |

---

## 📂 Estructura del Proyecto

```text
/aws-serverless-snake
│
├── /backend            # Código Fuente de las Lambdas (Python)
│   ├── gameConnectionHandler.py   # Gestión de WebSockets
│   ├── gameMatchmaker.py          # Lógica de Emparejamiento
│   └── gameLoopHandler.py         # Motor de Física y Estado
│
├── /frontend           # Cliente Web
│   └── index.html                 # Juego completo (HTML5 Canvas + JS)
│
└── /docs               # Documentación y Diagramas
```

---

## 🚀 Despliegue (Cómo instalarlo)

### Requisitos Previos
*   Cuenta de AWS activa.
*   Rol de IAM con permisos para Lambda, DynamoDB y API Gateway.

### Paso 1: Backend
1.  Crear una tabla en **DynamoDB** llamada `GameMatches` (Partition Key: `matchId`).
2.  Crear una API WebSocket en **API Gateway**.
3.  Desplegar las 3 funciones Lambda ubicadas en `/backend`.
    *   **Importante:** Aumentar el Timeout de `gameLoopHandler` a **10 minutos**.
4.  Configurar las rutas de la API (`$connect`, `$disconnect`, `joinQueue`, `playerMove`) para apuntar a las Lambdas correspondientes.

### Paso 2: Frontend
1.  Editar `/frontend/index.html` y actualizar la variable `WSS_URL` con tu endpoint de API Gateway.
2.  Subir el archivo a un **S3 Bucket** configurado como Website Hosting Público.

---

## 🎮 Instrucciones de Juego
1.  Entra a la URL proporcionada.
2.  Haz clic en **"Unirse a la Cola"**.
3.  Espera a un oponente (o abre una pestaña de Incógnito para jugar contra ti mismo).
4.  Usa las **Flechas del Teclado** para moverte.
5.  Come los ratones grises 🐭 para crecer.
6.  ¡Evita chocar! El primero en llegar a **500 Puntos** GANA. 🏆

---

## 👥 Créditos y Autores
Este proyecto fue diseñado e implementado como parte del curso de Computación en la Nube.

*   **Santiago**
*   **Andru**
*   **Antigravity AI**

---
*Hecho con ❤️ y Python.*
