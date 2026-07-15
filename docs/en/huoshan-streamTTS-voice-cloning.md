# Smart Console Volcano Double-Stream Speech Synthesis + Voice Cloning Configuration Tutorial

This tutorial is divided into 4 stages: Preparation Stage, Configuration Stage, Cloning Stage, and Usage Stage. It mainly introduces the process of configuring Volcano Double-Stream Speech Synthesis + Voice Cloning through the Smart Console.

## Stage 1: Preparation Stage
The super administrator first needs to activate the Volcano Engine service and obtain the App Id and Access Token. By default, Volcano Engine will provide one voice resource. This voice resource needs to be copied to this project.

If you want to clone multiple voices, you need to purchase and activate multiple voice resources. Just copy each voice resource's voice ID (S_xxxxx) to this project and assign it to system accounts for use. Here are the detailed steps:

### 1. Activate Volcano Engine Service
Visit https://console.volcengine.com/speech/app Create an application in Application Management, check Speech Synthesis Large Model and Voice Replication Large Model.

### 2. Obtain Voice Resource ID
Visit https://console.volcengine.com/speech/service/9999 Copy three items: App Id, Access Token, and Voice ID (S_xxxxx). As shown in the image:

![Obtain Voice Resource](images/image-clone-integration-01.png)

## Stage 2: Configure Volcano Engine Service

### 1. Fill in Volcano Engine Configuration

Using super administrator account to log into Smart Console, click on the top 【Model Configuration】, then click on 【Speech Synthesis】 on the left side of the model configuration page. Search for "Volcano Double-Stream Speech Synthesis", click modify, and fill your Volcano Engine's `App Id` into the 【Application ID】 field, and fill `Access Token` into the 【Access Token】 field. Then save.

### 2. Assign Voice Resource ID to System Account

Using super administrator account to log into Smart Console, click on the top `Parameter Dictionary`, in the dropdown menu, click the `System Function Configuration` page. On the page, check `Voice Cloning`, click save configuration. You can then see the `Voice Cloning` button in the top menu.

Using super administrator account to log into Smart Console, click on the top 【Voice Cloning】、【Voice Resources】.

Click the Add button, select "Volcano Double-Stream Speech Synthesis" in 【Platform Name】;

Enter your Volcano Engine's voice resource ID (S_xxxxx) in 【Voice Resource ID】, and press Enter after entering;

Select the system account you want to assign to in 【Account Ownership】, you can assign it to yourself. Then click Save.

## Stage 3: Cloning Stage

If after logging in, you click on the top 【Voice Cloning】》【Voice Cloning】 and see 【Your account has no voice resources, please contact administrator to assign voice resources】, it means you haven't assigned the voice resource ID to this account in Stage 2. You need to go back to Stage 2 and assign voice resources to the corresponding account.

If after logging in, you click on the top 【Voice Cloning】》【Voice Cloning】 and see the corresponding voice list, please continue.

In the list you will see the corresponding voice list. Select one voice resource, click the 【Upload Audio】 button. After uploading, you can listen to the sound or extract a segment of the sound. After confirmation, click the 【Upload Audio】 button.

![Upload Audio](images/image-clone-integration-02.png)

After uploading audio, in the list you will see the corresponding voice will become "Pending Replication" status. Click the 【Replicate Now】 button. Wait 1-2 seconds for the result to return.

If replication fails, place your mouse over the "Error Message" icon to see the reason for failure.

If replication is successful, in the list you will see the corresponding voice will become "Training Successful" status. At this time you can click the modify button in the 【Voice Name】 column to modify the name of the voice resource for easy selection and use later.

## Stage 4: Usage Stage

Click on the top 【Agent Management】, select any agent, click the 【Configure Role】 button.

Speech Synthesis (TTS) select "Volcano Double-Stream Speech Synthesis". In the list, find the voice resource with "Cloned Voice" in the name (as shown in the figure), select it and click save.

![Select Voice](images/image-clone-integration-03.png)

Next, you can wake up Xiao Zhi and have a conversation with it.

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.