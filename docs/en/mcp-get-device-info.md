# How MCP Methods Can Get Device Information

This tutorial will guide you on how to use MCP methods to get device information.

Step 1: Customize your `agent-base-prompt.txt` file

Copy the content of the `agent-base-prompt.txt` file from the xiaozhi-server directory to your `data` directory and rename it to `.agent-base-prompt.txt`.

Step 2: Modify the `data/.agent-base-prompt.txt` file, find the `<context>` tag, and add the following code content within the tag content:
```
- **Device ID:** {{device_id}}
```

After adding, your `data/.agent-base-prompt.txt` file's `<context>` tag content should look roughly like this:
```
<context>
【Important! The following information is provided in real-time, no tool queries needed, please use directly:】
- **Device ID:** {{device_id}}
- **Current Time:** {{current_time}}
- **Today's Date:** {{today_date}} ({{today_weekday}})
- **Today's Lunar Date:** {{lunar_date}}
- **User's Location City:** {{local_address}}
- **Local Weather for Next 7 Days:** {{weather_info}}
</context>
```

Step 3: Modify the `data/.config.yaml` file, find the `agent-base-prompt` configuration, and change the content before modification:
```
prompt_template: agent-base-prompt.txt
```
Change to
```
prompt_template: data/.agent-base-prompt.txt
```

Step 4: Restart your xiaozhi-server service.

Step 5: In your MCP method, add a parameter named `device_id`, type `string`, description `Device ID`.

Step 6: Reactivate Xiao Zhi to have him call the MCP method and check if your MCP method can obtain the `Device ID`.

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.