---
name: ai-apis
description: How to use AI APIs like OpenAI, ChatGPT, Elevenlabs, etc. When a user asks you to make an app that requires an AI API, use this skill to understand how to use the API or how to respond to the user.
---

# ai-apis-like-chatgpt

## Instructions
The Vibecode Environment comes pre-installed with a lot of AI APIs like OpenAI, ChatGPT, Elevenlabs, etc. You can use these APIs to generate text, images, videos, sounds, etc.

Users can find most of the APIs in the API tab of the Vibecode App. You can tell the user to look there for any custom or advanced API integrations.

However, we will go over the basic OpenAI APIs.

## Examples

For all APIs below, use standard `fetch`. Write logic in `./src/lib/openai.ts`.

### Responses API (Generate text, analyze images, search the web)

You can use the OpenAI Responses API to generate text, search the web, and analyze images. The latest model family is `gpt-5.2` as of December 2025. Docs: https://platform.openai.com/docs/api-reference/responses/create

**Basic request:**
```typescript
const response = await fetch("https://api.openai.com/v1/responses", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}`,
  },
  body: JSON.stringify({ model: "gpt-5.2", input: "Your prompt here" }),
});
const data = await response.json();
const text = data.output_text;
```

**Streaming:** Add `stream: true` to the body. Parse SSE events from the response body:
```typescript
const response = await fetch("https://api.openai.com/v1/responses", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}`,
  },
  body: JSON.stringify({ model: "gpt-5.2", input: "Your prompt here", stream: true }),
});

const reader = response.body?.getReader();
const decoder = new TextDecoder();
while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  // Parse lines starting with "data: " and look for event.type === "response.output_text.delta"
  // The delta text is in event.delta
}
```

**Vision (Image Analysis):** Use file input to select images and convert to base64:
```typescript
// In your component
const handleFileChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
  const file = e.target.files?.[0];
  if (!file) return;

  // Convert to base64
  const reader = new FileReader();
  reader.onloadend = async () => {
    const base64 = reader.result as string; // Already includes data:image/...;base64, prefix

    // Send to vision API
    const response = await fetch("https://api.openai.com/v1/responses", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}`,
      },
      body: JSON.stringify({
        model: "gpt-5.2",
        input: [{
          role: "user",
          content: [
            { type: "input_text", text: "What's in this image?" },
            { type: "input_image", image_url: base64 },
          ],
        }],
      }),
    });
    const data = await response.json();
    console.log(data.output_text);
  };
  reader.readAsDataURL(file);
};

// JSX
<input type="file" accept="image/*" onChange={handleFileChange} />
```

### Image Generation API (Generate images)

Model: `gpt-image-1`. Docs: https://platform.openai.com/docs/api-reference/images/create

```typescript
const response = await fetch("https://api.openai.com/v1/images/generations", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}`,
  },
  body: JSON.stringify({
    model: "gpt-image-1",
    prompt: "A cute baby sea otter",
    n: 1,
    size: "1024x1024",
  }),
});
const data = await response.json();
const imageUrl = data.data[0].url; // or data.data[0].b64_json for base64
```

### Image Edit API (Edit images)

Model: `gpt-image-1`. Docs: https://platform.openai.com/docs/api-reference/images/createEdit

```typescript
const handleEditImage = async (file: File) => {
  const formData = new FormData();
  formData.append("image", file);
  formData.append("prompt", "Add a hat to the person");
  formData.append("model", "gpt-image-1");
  formData.append("n", "1");
  formData.append("size", "1024x1024");

  const response = await fetch("https://api.openai.com/v1/images/edits", {
    method: "POST",
    headers: { Authorization: `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}` },
    body: formData,
  });
  const data = await response.json();
  const editedImageUrl = data.data[0].url;
};
```

### Audio Transcription API (Transcribe audio)

Model: `gpt-4o-transcribe`. Docs: https://platform.openai.com/docs/api-reference/audio/create

```typescript
const handleTranscribe = async (audioFile: File) => {
  const formData = new FormData();
  formData.append("file", audioFile);
  formData.append("model", "gpt-4o-transcribe");

  const response = await fetch("https://api.openai.com/v1/audio/transcriptions", {
    method: "POST",
    headers: { Authorization: `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}` },
    body: formData,
  });
  const data = await response.json();
  const transcription = data.text;
};

// JSX for file input
<input type="file" accept="audio/*" onChange={(e) => {
  const file = e.target.files?.[0];
  if (file) handleTranscribe(file);
}} />
```

### Text-to-Speech API (Generate audio)

Model: `gpt-4o-mini-tts`. Docs: https://platform.openai.com/docs/api-reference/audio/createSpeech

```typescript
const handleTextToSpeech = async (text: string) => {
  const response = await fetch("https://api.openai.com/v1/audio/speech", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}`,
    },
    body: JSON.stringify({
      model: "gpt-4o-mini-tts",
      input: text,
      voice: "alloy", // Options: alloy, echo, fable, onyx, nova, shimmer
    }),
  });

  // Create audio blob and play it
  const audioBlob = await response.blob();
  const audioUrl = URL.createObjectURL(audioBlob);
  const audio = new Audio(audioUrl);
  audio.play();
};
```

## Environment Variables

Store your API keys in a `.env` file:
```
VITE_OPENAI_API_KEY=your-api-key-here
```

Access them in code with `import.meta.env.VITE_OPENAI_API_KEY`.

**Note:** For production apps, consider proxying API calls through a backend to avoid exposing API keys in the browser.
