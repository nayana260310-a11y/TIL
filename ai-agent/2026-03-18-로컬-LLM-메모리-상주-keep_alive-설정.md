### Context

로컬 LLM (Ollama 등) 을 LangGraph 기반의 에이전트 워크플로우에 통합할 때, 모델이 메모리에서 자동으로 해제되는 현상으로 인해 실행이 중단되는 문제가 발생했습니다. LangGraph 가 전체 그래프를 실행하는 과정에서 로컬 모델이 메모리에서 내려가 데이터를 반환하지 못하면, 시스템은 이를 무한 루프 또는 시간 초과 (Timeout) 로 인식하여 작업을 강제 종료 (🚫 금지 마크) 시킵니다. 이는 특히 긴 토큰 생성이나 복잡한 에이전트 루프에서 빈번하게 발생합니다.

### Core

Ollama 에서 모델이 메모리에 상주하도록 유지하기 위해 `keep_alive` 파라미터를 `-1`로 설정해야 합니다. 이 설정은 모델이 요청이 끝난 후에도 명시적으로 제거될 때까지 메모리에 남아 있도록 합니다.

**Ollama API 호출 시 `keep_alive` 설정 예시 (Python):**

```python
from langchain_ollama import ChatOllama

# keep_alive=-1 을 설정하여 모델이 메모리에 상주하도록 함
llm = ChatOllama(
    model="llama3.2",
    base_url="http://localhost:11434",
    keep_alive=-1  # 모델이 메모리에서 내려가지 않도록 영구 유지
)

# LangGraph 에이전트에서 사용
# ... (에이전트 정의 및 실행 코드)
```

**cURL 명령어 예시:**

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Hello",
  "keep_alive": -1
}'
```

### Insight

로컬 LLM 을 에이전트 시스템에 통합할 때 `keep_alive` 파라미터의 설정은 시스템 안정성에 결정적인 역할을 합니다. 기본 설정 (보통 5 분) 은 짧은 대화에는 적합하지만, LangGraph 와 같은 복잡한 워크플로우에서는 모델이 중간에 메모리에서 해제되어 응답 지연이나 연결 끊김을 유발할 수 있습니다. `keep_alive=-1` 설정은 모델이 메모리에 상주하게 하여 응답 시간을 단축하고, LangGraph 의 타임아웃 오류를 방지하여 에이전트의 원활한 작동을 보장합니다. 다만, 이 설정은 시스템 메모리 사용량을 증가시킬 수 있으므로, 리소스 관리가 필요한 환경에서는 적절한 시간 (예: `keep_alive="24h"`) 으로 설정하는 것이 좋습니다.

**출처:** [Ollama Documentation - keep_alive parameter](https://github.com/ollama/ollama/blob/main/docs/api.md#generate-a-completion)
**출처:** [LangChain Ollama Integration Guide](https://python.langchain.com/docs/integrations/chat/ollama/)