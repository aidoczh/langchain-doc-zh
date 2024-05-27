# Nebula (Symbl.ai)

[Nebula](https://symbl.ai/nebula/) 是由 [Symbl.ai](https://symbl.ai) 构建的大型语言模型（LLM）。它经过训练，可以执行人类对话中的生成任务。Nebula 擅长对话的微妙细节建模，并执行对话中的任务。

Nebula 文档：https://docs.symbl.ai/docs/nebula-llm

这个示例介绍了如何使用 LangChain 与 [Nebula 平台](https://docs.symbl.ai/docs/nebula-llm) 进行交互。

确保你拥有 API 密钥。如果没有，请[申请一个](https://info.symbl.ai/Nebula_Private_Beta.html)。

```python
from langchain_community.llms.symblai_nebula import Nebula
llm = Nebula(nebula_api_key="<your_api_key>")
```

使用对话文本和指示构建提示。

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
conversation = """Sam: Good morning, team! Let's keep this standup concise. We'll go in the usual order: what you did yesterday, what you plan to do today, and any blockers. Alex, kick us off.
Alex: Morning! Yesterday, I wrapped up the UI for the user dashboard. The new charts and widgets are now responsive. I also had a sync with the design team to ensure the final touchups are in line with the brand guidelines. Today, I'll start integrating the frontend with the new API endpoints Rhea was working on. The only blocker is waiting for some final API documentation, but I guess Rhea can update on that.
Rhea: Hey, all! Yep, about the API documentation - I completed the majority of the backend work for user data retrieval yesterday. The endpoints are mostly set up, but I need to do a bit more testing today. I'll finalize the API documentation by noon, so that should unblock Alex. After that, I’ll be working on optimizing the database queries for faster data fetching. No other blockers on my end.
Sam: Great, thanks Rhea. Do reach out if you need any testing assistance or if there are any hitches with the database. Now, my update: Yesterday, I coordinated with the client to get clarity on some feature requirements. Today, I'll be updating our project roadmap and timelines based on their feedback. Additionally, I'll be sitting with the QA team in the afternoon for preliminary testing. Blocker: I might need both of you to be available for a quick call in case the client wants to discuss the changes live.
Alex: Sounds good, Sam. Just let us know a little in advance for the call.
Rhea: Agreed. We can make time for that.
Sam: Perfect! Let's keep the momentum going. Reach out if there are any sudden issues or support needed. Have a productive day!
Alex: You too.
Rhea: Thanks, bye!"""
instruction = "Identify the main objectives mentioned in this conversation."
prompt = PromptTemplate.from_template("{instruction}\n{conversation}")
llm_chain = LLMChain(prompt=prompt, llm=llm)
llm_chain.run(instruction=instruction, conversation=conversation)
```