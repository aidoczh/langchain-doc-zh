# Doctran: 语言翻译

通过嵌入式比较文档的好处在于可以跨多种语言进行工作。"Harrison says hello" 和 "Harrison dice hola" 将在向量空间中占据相似的位置，因为它们在语义上具有相同的含义。

然而，在将文档向量化之前，使用 LLM **将文档翻译成其他语言** 仍然是有用的。当用户需要用不同语言查询知识库时，或者当给定语言没有最先进的嵌入模型可用时，这一点尤其有帮助。

我们可以使用 [Doctran](https://github.com/psychic-api/doctran) 库来实现这一点，该库使用 OpenAI 的函数调用功能来在不同语言之间进行文档翻译。

```python
%pip install --upgrade --quiet  doctran
```

```python
from langchain_community.document_transformers import DoctranTextTranslator
from langchain_core.documents import Document
```

```python
from dotenv import load_dotenv
load_dotenv()
```

```output
True
```

## 输入

这是我们将要翻译的文档

```python
sample_text = """[Generated with ChatGPT]
机密文件 - 仅供内部使用
日期：2023年7月1日
主题：各种话题的更新和讨论
亲爱的团队，
希望这封邮件能够找到你们一切安好。在这份文件中，我想向你们提供一些重要的更新，并讨论需要我们关注的各种话题。请将此处包含的信息视为高度机密。
安全和隐私措施
作为我们不断致力于确保客户数据的安全和隐私的一部分，我们已在所有系统中实施了强有力的措施。我们要赞扬IT部门的John Doe（电子邮件：john.doe@example.com）在增强我们的网络安全方面的勤奋工作。展望未来，我们要求每个人严格遵守我们的数据保护政策和准则。此外，如果您发现任何潜在的安全风险或事件，请立即向我们专门的团队报告，邮箱地址为security@example.com。
人力资源更新和员工福利
最近，我们迎来了几位新的团队成员，他们在各自的部门做出了重大贡献。我要表彰Jane Smith（社会安全号码：049-45-5928）在客户服务方面的杰出表现。Jane一直从我们的客户那里收到积极的反馈。此外，请记住，我们的员工福利计划的开放报名期即将到来。如果您有任何问题或需要帮助，请联系我们的人力资源代表Michael Johnson（电话：418-492-3850，电子邮件：michael.johnson@example.com）。
营销倡议和活动
我们的营销团队一直在努力制定新的策略，以增加品牌知名度并推动客户参与。我们要感谢Sarah Thompson（电话：415-555-1234）在管理我们的社交媒体平台方面的卓越努力。Sarah在过去一个月内成功将我们的关注者基数增加了20%。此外，请记下7月15日即将举行的产品发布活动。我们鼓励所有团队成员参加并支持我们公司的这一激动人心的里程碑。
研发项目
在追求创新的过程中，我们的研发部门一直在不同的项目上不知疲倦地工作。我要承认David Rodriguez（电子邮件：david.rodriguez@example.com）在他作为项目负责人的工作中所做出的杰出贡献。David对我们尖端技术的发展做出了重要贡献。此外，我们要提醒每个人在7月10日安排的我们每月的研发头脑风暴会议上分享他们的想法和建议，以便开展潜在的新项目。
请将此文件中的信息视为最高机密，并确保不与未经授权的人员分享。如果您对讨论的话题有任何问题或疑虑，请随时直接联系我。
感谢您的关注，让我们继续共同努力实现我们的目标。
此致，
Jason Fan
创始人兼首席执行官
Psychic
jason@psychic.dev
"""
```

```python
documents = [Document(page_content=sample_text)]
qa_translator = DoctranTextTranslator(language="spanish")
```

## 输出

翻译文档后，结果将作为新文档返回，其中的页面内容已翻译成目标语言

```python
translated_document = qa_translator.transform_documents(documents)
```

```python
print(translated_document[0].page_content)
```

```output
Documento Confidencial - Solo para Uso Interno
Fecha: 1 de Julio de 2023
Asunto: Actualizaciones y Discusiones sobre Varios Temas
Estimado Equipo,
Espero que este correo electrónico les encuentre bien. En este documento, me gustaría proporcionarles algunas actualizaciones importantes y discutir varios temas que requieren nuestra atención. Por favor, traten la información contenida aquí como altamente confidencial.
Medidas de Seguridad y Privacidad
Como parte de nuestro compromiso continuo de garantizar la seguridad y privacidad de los datos de nuestros clientes, hemos implementado medidas sólidas en todos nuestros sistemas. Nos gustaría elogiar a John Doe (correo electrónico: john.doe@example.com) del departamento de TI por su diligente trabajo en mejorar nuestra seguridad de red. En el futuro, recordamos amablemente a todos que se adhieran estrictamente a nuestras políticas y pautas de protección de datos. Además, si encuentran algún riesgo o incidente de seguridad potencial, por favor, repórtelo de inmediato a nuestro equipo dedicado en security@example.com.
Actualizaciones de Recursos Humanos y Beneficios para Empleados
Recientemente, dimos la bienvenida a varios nuevos miembros del equipo que han realizado contribuciones significativas en sus respectivos departamentos. Me gustaría reconocer a Jane Smith (SSN: 049-45-5928) por su destacado desempeño en servicio al cliente. Jane ha recibido consistentemente comentarios positivos de nuestros clientes. Además, recuerden que el período de inscripción abierta para nuestro programa de beneficios para empleados se acerca rápidamente. Si tienen alguna pregunta o necesitan ayuda, por favor, contacten a nuestro representante de Recursos Humanos, Michael Johnson (teléfono: 418-492-3850, correo electrónico: michael.johnson@example.com).
Iniciativas y Campañas de Marketing
Nuestro equipo de marketing ha estado trabajando activamente en el desarrollo de nuevas estrategias para aumentar el conocimiento de nuestra marca y fomentar la participación de los clientes. Nos gustaría agradecer a Sarah Thompson (teléfono: 415-555-1234) por sus esfuerzos excepcionales en la gestión de nuestras plataformas de redes sociales. Sarah ha logrado aumentar nuestra base de seguidores en un 20% solo en el último mes. Además, marquen sus calendarios para el próximo evento de lanzamiento de productos el 15 de Julio. Animamos a todos los miembros del equipo a asistir y apoyar este emocionante hito para nuestra empresa.
Proyectos de Investigación y Desarrollo
En nuestra búsqueda de la innovación, nuestro departamento de investigación y desarrollo ha estado trabajando incansablemente en varios proyectos. Me gustaría reconocer el trabajo excepcional de David Rodriguez (correo electrónico: david.rodriguez@example.com) en su papel de líder de proyecto. Las contribuciones de David al desarrollo de nuestra tecnología de vanguardia han sido fundamentales. Además, nos gustaría recordar a todos que compartan sus ideas y sugerencias para posibles nuevos proyectos durante nuestra sesión mensual de lluvia de ideas de I+D, programada para el 10 de Julio.
Por favor, traten la información de este documento con la máxima confidencialidad y asegúrense de no compartirla con personas no autorizadas. Si tienen alguna pregunta o inquietud sobre los temas discutidos, por favor, no duden en comunicarse directamente conmigo.
Gracias por su atención y sigamos trabajando juntos para alcanzar nuestros objetivos.
Atentamente,
Jason Fan
Cofundador y CEO
Psychic
jason@psychic.dev
```