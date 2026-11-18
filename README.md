# FaceTag - AI Networking Assistant for Smart Glasses

**Built at Cal Hacks 13.0 | October 2026 | UC Berkeley**

[**View our Devpost submission and hackathon demo**](https://devpost.com/software/meta-glasses-v2kp8a)

FaceTag is an AI-powered networking assistant built for Meta Ray-Ban Display smart glasses. It combines **on-device facial recognition, contextual contact information and real-time AI assistance** to make conversations at networking events more seamless.

The idea is simple: when you meet someone, your glasses can recognise them, display useful background information and help you answer questions during a conversation, without needing to take out your phone.

Developed by **Toby, Joaquin, Mirabelle and Avi** at Cal Hacks 13.0.

## What We Built

### 1. Facial Recognition and Contact Information

- **On-device facial recognition:** Identify enrolled participants using Apple's Vision framework, without sending face embeddings to a server.
- **Smart glasses integration:** Display a person's name, role and relevant information directly on the Meta Ray-Ban Display.
- **Contact management:** A companion iPhone interface stores profiles, notes and face samples.
- **Manual identification:** Trigger recognition from the phone, glasses display or glasses capture button.
- **Unknown faces:** If someone isn't recognised, the interface allows their face to be associated with a contact.

For the hackathon, we used a small gallery of consenting teammates.

**Recognition pipeline:**

`Face detection → Face crop → Vision feature extraction → Similarity matching → Contact display`

The system compares facial feature representations with stored samples and applies confidence and runner-up thresholds to avoid incorrectly identifying people.

### 2. Real-Time Conversation Assistance

FaceTag listens for factual questions during conversations and displays AI-generated answers on the glasses.

For example, if someone asks *"How tall is the Eiffel Tower?"*, the assistant can begin generating an answer before they've finished speaking.

Rather than waiting for a complete sentence, the system uses **speculative inference on partial speech transcripts** to reduce perceived latency.

**Conversation pipeline:**

`Live audio → On-device transcription → Question detection → AI inference → Smart glasses display`

The assistant can:

- Identify factual questions from live speech.
- Ignore statements and questions that are too incomplete to answer.
- Generate answers using a fast language model.
- Re-evaluate an answer as additional words arrive.
- Display answers through the glasses' heads-up interface.

The aim is to deliver useful information *during* a conversation rather than after the moment has passed.

## Technical Architecture

| Component          | Technology           | Purpose                                         |
| ------------------ | -------------------- | ----------------------------------------------- |
| Mobile application | Swift, SwiftUI       | Companion iPhone application                    |
| Smart glasses      | Meta Ray-Ban Display | Heads-up information and interaction            |
| Face recognition   | Apple Vision         | On-device detection and feature matching        |
| Speech recognition | Apple SpeechAnalyzer | Continuous transcription with partial results   |
| AI inference       | Groq, Qwen           | Low-latency question answering                  |
| Contact storage    | Local device storage | Profiles, photos and contextual information     |
| Shared logic       | Swift Package        | Speech processing and question-answering engine |

### Conversation Assistant Implementation

The conversation system is implemented as a Swift package in `Assist/`, shared between the iPhone application and a Mac command-line tool.

| File                             | Function                                                                            |
| -------------------------------- | ----------------------------------------------------------------------------------- |
| `AssistSpeech/Transcriber.swift` | On-device speech transcription using Apple's SpeechAnalyzer                         |
| `AssistCore/QuestionGate.swift`  | Filters partial transcripts and determines when a question is sufficiently complete |
| `AssistCore/Engine.swift`        | Manages early inference, retries and conversation context                           |
| `AssistCore/Answerer.swift`      | Handles local answers and streaming model responses                                 |
| `AssistCore/EventLog.swift`      | Records decisions and timestamps for debugging and evaluation                       |

The engine can produce local answers for simple tasks such as arithmetic, time and date queries. Other factual questions are sent to Groq using `qwen/qwen3.8-27b`.

The model can return `SKIP` for questions it should not answer or `WAIT` when the transcript is too incomplete.

### Latency Optimisation

One of the main engineering challenges was reducing the time between hearing a question and displaying an answer.

Instead of waiting for the full question, FaceTag can initiate inference when the partial transcript contains sufficient information.

In local testing on an M-series Mac mini:

- **11/11** spoken test scenarios passed.
- Model time to first token was approximately **80-170 ms**.
- For some longer questions, answers were available before the speaker had finished their sentence.

These are local test results, not production latency guarantees.

## Running the Project

### iPhone Application

Requires a Mac with Xcode, XcodeGen and an iOS 26-compatible development environment.

Configure the local credentials file:

```bash
cp Secrets.example.xcconfig Secrets.xcconfig
```

Then generate the Xcode project:

```bash
xcodegen
open CalHacks.xcodeproj
```

Run the application on a compatible iPhone. The glasses functionality requires supported Meta Ray-Ban Display hardware and developer configuration.

### Conversation Assistant on Mac

Requires macOS 26 and Swift 6.2+.

To launch the live assistant with a local visualisation and evaluation interface:

```bash
cd Assist
swift run assist live
```

Open `http://localhost:8765`.

The interface shows a simulated glasses display, live transcription, question detection, timing information and model responses.

Other available commands:

```bash
cd Assist

# Live microphone input
swift run assist mic

# Run the spoken test suite
swift run assist suite

# Test a spoken question
swift run assist say "How tall is the Eiffel Tower?"

# Replay a recorded audio file
swift run assist file clip.m4a

# Run offline logic checks
swift run assist-check
```

Model-backed functions require appropriate API credentials. The first microphone run may also require granting microphone permissions.

## Limitations

FaceTag was built as a hackathon prototype and is not a production-ready wearable assistant.

- Facial recognition is limited to enrolled participants with reference photos.
- The conversation assistant currently supports English.
- The documented conversation pipeline does not reliably distinguish between speakers, including the glasses wearer.
- Model-backed answers require network connectivity and may incur API costs.
- The smart glasses integration requires compatible hardware and developer access.

Facial recognition should only be used with the appropriate consent and in accordance with applicable platform requirements.

## Team

Built collaboratively at **Cal Hacks 13.0**, October 2026.

**Contributors:** Toby, Joaquin, Mirabelle and Avi.

[**Official Devpost project submission**](https://devpost.com/software/meta-glasses-v2kp8a)
