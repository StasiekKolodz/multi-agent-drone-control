# 🚁 AI-Powered Autonomous Drone Control System

*An intelligent multi-agent system that enables natural language control of drones through advanced AI reasoning, computer vision, and safety validation.*

> **Note:** This project interfaces with physical drone hardware via ROS 2 and requires a compatible flight controller (ArduPilot/PX4) for live deployment.

---

## 📖 About This Project

Traditional drone control requires technical expertise in flight systems and manual piloting skills. This project eliminates that barrier by allowing operators to control drones using simple natural language commands like *"Fly 10 meters west and land"*. The system autonomously interprets commands, analyzes real-time camera feeds, plans safe flight paths, and executes missions with built-in safety validation—all powered by GPT-4 and a sophisticated multi-agent architecture.

---

## ✨ Key Features

- 🗣️ **Natural Language Command Interface** – Control your drone by simply typing commands in plain language, no technical knowledge required
- 🤖 **Intelligent Mission Planning** – AI automatically breaks down complex commands into executable flight steps with context awareness
- 👁️ **Real-Time Vision Analysis** – GPT-4 Vision processes live camera feeds to provide spatial awareness for obstacle detection and navigation
- 🛡️ **Multi-Layer Safety Validation** – Every command is validated by an AI guardian agent before execution, with battery monitoring and emergency abort capabilities
- 🧠 **Learning from Experience** – The system stores mission outcomes in a vector database to improve future decision-making
- 🎯 **Precise Navigation Control** – Supports takeoff, landing, relative positioning (NED coordinates), and autonomous waypoint navigation
- 📊 **Live Telemetry Monitoring** – Real-time tracking of GPS position, battery status, altitude, and flight mode
- 🔄 **Asynchronous Multi-Agent Coordination** – Five specialized AI agents work together through a message-passing architecture

---

## 🏗️ System Architecture

The system uses a **choreography-based multi-agent architecture** where specialized AI agents communicate through a shared message pool:

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────┐
│ Mission Planner │────▶│ Vision Agent │────▶│  Navigator  │
│   (GPT-4)       │     │  (GPT-4V)    │     │  (GPT-4)    │
└─────────────────┘     └──────────────┘     └──────┬──────┘
         ▲                                           │
         │                                           ▼
         │                                    ┌─────────────┐
         └────────────────────────────────────┤  Guardian   │
                                              │ (Validator) │
                                              └──────┬──────┘
                                                     │
                                              ┌──────▼──────┐
                                              │ ROS 2 Drone │
                                              │  Controller │
                                              └─────────────┘
```

### Agent Responsibilities

| Agent | Role | Technology |
|-------|------|------------|
| **Mission Planner** | Converts natural language to structured mission steps | GPT-4, LangChain Conversational Agent |
| **Vision Agent** | Analyzes drone camera feed for spatial awareness | GPT-4 Turbo with Vision |
| **Navigator** | Plans and sequences drone actions for each mission step | GPT-4 with LangGraph ReAct |
| **Guardian** | Validates action safety and executes approved commands | GPT-4-mini + Safety Logic |
| **Reflection Agent** | Stores mission outcomes for future learning | FAISS Vector Store |

---

## 🛠️ Tech Stack

| Category | Technologies |
| :--- | :--- |
| **AI & LLM** | OpenAI GPT-4, GPT-4 Turbo (Vision), GPT-4-mini, LangChain, LangGraph |
| **Multi-Agent Framework** | Custom message-passing architecture, Python threading |
| **Computer Vision** | OpenCV, CV Bridge, GPT-4 Vision API |
| **Drone Control** | ROS 2 (Robot Operating System), MAVLink Protocol, ArduPilot/PX4 |
| **Backend API** | Flask REST API, Python Requests |
| **Vector Database** | FAISS (Facebook AI Similarity Search) with OpenAI Embeddings |
| **Action Framework** | ROS 2 Action Clients (Goal-Feedback-Result pattern) |
| **Telemetry** | ROS 2 Services & Subscribers, Real-time sensor fusion |
| **Deployment** | Python 3.x, Multi-threaded execution, Daemon processes |

---

## 🚀 How It Works

### 1. **Natural Language Input**
The operator types a command like: *"Take off to 3 meters, fly 5 meters north, then land"*

### 2. **Mission Decomposition**
The Mission Planner agent uses GPT-4 to break the command into discrete steps:
```json
[
  {"id": 1, "cel": "Wystartuj do 3 metrów"},
  {"id": 2, "cel": "Leć 5 metrów na północ"},
  {"id": 3, "cel": "Wyląduj"}
]
```

### 3. **Vision Context**
The Vision Agent fetches the current camera frame and generates a spatial description:
> *"Large object on left at 2 meters, open space ahead, ground visible below"*

### 4. **Action Planning**
The Navigator uses GPT-4 ReAct to select appropriate drone primitives:
- `Takeoff(altitude=3.0)`
- `FlyTo(north=5, east=0, down=0)`
- `Land()`

### 5. **Safety Validation**
The Guardian Agent validates each action:
- ✅ Checks if action matches mission context
- ✅ Verifies vision context is safe
- ✅ Monitors battery levels
- ❌ Rejects illogical or dangerous commands

### 6. **Physical Execution**
Approved commands flow through:
```
HTTP API → Flask Server → ROS 2 DroneController → MAVLink → Flight Controller → Motors
```

### 7. **Learning Loop**
After mission completion, the Reflection Agent:
- Prompts user for success/failure feedback
- Stores entire mission transcript in vector database
- Future missions can query past experiences

---

## 🚀 Run This Project Locally

### Prerequisites
- Python 3.8+
- ROS 2 (Humble/Foxy) installed
- Compatible drone with ArduPilot/PX4 firmware
- OpenAI API key

### Setup Instructions

1. **Clone the repository:**
   ```bash
   git clone https://github.com/StasiekKolodz/projekt_nlp.git
   cd projekt_nlp
   ```

2. **Install Python dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Configure environment variables:**
   ```bash
   cp .env.example .env
   # Edit .env and add your OpenAI API key:
   # OPENAI_API_KEY=sk-your-api-key-here
   ```

4. **Start the ROS 2 backend (in a separate terminal):**
   ```bash
   cd controll_backend
   python app.py
   ```

5. **Launch the AI agent system:**
   ```bash
   python main.py
   ```

6. **Start issuing commands:**
   ```
   [MISSION PLANNER] Enter your command: Take off and fly forward 5 meters
   ```

### Testing Without Physical Drone
To test the AI logic without hardware, you can modify `drone_tools.py` to use a simulator or mock responses.

---

## 📚 Example Commands

| Command | What Happens |
|---------|--------------|
| `"Take off to 2 meters"` | Drone arms, enters guided mode, and ascends to 2m altitude |
| `"Fly 10 meters west"` | Moves 10m in the westward direction using NED coordinates |
| `"If you see an obstacle, move up 1 meter"` | Vision agent detects objects, conditionally adjusts altitude |
| `"Land"` | Initiates controlled landing sequence |

---

## 🌱 Lessons Learned

- **Multi-Agent Orchestration Complexity** – Designed a choreography-based system where agents coordinate through message passing rather than direct calls, enabling loose coupling and easier debugging. Learned to manage race conditions with thread-safe message pools and state flags.

- **LLM Reliability in Safety-Critical Systems** – Discovered that LLMs can hallucinate unsafe actions, which led to implementing a dedicated Guardian validation layer. This taught me the importance of multiple validation gates when AI controls physical hardware.

- **Bridging Symbolic AI and Low-Level Control** – Integrated GPT-4's high-level reasoning with ROS 2's real-time hardware control, requiring careful attention to latency, error propagation, and synchronization between asynchronous LLM calls and blocking drone operations.

- **Vision-Guided Navigation Challenges** – Learned that raw GPT-4 Vision descriptions can be verbose and unreliable for navigation. Solved this by crafting specialized prompts that extract only spatial information (position, distance, size) while ignoring irrelevant details like color or object type.

- **Error Handling Across Abstraction Layers** – Dealt with failures at multiple levels (LLM API timeouts, ROS 2 service failures, MAVLink disconnects). Implemented defensive programming with timeouts, validation checks, and graceful degradation to prevent cascading failures.

---

## 🔮 Future Enhancements

- [ ] Add support for autonomous obstacle avoidance using SLAM (Simultaneous Localization and Mapping)
- [ ] Implement voice command interface for hands-free operation
- [ ] Create a web dashboard for real-time mission monitoring and telemetry visualization
- [ ] Add support for multi-drone coordination and swarm behavior
- [ ] Integrate reinforcement learning for optimized flight path planning
- [ ] Expand vector store with pre-trained mission templates for common scenarios
