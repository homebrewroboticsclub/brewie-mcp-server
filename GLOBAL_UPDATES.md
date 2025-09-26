# GLOBAL_UPDATES.md

## Overview of Changes Since Forking the Original Project

This document describes all changes made to the fork of the [ros-mcp-server](https://github.com/robotmcp/ros-mcp-server) project for adaptation to the Brewie robot.

---

##  Summary

This fork takes the original ros-mcp-server and makes it work perfectly with the Brewie robot. We added voice control, AI vision, crypto payments, and security features while keeping it simple to use.

The robot can now understand you, see what's around it, and do complex tasks safely - all controlled by your voice.

### New Capabilities:
-  Voice robot control
-  AI-powered target detection
-  Cryptocurrency payments via QR codes (BrewPay)
-  Voice authentication for safety
-  Integration with robot's action system
-  Simple control interface

## 1. ROS Connection Method Changes

### What Changed
Switched from direct WebSocket JSON communication to `roslibpy` library:
```python
# Original approach
json_msg = json.dumps(message)
self.ws.send(json_msg)

# New approach  
ROSclient = roslibpy.Ros(host='localhost', port=9090)
topic = roslibpy.Topic(ROSclient, '/topic_name', 'message_type')
```

### Why This Works Better for Brewie
- More reliable connection for our specific robot setup
- Built-in message handling that fits our use case
- Easier to work with Brewie's ROS topics

---

## 2. MCP Server Startup Method Changes

### What We Added
Created simple startup scripts to make things easier:

#### Launch Files:
- `SART.bat` - main launch script with API keys
- `SART_EX.bat` - template for users  
- `voice_agent.py` - integrated voice agent

#### Simple Process:
```bash
# Just run this
uv run voice_agent.py
```

### What This Gives Us
- One command to start everything
- Voice control built-in
- Less setup hassle

---

## 3. Voice Agent Integration

### What We Built
Added a complete voice control system that lets you talk to the robot naturally.

#### **Voice Activation**
- Say "Brewie" to wake it up
- Works on both Raspberry Pi and Windows
- Low CPU usage for always-on listening

#### **Speech Processing**
- Converts your speech to text
- Understands natural language commands
- Talks back to you with voice responses
- Remembers conversation context

#### ** Flexible LLM Model Switching**
You can easily switch between different AI models on Together AI:

```python
# Just change this line in voice_agent.py:
MODEL_NAME = "meta-llama/Llama-3.3-70B-Instruct-Turbo"

# Examples:
MODEL_NAME = "mistralai/Mistral-7B-Instruct-v0.3"  # Faster
MODEL_NAME = "meta-llama/Llama-3.3-70B-Instruct-Turbo-Free"  # Free
MODEL_NAME = "Qwen/Qwen2.5-72B-Instruct"  # Alternative
```

**Available Models:**
- Meta Llama, Mistral, Qwen, Code models, and more
- Choose based on speed, cost, or accuracy needs
- Switch anytime without code changes

#### ** Security: Voice-Based Access Control**
Some commands require voice authentication for safety:

**Public Commands** (anyone can use):
- `make_step` - basic movement
- `get_image` - camera capture  
- `get_available_actions` - action listing
- `run_action` - standard actions

**Restricted Commands** (master voice required):
- `sniper` - autonomous targeting and shooting
- `defend` - defensive combat actions
- `BrewPay` - cryptocurrency transactions

The system recognizes your voice and only allows dangerous operations if you're the registered master.

#### **How It Works**
1. Always listening for "Brewie" wake word
2. Captures your voice command
3. Checks if you're authorized for dangerous commands
4. Processes command with AI
5. Executes robot action
6. Gives you voice feedback

---

## 4. Brewie Robot-Specific Functions

### What We Removed
Removed generic functions that didn't fit Brewie:
- `pub_twist` - twist message control
- `pub_twist_seq` - twist sequences  
- `pub_jointstate` - joint control
- `sub_jointstate` - joint state reading

### What We Added for Brewie

#### `make_step(x: float, z: float)`
- Move robot with joystick control
- x = left/right, z = forward/backward

#### `defend(rotate: float, UPDOWN: float)` 🔒
- Aim and shoot at targets
- Requires master voice authentication

#### `sniper(targediscr: str)` 🔒
- AI-powered target detection and shooting
- Uses camera + AI to find targets
- Requires master voice authentication

#### `BrewPay(amount: float)` 🔒
- Send SOL cryptocurrency via QR codes
- Takes photo, reads QR code, sends money
- Requires master voice authentication

#### `run_action(action_name: str)`
- Run pre-made robot actions
- Works with Brewie's ActionGroups

#### `get_available_actions()`
- List all available robot actions

---

## 5. VLM (Vision Language Model) Integration

### How AI Vision Works
The `sniper` function uses AI to analyze camera images and find targets:

```python
# AI analyzes robot's camera images
respons = LLMclient.chat.completions.create(
    model="Qwen/Qwen2.5-VL-72B-Instruct",
    messages=[{
        "role": "user", 
        "content": [
            {"type": "text", "text": "Find the purple ball"},
            {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{image}"}}
        ]
    }]
)
```

### ** Flexible VLM Model Switching**
You can switch vision models too:

```python
# In server.py, change the model:
model="Qwen/Qwen2.5-VL-72B-Instruct"  # High accuracy
# model="llava-hf/llava-1.5-7b-hf"    # Faster
# model="Salesforce/blip2-opt-2.7b"   # Lightweight
```

**Available Vision Models:**
- Qwen Vision, LLaVA, InstructBLIP, BLIP models
- Choose based on speed vs accuracy needs

### What It Can Do:
- Analyze robot camera images
- Find objects by description  
- Auto-aim at detected targets

---

## 6. SOL Blockchain Integration

### What We Added
Built-in cryptocurrency payments using SOL:

### `BrewPay` Function:
- Takes photo of QR code with wallet address
- Validates the Solana address
- Sends real SOL cryptocurrency
- Stores private key locally for security

### How It Works:
1. Clear camera folder
2. Take photo of QR code
3. Read wallet address from QR code
4. Load your private key
5. Send SOL to that address
6. Wait for confirmation

### Setup Files:
- `SOL_SETUP_INSTRUCTIONS.md` - how to set up
- `requirements_sol.txt` - extra dependencies needed
- `master_sh/sol_private_key` - your private key (keep safe!)

---

## 7. Security and Authentication System

### Voice-Based Security
The system recognizes your voice to prevent unauthorized access to dangerous commands.

### Access Levels:
- **Public Commands**: Anyone can use (movement, camera, basic actions)
- **Restricted Commands**: Need master voice authentication
  - `sniper` - shooting
  - `defend` - defense  
  - `BrewPay` - cryptocurrency transfers

---

## 8. New Project Structure

### What We Added:
```
brewie-mcp-server/
├── voice_agent.py              # Voice control system
├── ROS/action_groups.py        # ROS integration
├── master_sh/                  # Security storage
│   ├── sol_private_key        # Your SOL wallet key
│   └── master_voice           # Your voice profile
├── audio_in/                   # Voice recordings
├── audio_out/                  # Speech cache
├── photos/environment/        # Camera photos
├── SART.bat                   # Easy startup
├── SART_EX.bat               # Startup template
├── MCPFUNCTIONS.md           # Function docs
├── SOL_SETUP_INSTRUCTIONS.md # SOL setup guide
└── requirements_sol.txt      # Extra dependencies
```

### Wake Word Models:
- `Brewie_en_raspberry-pi_v3_0_0.ppn` - for Raspberry Pi
- `Brewie_en_windows_v3_0_0.ppn` - for Windows

---

## 9. Technical Improvements

### What We Optimized:
- **TTS Caching**: Faster repeated phrases
- **Async Processing**: Non-blocking operations
- **Thread Safety**: Proper multi-threading
- **Error Handling**: Better diagnostics

### New Dependencies Added:
```toml
# Voice tech
"pvporcupine>=3.0.5",      # Wake word detection
"pveagle>=1.0.4",          # Voice authentication  
"pvrecorder>=1.2.7",       # Audio recording
"speechrecognition>=3.14.3", # Speech recognition
"gtts>=2.5.4",             # Text-to-speech
"pygame>=2.6.1",           # Audio playback

# AI integration
"together>=1.5.13",        # Together API
"fastmcp>=2.8.1",          # MCP client

# Blockchain
"solana>=0.30.2",          # Solana SDK
"solders>=0.21.0",         # Solana utilities
"qrcode[pil]>=8.2",        # QR code generation
"pyzbar>=0.1.9",           # QR code recognition
"base58>=2.1.1",           # Base58 encoding
```

---

## 10. What This Fork Gives You

1. **Brewie-Specific**: Built for this robot, not generic
2. **Voice Control**: Talk to your robot naturally
3. **AI Vision**: Robot can see and understand what's around it
4. **Crypto Payments**: Transfer tokens via QR codes
5. **Voice Security**: Only authorized users can do dangerous stuff
6. **Easy Setup**: One command to start everything
7. **Action Integration**: Works with Brewie's existing actions

---


