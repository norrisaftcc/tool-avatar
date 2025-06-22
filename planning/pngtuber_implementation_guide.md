# PNGtuber Implementation Guide - Phase 0

## 🎯 Strategy: Validate Fast, Upgrade Later

**Why PNGtuber First**: Prove all AI integrations, educational workflows, and system architecture with simple image swapping before investing in complex Live2D animation.

**Success Definition**: If PNGtuber version effectively teaches students and validates the concept, Live2D upgrade becomes pure polish rather than experimental development.

## 🎨 Asset Requirements

### Llama Expression Set (7 PNGs)
```yaml
Required Expressions:
  llama_neutral.png:     # Default peaceful teaching pose
  llama_happy.png:       # Positive reinforcement, encouragement
  llama_thinking.png:    # Processing complex questions, thoughtful
  llama_explaining.png:  # Active teaching, engaged with student
  llama_excited.png:     # Student breakthrough celebration
  llama_confused.png:    # Needs clarification, doesn't understand
  llama_talking.png:     # Simple mouth-open version for speech

Specifications:
  - Resolution: 512x512px (lighter than Live2D requirements)
  - Format: PNG with transparency
  - Style: Consistent with Three LLaMa Moon aesthetic
  - Background: Fully transparent
  - Expression clarity: Obvious differences for easy recognition
```

### TeacherBot-77 Expression Set (7 PNGs)
```yaml
Required Expressions:
  teacherbot_neutral.png:    # Default technical analysis stance
  teacherbot_analyzing.png:  # Code review, scanning for issues
  teacherbot_found_bug.png:  # Issue detected, alert expression
  teacherbot_explaining.png: # Technical explanation mode
  teacherbot_celebrating.png:# Problem solved successfully
  teacherbot_error.png:      # System confusion, processing failure
  teacherbot_talking.png:    # Speaking state with different screen

Specifications:
  - Resolution: 512x512px
  - Retro computer aesthetic maintained
  - Screen area can show different emoji/symbols
  - Tank treads and arms in appropriate positions
  - Clear emotional distinction between states
```

## 🔧 Technical Implementation

### Simple Renderer Architecture
```python
# pngtuber_engine.py
import pygame
import asyncio
from pathlib import Path

class PNGtuberCharacter:
    def __init__(self, character_name: str):
        self.name = character_name
        self.sprites = self.load_sprites()
        self.current_expression = "neutral"
        self.is_talking = False
        self.talk_timer = 0.0
        self.talk_interval = 0.3  # mouth flap speed
        
    def load_sprites(self) -> dict:
        """Load all expression sprites for this character"""
        sprites = {}
        sprite_dir = Path(f"assets/characters/{self.name}")
        
        for sprite_file in sprite_dir.glob("*.png"):
            expression_name = sprite_file.stem.replace(f"{self.name}_", "")
            sprites[expression_name] = pygame.image.load(sprite_file)
        
        return sprites
    
    def set_expression(self, expression: str):
        """Instantly switch to new expression"""
        if expression in self.sprites:
            self.current_expression = expression
            
    def start_talking(self):
        """Begin talking animation"""
        self.is_talking = True
        self.talk_timer = 0.0
        
    def stop_talking(self):
        """End talking animation"""
        self.is_talking = False
        
    def update(self, dt: float):
        """Update talking animation"""
        if self.is_talking:
            self.talk_timer += dt
            
    def render(self, screen, position=(0, 0)):
        """Render current state to screen"""
        # Determine which sprite to show
        if self.is_talking and (self.talk_timer % self.talk_interval) < (self.talk_interval / 2):
            # Show talking variant during speech
            sprite = self.sprites.get("talking", self.sprites[self.current_expression])
        else:
            sprite = self.sprites[self.current_expression]
            
        screen.blit(sprite, position)

class PNGtuberEngine:
    def __init__(self):
        pygame.init()
        self.screen = pygame.display.set_mode((800, 600))
        pygame.display.set_caption("Educational AI Characters")
        
        # Load characters
        self.llama = PNGtuberCharacter("llama")
        self.teacherbot = PNGtuberCharacter("teacherbot")
        self.current_character = self.llama
        
        # AI integration
        self.ai_backend = None
        self.running = True
        
    def switch_character(self, character_name: str):
        """Switch between llama and teacherbot"""
        if character_name == "llama":
            self.current_character = self.llama
        elif character_name == "teacherbot":
            self.current_character = self.teacherbot
            
    async def process_ai_response(self, response_data: dict):
        """Handle AI backend response"""
        # Switch character if needed
        if "character" in response_data:
            self.switch_character(response_data["character"])
            
        # Set expression
        if "expression" in response_data:
            self.current_character.set_expression(response_data["expression"])
            
        # Handle speech
        if "text" in response_data:
            await self.speak_text(response_data["text"], response_data.get("duration", 3.0))
            
    async def speak_text(self, text: str, duration: float):
        """Simulate speaking with talking animation"""
        self.current_character.start_talking()
        
        # TODO: Integrate with TTS system
        await asyncio.sleep(duration)
        
        self.current_character.stop_talking()
        
    def run(self):
        """Main game loop"""
        clock = pygame.time.Clock()
        
        while self.running:
            dt = clock.tick(60) / 1000.0  # 60 FPS
            
            # Handle events
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    self.running = False
                    
            # Update characters
            self.current_character.update(dt)
            
            # Render
            self.screen.fill((0, 0, 0))  # Black background
            self.current_character.render(self.screen, (150, 100))  # Center character
            pygame.display.flip()
            
        pygame.quit()
```

### AI Integration Layer
```python
# ai_coordinator.py
import asyncio
import openai
from typing import Dict, Any

class EducationalAICoordinator:
    def __init__(self):
        self.openai_client = openai.OpenAI()
        self.conversation_history = []
        self.current_topic = None
        
        # Character specializations
        self.character_specialties = {
            "llama": [
                "creative thinking", "algorithm concepts", "motivation", 
                "complex theory", "inspiration", "encouragement"
            ],
            "teacherbot": [
                "syntax errors", "debugging", "step-by-step", "technical details",
                "code review", "tools and environment", "precise instructions"
            ]
        }
        
    def select_character(self, question: str) -> str:
        """Determine which character should handle this question"""
        question_lower = question.lower()
        
        # Technical keywords suggest TeacherBot-77
        technical_keywords = [
            "error", "bug", "debug", "syntax", "compile", "install",
            "environment", "tool", "command", "terminal", "fix"
        ]
        
        # Creative keywords suggest Llama
        creative_keywords = [
            "algorithm", "approach", "design", "concept", "theory",
            "explain", "understand", "why", "how does", "what is"
        ]
        
        technical_score = sum(1 for keyword in technical_keywords if keyword in question_lower)
        creative_score = sum(1 for keyword in creative_keywords if keyword in question_lower)
        
        if technical_score > creative_score:
            return "teacherbot"
        else:
            return "llama"
            
    def determine_expression(self, character: str, response_text: str, question_type: str) -> str:
        """Map AI response to appropriate expression"""
        response_lower = response_text.lower()
        
        if character == "llama":
            if any(word in response_lower for word in ["think", "consider", "hmm"]):
                return "thinking"
            elif any(word in response_lower for word in ["great", "excellent", "perfect"]):
                return "excited"
            elif any(word in response_lower for word in ["let me explain", "so here"]):
                return "explaining"
            elif any(word in response_lower for word in ["confused", "not sure", "clarify"]):
                return "confused"
            elif any(word in response_lower for word in ["good job", "well done"]):
                return "happy"
            else:
                return "neutral"
                
        elif character == "teacherbot":
            if "analyzing" in response_lower or "checking" in response_lower:
                return "analyzing"
            elif any(word in response_lower for word in ["found", "error", "issue", "problem"]):
                return "found_bug"
            elif any(word in response_lower for word in ["fixed", "solved", "working"]):
                return "celebrating"
            elif any(word in response_lower for word in ["explain", "because", "the reason"]):
                return "explaining"
            elif "error" in response_lower and "cannot" in response_lower:
                return "error"
            else:
                return "neutral"
        
        return "neutral"
    
    async def process_question(self, question: str) -> Dict[str, Any]:
        """Process student question and return character response"""
        # Select appropriate character
        character = self.select_character(question)
        
        # Create character-specific system prompt
        if character == "llama":
            system_prompt = """You are an inspiring programming instructor who helps students 
            understand concepts creatively. Focus on the bigger picture, motivation, and 
            conceptual understanding. Be encouraging and mystical."""
        else:
            system_prompt = """You are a precise technical assistant who helps with specific 
            programming problems. Focus on exact solutions, debugging steps, and technical 
            accuracy. Be systematic and thorough."""
        
        # Generate AI response
        response = await self.openai_client.chat.completions.acreate(
            model="gpt-4o-mini",
            messages=[
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": question}
            ],
            max_tokens=150
        )
        
        response_text = response.choices[0].message.content
        
        # Determine expression
        expression = self.determine_expression(character, response_text, "general")
        
        # Calculate speaking duration (rough estimate)
        duration = len(response_text.split()) * 0.4  # ~0.4 seconds per word
        
        return {
            "character": character,
            "expression": expression,
            "text": response_text,
            "duration": duration,
            "timestamp": asyncio.get_event_loop().time()
        }
```

### WebSocket Communication
```python
# websocket_server.py
import asyncio
import websockets
import json
from ai_coordinator import EducationalAICoordinator

class PNGtuberWebSocketServer:
    def __init__(self):
        self.ai_coordinator = EducationalAICoordinator()
        self.connected_clients = set()
        
    async def register_client(self, websocket):
        """Register new client connection"""
        self.connected_clients.add(websocket)
        try:
            await websocket.wait_closed()
        finally:
            self.connected_clients.remove(websocket)
    
    async def broadcast_to_clients(self, message: dict):
        """Send message to all connected clients"""
        if self.connected_clients:
            message_json = json.dumps(message)
            await asyncio.gather(
                *[client.send(message_json) for client in self.connected_clients],
                return_exceptions=True
            )
    
    async def handle_message(self, websocket, path):
        """Handle incoming WebSocket messages"""
        await self.register_client(websocket)
        
        async for message in websocket:
            try:
                data = json.loads(message)
                
                if data["type"] == "question":
                    # Process student question
                    response = await self.ai_coordinator.process_question(data["text"])
                    
                    # Broadcast response to all clients
                    await self.broadcast_to_clients({
                        "type": "ai_response",
                        "character": response["character"],
                        "expression": response["expression"],
                        "text": response["text"],
                        "duration": response["duration"]
                    })
                    
                elif data["type"] == "manual_control":
                    # Manual character control for testing
                    await self.broadcast_to_clients({
                        "type": "manual_update",
                        "character": data.get("character", "llama"),
                        "expression": data.get("expression", "neutral")
                    })
                    
            except json.JSONDecodeError:
                await websocket.send(json.dumps({"error": "Invalid JSON"}))
            except Exception as e:
                await websocket.send(json.dumps({"error": str(e)}))

# Start server
async def main():
    server = PNGtuberWebSocketServer()
    async with websockets.serve(server.handle_message, "localhost", 8765):
        print("PNGtuber WebSocket server running on ws://localhost:8765")
        await asyncio.Future()  # Run forever

if __name__ == "__main__":
    asyncio.run(main())
```

## 📋 Week 1 Implementation Checklist

### Day 1: Asset Creation
- [ ] Extract base llama design from Three LLaMa Moon SVG
- [ ] Create 7 llama expression variants in image editor
- [ ] Extract TeacherBot-77 design and create 7 expressions
- [ ] Validate all PNGs are 512x512 with transparency
- [ ] Test load all sprites in pygame

### Day 2: Basic Renderer
- [ ] Implement PNGtuberCharacter class
- [ ] Create simple talking mouth animation
- [ ] Build main pygame rendering loop
- [ ] Test expression switching manually
- [ ] Verify 60fps performance

### Day 3: AI Backend
- [ ] Set up FastAPI server with OpenAI integration
- [ ] Implement character selection logic
- [ ] Create expression mapping system
- [ ] Test AI responses with manual character control

### Day 4: WebSocket Integration
- [ ] Build WebSocket server for real-time communication
- [ ] Connect pygame renderer to WebSocket client
- [ ] Test end-to-end: question → AI → character response
- [ ] Debug timing and synchronization issues

### Day 5: Dual Character System
- [ ] Implement smart character switching
- [ ] Test question routing (creative → llama, technical → teacherbot)
- [ ] Verify expression accuracy for each character type
- [ ] Optimize character transition timing

### Weekend: Polish & Testing
- [ ] Add error handling and fallback systems
- [ ] Create simple web interface for testing
- [ ] Document setup process for Week 2 team members
- [ ] Prepare demo scenarios for Monday presentation

## 🎯 Success Metrics - Week 1

### Technical Validation
- [ ] System runs stable at 60fps for 1+ hours
- [ ] AI response time consistently under 3 seconds
- [ ] Character switching works reliably
- [ ] Expression mapping feels natural and appropriate

### Educational Validation
- [ ] Students can distinguish between llama and TeacherBot-77 personalities
- [ ] Expression changes enhance understanding vs. distract
- [ ] Technical questions route correctly to TeacherBot-77
- [ ] Creative questions route correctly to llama

### User Experience Validation
- [ ] Interface is intuitive for both students and instructors
- [ ] Characters feel helpful rather than gimmicky
- [ ] System adds value to programming education
- [ ] Students request to use it again

**If Week 1 succeeds, Live2D upgrade becomes refinement rather than risky experiment!**