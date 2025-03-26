# Milestone 3: Voice Capabilities Implementation - Architecture Details

## Azure Speech Services Setup

### Azure Speech Services Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Azure Speech Services                       │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Speech-to-Text │   │  Text-to-Speech │   │  Speech         ││
│  │  Service        │   │  Service        │   │  Translation    ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Azure Speech Configuration

#### Speech-to-Text Configuration
- **Region**: East US (primary), West US (backup)
- **Pricing Tier**: Standard
- **Language Models**: English (US, UK, AU), Spanish, French
- **Custom Speech**: None initially, potential for future customization
- **Acoustic Models**: Default with noise reduction
- **Endpoint Configuration**: REST API with WebSocket support

#### Text-to-Speech Configuration
- **Region**: East US (primary), West US (backup)
- **Pricing Tier**: Standard
- **Voice Selection**: Neural voices (en-US-JennyNeural, en-US-GuyNeural)
- **Custom Voice**: None initially
- **SSML Support**: Enabled for prosody control
- **Endpoint Configuration**: REST API

### Credential Management

```
┌─────────────────────────────────────────────────────────────────┐
│                     Credential Management                       │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Cloudflare     │   │  Credential     │   │  Rotation       ││
│  │  KV Store       │   │  Accessor       │   │  Service        ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Voice Processing Worker

### Voice Processing Worker Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Voice Processing Worker                     │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Audio          │   │  Speech-to-Text │   │  Text           ││
│  │  Receiver       │   │  Client         │   │  Processor      ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  LLM Query      ││
│                                              │  Worker         ││
│                                              │                 ││
│                                              └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  Text-to-Speech ││
│                                              │  Client         ││
│                                              │                 ││
│                                              └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  Audio          ││
│                                              │  Responder      ││
│                                              │                 ││
│                                              └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Audio File Handling

```
┌─────────────────────────────────────────────────────────────────┐
│                     Audio File Flow                             │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Browser        │──►│  Cloudflare     │──►│  R2 Storage     ││
│  │  Recording      │   │  Worker         │   │  (Temporary)    ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └───────┬─────────┘   └───────┬─────────┘│
│                                │                     │          │
│                                │                     │          │
│                                ▼                     ▼          │
│                      ┌─────────────────┐   ┌─────────────────┐  │
│                      │                 │   │                 │  │
│                      │  Azure          │◄──┤  Audio          │  │
│                      │  Speech-to-Text │   │  Processor      │  │
│                      │                 │   │                 │  │
│                      └───────┬─────────┘   └─────────────────┘  │
│                              │                                  │
│                              │                                  │
│                              ▼                                  │
│                      ┌─────────────────┐                        │
│                      │                 │                        │
│                      │  Text           │                        │
│                      │  Processing     │                        │
│                      │                 │                        │
│                      └───────┬─────────┘                        │
│                              │                                  │
│                              │                                  │
│                              ▼                                  │
│                      ┌─────────────────┐                        │
│                      │                 │                        │
│                      │  Azure          │                        │
│                      │  Text-to-Speech │                        │
│                      │                 │                        │
│                      └───────┬─────────┘                        │
│                              │                                  │
│                              │                                  │
│                              ▼                                  │
│                      ┌─────────────────┐   ┌─────────────────┐  │
│                      │                 │   │                 │  │
│                      │  R2 Storage     │──►│  Browser        │  │
│                      │  (Temporary)    │   │  Playback       │  │
│                      │                 │   │                 │  │
│                      └─────────────────┘   └─────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Audio Security

```
┌─────────────────────────────────────────────────────────────────┐
│                     Audio Security                              │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  TLS            │   │  Signed         │   │  Encrypted      ││
│  │  Transport      │   │  URLs           │   │  Storage        ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Automatic      │   │  Access         │   │  Audit          ││
│  │  Expiration     │   │  Control        │   │  Logging        ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Frontend Voice Integration

### Click-to-Talk Interface

```
┌─────────────────────────────────────────────────────────────────┐
│                     Click-to-Talk Interface                     │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Microphone     │   │  Recording      │   │  Audio          ││
│  │  Button         │   │  Indicator      │   │  Visualizer     ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Playback       │   │  Processing     │   │  Error          ││
│  │  Controls       │   │  Indicator      │   │  Handling       ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Frontend Components

```javascript
// Voice Recording Component
const VoiceRecorder = () => {
  const [isRecording, setIsRecording] = useState(false);
  const [audioBlob, setAudioBlob] = useState(null);
  const [isProcessing, setIsProcessing] = useState(false);
  const [error, setError] = useState(null);
  
  // MediaRecorder instance
  const recorderRef = useRef(null);
  // Audio chunks during recording
  const chunksRef = useRef([]);
  
  const startRecording = async () => {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
      setIsRecording(true);
      
      recorderRef.current = new MediaRecorder(stream);
      chunksRef.current = [];
      
      recorderRef.current.ondataavailable = (e) => {
        chunksRef.current.push(e.data);
      };
      
      recorderRef.current.onstop = () => {
        const blob = new Blob(chunksRef.current, { type: 'audio/webm' });
        setAudioBlob(blob);
        sendAudioToServer(blob);
      };
      
      recorderRef.current.start();
    } catch (err) {
      setError("Microphone access denied or not available");
      console.error(err);
    }
  };
  
  const stopRecording = () => {
    if (recorderRef.current && isRecording) {
      recorderRef.current.stop();
      setIsRecording(false);
      
      // Stop all audio tracks
      recorderRef.current.stream.getTracks().forEach(track => track.stop());
    }
  };
  
  const sendAudioToServer = async (blob) => {
    setIsProcessing(true);
    
    try {
      const formData = new FormData();
      formData.append('audio', blob, 'recording.webm');
      
      const response = await fetch('/api/voice-processing', {
        method: 'POST',
        body: formData,
      });
      
      if (!response.ok) {
        throw new Error('Server error processing audio');
      }
      
      const data = await response.json();
      
      // Handle response with audio URL
      if (data.audioUrl) {
        playResponseAudio(data.audioUrl);
      }
    } catch (err) {
      setError("Error processing voice: " + err.message);
      console.error(err);
    } finally {
      setIsProcessing(false);
    }
  };
  
  const playResponseAudio = (url) => {
    const audio = new Audio(url);
    audio.play();
  };
  
  return (
    <div className="voice-recorder">
      <button 
        className={`mic-button ${isRecording ? 'recording' : ''}`}
        onMouseDown={startRecording}
        onMouseUp={stopRecording}
        onTouchStart={startRecording}
        onTouchEnd={stopRecording}
        disabled={isProcessing}
      >
        {isRecording ? 'Recording...' : 'Hold to Talk'}
      </button>
      
      {isProcessing && (
        <div className="processing-indicator">
          Processing your request...
        </div>
      )}
      
      {error && (
        <div className="error-message">
          {error}
          <button onClick={() => setError(null)}>Dismiss</button>
        </div>
      )}
    </div>
  );
};
```

### Chatwoot Integration

```javascript
// Chatwoot Voice Integration
const injectVoiceFeatures = () => {
  // Find the message input container
  const messageContainer = document.querySelector('.message-input-container');
  
  if (!messageContainer) return;
  
  // Create voice button element
  const voiceButton = document.createElement('div');
  voiceButton.className = 'voice-input-button';
  voiceButton.innerHTML = `
    <button class="mic-button">
      <svg width="24" height="24" viewBox="0 0 24 24">
        <path d="M12 14c1.66 0 3-1.34 3-3V5c0-1.66-1.34-3-3-3S9 3.34 9 5v6c0 1.66 1.34 3 3 3z"></path>
        <path d="M17 11c0 2.76-2.24 5-5 5s-5-2.24-5-5H5c0 3.53 2.61 6.43 6 6.92V21h2v-3.08c3.39-.49 6-3.39 6-6.92h-2z"></path>
      </svg>
    </button>
  `;
  
  // Add event listeners
  const micButton = voiceButton.querySelector('.mic-button');
  
  let recorder = null;
  let audioChunks = [];
  
  micButton.addEventListener('mousedown', startRecording);
  micButton.addEventListener('mouseup', stopRecording);
  micButton.addEventListener('touchstart', startRecording);
  micButton.addEventListener('touchend', stopRecording);
  
  // Insert voice button into the UI
  messageContainer.appendChild(voiceButton);
  
  async function startRecording() {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
      recorder = new MediaRecorder(stream);
      audioChunks = [];
      
      recorder.ondataavailable = (e) => {
        audioChunks.push(e.data);
      };
      
      recorder.onstop = async () => {
        const audioBlob = new Blob(audioChunks, { type: 'audio/webm' });
        await processAudio(audioBlob);
        
        // Stop all tracks
        stream.getTracks().forEach(track => track.stop());
      };
      
      micButton.classList.add('recording');
      recorder.start();
    } catch (err) {
      console.error('Error accessing microphone:', err);
      showError('Could not access microphone');
    }
  }
  
  function stopRecording() {
    if (recorder && recorder.state === 'recording') {
      micButton.classList.remove('recording');
      recorder.stop();
    }
  }
  
  async function processAudio(blob) {
    try {
      micButton.classList.add('processing');
      
      const formData = new FormData();
      formData.append('audio', blob);
      formData.append('conversationId', getConversationId());
      
      const response = await fetch('/api/voice-processing', {
        method: 'POST',
        body: formData
      });
      
      if (!response.ok) {
        throw new Error('Failed to process audio');
      }
      
      const data = await response.json();
      
      if (data.audioUrl) {
        playResponseAudio(data.audioUrl);
      }
    } catch (err) {
      console.error('Error processing audio:', err);
      showError('Error processing your voice message');
    } finally {
      micButton.classList.remove('processing');
    }
  }
  
  function playResponseAudio(url) {
    const audio = new Audio(url);
    audio.play();
  }
  
  function showError(message) {
    // Show error message in UI
    const errorElement = document.createElement('div');
    errorElement.className = 'voice-error-message';
    errorElement.textContent = message;
    
    messageContainer.appendChild(errorElement);
    
    setTimeout(() => {
      errorElement.remove();
    }, 5000);
  }
  
  function getConversationId() {
    // Extract conversation ID from URL or data attributes
    const conversationElement = document.querySelector('[data-conversation-id]');
    return conversationElement ? conversationElement.dataset.conversationId : null;
  }
};

// Initialize voice features when Chatwoot is loaded
document.addEventListener('DOMContentLoaded', () => {
  // Wait for Chatwoot UI to be fully loaded
  setTimeout(injectVoiceFeatures, 1000);
});
```

## Voice Processing Flow

### Complete Voice Processing Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     Voice Processing Flow                       │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  User           │──►│  Chatwoot       │──►│  Voice          ││
│  │  (Speaking)     │   │  Interface      │   │  Processing     ││
│  │                 │   │                 │   │  Worker         ││
│  └─────────────────┘   └─────────────────┘   └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  Azure          ││
│                                              │  Speech-to-Text ││
│                                              │                 ││
│                                              └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  LLM Query      ││
│                                              │  Worker         ││
│                                              │                 ││
│                                              └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  Qdrant         ││
│                                              │  (Context)      ││
│                                              │                 ││
│                                              └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  Groq LLM       ││
│                                              │                 ││
│                                              └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  Azure          ││
│                                              │  Text-to-Speech ││
│                                              │                 ││
│                                              └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  Voice          ││
│                                              │  Processing     ││
│                                              │  Worker         ││
│                                              │                 ││
│                                              └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  Chatwoot       ││
│                                              │  Interface      ││
│                                              │                 ││
│                                              └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  User           ││
│                                              │  (Listening)    ││
│                                              │                 ││
│                                              └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Voice Testing and Optimization

### Testing Framework

```
┌─────────────────────────────────────────────────────────────────┐
│                     Voice Testing Framework                     │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Automated      │   │  Manual         │   │  A/B            ││
│  │  Tests          │   │  Tests          │   │  Testing        ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Performance    │   │  Accuracy       │   │  User           ││
│  │  Testing        │   │  Testing        │   │  Feedback       ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Optimization Areas

```
┌─────────────────────────────────────────────────────────────────┐
│                     Voice Optimization Areas                    │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Audio          │   │  Latency        │   │  Accuracy       ││
│  │  Quality        │   │  Reduction      │   │  Improvement    ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Noise          │   │  Voice          │   │  Error          ││
│  │  Reduction      │   │  Customization  │   │  Handling       ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```