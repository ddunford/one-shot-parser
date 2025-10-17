# ADR-0001: Adapter Pattern for LLM Integration

**Status**: Accepted

**Date**: 2025-10-16

**Decision Makers**: Engineering Team

---

## Context

The CV Parser needs to support multiple LLM providers (Ollama, OpenAI, AWS Bedrock) for parsing CV documents. Each provider has a different API, authentication mechanism, and response format. We need a flexible architecture that allows:

1. Easy switching between providers without code changes
2. Adding new providers without modifying existing code
3. Consistent behavior regardless of provider
4. Testability with mock providers

##

 Decision

We will use the **Adapter Pattern** to abstract LLM provider differences behind a common interface.

**Implementation**:
- Define `ILLMAdapter` interface with `parseCV()`, `getProviderInfo()`, and `checkHealth()` methods
- Implement provider-specific adapters: `OllamaAdapter`, `OpenAIAdapter`, `BedrockAdapter`
- Use `AdapterFactory` to instantiate the correct adapter based on configuration
- All adapters return the same `CVData` schema regardless of internal provider format

---

## Consequences

### Positive

✅ **Provider Independence**: Application logic doesn't depend on specific LLM APIs
✅ **Easy Extension**: New providers require only a new adapter class
✅ **Testability**: Mock adapters can be injected for testing
✅ **Maintainability**: Changes to one provider don't affect others
✅ **Flexibility**: Users can switch providers at runtime

### Negative

❌ **Abstraction Overhead**: Extra layer between application and LLM APIs
❌ **Prompt Inconsistency Risk**: Each provider may interpret prompts differently
❌ **Testing Complexity**: Must test each adapter independently

### Neutral

🔵 **Performance**: Minimal overhead (single function call indirection)
🔵 **Learning Curve**: Developers must understand adapter pattern

---

## Alternatives Considered

### Alternative 1: Direct API Integration

**Approach**: Call Ollama/OpenAI/Bedrock APIs directly in business logic

**Rejected Because**:
- Tight coupling to specific providers
- Difficult to switch providers
- Hard to test without real API calls
- Violates Open/Closed Principle

### Alternative 2: Strategy Pattern with Separate Services

**Approach**: Create `OllamaService`, `OpenAIService`, `BedrockService` as separate services

**Rejected Because**:
- More complex than needed
- No clear benefit over Adapter Pattern
- Services imply broader responsibilities beyond API adaptation

### Alternative 3: Plugin System

**Approach**: Dynamically load provider implementations as plugins

**Rejected Because**:
- Over-engineered for 3 known providers
- Adds deployment complexity
- MVP doesn't require runtime plugin loading

---

## Implementation Notes

**Interface** (src/backend/services/llm/ILLMAdapter.ts):
```typescript
interface ILLMAdapter {
  parseCV(text: string, options?: ParseOptions): Promise<CVData>;
  getProviderInfo(): ProviderInfo;
  checkHealth(): Promise<boolean>;
}
```

**Factory** (src/backend/services/llm/AdapterFactory.ts):
```typescript
class AdapterFactory {
  getAdapter(provider: string): ILLMAdapter {
    switch (provider) {
      case 'ollama': return new OllamaAdapter(config.ollama);
      case 'openai': return new OpenAIAdapter(config.openai);
      case 'bedrock': return new BedrockAdapter(config.bedrock);
      default: throw new Error(`Unknown provider: ${provider}`);
    }
  }
}
```

---

## References

- ARCHITECTURE.md § LLM Adapter (Strategy Pattern)
- Gang of Four Design Patterns: Adapter Pattern
- Martin Fowler: Gateway Pattern (similar concept)

---

**Related ADRs**:
- ADR-0009: Prompt Engineering Strategy (depends on this)
- ADR-0006: Express Backend Framework (context for this decision)
