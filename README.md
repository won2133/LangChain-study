# 랭체인 공부하기

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
### 맥락 인식 챗봇
* ConversationSummaryBufferMemory, SQLChatMessageHistory 이용하여 이전 대화 기억하는 챗봇
* SQL에는 전체 내용을 저장하고, ConversationSummaryBufferMemory를 이용해 이전 대화를 요약하여 llm에 전달한다.
* 공부 목적이기 때문에 ConversationChain를 사용하는 것과 메모리를 직접 사용하는 것, RunnableWithMessageHistory를 사용하는 것과 SQLChatMessageHistory로 직접 저장하는 것을 조합하여 모두 구현해보았다. 
* 원래 하나의 파일로 했었는데 실행하는 순서에 따라, 입력하는 것에 따라 조금씩 결과가 다르게 나와서 세 개의 파일로 나누어 따로 실행했다.
* SQLChatMessageHistory 파일
  * ConversationChain와 함께 사용하는 방식과 메모리와 함께 사용하는 방식
  * 데이터베이스에는 잘 분리되어 저장되었으나, 세션 아이디와 상관없이 이전 대화를 기억하였다. 
* RunnableWithMessageHistory 파일
  * 역시 ConversationChain와 함께 사용하는 방식과 메모리와 함께 사용하는 방식
  * conversationChain을 사용했을 때는 세션 아이디가 달라져도 대화를 기억했다.
  * 메모리를 직접 사용하는 방식에서는 세션 아이디가 다를 경우 이전 대화를 기억하지 못했고, 같은 경우 대화는 이어졌지만 직접적으로 기억하냐고 물었을 때는 기억하지 못한다고 답한 경우가 많았다.
* 세션 관리 파일
  * 메모리를 세션 아이디 별로 초기화하는 코드를 추가하여, 위 네 가지 조합 별로 코드를 작성하고 실행해보았다.
  * 대체로 세션 아이디별로 나누어서 잘 저장되고 잘 대답하였다. 그러나 SQLChatMessageHistory와 메모리의 조합으로 사용했을 때는 이전 대화를 기억하지 못했다. 메모리를 직접 사용하는 코드를 작성할 때 메모리를 저장할 변수를 초기화하였는데, RunnableWithMessageHistory는 데이터베이스를 참고하여 잘 대답하였는데 그걸 쓰지 않아서 기억을 못한 것으로 보인다.
