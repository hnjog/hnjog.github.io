---
title: "AssetManager"
date : "2025-08-26 17:00:00 +0900"
last_modified_at: "2025-08-26T17:00:00"
categories:
  - Unreal
tags:
  - AssetManager
---

## AssetManager
언리얼 엔진 UE4.18 이후에 도입된<br>
에셋 매니저는 '게임 리소스(에셋)'을 효율적으로 관리하고 로드하기 위해<br>
제공되는 시스템<br>

에셋 매니저의 핵심 개념들<br>

- Manager를 통한 집중적인 에셋관리<br>
  : '에셋의 논리적 이름(Primary Asset ID)'를 통한 에셋 접근을 통해<br>
  '데이터' 중심의 에셋 관리를 가능하게 함<br>

  (기존에는 StaticLoadObject 나 LoadObject 등을 통한 로딩 방식)<br>
  그런데, 수백~수천 개가 있는 경로에서<br>
  '새로운 파일 정리'를 하려 하면<br>
  말 그대로 난리가 나게 됨<br>
  (-> 사실상 하드 코딩)<br>

- Primary Asset<br>
  : AssetManager가 관리하는 기본적인 단위<br>
    (Primary Asset Type + PrimaryAssetID의 조합)<br>
    (경로 나 직접적인 참조 x)
    게임의 핵심적인 자원을 해당 타입으로 등록하여 관리<br>

    - UDataAsset vs UPrimaryDataAsset<br>
      - UDataAsset <br>
        : '데이터 컨테이너'로서 데이터만 넣어두고 BP나 코드에서 불러서 사용(에셋 매니저가 스캔,관리하지 않음)<br>
      - UPrimaryDataAsset<br>
        : 에셋 매니저가 스캔할 수 있도록 특별히 설계된 UDataAsset(상속받음)<br>
          내부적으로 PrimaryAssetType / PrimaryAssetId 를 자동 생성<br>
          (AssetManager가 전역적으로 추적 가능)<br>

- 에셋의 로딩 방식<br>
  : 동기/비동기 로딩 양측을 지원하고 표준화<br>

- 에셋 번들(Asset Bundle)<br>
  : Primary Asset에 '묶여서 함께 로딩'할 수 있는 서브 에셋 모음<br>
    (ex : 무기 데이터를 로드할 때, Mesh,Sound,Particle을 같이 로드)<br>
    (데이터를 로딩할 타이밍을 계획할 수 있음)<br>

- 에셋 레지스트리 (Asset Registry)<br>
  : 엔진 내부의 '에셋 정보 데이터베이스'<br>
    실제 메모리엔 로드하지 않고<br>
    '경로','클래스','태그'와 같은 메타데이터를 빠르게 검색하는 용도<br>

---

### Scan
에셋 매니저가 '프로젝트 폴더'에서 '특정 타입'의 에셋들을<br>
찾아 '에셋 레지스트리'에 등록하는 과정<br>
(데이터를 '로드'하기 이전에 '어떤 에셋'들을 로드할지<br>
미리 한번 살펴보는 과정)<br>
(Project Settings에서 AssetManager 세팅을 통해<br>
Scan할 Type을 골라준다)<br>

<img width="3169" height="1893" alt="Image" src="https://github.com/user-attachments/assets/d17a6ff4-5dba-4c26-ae9c-eadcc6698c4b" /><br>

- Primary Asset Type<br>
  : 해당 셋팅에서 넣어주는 Type 문자열이 Primary Data Asset의 Type으로 설정됨<br>
    (정확히는 FPrimaryDataAssetId의 Type 에 들어간다)<br>

- Base Class<br>
  : 지정한 클래스 기반으로 검색한다<br>
  (예를 들어 UWeaponData : UPrimaryDataAsset 라는 클래스는<br>
  Base Class에 UWeaponData 를 지정 시,<br>
  해당 클래스를 기반으로 만든 모든 DataAsset을 스캔함)<br>

- Directory<br>
  : '검색'할 폴더 지정<br>
  (아무것도 없으면 아예 스캔을 안한다)<br>
  옵션에 따라 하위 디렉토리 지정 가능<br>

(이렇게 해줘도 되고 아니면<br>
Default.ini에 직접 설정할수도 있음)<br>

```
[/Script/Engine.Engine]
...
AssetManagerClassName=/Script/Samples.SampleAssetManager
```

---

### Bundle
Primary Asset 내부에서 '함께 로드/언로드' 하라 설정한 리소스 묶음<br>
(FAssetBundleEntry 구조체)<br>

내부적으로 자신의 SoftObjectPtr 멤버들을 등록하는 편<br>

UPrimaryDataAsset 에서 번들을 등록하는 예시 코드<br>

```
UCLASS(BlueprintType)
class UWeaponData : public UPrimaryDataAsset
{
    GENERATED_BODY()
public:
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    TSoftObjectPtr<USkeletalMesh> Mesh;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    TSoftObjectPtr<USoundBase> FireSound;

    // PrimaryAssetInterface 구현
    virtual void GetPrimaryAssetBundles(FPrimaryAssetId AssetId, TArray<FAssetBundleEntry>& OutBundles) const override
    {
        FAssetBundleEntry DefaultBundle;
        DefaultBundle.BundleName = "Default";
        DefaultBundle.BundleScope = AssetId;
        DefaultBundle.AssetPaths.Add(Mesh.ToSoftObjectPath());
        DefaultBundle.AssetPaths.Add(FireSound.ToSoftObjectPath());

        OutBundles.Add(DefaultBundle);
    }
};
```

- AssetManager가 저 WeaponData를 로드할때 Mesh와 FireSound를 같이 불러오게 됨<br>

- TSoftObjectPtr?<br>
  : 경로 기반의 '참조'<br>
   (내부적으로 경로(FSoftObjectPath)만 저장해두고<br>
   실제 객체는 필요할때 LoadSynchronous 나 AssetManager의 비동기 로드로 불러옴)<br>
   Lazy Loading 이기에, 필요할 때만 리소스를 불러오는 방식<br>

#### UAssetManager는 Singleton
- 전역적인 접근이 필요 : 게임의 어느 타이밍에서든 '자원 관리'는 필요<br>
- 여러 관리자의 필요성 x : 동일한 에셋을 '여러 관리자'가 관리할 필요 없음<br>


---

### Asset Manager를 상속받아 커스텀하는 경우

<img width="1789" height="261" alt="Image" src="https://github.com/user-attachments/assets/42c6cbef-dc5c-4435-bdd7-e3760f97ef4a" /><br>

이렇게 프로젝트 세팅에서 해당 AssetManager를 바꿔줄것<br>

```
USampleAssetManager& USampleAssetManager::Get()
{
	check(GEngine);

	// 이 클래스가 오버라이드 된 클래스이기에, GEngine에 AssetManager가 존재
	if (USampleAssetManager* Singleton = Cast<USampleAssetManager>(GEngine->AssetManager))
	{
		return *Singleton;
	}

	UE_LOG(LogSample, Fatal, TEXT("invalid AssetManagerClassname in DefaultEngine.ini(project settings); it must be SampleAssetManager"));

	// crash가 나기에 도달할 수 없는 상황이지만 컴파일을 위해
	return *NewObject<USampleAssetManager>();
}
```

그래야 Singleton 방식으로 Get을 만들때<br>
기존 Engine에 존재하는 AssetManager를 재이용할 수 있다<br>
(+ 프로젝트 세팅을 안한 경우, nullptr이 반환)<br>
(-> 그렇기에 AssetManager가 2개 생성되므로 Fatal Error 를 남김)<br>
(차라리 크래시를 내고 싶다면 CastCheck 를 사용하는 방법도 존재함)<br>

- Cast<T> <br>
  : 내부에서 IsA() 를 사용하여 체크<br>
    '부모 클래스 객체'를 자식 클래스로 캐스팅하지 못하게 함<br>
    (잘못된 다운 캐스팅에 대한 대처)<br>
    (C++에서는 RTTI가 켜져있어야 dynamic_cast가 잡아낼 수 있음)<br>
    (Unreal에선 리플렉션을 통해 RTTI를 키지 않아도 잡아낼 수 있음!)<br>


---

## 동기 / 비동기 로딩
에셋의 로딩 방식<br>

- 동기<br>
 : 완료될때까지 대기<br>
   - 초기 게임 데이터들을 로딩할때 많이 사용<br>
   - 빠른 편이며, 유저가 기다림을 '이미 느끼는 상황' 등에는 사용해도<br>
     상관없는 편<br>
     (그러나 인게임중에 막쓰면 게임이 멈추거나 프레임 드랍)<br>
  
- 비동기<br>
 : 로딩을 호출시킨 후<br>
 다시 다른 작업을 하러 떠남<br>
  - 인게임 중에 많이 사용하는 로딩 방식<br>
  - 필요한 상황에서만 로딩하기에 메모리가 절약된다<br>


동기식 로딩 예시 코드<br>

```
// synchronous load asset을 불필요하게 호출하는 경우를 확인하기 위함
// (동기 로딩이 호출되면 게임이 끊기거나 프레임 드랍이 발생하므로 그러한 경우를 체크하기 위한 용)
// 동기식 에셋 로드
// 동기 : 완전히 끝날때까지 기다린 후, 다음 작업 진행 (게임 시작 시, 대량 로딩) -> 빠름, 직관적
// 비동기 : 일정 시간 진행 후, 다른 스레드 등을 통하여 타 작업을 진행 (인게임 중 로딩) -> 유저 경험
UObject* USampleAssetManager::SynchronousLoadAsset(const FSoftObjectPath& AssetPath)
{
	// FSoftObjectPath : 실제로 들고 있는 것이 아닌, 해당 에셋의 파일 경로를 들고 있는 용도

	if (AssetPath.IsValid())
	{
		//FScopeLogTime 확인
		TUniquePtr<FScopeLogTime> LogTimePtr;
		if (ShouldLogAssetLoads())
		{
			// 단순히 로깅하면서, 초단위로 로깅 진행
			// 동기 로딩의 시간 체크
			LogTimePtr = MakeUnique<FScopeLogTime>(*FString::Printf(TEXT("synchronous loaded assets [%s]"), *AssetPath.ToString()), nullptr, FScopeLogTime::ScopeLog_Seconds);
		}

		// 아래의 두 함수는 정말로 로딩이 안되어 있는 경우에만 동적 로딩이 호출됨
		// 캐싱이 되어있다면 그걸 그대로 가져옴

		// 여기서 두가지 분기:
		// 1. AssetManager가 있으면, AssetManager의 StreamableManager를 통해 정적 로딩
		// 2. 아니면, FSoftObjectPath를 통해 바로 정적 로딩
		if (UAssetManager::IsValid())
		{
			// 진짜 로딩을 진행
			return UAssetManager::GetStreamableManager().LoadSynchronous(AssetPath);
		}

		// if asset manager is not ready, use LoadObject()
		// - 슥 보면, StaticLoadObject가 보인다: 
		// - 참고로, 항상 StaticLoadObject하기 전에 StaticFindObject를 통해 확인하고 실패하면 진짜 로딩함
		// 매우 느린 로딩 방식(어쩔수 없는 경우에만 사용)
		return AssetPath.TryLoad();
	}
	return nullptr;
}

---
해당 함수를 사용하는 템플릿 방식

template <typename AssetType>
AssetType* USampleAssetManager::GetAsset(const TSoftObjectPtr<AssetType>& AssetPointer, bool bKeepsInMemory)
{
	AssetType* LoadedAsset = nullptr;
	const FSoftObjectPath& AssetPath = AssetPointer.ToSoftObjectPath();

	if (AssetPath.IsValid())
	{
		// 로딩이 되어있다? -> 바로 가져옴
		// 로딩이 안되어 있다 -> Null
		LoadedAsset = AssetPointer.Get();
		if (!LoadedAsset)
		{
			// 동기 에셋 로드
			LoadedAsset = Cast<AssetType>(SynchronousLoadAsset(AssetPath));
			ensureAlwaysMsgf(LoadedAsset, TEXT("Failed to load asset [%s]"), *AssetPointer.ToString());
		}

		// 메모리에 남겨두는지 확인
		if (LoadedAsset && bKeepsInMemory)
		{
			// 여기서 AddLoadAsset은 메모리에 상주하기 위한 장치라고 생각하면 됨:
			// - 한번 등록되면 직접 내리지 않는한 Unload가 되지 않음 (== 캐싱)
			Get().AddLoadedAsset(Cast<UObject>(LoadedAsset));
		}
	}

	return LoadedAsset;
}

// 반환을 클래스 타입으로
template <typename AssetType>
TSubclassOf<AssetType> USampleAssetManager::GetSubclass(const TSoftClassPtr<AssetType>& AssetPointer, bool bKeepInMemory)
{
	// AssetType 에 해당하는 녀석들만 참조로 가지려 함
	TSubclassOf<AssetType> LoadedSubclass;

	const FSoftObjectPath& AssetPath = AssetPointer.ToSoftObjectPath();
	if (AssetPath.IsValid())
	{
		// 적절한 클래스가 매칭되었는지를 확인
		LoadedSubclass = AssetPointer.Get();
		if (!LoadedSubclass)
		{
			LoadedSubclass = Cast<UClass>(SynchronousLoadAsset(AssetPath));
			ensureAlwaysMsgf(LoadedSubclass, TEXT("Failed to load asset class [%s]"), *AssetPointer.ToString());
		}

		if (LoadedSubclass && bKeepInMemory)
		{
			Get().AddLoadedAsset(Cast<UObject>(LoadedSubclass));
		}
	}

	return LoadedSubclass;
}

```

- 해당 함수는 static<br>
  따라서 UAssetManager::IsValid() 같은 함수를 사용한다<br>

- 경로가 적절하다면 동기 로딩 실행<br>
  이 때, ShouldLogAssetLoads()을 통해 '동기 로딩'하는 로그를 남겨둔다<br>
  (동기 로딩 체크용)<br>

- GetStreamableManager()::LoadSynchronous<br>
  : 에셋 매니저 내부의 로딩 관리 매니저를 불러와 동기 로딩 요청<br>
    (캐시에 있다면 그대로 return 하지만, 없다면 로드 완료될때까지 대기시킴)<br>
    (불필요하거나 너무 오래걸리면, 비동기 로딩이나 다른 방식으로 바꿀 수 있도록)<br>

- TryLoad?<br>
  : 에셋 매니저가 준비되지 않은 상황임에도 에셋을 로드하는 상황<br>
    매우 로딩이 느리기에, 어쩔수 없는 상황에만 사용한다<br>

  -> 두 동기 로딩 함수는 캐싱된 메모리를 먼저 확인함<br>

- TSoftObjectPtr::Get()<br>
  : 로딩되어 있다면 바로 가져옴<br>
  그렇지 않다면 NULL<br>

- Unreal 엔진 내부의 매니저 클래스이므로<br>
  에디터가 켜지기 전처럼 다양한 환경에서 호출될 수 있음<br>
  그리고 Unreal Engine이 멀티스레드 환경이기에<br>
  에셋이 멀정한지 확인한후, 락을 얻어<br>
  에셋을 캐싱해준다<br>

```
void USampleAssetManager::AddLoadedAsset(const UObject* Asset)
{
	// 주어진 조건이 항상 참인지 확인할때 사용하는 매크로 (디버깅 모드)
	// Asset null 체크
	if (ensureAlways(Asset))
	{
		// 임계영역 잠금 관리용 클래스
		// 생성될때 lock 획득, 범위 종료 시 잠금 해제 (Thread-Safe)
		// SyncObject가 임계영역이므로, 그것에 Lock을 걸고
		// 안전하게 Set에 Add
		FScopeLock Lock(&SyncObject);
		LoadedAssets.Add(Asset);
	}
}
```

## 자주 사용되는 개념들

### Asset
언리얼 에디터 안에서 다루는 (.uasset) 파일 단위를 뜻함<br>
(Texture, Mesh, Sound, BP, DataAsset 등)<br>

- UObject를 상속받음<br>
- 완성된 객체를 저장하는 단위이다<br>
  (그렇기에 여러 데이터의 원본)<br>

실제 데이터 리소스 자체를 의미<br>

### StaticClass
UClass를 얻기 위한 정적 함수<br>
(보통 Class::StaticClass() 방식으로 사용)<br>

- UClass는 리플렉션의 메타데이터<br>
  (프로젝트 실행시, 메모리에 상주)<br>
- GC 대상이 아니므로, 엔진 종료까지 유지<br>

아래의 TSubClassof 혹은 클래스 포인터와 연동하여<br>
객체 생성/ 타입 체크 등에 사용 가능<br>

ex) UClass 의 사용처들<br>
- IsA<> : 객체 타입 확인<br>
- GetProperty() : 속성 접근, 동적으로 읽거나 수정<br>
- NewObject<> 함수를 통해 동적 객체 생성<br>

### TSubclassOf
UClass* 를 감싸는 포인터<br>
(제네릭 안정성을 제공하기에 코드 / 에디터 에서<br>
그 클래스와 자식 클래스만 선택 가능하도록 제한)<br>

- 클래스 선택 슬롯을 에디터에 노출할때 유용<br>
- 런타임 등에서 '스폰할 클래스' 선택 등<br>

특정 계열의 클래스를 안전하게 가리키는 포인터<br>

[리플렉션 참고용 포스팅](https://hnjog.github.io/unreal/UE_GC/){:target="_blank" rel="noopener noreferrer"}<br>

### CDO(Class Default Object)
각 UClass가 하나씩 가지고 있는 '기본 객체'<br>

- 클래스 로드될때 자동으로 생성되며, 항상 메모리에 상주<br>
- 엔진이 새 인스턴스를 만들때 CDO를 복사하여 초기값 세팅<br>
  (즉 CDO를 변경하면 클래스 기본값을 수정하는 것과 동일)<br>
  ('기본값'이기에 save/load 나 네트워크 데이터 등에서<br>
  '달라진 데이터'만 찾기 수월하고, 그 데이터만 별도로 관리함으로서<br>
  최적화가 가능)<br>
- 별도의 인스턴스를 만들지 않아도 기본값을 확인할 수 있음<br>


클래스 단위로 하나 존재하는 기본 객체 자원<br>

## 표로 보는 각 지정 Type들

| 항목                                 | 무엇을 담나                 | 로딩 시점                      | 쿡/패키징 영향      | **생명 주기(Lifecycle)**                                               | 대표 사용처                    | 예시                                    |
| ---------------------------------- | ---------------------- | -------------------------- | ------------- | ------------------------------------------------------------------ | ------------------------- | ------------------------------------- |
| **UObject\*** / **TObjectPtr\<T>** | 이미 로드된 객체 인스턴스         | 즉시 사용(로드 필요)               | 하드 참조 → 항상 포함 | ✅ 객체를 **강하게 붙잡음**. 이 참조가 남아있는 동안 GC가 해제 불가.                        | 컴포넌트, 항상 필요한 리소스          | `UTexture2D* Icon;`                   |
| **TSoftObjectPtr\<T>**             | 객체 경로(FSoftObjectPath) | 필요 시 로드 (`LoadSync/Async`) | 패키지/메모리 부담 ↓  | ⚠️ 객체를 **붙잡지 않음**. 로드 후에도 다른 강참조가 없으면 GC에 의해 해제 가능.                | AssetManager 번들, 지연 로딩 자원 | `TSoftObjectPtr<USoundBase> FireSFX;` |
| **UClass\***                       | 클래스 메타객체               | 항상 상주                      | 쿡 영향 없음       | ♾️ 엔진 메타데이터라 **항상 메모리에 상주**. GC와 무관.                               | 타입 비교, 인스턴스 생성            | `AActor::StaticClass()`               |
| **TSubclassOf\<T>**                | 특정 타입 제한된 UClass\*     | 즉시 사용                      | 클래스 에셋 로드 필요  | ♾️ `UClass*`를 담는 래퍼라 **클래스가 로드되는 한 항상 유지**.                        | 스폰할 클래스 지정                | `TSubclassOf<AActor> EnemyClass;`     |
| **TSoftClassPtr\<T>**              | 클래스 에셋 경로              | 필요 시 로드                    | 패키지/메모리 부담 ↓  | ⚠️ 경로만 저장 → 클래스 로드 시 UClass\*로 변환됨. 하지만 **붙잡지 않음** → 필요 없으면 해제 가능. | BP 클래스 지연 로딩              | `TSoftClassPtr<AMyEnemy> EnemyBP;`    |
| **CDO** (`GetDefaultObject`)       | 클래스 디폴트 객체             | 클래스 로드 시 자동 생성             | 쿡 영향 거의 없음    | ♾️ 클래스 생명 주기와 동일. 클래스가 살아있는 한 **절대 해제되지 않음**.                      | 기본값 확인, 인스턴스 초기화          | `Cls->GetDefaultObject()`             |

