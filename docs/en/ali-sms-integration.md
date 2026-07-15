# Alibaba Cloud SMS Integration Guide

Login to the Alibaba Cloud console and go to the "SMS Service" page: https://dysms.console.aliyun.com/overview

## Step 1 Add Signature
![Step](images/alisms/sms-01.png)
![Step](images/alisms/sms-02.png)

The above steps will obtain a signature. Please write it into the smart control panel parameter, `aliyun.sms.sign_name`

## Step 2 Add Template
![Step](images/alisms/sms-11.png)

The above steps will obtain a template code. Please write it into the smart control panel parameter, `aliyun.sms.sms_code_template_code`

Note: The signature needs 7 working days to be effective. It can only be sent successfully after the operator has completed the filing.

Note: The signature needs 7 working days to be effective. It can only be sent successfully after the operator has completed the filing.

Note: The signature needs 7 working days to be effective. It can only be sent successfully after the operator has completed the filing.

You can proceed with the following operations after the filing is successful.

## Step 3 Create SMS Account and Enable Permissions
Login to the Alibaba Cloud console and go to the "Access Control" page: https://ram.console.aliyun.com/overview?activeTab=overview

![Step](images/alisms/sms-21.png)
![Step](images/alisms/sms-22.png)
![Step](images/alisms/sms-23.png)
![Step](images/alisms/sms-24.png)
![Step](images/alisms/sms-25.png)

The above steps will obtain access_key_id and access_key_secret. Please write them into the smart control panel parameters, `aliyun.sms.access_key_id` and `aliyun.sms.access_key_secret`

## Step 4 Enable Mobile Registration Function

1. Normally, after all the above information is filled in, there will be this effect. If not, a step may be missing.
![Step](images/alisms/sms-31.png)

2. Enable non-administrator users to register. Set the parameter `server.allow_user_register` to `true`

3. Enable mobile registration function. Set the parameter `server.enable_mobile_register` to `true`
![Step](images/alisms/sms-32.png)

> [!NOTE] 
> This translation requires manual review as it involves technical documentation that may benefit from human verification for accuracy.