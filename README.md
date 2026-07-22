# 저장소 소개

이 저장소는 AI를 활용해 Unreal Engine 내부를 공부한 내용을 노트로 남긴 저장소입니다.  
AI를 어떻게 활용했는지는 [AI 활용방법](#ai-활용방법)에서 소개합니다.  

## 지금까지 다룬 것들

#### 렌더링 인프라

- Render Thread
- FPrimitiveSceneProxy, FScene, FGPUScene
- FSceneRenderer, FMeshPassProcessor, FMeshDrawCommand
- FLocalVertexFactory, FStaticMeshRenderData

#### 물리 인프라

- FPhysScene, FBodyInstance
- FPBDRigidsSolver, FPBDRigidsSOAs
- Physics Thread, FChaosMarshallingManager (push/pull)
- FSingleParticlePhysicsProxy, FImplicitObject

#### 네트워크 인프라

- UNetDriver, UNetConnection, UChannel
- FObjectReplicator, ServerReplicateActors
- Iris: FNetRefHandleManager, FReplicationProtocol, FReplicationFragment

#### 에셋 로딩 인프라

- FLinkerLoad
- FAsyncLoadingThread2

#### Task 인프라

- Task Graph, UE::Tasks
- Prerequisites, Subsequents
- Work Stealing, Retraction

#### Slate 인프라

- FWindowsApplication, FSlateApplication
- SViewport, FSceneViewport, UGameViewportClient

#### Reflection 인프라

- UClass, UStruct, FProperty
- UFunction

#### 블루프린트 인프라

- Blueprint VM
- UBlueprintGeneratedClass

#### 엔진 실행 구조

- FEngineLoop, 엔진 부팅
- Engine Tick, World Tick, Tick Group

#### Audio 인프라

- Audio Thread
- FAudioDevice, FActiveSound, FWaveInstance

## AI 활용방법

한 마디로 말씀드리자면, Warm-up 후 전체 구조를 드러내는 표기법을 프롬프팅한 후, 후속 질문을 반복하는 방식으로 AI를 활용했습니다.

#### 하네스와 Warm-up

LLM 자체는 상태 없는 함수입니다. 토큰 열을 받아 다음 토큰의 확률 분포를 낼 뿐, 파일을 열거나 명령을 실행하지는 못합니다. 실제 탐색은 하네스(에이전트)가 담당합니다.

```text
사용자 질문
→ LLM이 필요한 검색·도구 호출을 텍스트로 생성
→ 하네스가 실행
→ 결과를 Context에 추가
```

검색 대상을 고르는 것은 LLM이지만, 실제 파일 탐색과 문자열 매칭은 하네스의 도구(`rg`, `grep`, 파일시스템 API)가 수행하고, 그 결과 토큰이 다시 Context로 들어옵니다.

이것을 활용해서 처음 다루는 주제는 먼저 LLM을 Warm-up 시킵니다. 분석 질문을 던지기 전에, 하네스로 실제 소스를 Context에 먼저 적재하는 과정입니다.

왜 필요한가:

- 모델의 사전 지식(=가중치)은 학습 데이터를 손실 압축한 결과입니다. 드물게 등장한 사실일수록 그럴듯하지만 틀린 답, hallucination이 나오기 쉽습니다.
- 추론 시점에는 weight가 동결되어 있습니다. 그래서 세션 중에 소스를 읽어도 그것은 학습(가중치 갱신)이 아니라 conditioning입니다. In-Context Learning은 가중치를 바꾸지 않고, 프롬프트에 놓인 토큰이 attention을 통해 다음 토큰의 분포를 바꿉니다.

Warm-up을 시킨 뒤, 제가 이해할 때까지 반복 후속 질문으로 파고드는 방식으로 학습에 AI를 활용했습니다. 
LLM 출력의 정보량과 정확성을 높이기 위해 필요한 경우 아래 표기 규칙을 요구했습니다.

#### 클래스 관계를 정리할 때 사용하는 표기 규칙

- 멤버: `Owner::Member:Type`; `FDirtyProxy::Proxy:IPhysicsProxyBase*`, `FDirtySet::DirtyProxyBuckets:FDirtyProxiesBucket[]`
- 타입 변종: 포인터 `Type*`, 리스트류(array/set/queue) `Type[]`, 맵 `<Key->Value>`, 조합 `Type*[]`
- 소유 포인터: ◦ (예: MyAsset: FMyAsset◦)
- 그 외 포인터(참조, 사용): *
- 인덱싱: `Owner::Member:ElementType[Idx]`

#### 코드 흐름 표기 규칙

  ```
  AAClass::AAMethod()
  {
    Var:Type::Method();           // local/param 의 메서드
    this::Member:Type::Method();  // 멤버의 메서드. 이 표기법 덕분에 local/param 하고 구별할 수 있다.
    this::OwnMethod();            // 자신의 메서드
    ::FreeFunc();                 // 자유 함수
    Class::StaticMethod();        // static method
    this::Member:Type = Value;    // 멤버 대입
  }
  ```

  중첩(호출 트리)
  ```
  FEndPhysicsTickFunction::ExecuteTick()
  {
    Target:UWorld*::FinishPhysicsSim();
      PhysScene:FPhysScene*::EndFrame();
        FChaosScene::EndFrame()
        {
          this::SyncBodies(Solver);
          Solver:FPBDRigidsSolver::SyncEvents_GameThread()
          {
            this::MEventManager:FEventManager::DispatchEvents();
          }
        }
  }
  ```

## 노트 파일 규칙

노트는 [`note/`](note/) 폴더에 있으며 다음 형식을 사용합니다.

```text
NNN_YYMMDD_[주제][세부주제].txt
```

- `NNN`: 학습 순서
- `YYMMDD`: 작성 날짜
- `[키워드]`: 학습한 주제

