> A lightweight, local-first AI desktop companion for Windows that can see your screen, hear your questions, understand your context, and speak its answers.

Slimey is a Windows desktop AI companion designed to provide contextual assistance without requiring the user to constantly switch between their application and an external chatbot.

It runs locally with **Ollama** and a vision-capable model, allowing Slimey to use the user's screen as visual context while answering questions.

---

## ✨ What is Slimey?

Slimey is designed to sit unobtrusively on the user's desktop and become active when the user intentionally interacts with it.

Instead of:

```text
User → Switch application → Open browser → Open AI chatbot → Explain problem

Slimey aims for:
User → Ask Slimey → Slimey sees the screen → Understands context → Responds

The goal is to make AI assistance feel like a lightweight desktop companion rather than another application the user has to constantly manage.
🚀 Core Features
🎙️ Voice Interaction
Slimey supports voice-based interaction through the existing desktop interaction system.
The user can activate Slimey using the configured keyboard shortcut, speak naturally, and receive an answer.
The MVP is designed around intentional activation rather than continuously listening to the microphone.
👁️ Screen-Aware Assistance
Slimey can capture the visible screen and provide that visual information to the local vision model.
This allows questions such as:
"What is this error?"

"What am I looking at?"

"How do I do this?"

"Where is the option I need?"

to be answered using the actual screen as context rather than relying only on a text description from the user.
🧠 Local AI
Slimey can use Ollama to run AI models locally.
For the current MVP, the primary model is:
qwen2.5vl:3b

This provides vision capabilities required for screen-aware assistance.
Because the model runs locally through Ollama, the MVP does not require a paid AI API key.
🔊 Spoken Responses
Slimey can provide spoken responses through the existing text-to-speech system.
This allows interaction to remain hands-free instead of requiring the user to constantly read a chatbot panel.
🖥️ Application Context
Slimey can use information about the currently active application as additional context.
For example:
User is working in Visual Studio Code.

User:
"Why am I getting this error?"

Slimey:
Uses the visible screen + active application context
to explain the relevant problem.

This allows Slimey to provide more useful assistance than a generic chatbot that only receives the user's sentence.
🎯 Contextual Guidance
When appropriate, Slimey can provide actionable guidance based on what it sees on the screen.
For example:
User:
"Where do I click to export this?"

Slimey:
Identifies the relevant UI element and provides
direction toward it.

The MVP intentionally keeps this interaction simple rather than attempting to automate every mouse action.
🏗️ High-Level Architecture
Slimey follows a local desktop-assistant pipeline:
                    ┌─────────────────┐
                    │      User       │
                    │ Voice / Hotkey  │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │    Slimey UI    │
                    │ Desktop Companion│
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
      ┌─────────────────┐          ┌─────────────────┐
      │ Screen Context  │          │ Voice / Text    │
      │   Capture       │          │     Input       │
      └────────┬────────┘          └────────┬────────┘
               │                            │
               └──────────────┬─────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │ Context Builder │
                    │ Screen + App +  │
                    │ User Question   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │     Ollama      │
                    │ Local Vision AI │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │     Slimey      │
                    │ Response / TTS  │
                    └─────────────────┘

The important principle is that the AI receives more than just the user's question.
It can receive:
- User's spoken/text question
- Current screen context
- Active application context
- Relevant interaction state
This allows Slimey to generate more context-aware responses.
🛠️ Technology Stack
Component	Technology
Platform	Windows 10 / 11
Language	Python
Desktop UI	PyQt6
Local AI Runtime	Ollama
Vision Model	Qwen2.5-VL 3B
Screen Understanding	Local screenshot + vision model
Speech Input	Local speech recognition / existing STT pipeline
Speech Output	Edge TTS / existing TTS pipeline
Packaging	PyInstaller
Installer	Inno Setup
Version Control	Git / GitHub


📋 Requirements
Operating System
- Windows 10 or Windows 11
- 64-bit recommended
Development Environment
- Python 3.11+
- Git
- Working microphone
- Working speakers/headphones
AI
- Ollama
- qwen2.5vl:3b
Hardware
Slimey is designed to work without a dedicated high-end GPU because Ollama can run the model using available CPU/GPU resources.
Performance depends heavily on the user's hardware.
📥 Installation
1. Clone Slimey
git clone https://github.com/vaibhavgaur44/Slimey.git
cd Slimey

2. Create a virtual environment
python -m venv .venv

Activate it:
Command Prompt
.venv\Scripts\activate

PowerShell
.venv\Scripts\Activate.ps1

3. Install dependencies
pip install -r requirements.txt

🧠 Ollama Setup
Install Ollama for Windows from:
https://ollama.com/
After installation, verify:
ollama --version

Pull the Slimey vision model:
ollama pull qwen2.5vl:3b

Verify that the model is available:
ollama list

You should see:
qwen2.5vl:3b

Ollama normally exposes its local service at:
http://localhost:11434

No paid API key is required for the local Ollama setup.
⚙️ Configuration
If the project contains an .env.example, create your local environment file from it.
For the local Ollama setup, the relevant configuration is:
OLLAMA_HOST=http://localhost:11434
OLLAMA_MODEL=qwen2.5vl:3b

Do not commit personal API keys or private credentials to the repository.
▶️ Running Slimey
From the project directory:
python main.py

Slimey will start as a Windows desktop application.
The application runs in the background and can be activated through its configured interaction method.
🎙️ Basic Interaction
The intended interaction flow is:
1. Activate Slimey
2. Speak your question
3. Slimey captures the relevant context
4. The local vision model processes the context
5. Slimey generates a response
6. The response is presented/spoken to the user

Example:
User:
"Why is this code giving me an error?"

Slimey:
Analyzes the visible screen and explains
the likely cause.

Another example:
User:
"How do I export this video?"

Slimey:
Uses the current application and visible UI
to provide contextual guidance.

🔐 Privacy / Local AI
One of the main design goals of Slimey is local-first AI assistance.
When using the Ollama configuration:
User
 ↓
Slimey
 ↓
Local screen/context processing
 ↓
Ollama
 ↓
Local AI model
 ↓
Slimey

The AI inference does not require sending the question to a paid cloud LLM API.
Users should still be careful when sharing sensitive information through screen-aware applications.
🎨 Slimey
Slimey is designed as a small desktop companion rather than a traditional chatbot window.
Its visual identity is based around a simple 2D lime-green slime character.
The companion is intended to remain unobtrusive while the user works and become active when assistance is requested.
🎯 MVP Scope
The hackathon MVP prioritizes:
- Windows desktop companion
- Slimey visual identity
- Voice activation
- Screen capture/context
- Local Ollama inference
- Vision-based screen understanding
- Active application context
- Spoken responses
- Basic contextual guidance
The MVP intentionally avoids unnecessary complexity.
Features such as advanced proximity detection, elaborate animation systems, continuous autonomous interaction, and extensive automation are outside the core MVP scope.
⚠️ Current Limitations
Because Slimey uses local AI, response speed and quality depend on the user's hardware and selected model.
Smaller local models may:
- take longer to process screenshots
- occasionally misunderstand UI elements
- provide less detailed reasoning than larger cloud models
- struggle with very small text or complex interfaces
Screen understanding is also dependent on the quality and visibility of the captured screen content.
🔨 Building a Windows Application
Slimey can be packaged into a standalone Windows application using the project's existing build configuration.
The intended distribution format is:
Slimey.exe

and, where applicable:
Slimey Setup.exe

The packaged application is intended to reduce the need for end users to manually configure the Python environment.
The exact build process is maintained in the repository's build configuration.
🧪 Development
The project is developed using:
Python
PyQt6
Ollama
Git
GitHub

Development follows a local-first workflow so that the application can be tested without requiring paid cloud services.
📁 Project Structure
The repository is organized into major components including:
Slimey/
│
├── ai/                  # AI/provider functionality
├── assets/              # Application assets and icons
├── audio/               # Speech input/output functionality
├── screen/              # Screen/context functionality
├── skills/              # Extensible interaction skills
├── tutor_features/      # Contextual assistance features
├── ui/                  # Desktop UI
│
├── companion_manager.py
├── config.py
├── hotkey.py
├── main.py
├── tutor.py
│
├── requirements.txt
├── build.bat
├── installer.iss
└── README.md

Individual components may evolve as Slimey continues to be developed.
🌱 Project Vision
Slimey is built around a simple idea:
AI should understand what you are doing, not just what you type.

Instead of forcing users to describe their problem manually, Slimey can use the surrounding visual and application context to provide assistance directly where the user is working.
The long-term goal is to make Slimey feel less like a chatbot and more like a lightweight AI companion that understands the user's workflow.
📄 License
See the LICENSE file included with this repository for licensing information.
Built for the Hackathon
Slimey is a hackathon project focused on exploring how local vision-language models can be integrated into a lightweight Windows desktop companion.
Slimey — See it. Understand it. Help with it.
