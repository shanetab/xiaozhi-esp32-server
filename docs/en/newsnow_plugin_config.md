# get_news_from_newsnow Plugin News Source Configuration Guide

## Overview

The `get_news_from_newsnow` plugin now supports dynamic configuration of news sources through the web management interface, without requiring code modifications. Users can configure different news sources for each agent in the Smart Console.

## Configuration Method

### 1. Configure via Web Management Interface (Recommended)

1. Log into the Smart Console
2. Go to the "Role Configuration" page
3. Select the agent to configure
4. Click the "Edit Functions" button
5. In the right parameter configuration area, find the "newsnow news aggregation" plugin
6. Enter the Chinese names separated by semicolons in the "News Source Configuration" field

### 2. Configuration File Method

Configure in `config.yaml`:

```yaml
plugins:
  get_news_from_newsnow:
    url: "https://newsnow.busiyi.world/api/s?id="
    news_sources: "ThePaper;Baidu Hot Search;CaiLian;Weibo;Douyin"
```

## News Source Configuration Format

The news source configuration uses Chinese names separated by semicolons, in the format:

```
Chinese Name 1;Chinese Name 2;Chinese Name 3
```

### Configuration Example

```
ThePaper;Baidu Hot Search;CaiLian;Weibo;Douyin;Zhihu;36Kr
```

## Supported News Sources

The plugin supports the following Chinese names for news sources:

- ThePaper
- Baidu Hot Search
- CaiLian
- Weibo
- Douyin
- Zhihu
- 36Kr
- Wall Street Seen
- IT Home
- Toutiao
- Hupu
- Bilibili
- Kuaishou
- Xueqiu
- Gelonghui
- FaBu Finance
- Jinshi Data
- Niuke
- Shaoershou
- Xi Dou Jin
- Phoenix News
- Chongbuluo
- Lianhe Zaobao
- Kouan
- Yuanjing Forum
- Reference News
- Satellite News
- Baidu Tieba
- Kaopu News
- And more...

## Default Configuration

If no news sources are configured, the plugin will use the following default configuration:

```
ThePaper;Baidu Hot Search;CaiLian
```

## Usage Instructions

1. **Configure News Sources**: Set the Chinese names of news sources in the web interface or configuration file, separated by semicolons
2. **Invoke Plugin**: Users can say "report news" or "get news"
3. **Specify News Source**: Users can say "report ThePaper" or "get Baidu Hot Search"
4. **Get Details**: Users can say "give detailed information about this news"

## How It Works

1. The plugin accepts Chinese names as parameters (such as "ThePaper")
2. Based on the configured news source list, it converts the Chinese name to the corresponding English ID (such as "thepaper")
3. It calls the API with the English ID to retrieve news data
4. It returns the news content to the user

## Important Notes

1. The configured Chinese names must exactly match the names defined in CHANNEL_MAP
2. Configuration changes require restarting the service or reloading the configuration
3. If configured news sources are invalid, the plugin will automatically use the default news sources
4. Multiple news sources are separated by English semicolons(;), not Chinese semicolons(；)

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.