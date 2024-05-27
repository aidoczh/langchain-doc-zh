# Twilio

本文介绍如何使用[Twilio](https://www.twilio.com) API包装器通过短信或[Twilio消息通道](https://www.twilio.com/docs/messaging/channels)发送消息。

Twilio消息通道可以方便地与第三方消息应用集成，并允许您通过WhatsApp Business平台（GA）、Facebook Messenger（公测版）和Google Business消息（私测版）发送消息。

## 设置

要使用此工具，您需要安装Python Twilio包`twilio`

```python
%pip install --upgrade --quiet  twilio
```

您还需要设置Twilio帐户并获取您的凭据。您将需要您的帐户字符串标识符（SID）和您的身份验证令牌。您还需要一个用于发送消息的号码。

您可以将这些作为命名参数`account_sid`，`auth_token`，`from_number`传递给TwilioAPIWrapper，或者您可以设置环境变量`TWILIO_ACCOUNT_SID`，`TWILIO_AUTH_TOKEN`，`TWILIO_FROM_NUMBER`。

## 发送短信

```python
from langchain_community.utilities.twilio import TwilioAPIWrapper
```

```python
twilio = TwilioAPIWrapper(
    #     account_sid="foo",
    #     auth_token="bar",
    #     from_number="baz,"
)
```

```python
twilio.run("hello world", "+16162904619")
```

## 发送WhatsApp消息

您需要将您的WhatsApp Business帐户与Twilio链接起来。您还需要确保用于发送消息的号码在Twilio上配置为WhatsApp启用发送者，并已在WhatsApp上注册。

```python
from langchain_community.utilities.twilio import TwilioAPIWrapper
```

```python
twilio = TwilioAPIWrapper(
    #     account_sid="foo",
    #     auth_token="bar",
    #     from_number="whatsapp: baz,"
)
```

```python
twilio.run("hello world", "whatsapp: +16162904619")
```