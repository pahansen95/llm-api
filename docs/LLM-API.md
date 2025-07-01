# Universal LLM API

The universal LLM API provides a single consistent API for interacting with multiple LLM Providers.

## User Expectations

From the user (developer) experience, we mainly want to:

1. Send the model prepared inputs (text, image, video, etc...)
2. Recieve the generated outputs (text, image, video, etc...)

We want the API to be straightforward & cognitively simple to understand while still affording us granular control every so often. We also want the freedom to use any model from any provider. Integration of LLMs into our software systems should be easy; we don't want to setup entire infrastructure just to ask "is this a hotdog?".

## Design

We open with a brief overview of our mental models of integrating LLMs into our software:

1. You "chat" with models; this implies a sustained conversational state. Models can also "take action". Simply put, Models can invoke tool calls that either observe the world or produce side effects.
2. Conversations should include Text (Natural or Structured), Images & Audio. Tool invocations require structured inputs & (mostly) produce structured outputs. Tools invocations are embedded as part of a conversation; the model either chooses to, or is directed by the user, to take action.
3. There are many different LLMs all having different internal functionality. There are many different LLM Providers, each having their own way of doing things. By & large, we treat all models functionally the same, but select which models to use based on A) the task at hand, B) ease of use & C) costs.

We now generally articulate the expected system structural & behavioral layout of the Universal API:

- At startup, we configure the API with the set of Model Providers, Model Kinds, Model Instances & Model Configurations:
  - A Model Provider is some local or remote system that operates & provides programmatic access to LLMs
  - A Model Kind is a named LLM having some set of functionality; usually classified within a "family" or "type" of models.
  - A Model Instance is a deployed Model Kind that can be "chatted" with; the exact details of how an Instance are provided are irrelevant; we only care about the functional facade.
  - A Model Configuration is a specific declaration of runtime state for a Model Instance; it is applied when "chatting" with the model
- Model Providers
  - Own the Resources & Lifetimes associated with Model Instances
  - Initialize & Teardown Resources
  - Manage sessions with their appropriate backend
  - Decouple the application integration patterns from the backend implementation
- During exectuion of our business logic:
  - We maintian the state of our chat in a "Conversation".
  - We chat with multiple Model Instances having multiple Model Configurations.
- Conversations...
  - track all discrete "chats" we had; each "chat" would consist of chat inputs & output(s), & chat metadata including provider, model, config, timestamps, etc...
  - track the various sequences of the conversations: ie. a log of chats stringed together. This sequencing forms a DAG, where every path is a single "chat log". We may need to branch out from a single chat into multiple paths.
- A "LLM API" will provide
  - Realtime Prompting interfaces including chunked & streaming response
  - Batched & Asynchronous Prompting interfaces
    - State of inflight prompts will be saved to a repository.
  - Model Query & Inspection
  - Ability to contextually apply Model Configurations to Model Instances
- The State Repository
  - Persists Conversational State & Inflight Prompts.
  - Decouples CRUD & persistence of state through.

The LLM API is both a Local Library & a Remote API:

- The Library can be imported directly from an application for direct integration
- The Library can run in "remote" mode to provide IPC capabilities:
  - Stateful API mirroring library functionality
  - OpenAI API Compatibility Shim
  - HTTP, Unix Sockets, Local IPC, etc...