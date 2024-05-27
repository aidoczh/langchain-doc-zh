# Datadog 日志

>[Datadog](https://www.datadoghq.com/) 是一个针对云规模应用的监控和分析平台。

此加载器使用 `datadog_api_client` Python 包从 Datadog 中获取应用程序的日志。您必须使用您的 `Datadog API 密钥` 和 `APP 密钥` 初始化加载器，并且需要传入要提取的所需日志的查询。

```python
from langchain_community.document_loaders import DatadogLogsLoader
```

```python
%pip install --upgrade --quiet  datadog-api-client
```

```python
DD_API_KEY = "..."
DD_APP_KEY = "..."
```

```python
query = "service:agent status:error"
loader = DatadogLogsLoader(
    query=query,
    api_key=DD_API_KEY,
    app_key=DD_APP_KEY,
    from_time=1688732708951,  # 可选，毫秒级时间戳
    to_time=1688736308951,  # 可选，毫秒级时间戳
    limit=100,  # 可选，默认为 100
)
```

```python
documents = loader.load()
documents
```

```output
[Document(page_content='message: grep: /etc/datadog-agent/system-probe.yaml: No such file or directory', metadata={'id': 'AgAAAYkwpLImvkjRpQAAAAAAAAAYAAAAAEFZa3dwTUFsQUFEWmZfLU5QdElnM3dBWQAAACQAAAAAMDE4OTMwYTQtYzk3OS00MmJjLTlhNDAtOTY4N2EwY2I5ZDdk', 'status': 'error', 'service': 'agent', 'tags': ['accessible-from-goog-gke-node', 'allow-external-ingress-high-ports', 'allow-external-ingress-http', 'allow-external-ingress-https', 'container_id:c7d8ecd27b5b3cfdf3b0df04b8965af6f233f56b7c3c2ffabfab5e3b6ccbd6a5', 'container_name:lab_datadog_1', 'datadog.pipelines:false', 'datadog.submission_auth:private_api_key', 'docker_image:datadog/agent:7.41.1', 'env:dd101-dev', 'hostname:lab-host', 'image_name:datadog/agent', 'image_tag:7.41.1', 'instance-id:7497601202021312403', 'instance-type:custom-1-4096', 'instruqt_aws_accounts:', 'instruqt_azure_subscriptions:', 'instruqt_gcp_projects:', 'internal-hostname:lab-host.d4rjybavkary.svc.cluster.local', 'numeric_project_id:3390740675', 'p-d4rjybavkary', 'project:instruqt-prod', 'service:agent', 'short_image:agent', 'source:agent', 'zone:europe-west1-b'], 'timestamp': datetime.datetime(2023, 7, 7, 13, 57, 27, 206000, tzinfo=tzutc())}),
 Document(page_content='message: grep: /etc/datadog-agent/system-probe.yaml: No such file or directory', metadata={'id': 'AgAAAYkwpLImvkjRpgAAAAAAAAAYAAAAAEFZa3dwTUFsQUFEWmZfLU5QdElnM3dBWgAAACQAAAAAMDE4OTMwYTQtYzk3OS00MmJjLTlhNDAtOTY4N2EwY2I5ZDdk', 'status': 'error', 'service': 'agent', 'tags': ['accessible-from-goog-gke-node', 'allow-external-ingress-high-ports', 'allow-external-ingress-http', 'allow-external-ingress-https', 'container_id:c7d8ecd27b5b3cfdf3b0df04b8965af6f233f56b7c3c2ffabfab5e3b6ccbd6a5', 'container_name:lab_datadog_1', 'datadog.pipelines:false', 'datadog.submission_auth:private_api_key', 'docker_image:datadog/agent:7.41.1', 'env:dd101-dev', 'hostname:lab-host', 'image_name:datadog/agent', 'image_tag:7.41.1', 'instance-id:7497601202021312403', 'instance-type:custom-1-4096', 'instruqt_aws_accounts:', 'instruqt_azure_subscriptions:', 'instruqt_gcp_projects:', 'internal-hostname:lab-host.d4rjybavkary.svc.cluster.local', 'numeric_project_id:3390740675', 'p-d4rjybavkary', 'project:instruqt-prod', 'service:agent', 'short_image:agent', 'source:agent', 'zone:europe-west1-b'], 'timestamp': datetime.datetime(2023, 7, 7, 13, 57, 27, 206000, tzinfo=tzutc())})]
```