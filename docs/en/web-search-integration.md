# Online Search Plugin Usage Guide

## Feature Overview

The online search plugin `web_search` supports real-time online searching for information during conversations and returning the results. The plugin supports two search sources: Metaso and Tavily, allowing users to choose one based on their needs.

## API Key Application Guide

We currently support `Metaso Search` and `Tavily Search`.
- Tavily Search: 1000 free searches per month.
- Metaso Search: Offers high-quality domestic data sources.

## API Key Application Guide

### Method 1: Using Metaso Search

- Visit [Metaso Search API](https://metaso.cn/search-api/api-keys), register and log into your account
- On the API key management page, click "Create New Key"
- Copy the generated API Key (prefixed with `mk-`), this is the key information needed for configuration

### Method 2: Using Tavily Search

- Visit [Tavily Console](https://app.tavily.com/home), register and log into your account
- Create an API Key in the console
- Copy the generated API Key (prefixed with `tvly-`), this is the key information needed for configuration

## Configuration Methods

### Method 1. Using Xiaozhi Console Deployment (Recommended)

- Log into Xiaozhi Console
- Enter the "Configure Role" page and select the agent to configure
- Click the "Edit Functions" button, and find the "Online Search" plugin in the parameter configuration area on the right
- Check "Online Search"
- Fill in the search source (`metaso` or `tavily`) and the corresponding `API Key` into the configuration fields
- Save the configuration, then save the agent configuration

### Method 2. Single Module xiaozhi-server Deployment

Configure in `data/.config.yaml`:

- Fill the search source into `provider`, with optional values `metaso` or `tavily`
- Fill the acquired API Key into `api_key`

```yaml
plugins:
  web_search:
    provider: "metaso"
    api_key: "Your API Key"
```

To customize the number of returned results and tool description, additional configuration of `max_results` and `description` is possible:

```yaml
plugins:
  web_search:
    provider: "metaso"
    description: "Online search tool. Use this tool when users explicitly need online search for questions."
    max_results: 5
    api_key: "Your API Key"
```

Also, ensure `web_search` is enabled in the `functions` list:

```yaml
plugins:
  functions:
    - web_search
```

After configuration, restart the service to take effect.

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.