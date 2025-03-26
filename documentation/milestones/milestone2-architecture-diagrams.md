# Milestone 2: AI Integration and Basic Voice Capabilities - Architecture and Diagrams

## Enhanced System Architecture

Milestone 2 builds upon the core Chatwoot infrastructure by adding AI integration and basic voice capabilities through Azure's speech services.

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                        Client Devices                           │
│                                                                 │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                      Load Balancer / CDN                        │
│                                                                 │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                     Chatwoot Web Application                    │
│                     (with Voice UI Components)                  │
│                                                                 │
└───────────────┬─────────────────────────────────┬───────────────┘
                │                                 │
                ▼                                 ▼
┌───────────────────────────────┐   ┌───────────────────────────────┐
│                               │   │                               │
│      Chatwoot API Server      │   │       Redis (Caching)         │
│                               │   │                               │
└───────────────┬───────────────┘   └───────────────────────────────┘
                │
                ▼
┌───────────────────────────────┐   ┌───────────────────────────────┐
│                               │   │                               │
│     PostgreSQL Database       │◄──┤    Customer Data Connector    │
│                               │   │                               │
└───────────────┬───────────────┘   └───────────────────────────────┘
                │
                ▼
┌───────────────────────────────┐   ┌───────────────────────────────┐
│                               │   │                               │
│      AI Integration Layer     │◄──┤     Voice Processing API      │
│                               │   │                               │
└───────────────┬───────────────┘   └───────────────┬───────────────┘
                │                                   │
                ▼                                   ▼
┌───────────────────────────────┐   ┌───────────────────────────────┐
│                               │   │                               │
│      AI Backend Service       │   │     Azure Speech Services     │
│                               │   │     (STT and TTS APIs)        │
└───────────────────────────────┘   └───────────────────────────────┘
```

## New Component Details

### Voice UI Components
- Microphone button in Chatwoot interface
- Audio recording using browser's MediaRecorder API
- Audio playback for voice responses
- Visual indicators for recording/playback status
- Basic error handling for audio issues

### Voice Processing API
- Handles audio file uploads from frontend
- Manages communication with Azure Speech Services
- Processes audio transcription results
- Sends text to AI backend for processing
- Requests text-to-speech conversion for responses
- Returns audio responses to frontend

### Azure Speech Services Integration
- Speech-to-Text (STT) service for transcribing user voice input
- Text-to-Speech (TTS) service for generating voice responses
- Configuration for voice selection and quality settings
- Error handling and fallback mechanisms

### AI Integration Layer
- Connects Chatwoot to AI backend services
- Manages context and conversation history
- Handles routing between human agents and AI
- Provides AI response formatting for Chatwoot display

## Data Flow for Voice Interaction

```
┌──────────────┐    ┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│              │    │               │    │               │    │               │
│    User      │───►│  Voice UI     │───►│ Voice         │───►│ Azure STT     │
│  (Speaking)  │    │ (Recording)   │    │ Processing API│    │ Service       │
│              │    │               │    │               │    │               │
└──────────────┘    └───────────────┘    └───────┬───────┘    └───────┬───────┘
                                                 │                    │
                                                 │                    │
                                                 ▼                    ▼
┌──────────────┐    ┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│              │    │               │    │               │    │               │
│    User      │◄───│  Voice UI     │◄───│ Voice         │◄───│ Azure TTS     │
│ (Listening)  │    │ (Playback)    │    │ Processing API│    │ Service       │
│              │    │               │    │               │    │               │
└──────────────┘    └───────────────┘    └───────┬───────┘    └───────┬───────┘
                                                 │                    │
                                                 │                    │
                                                 ▼                    ▼
                                         ┌───────────────┐    ┌───────────────┐
                                         │               │    │               │
                                         │ AI Integration│◄───┤ Chatwoot API  │
                                         │ Layer         │    │ Server        │
                                         │               │    │               │
                                         └───────┬───────┘    └───────────────┘
                                                 │
                                                 │
                                                 ▼
                                         ┌───────────────┐
                                         │               │
                                         │ AI Backend    │
                                         │ Service       │
                                         │               │
                                         └───────────────┘
```

## Click-to-Talk Implementation Details

### Frontend Components
- **Microphone Button**: Added to the Chatwoot conversation interface
- **Recording State Management**: Visual indicators for recording status
- **Audio Processing**: Browser-based audio capture and playback
- **Error Handling**: User feedback for audio permission issues or failures

### Backend Components
- **Audio File Handling**: Secure upload and storage of audio files
- **Transcription Service**: Integration with Azure Speech-to-Text
- **Response Generation**: Integration with AI backend
- **Voice Synthesis**: Integration with Azure Text-to-Speech
- **Caching**: Temporary storage of audio responses for performance

## Voice Processing Flow

1. **User Initiates Voice Input**:
   - User clicks and holds microphone button
   - Browser requests microphone permission (if not already granted)
   - Audio recording begins
   - Visual indicator shows recording in progress

2. **Voice Capture and Processing**:
   - User speaks query while holding button
   - Audio is captured using MediaRecorder API
   - User releases button to end recording
   - Audio file is created and sent to backend

3. **Speech-to-Text Conversion**:
   - Backend receives audio file
   - Audio is sent to Azure Speech-to-Text service
   - Transcribed text is returned
   - Text is processed and formatted

4. **AI Processing**:
   - Transcribed text is sent to AI backend
   - AI generates appropriate response
   - Response is formatted for Chatwoot display
   - Text is prepared for voice synthesis

5. **Text-to-Speech Conversion**:
   - Response text is sent to Azure Text-to-Speech service
   - Voice audio file is generated
   - Audio file is returned to frontend

6. **Response Playback**:
   - Frontend receives audio file
   - Audio playback begins automatically
   - Text response is displayed simultaneously
   - User can interrupt by clicking microphone again

## Security Considerations

- Secure audio file transmission using HTTPS
- Temporary audio file storage with appropriate access controls
- Compliance with data protection regulations for voice data
- User consent mechanisms for voice recording
- Secure API key management for Azure services

## Performance Considerations

- Audio compression for efficient transmission
- Caching of common responses for performance
- Optimized audio formats for different devices
- Fallback to text-only mode in case of voice service failures
- Monitoring of voice service latency and quality