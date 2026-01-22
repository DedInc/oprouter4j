# OpRouter4j

[![JitPack](https://jitpack.io/v/DedInc/oprouter4j.svg)](https://jitpack.io/#DedInc/oprouter4j)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A Java client for the OpenRouter API. This library handles the HTTP layer, conversation history management, and retry logic, allowing integration with OpenRouter-supported models without dealing with raw JSON requests.

## Installation

This library is hosted on JitPack. You need to add the JitPack repository to your build file.

### Gradle (Kotlin DSL)

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        mavenCentral()
        maven { url = uri("https://jitpack.io") }
    }
}

// build.gradle.kts
dependencies {
    implementation("com.github.DedInc:oprouter4j:1.0")
}
```

### Maven

```xml
<repositories>
    <repository>
        <id>jitpack.io</id>
        <url>https://jitpack.io</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>com.github.DedInc</groupId>
        <artifactId>oprouter4j</artifactId>
        <version>1.0</version>
    </dependency>
</dependencies>
```

## Configuration

The client looks for environment variables or a `.env` file in the project root.

**Required:**
```bash
OPENROUTER_API_KEY=sk-or-v1-...
```

**Optional defaults:**
```bash
DEFAULT_MODEL=x-ai/grok-4-fast:free
BASE_URL=https://openrouter.ai/api/v1
MAX_REQUESTS_PER_MINUTE=60
MAX_CONCURRENT_REQUESTS=5
MAX_RETRIES=5
LOG_LEVEL=INFO
STORAGE_TYPE=file 
```

## Usage

### Synchronous Chat

Basic implementation using `OpenRouterClient`. Note that the response data is returned as a raw Map/List structure.

```java
import com.github.dedinc.oprouter4j.client.OpenRouterClient;
import com.github.dedinc.oprouter4j.model.APIResponse;
import java.util.*;

public class Main {
    public static void main(String[] args) {
        try (OpenRouterClient client = new OpenRouterClient()) {
            List<Map<String, String>> messages = new ArrayList<>();
            Map<String, String> message = new HashMap<>();
            
            message.put("role", "user");
            message.put("content", "Print hello world in Java");
            messages.add(message);
            
            APIResponse response = client.chatCompletion(messages);
            
            if (response.isSuccess()) {
                // Parse the nested structure
                List<Map<String, Object>> choices = (List<Map<String, Object>>) response.getData().get("choices");
                Map<String, Object> msg = (Map<String, Object>) choices.get(0).get("message");
                System.out.println(msg.get("content"));
            } else {
                System.err.println("Error: " + response.getMessage());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Streaming

For real-time output generation.

```java
client.chatCompletionStream(messages, chunk -> {
    System.out.print(chunk);
});
```

### Conversation Management

The library includes a `Conversation` class to handle context windows and history.

```java
import com.github.dedinc.oprouter4j.storage.Conversation;
import com.github.dedinc.oprouter4j.model.MessageRole;

// Initialize conversation (id, title, specific model)
Conversation conv = new Conversation(null, "DevChat", "anthropic/claude-3-haiku");

// Add user message
conv.addMessage(MessageRole.USER, "Explain the volatile keyword");

// Send context to API
APIResponse response = client.chatCompletion(conv.getMessagesForApi());

// Append response to history
if (response.isSuccess()) {
    String content = extractContent(response); // helper method to parse JSON
    conv.addMessage(MessageRole.ASSISTANT, content);
    
    // Persist to disk (uses json storage by default)
    conv.save();
}
```

## Error Handling

The client implements exponential backoff for rate limits (HTTP 429) and network failures. 

*   **Max Retries:** Configurable via `MAX_RETRIES` (default: 5).
*   **Rate Limits:** Respects OpenRouter headers.

## Building from Source

Requirements: Java 11+.

```bash
git clone https://github.com/DedInc/oprouter4j.git
cd oprouter4j
./gradlew build
```

## License

MIT License. See [LICENSE](LICENSE) for details.
