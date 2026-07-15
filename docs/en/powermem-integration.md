# PowerMem Memory Component Integration Guide

## Introduction

[PowerMem](https://www.powermem.ai/) is an Agent memory component open-sourced by OceanBase, which uses local LLM for memory summarization and intelligent retrieval, providing efficient memory management functions for AI agents.

Cost Explanation: PowerMem itself is open-source and free, actual costs depend on the LLM and database you choose:
- Using SQLite + free LLM (such as Zhipu glm-4-flash) = **Completely free**
- Using cloud LLM or cloud database = charged according to corresponding service

> 💡 **Best Performance Tip**: PowerMem paired with OceanBase can achieve maximum performance release, SQLite is only recommended when resources are insufficient.

- **GitHub**: https://github.com/oceanbase/powermem
- **Official Website**: https://www.powermem.ai/
- **Usage Examples**: https://github.com/oceanbase/powermem/tree/main/examples

## Features

- **Local Summarization**: Memory summarization and extraction through LLM locally
- **User Profile**: Automatically extracts user information (name, profession, interests, etc.) through `UserMemory`, continuously updating user profiles
- **Intelligent Forgetting**: Automatically "forgets" outdated noise information based on Ebbinghaus forgetting curve
- **Multiple Storage Backends**: Supports OceanBase (recommended, best performance), SeekDB (recommended, AI application storage integration), PostgreSQL, SQLite (lightweight alternative)
- **Multiple LLM Support**: Qwen, Zhipu (glm-4-flash free), OpenAI, etc.
- **Intelligent Retrieval**: Semantic retrieval capability based on vector search
- **Private Deployment**: Fully supports local private deployment
- **Asynchronous Operations**: Efficient asynchronous memory management

## Installation

PowerMem has been added to project dependencies. If manual installation is needed:

```bash
pip install powermem
```

## Configuration Instructions

### Basic Configuration

Configure PowerMem in `config.yaml`:

```yaml
selected_module:
  Memory: powermem

Memory:
  powermem:
    type: powermem
    # Whether to enable user profile function
    # User profile support: oceanbase, seekdb, sqlite (powermem 0.3.0+)
    enable_user_profile: true
    
    # ========== LLM Configuration ==========
    llm:
      provider: openai  # Optional: qwen, openai, zhipu, etc.
      config:
        api_key: Your LLM API key
        model: qwen-plus
        # openai_base_url: https://api.openai.com/v1  # Optional, custom service address
    
    # ========== Embedding Configuration ==========
    embedder:
      provider: openai  # Optional: qwen, openai, etc.
      config:
        api_key: Your embedding model API key
        model: text-embedding-v4
        openai_base_url: https://dashscope.aliyuncs.com/compatible-mode/v1
        # embedding_dims: 1024  # Vector dimensions, configure when not 1536
    
    # ========== Database Configuration ==========
    vector_store:
      provider: sqlite  # Optional: oceanbase (recommended), seekdb (recommended), postgres, sqlite (lightweight)
      config: {}  # SQLite requires no additional configuration
```

### Configuration Parameter Details

#### LLM Configuration

| Parameter | Description | Options |
|-----------|-------------|---------|
| `llm.provider` | LLM provider | `qwen`, `openai`, `zhipu`, etc. |
| `llm.config.api_key` | API key | - |
| `llm.config.model` | Model name | Choose according to provider |
| `llm.config.openai_base_url` | Custom service address (optional) | - |

#### Embedding Configuration

| Parameter | Description | Options |
|-----------|-------------|---------|
| `embedder.provider` | Embedding model provider | `qwen`, `openai`, etc. |
| `embedder.config.api_key` | API key | - |
| `embedder.config.model` | Model name | Choose according to provider |
| `embedder.config.openai_base_url` | Custom service address (optional) | - |

#### Database Configuration

| Parameter | Description | Options |
|-----------|-------------|---------|
| `vector_store.provider` | Storage backend type | `oceanbase` (recommended), `seekdb` (recommended), `postgres`, `sqlite` (lightweight) |
| `vector_store.config` | Database connection configuration | Set according to provider |

### Memory Mode Explanation

PowerMem supports two memory modes:

| Mode | Configuration | Function | Storage Requirements |
|------|-------------|----------|---------------------|
| **Normal Memory** | `enable_user_profile: false` | Conversation memory storage and retrieval | Supports all databases |
| **User Profile** | `enable_user_profile: true` | Memory + automatic user profile extraction | oceanbase, seekdb, sqlite |

> 📌 **Version Note**: PowerMem 0.3.0+ version supports user profile functionality with OceanBase, SeekDB, and SQLite storage backends.

### Using Qwen (Recommended)

1. Visit [Alibaba Cloud Bailian Platform](https://bailian.console.aliyun.com/) to register an account
2. Get API key from [API Key Management](https://bailian.console.aliyun.com/?apiKey=1#/api-key) page
3. Configure as follows:

```yaml
Memory:
  powermem:
    type: powermem
    enable_user_profile: true
    llm:
      provider: qwen
      config:
        api_key: sk-xxxxxxxxxxxxxxxx
        model: qwen-plus
    embedder:
      provider: openai
      config:
        api_key: sk-xxxxxxxxxxxxxxxx
        model: text-embedding-v4
        openai_base_url: https://dashscope.aliyuncs.com/compatible-mode/v1
    vector_store:
      provider: sqlite
      config: {}
```

### Using Zhipu Free LLM (Completely Free Option)

Zhipu provides a free glm-4-flash model, combined with SQLite to achieve completely free usage:

1. Visit [Zhipu AI Open Platform](https://bigmodel.cn/) to register an account
2. Get API key from [API Keys](https://bigmodel.cn/usercenter/proj-mgmt/apikeys) page
3. Configure as follows:

```yaml
Memory:
  powermem:
    type: powermem
    enable_user_profile: true
    llm:
      provider: openai  # Using openai compatibility mode
      config:
        api_key: xxxxxxxxxxxxxxxx.xxxxxxxxxxxxxxxx
        model: glm-4-flash
        openai_base_url: https://open.bigmodel.cn/api/paas/v4/
    embedder:
      provider: openai
      config:
        api_key: xxxxxxxxxxxxxxxx.xxxxxxxxxxxxxxxx
        model: embedding-3
        openai_base_url: https://open.bigmodel.cn/api/paas/v4/
    vector_store:
      provider: sqlite
      config: {}
```

### Using OpenAI

```yaml
Memory:
  powermem:
    type: powermem
    enable_user_profile: true
    llm:
      provider: openai
      config:
        api_key: sk-xxxxxxxxxxxxxxxx
        model: gpt-4o-mini
        openai_base_url: https://api.openai.com/v1
    embedder:
      provider: openai
      config:
        api_key: sk-xxxxxxxxxxxxxxxx
        model: text-embedding-3-small
        openai_base_url: https://api.openai.com/v1
    vector_store:
      provider: sqlite
      config: {}
```

### Using OceanBase (Best Performance Option)

OceanBase is PowerMem's best match, achieving maximum performance release:

1. Deploy OceanBase database (supports open-source local deployment or cloud service)
   - Open-source deployment: https://github.com/oceanbase/oceanbase
   - Cloud service: https://www.oceanbase.com/
2. Configure as follows:

```yaml
Memory:
  powermem:
    type: powermem
    enable_user_profile: true
    llm:
      provider: qwen
      config:
        api_key: sk-xxxxxxxxxxxxxxxx
        model: qwen-plus
    embedder:
      provider: openai
      config:
        api_key: sk-xxxxxxxxxxxxxxxx
        model: text-embedding-v4
        openai_base_url: https://dashscope.aliyuncs.com/compatible-mode/v1
    vector_store:
      provider: oceanbase
      config:
        host: 127.0.0.1
        port: 2881
        user: root@test
        password: your_password
        db_name: powermem
        collection_name: memories  # Default value
        embedding_model_dims: 1536  # Embedding vector dimensions, required parameter
```

## Device Memory Isolation

PowerMem automatically uses device ID (`device_id`) as `user_id` for memory isolation. This means:

- Each device has an independent memory space
- Memories between different devices are completely isolated
- Multiple conversations from the same device can share memory context

## User Profile (UserMemory)

PowerMem provides the `UserMemory` class that can automatically extract user profile information from conversations.

> 📌 **Version Note**: PowerMem 0.3.0+ version supports user profile functionality with OceanBase, SeekDB, and SQLite storage backends.

### Enabling User Profile

Set `enable_user_profile: true` in configuration to enable:

```yaml
Memory:
  powermem:
    type: powermem
    enable_user_profile: true  # Enable user profile
    llm:
      provider: qwen
      config:
        api_key: sk-xxxxxxxxxxxxxxxx
        model: qwen-plus
    embedder:
      provider: openai
      config:
        api_key: sk-xxxxxxxxxxxxxxxx
        model: text-embedding-v4
        openai_base_url: https://dashscope.aliyuncs.com/compatible-mode/v1
    vector_store:
      provider: sqlite  # User profile support: oceanbase, seekdb, sqlite
      config: {}
```

### User Profile Capabilities

| Capability | Description |
|------------|-------------|
| **Information Extraction** | Automatically extracts names, ages, professions, interests, etc. |
| **Continuous Updates** | Continuously improves user profiles as conversations progress |
| **Profile Retrieval** | Combines user profiles with memory searches to improve retrieval relevance |
| **Intelligent Forgetting** | Based on Ebbinghaus forgetting curve, fades outdated information |

### Working Principle

When user profile is enabled, XiaoZhi automatically returns when querying memory:
1. **User Profile**: Basic user information, hobbies, etc.
2. **Related Memory**: Historical memory related to current conversation

> ✅ **Version Note**: PowerMem 0.3.0+ version supports user profile functionality with OceanBase, SeekDB, and SQLite storage backends.

## Comparison with Other Memory Components

| Feature | PowerMem | mem0ai | mem_local_short |
|---------|----------|--------|-----------------|
| Working Method | Local Summarization | Cloud Interface | Local Summarization |
| Storage Location | Local/Cloud DB | Cloud | Local YAML |
| Cost | Depends on LLM and DB | 1000 times/month free | Completely Free |
| Intelligent Retrieval | ✅ Vector Search | ✅ Vector Search | ❌ Full Return |
| User Profile | ✅ UserMemory | ❌ | ❌ |
| Intelligent Forgetting | ✅ Forgetting Curve | ❌ | ❌ |
| Private Deployment | ✅ Supported | ❌ Cloud Only | ✅ Supported |
| Database Support | OceanBase (Recommended)/SeekDB/PostgreSQL/SQLite | - | YAML File |

## Troubleshooting

### 1. API Key Error

If you encounter `API key is required` error, please check:
- Whether `llm_api_key` and `embedding_api_key` are correctly filled
- Whether API key is valid

### 2. Model Not Found

If you encounter model not found error, please confirm:
- Whether `llm_model` and `embedding_model` names are correct
- Whether corresponding model services are activated

### 3. Connection Timeout

If you encounter connection timeout, you can try:
- Checking network connection
- If using proxy, configure `llm_base_url` and `embedding_base_url`

## Testing Verification

You can test if PowerMem works properly in a virtual environment:

```bash
# Activate virtual environment
source .venv/bin/activate

# Test PowerMem import
python -c "from powermem import AsyncMemory; print('PowerMem import successful')"

# Test UserMemory import (user profile function)
python -c "from powermem import UserMemory; print('UserMemory import successful')"
```

## More Resources

- [PowerMem Official Documentation](https://www.powermem.ai/)
- [PowerMem GitHub Repository](https://github.com/oceanbase/powermem)
- [PowerMem Usage Examples](https://github.com/oceanbase/powermem/tree/main/examples)
- [OceanBase Official Website](https://www.oceanbase.com/)
- [OceanBase GitHub](https://github.com/oceanbase/oceanbase)
- [SeekDB GitHub](https://github.com/oceanbase/seekdb) (AI-native search database)
- [Alibaba Cloud Bailian Platform](https://bailian.console.aliyun.com/)

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.