# AWS Serverless Snake: Real-Time Multiplayer Architecture

![AWS](https://img.shields.io/badge/AWS-Serverless-orange?style=for-the-badge&logo=amazon-aws)
![Python](https://img.shields.io/badge/Backend-Python-blue?style=for-the-badge&logo=python)
![DynamoDB](https://img.shields.io/badge/Database-DynamoDB-4053D6?style=for-the-badge&logo=amazon-dynamodb)
![Status](https://img.shields.io/badge/Status-Operational-green?style=for-the-badge)

## Project Overview

**AWS Serverless Snake** is a real-time multiplayer online game (MMO-lite) engineered entirely on a Serverless architecture.

Unlike traditional multiplayer games that rely on dedicated fleets of EC2 instances running 24/7, this project demonstrates an **Event-Driven** approach. The infrastructure provisions resources dynamically only when active gameplay occurs, resulting in a "Scale-to-Zero" cost model with high operational efficiency.

### Key Features

*   **Real-Time Synchronization:** Two-player competitive environment with sub-second latency synchronization.
*   **Dynamic Matchmaking:** Automated lobby system utilizing asynchronous queues for player grouping.
*   **Cost Efficiency:** Fully serverless backend that incurs zero cost during idle periods.
*   **Stateless Game Loop:** innovative game engine design that maintains continuous state without persistent compute instances.
*   **Competitive Mechanics:** Implements server-side validation for physics, collisions, and scoring to prevent cheating.

---

## Technical Architecture

The system implements a **WebSocket API Gateway pattern** integrated with a **Lambda-based Game Loop**.

### Architecture Diagram

The data flow ensures low-latency communication while maintaining data integrity via high-performance NoSQL storage.

```mermaid
graph TD
    User((Player)) -->|WebSocket Connect| APIG[API Gateway (WSS)]
    
    subgraph "Event Layer"
        APIG -->|Connect/Disconnect| ConnectionHandler[Lambda ConnectionHandler]
        APIG -->|Input Action| DynamoDB
    end
    
    subgraph "Matchmaking Layer"
        ConnectionHandler -->|Join Queue| SQS[Amazon SQS]
        SQS -->|Trigger| Matchmaker[Lambda Matchmaker]
        Matchmaker -->|Create Match| DynamoMatches[(DynamoDB Matches)]
    end
    
    subgraph "Game Engine Layer"
        Matchmaker -->|Invoke Async| GameLoop[Lambda GameLoop Handler]
        GameLoop <-->|Read Inputs / Write State| DynamoMatches
        GameLoop -->|Broadcast State (200ms)| APIG
        APIG -->|Update UI| User
    end
```

### Component Breakdown

| Component | Functionality |
| :--- | :--- |
| **API Gateway (WebSocket)** | Manages persistent full-duplex connections between clients and the backend. |
| **Lambda ConnectionHandler** | Handles connection lifecycle events and queues join requests. |
| **Lambda Matchmaker** | Processes the SQS queue to group players and initializes match state in the database. |
| **Lambda GameLoopHandler** | **Core Engine.** Executes the physics simulation and state updates in a recursive cycle. |
| **DynamoDB** | Provides ultra-low latency storage for game state and player inputs, acting as the "memory" for the stateless functions. |

---

## Project Structure

The repository is organized into backend logic, frontend client code, and documentation.

```text
/aws-serverless-snake
│
├── /backend            # Serverless Functions (Python)
│   ├── gameConnectionHandler.py   # WebSocket Lifecycle Management
│   ├── gameMatchmaker.py          # Matchmaking Logic & Queue Processing
│   └── gameLoopHandler.py         # Physics Engine & State Broadcast
│
├── /frontend           # Client Application
│   └── index.html                 # Single Page Application (HTML5 Canvas + JS)
│
└── /docs               # supplementary Documentation
```

---

## Deployment Instructions

### Prerequisites

*   Active AWS Account.
*   IAM Role configured with least-privilege permissions for Lambda, DynamoDB, and API Gateway execution.

### Step 1: Backend Deployment

1.  **Database:** Create a **DynamoDB** table named `GameMatches` with `matchId` (String) as the Partition Key.
2.  **API:** Deploy a WebSocket API in **API Gateway**.
3.  **Functions:** Deploy the three Lambda functions found in the `/backend` directory.
    *   *Configuration Note:* Ensure the `gameLoopHandler` timeout is set to **10 minutes** (600 seconds) to accommodate full match duration.
4.  **Routing:** Configure the API Gateway routes (`$connect`, `$disconnect`, `joinQueue`, `playerMove`) to target the respective Lambda functions.

### Step 2: Frontend Configuration

1.  Open `/frontend/index.html`.
2.  Update the `WSS_URL` constant with your deployed API Gateway WebSocket Endpoint.
3.  Deploy the client file to an S3 Bucket configured for Static Website Hosting, or run locally for testing.

---

## Usage Guide

1.  Navigate to the deployed client URL.
2.  Select **"Join Queue"** to enter the matchmaking pool.
3.  Calculated waiting time depends on active player count (or use a secondary browser tab to simulate an opponent).
4.  **Controls:** Use the Arrow Keys to direct the snake.
5.  **Objective:** Consume targets to increase length and score. Avoid collisions with walls and opponents.
6.  **Victory Condition:** Reach 500 points to terminate the match.

---

## Acknowledgments

This project was architected and implemented as a capstone for the Cloud Computing curriculum.

**Authors:**
*   Santiago
*   Andru
*   Antigravity AI

---

*Implemented in Python.*
