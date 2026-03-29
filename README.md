# 🏛️ InConQuest — Gamified Constitutional Literacy

> **Learn by playing. Compete while understanding. Remember through experience.**

A full-stack, AI-powered, competition-first learning platform that makes the **Indian Constitution** engaging, interactive, and memorable through games, multiplayer quizzes, and real-time leaderboards.

---

## 📚 Table of Contents

- [Project Summary](#-project-summary)
- [Key Features](#-key-features)
- [Tech Stack](#️-tech-stack)
- [Architecture Overview](#️-architecture-overview)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start-development)
- [Environment Variables](#-environment-variables)
- [Kafka Setup](#-kafka--topic-verification)
- [Score Event Flow](#-score-event-flow-developer-view)
- [Verification & Debugging](#-verification--debugging)
- [Contributing](#-contributing)
- [License](#-license)

---

## 📌 Project Summary

**InConQuest** is a gamified constitutional literacy platform designed to address low awareness and poor engagement with civics education.

Instead of static textbooks or basic quizzes, InConQuest uses:

- 🤖 AI-generated explanations
- 🎮 Multiple single-player and multiplayer games
- ⚡ Event-driven scoring using **Apache Kafka**
- 🏆 Real-time leaderboards, badges, and streaks

Every meaningful user action produces a **score event**, making competition and rewards the core driver of learning.

### Core Goals

| Goal | Description |
|------|-------------|
| 📖 Simplified Learning | AI-written explanations of Constitutional Articles |
| 🎮 Engaging Games | Multiple game modes that reinforce learning |
| 🤝 Multiplayer | Real-time competitive play with instant feedback |
| ⚡ Scalable Architecture | Event-driven leaderboard system via Kafka |

---

## ✨ Key Features

### 🧠 Gemini-Powered Article Explanations
Clear, simplified explanations of Constitutional Articles generated directly by **Gemini AI**.

### 🎮 Multiple Game Modes

| Game | Description |
|------|-------------|
| 🐍 Snakes & Ladders | Question-based board progression |
| 🎨 Pictionary | Indian leaders & constitutional figures |
| 🌀 Maze Game | Wumpus World inspired exploration |
| ⚖️ In the Courtroom | Real cases with AI-generated plots |
| 📅 Daily Quiz | Consti-Word (Wordle-inspired) |
| 👥 Multiplayer Quiz | Real-time competitive play |

### 🤝 Multiplayer Gameplay
- WebSocket-based live quizzes
- Instant score updates across all players

### 🤖 AI Chatbot (DistilBART)
- Simplifies complex legal language
- Helps users clarify doubts and improve performance

### ⚡ Event-Driven Scoring
- Every game emits **Kafka score events**
- No direct leaderboard writes from games (fully decoupled)

### 🏆 Leaderboards, Badges & Streaks
- Near real-time leaderboard updates
- Badge milestones at 50, 100, 200+ points
- Daily streak tracking

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| **Frontend** | React |
| **Backend** | Node.js, Express |
| **Real-Time** | WebSockets (Socket.IO) |
| **Database** | MongoDB (Atlas for prod, local for dev) |
| **Event Streaming** | Apache Kafka (Dockerized) |
| **AI — Explanations** | Gemini (article explanations & case generation) |
| **AI — Chatbot** | DistilBART (summarization & simplification) |
| **DevOps** | Docker, Docker Compose |

---

## 🏗️ Architecture Overview

```
Frontend (React Games & UI)
        |
        v
Backend APIs (Node + Express)
        |
        v
Kafka Producer → Kafka Topic (score events)
        |
        v
Kafka Consumer (Leaderboard Processor)
        |
        v
MongoDB (leaderboard_cache, badges, streaks)
        |
        v
Frontend reads cached leaderboard (fast)
```

> 🔑 **Key Design Principle:** Games **never** update leaderboards directly. All scoring flows through Kafka to ensure scalability and decoupling.

---

## ✅ Prerequisites

Make sure you have the following installed:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) + Docker Compose
- [Node.js](https://nodejs.org/) (v18+ recommended)
- npm
- MongoDB (Atlas or local)
- `mongosh` *(optional, for inspection)*

---

## ⚡ Quick Start (Development)

Open **PowerShell** in the project root (where `docker-compose.kafka.yml` exists).

### 1️⃣ Create `.env` File
*(See [Environment Variables](#-environment-variables) section below)*

### 2️⃣ Start Kafka & Zookeeper

```bash
docker compose -f .\docker-compose.kafka.yml up -d
```

### 3️⃣ (Optional) Run Local MongoDB

> Recommended for Windows dev to avoid SRV DNS issues.

```bash
docker run -d --name local-mongo -p 27017:27017 mongo:6
```

### 4️⃣ Start Backend (Auth Server)

```bash
cd backend\auth-server
npm install
npm start
```

### 5️⃣ Start Kafka Producer & Consumer

```bash
docker compose -f .\docker-compose.kafka.yml up -d kafka-producer kafka-consumer
```

### 6️⃣ Start Frontend

```bash
cd src
npm install
$env:PORT=3000; npm start
```

🌐 Visit: [http://localhost:3000](http://localhost:3000)

---

## 🔐 Environment Variables

Create a `.env` file in the project root:

```env
# MongoDB
MONGODB_URI=mongodb://host.docker.internal:27017/inconquest

# Auth
JWT_SECRET=some-strong-secret
JWT_EXPIRE=7d
PORT=3001

# AI
GEMINI_API_KEY=your_gemini_key_here
```

> ⚠️ **Important:** Avoid `mongodb+srv://` inside Docker on Windows if DNS fails. Use a non-SRV Atlas URI or local MongoDB instead.

---

## 📡 Kafka & Topic Verification

```bash
# List running containers
docker compose -f .\docker-compose.kafka.yml ps

# Create a topic
docker compose -f .\docker-compose.kafka.yml exec kafka \
kafka-topics --bootstrap-server localhost:9092 \
--create --topic test-topic --partitions 1 --replication-factor 1
```

> 🔁 **Broker mapping:** `localhost:29092` → `kafka:9092`

---

## 🔁 Score Event Flow (Developer View)

```
1. Player completes an action (quiz/game)
2. Frontend sends request to backend
3. Backend calculates score
4. Backend publishes Kafka score event
5. Kafka consumer processes event
6. MongoDB leaderboard cache updated
7. Frontend fetches cached leaderboard
```

> 📌 Games **never** write to the leaderboard DB directly.

---

## 🧪 Verification & Debugging

### Common Checks

- ✅ Confirm producer logs show `"Score event published"`
- ✅ Confirm consumer logs show `"Leaderboard cache updated"`
- ✅ Verify both services use the same `MONGODB_URI`

### Inspect Leaderboard Cache

```bash
mongosh "mongodb://localhost:27017/inconquest" \
--eval "db.leaderboard_cache.findOne({_id:'top5'})"
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the **MIT License** — feel free to use and modify!

---

<div align="center">
  <strong>Built with ❤️ to make the Indian Constitution accessible to everyone.</strong><br/>
  ⭐ Star this repo if you found it helpful!
</div>
