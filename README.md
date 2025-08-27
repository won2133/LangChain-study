# 랭체인 공부하기


## 목차
* 개요
* 학습 기록
* 간단한 프로젝트
  * LLM 멀티턴 대화(+세션 관리)
  * SLM 멀티턴 대화

## 개요
* 위키독스에 있는 테디노트의 <랭체인LangChain 노트>를 기반으로 공부한 기록이다.
* llm 모델로는 대부분 구글의 gemini 모델을 사용하였다.
* 공부는 진행 중이며, 계속해서 업데이트될 예정이다.

## 학습 기록
* 프롬프트와 출력 파서: 프롬프트, 퓨샷 프롬프트, 다양한 출력 파서들
* 모델: 캐시, 모델 직렬화, 비디오 질의응답 등
* runnable: RunnablePassthrough, RunnableParallel, RunnableLambda 사용하기
* 메모리: 다양한 기준으로 메모리에 저장하기, 데이터베이스에 저장하기

## 간단한 프로젝트
### LLM 멀티턴 대화(+세션 관리)
#### 개요
* ConversationSummaryBufferMemory, SQLChatMessageHistory, RunnableWithMessageHistory 이용하여 이전 대화 기억하는 챗봇
* 모델: gemini-2.0-flash
* 공부하는 게 목적이기 때문에 다양한 시도를 해봤다. 우선은 메모리에만 저장하는 것과 메모리와 데이터베이스에 각각 저장하는 것을 차례로 구현했다. 이때 두 가지 모두 ConversationChain을 사용하는 것과 메모리에 직접 저장하는 것의 두 방식을 이용해 구현해보았다.
* 이전 대화를 기억하는지, 세션 아이디 별로 구별하여 저장하는지, 그리고 메모리를 초기화 했을 때 기억하는지를 중점적으로 확인하였다.
* 세션1에서는 테디라는 이름으로 서울에 관한 이야기를, 세션2에서는 디테라는 이름으로 제주도에 관한 이야기를 나누었다.
#### 메모리에만 저장했을 때
* ConversationSummaryBufferMemory 사용
* ConversationChain 쓰는 것, 쓰지 않고 메모리에 직접 저장하는 것 모두 세션 별로 분리하여 잘 저장하고 불러오는 것을 확인하였다.
* 데이터베이스에 저장하는 것과 비교하기 위해 메모리를 초기화한 후에 이전 대화 기록을 기억하는지 확인했고, 당연하게도 기억하지 못했다.
#### 데이터베이스에도 같이 저장했을 때
* RunnableWithMessageHistory 추가
* 메모리는 요약 메모리를 써서 요약본을 저장하였고, 데이터베이스에는 원본을 그대로 저장하였다.
* 마찬가지로 두 가지 방식 모두 세션 별로 잘 저장하고 불러왔다.
* 다만 ConversationChain을 사용했을 때, 메모리를 초기화한 이후에는 이전 대화를 기억하지 못했다.


### SLM 멀티턴 대화
#### 개요
* SLM을 로컬에 저장한 뒤 랭체인과 연결해 멀티턴 대화 구현
* 모델: 허깅페이스의 'MLP-KTLim/llama-3-Korean-Bllossom-8B'
* 역시 공부 목적이기 때문에 다양한 방법으로 시도해보았다.
#### 준비
* 프롬프트: 허깅페이스 모델 설명 페이지의 파이썬 파이프라인 코드 실행 후 사용된 프롬프트를 출력하여 문자열 그대로 템플릿으로 정의하였다.
  <pre><code>template = '''
  <|begin_of_text|><|start_header_id|>system<|end_header_id|>

  You are a helpful AI assistant. Please answer the user's questions kindly. 당신은 유능한 AI 어시스턴트 입니다. 사용자의 질문에 대해 친절하게 답변해주세요.<|eot_id|><|start_header_id|>user<|end_header_id|>
  {question}<|eot_id|><|start_header_id|>assistant<|end_header_id|>
  ''' </code></pre>
* 종료 토큰 확인
  * 모델 설명 페이지 파이썬 파이프라인 코드 중 종료 토큰을 담은 terminators
    <pre><code>terminators = [
    pipe.tokenizer.eos_token_id,
    pipe.tokenizer.convert_tokens_to_ids("<|eot_id|>")
    ]
    </code></pre>
  * terminators(=종료 토큰) 확인
    <pre>[128009, 128009]</pre>
  * 앞서 확인한 종료 토큰을 바탕으로 파이프라인 정의 후 랭체인의 HuggingFacePipeline과 연결
    <pre><code>pipe = pipeline(
        "text-generation", 
        model=model,
        tokenizer=tokenizer, 
        max_new_tokens=512, 
        eos_token_id=[128009, 128009] #종료 토큰 설정
    )
    hf = HuggingFacePipeline(pipeline=pipe)
    </code></pre> 

* ConversationChain 사용
  * 템플릿이 그대로 저장됨
#### SLM 멀티턴 대화
* 입력한 대화
  1. 안녕 내 이름은 '테디'야
  2. 파리 여행할 때 꼭 가야 할 곳 추천해줄래?
  3. 음식도 추천해줘
  4. 대한민국의 수도는 어디야?
  5. 내 이름 기억해?
  6. 지난 번에 어떤 도시에 대해 얘기를 나눴지?
* 메모리에 직접 저장
  * ConversationBufferMemory 이용
  * ConversationSummaryBufferMemory 이용
* 메모리 사용x, 직접 요약
* RAG 기반: 이전 대화 기록들 중 가장 유사한 대화 뽑아서 대화하기