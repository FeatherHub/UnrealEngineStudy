# 저장소 소개

이 저장소는 AI를 활용해 Unreal Engine 내부를 탐험한 내용을 노트로 남긴 저장소입니다.  

## 지금까지 다룬 것들

#### 렌더링 인프라

- Render Thread
- FPrimitiveSceneProxy, FScene, FGPUScene
- FSceneRenderer, FMeshPassProcessor, FMeshDrawCommand
- FLocalVertexFactory, FStaticMeshRenderData
- 관련 노트: [150 [렌더링 인프라]](<note/150_260629_[렌더링 인프라].txt>), [149 [렌더링 인프라]](<note/149_260628_[렌더링 인프라].txt>), [148 [렌더링 인프라]](<note/148_260627_[렌더링 인프라].txt>), [147 [렌더링 인프라]](<note/147_260625_[렌더링 인프라].txt>), [146 [렌더링 인프라]](<note/146_260624_[렌더링 인프라].txt>)

#### 물리 인프라

- FPhysScene, FBodyInstance
- FPBDRigidsSolver, FPBDRigidsSOAs
- Physics Thread, FChaosMarshallingManager (push/pull)
- FSingleParticlePhysicsProxy, FImplicitObject
- 관련 노트: [163 [물리 인프라]](<note/163_260712_[물리 인프라].txt>), [162 [물리 인프라]](<note/162_260711_[물리 인프라].txt>), [161 [물리 인프라]](<note/161_260710_[물리 인프라].txt>), [160 [물리 인프라]](<note/160_260709_[물리 인프라].txt>)

#### 네트워크 인프라

- UNetDriver, UNetConnection, UChannel
- FObjectReplicator, ServerReplicateActors
- Iris: FNetRefHandleManager, FReplicationProtocol, FReplicationFragment
- 관련 노트: [167 [네트워크 인프라][Iris]](<note/167_260717_[네트워크 인프라][Iris].txt>), [166 [네트워크 인프라][Iris]](<note/166_260716_[네트워크 인프라][Iris].txt>), [165 [네트워크 인프라][Iris]](<note/165_260715_[네트워크 인프라][Iris].txt>), [164 [네트워크 인프라]](<note/164_260714_[네트워크 인프라].txt>)

#### 에셋 로딩 인프라

- FLinkerLoad
- FAsyncLoadingThread2
- 관련 노트: [159 [AssetLoading][FAsyncLoadingThread2]](<note/159_260708_[AssetLoading][FAsyncLoadingThread2].txt>), [158 [AssetLoading][FLinkerLoad]](<note/158_260707_[AssetLoading][FLinkerLoad].txt>)

#### Task 인프라

- Task Graph, UE::Tasks
- Prerequisites, Subsequents
- Work Stealing, Retraction
- 관련 노트: [172 [Task 인프라]](<note/172_260722_[Task 인프라].txt>), [171 [Task 인프라]](<note/171_260721_[Task 인프라].txt>), [170 [Task 인프라]](<note/170_260720_[Task 인프라].txt>), [168 [Task 인프라]](<note/168_260718_[Task 인프라].txt>)

#### Slate 인프라

- FWindowsApplication, FSlateApplication
- SViewport, FSceneViewport, UGameViewportClient
- 관련 노트: [145 [Slate 인프라]](<note/145_260624_[Slate 인프라].txt>), [138 [Slate][UMG]](<note/138_260613_[Slate]UMG].txt>), [136 [Slate][Viewport]](<note/136_260611_[Slate][Viewport].txt>)

#### Reflection 인프라

- UClass, UStruct, FProperty
- UFunction
- 관련 노트: [142 [Reflection 인프라]](<note/142_260616_[Reflection 인프라].txt>), [141 [Reflection 인프라]](<note/141_260615_[Reflection 인프라].txt>), [123 [Reflection 인프라][GC]](<note/123_260508_[Reflection 인프라][GC].txt>), [122 [Reflection 인프라]](<note/122_260507_[Reflection 인프라].txt>)

#### 블루프린트 인프라

- Blueprint VM
- UBlueprintGeneratedClass
- 관련 노트: [132 [Blueprint]](<note/132_260523_[Blueprint].txt>), [076 [Blueprint][MaterialEditor]](<note/076_260403_[Blueprint][MaterialEditor].txt>), [073 [BP, Function Reflection]](<note/073_260401_[BP, Function Reflection].txt>), [052 [Blueprint 인프라]](<note/052_260318_[Blueprint 인프라].txt>)

#### 엔진 실행 구조

- FEngineLoop, 엔진 부팅
- Engine Tick, World Tick, Tick Group
- 관련 노트: [139 [엔진 인프라]](<note/139_260613_[엔진 인프라].txt>), [135 [엔진부팅]](<note/135_260607_[엔진부팅].txt>), [104 [엔진 Tick][엔진구조]](<note/104_260418_[엔진 Tick][엔진구조].txt>), [069 [Engine Tick][World Tick]](<note/069_260330_[Engine Tick][World Tick].txt>)

#### Audio 인프라

- Audio Thread
- FAudioDevice, FActiveSound, FWaveInstance
- 관련 노트: [091 [Audio]](<note/091_260413_[Audio].txt>), [047 [Audio]](<note/047_260309_[Audio].txt>)

---  

## AI 활용방법

Claude Cowork를 사용했습니다. 하네스가 로컬에서 실제 소스코드를 검색하고, 필요한 부분을 LLM의 Context에 넣을 수 있기 때문입니다. 

```text
사용자 질문
→ LLM이 필요한 Tool Call을 생성
→ 하네스가 실행
→ 결과를 LLM의 Context에 추가
```

처음 몇 턴은 주제에 대한 개요를 질문하면서 관련 클래스들을 LLM Context에 쌓은 뒤, 후속 질문을 반복하는 방식으로 AI를 학습에 활용했습니다.
클래스 관계나 호출 흐름을 한눈에 보기 위해, 필요한 경우 아래 표기 규칙을 요구했습니다.

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

  중첩 호출 트리
  ```
  FEndPhysicsTickFunction::ExecuteTick()
  {
    Target:UWorld*::FinishPhysicsSim();
      PhysScene:FPhysScene*::EndFrame();
      {
        this::SyncBodies(Solver);
        Solver:FPBDRigidsSolver::SyncEvents_GameThread()
        {
          this::MEventManager:FEventManager::DispatchEvents();
        }
      }
  }
  ```

## 노트 제목 규칙

```text
NNN_YYMMDD_[주제][세부주제].txt
```

- `NNN`: 학습 순서
- `YYMMDD`: 작성 날짜
- `[키워드]`: 학습한 주제
