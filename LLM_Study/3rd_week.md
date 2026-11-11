체인
- 정해진 순서 실행
- 분기 = 명시적 RunnableBranch
- 디버깅 쉬움
- 단순 작업에 적합

에이전트
- LLM이 도구 선택/반복
- 분기 = LLM의 판단
- 비결정적 - 추적 필수
- 복합/자율 작업에 적합

```
from langchain.agents import create_agent
agent = create_agent(
    model=llm,
    tools=[get_weather, search],
    prompt="당신은 한국어 비서입니다."
    checkpointer=MemorySavor(), #랭그래플의 단기 메모리 시스템: 히스토리 관리
    response_format=Pydantic스키마, # 구조화된 출력
    middleware=[dynamic_prompt(...)],
)
```

인보크 메소드: 전체 실행된 후에 최종적으로 상태를 확인하는 경우에 사용
에이전트의 사고 과정을 실시간으로 관찰하고 디버깅할 때는: stream method 사용
- values: 매 step의 전체 state 반환: 전체 흐름 파악
- updates: 변경된 부분만 반환: 효율적인 상태 추적
- messages: LLM 토큰 단위 스트리밍: UI 실시간 표시에 적합

에이전트의 응답을 Pydantci 스키마로 구조화하기
1. 별도 tool을 호출
2. 모델 자체의 structured_output 기능 활용 