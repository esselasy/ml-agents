# 새로운 학습 환경 만들기

이 튜토리얼은 유니티 환경을 만드는 과정을 안내합니다. 유니티 환경은 유니티 엔진을 사용하여 빌드 된 애플리케이션으로, 강화 학습 에이전트를 교육하는 데 사용할 수 있습니다.

![A simple ML-Agents environment](images/mlagents-NewTutSplash.png)

이 예에서는 공을 무작위로 배치된 큐브를 향해 굴리도록 훈련시킵니다. 공은 플랫폼에서 떨어지지 않는 방법을 배웁니다.

## 개요

유니티 프로젝트에서 ML-Agents 툴킷을 사용하려면 다음과 같은 기본 단계가 필요합니다:

1. 에이전트가 살 수 있는 환경을 만듭니다. 환경은 몇 가지 오브젝트를 포함하는 간단한 물리적 시뮬레이션에서부터 전체 게임 또는 생태계까지 다양합니다.
2. Academy를 상속하는 서브 클래스를 구현하고 환경으로 사용할 유니티 씬(Scene)의 게임 오브젝트에 추가하십시오. 아카데미 서브 클래스에는 에이전트와 독립적으로 Scene을 업데이트하는 몇 가지 메소드를 선택적으로 구현할 수 있습니다. 예를 들어, 환경에서 에이전트 및 기타 엔티티를 추가, 이동 또는 삭제할 수 있습니다.
3. **Assets** > **Create** > **ML-Agents** > **Brain**을 클릭하고 적당한 이름을 지정하여 한 개 이상의 브레인 에셋을 만듭니다.
4. Agent를 상속하는 서브 클래스를 구현합니다. 환경을 관찰하고, 할당된 동작를 실행하고, 강화 훈련에 사용되는 보상을 계산하는 코드를 에이전트 서브 클래스에 정의합니다. 에이전트가 작업을 완료하거나 실패한 경우 에이전트를 재설정하는 메소드를 구현할 수도 있습니다.
5. 에이전트 서브 클래스를 적당한 게임 오브젝트, 일반적으로 시뮬레이션에서 에이전트를 나타내는 Scene의 오브젝트에 추가하십시오. 각 에이전트에는 브레인이 할당되어야 합니다.
6. 훈련을 시작하려면 아카데미의 Broadcast Hub에 있는 `Control` 체크 박스를 체크해야 합니다. [훈련을 실행합니다](Training-ML-Agents.md).

**참고:** 유니티에 익숙하지 않은 경우, 이 튜토리얼에서 설명하는 에디터 작업을 따라하기 어렵다면 Unity 매뉴얼 [인터페이스 배우기](https://docs.unity3d.com/Manual/LearningtheInterface.html)을 참조하십시오.

아직 설치하지 않은 경우, [설치 방법](Installation.md)을 따르십시오.

## 유니티 프로젝트 설정

가장 먼저 할 일은 새로운 유니티 프로젝트를 만들고 ML-Agents 에셋을 프로젝트로 가져오는 것입니다:

1. 유니티 에디터를 시작하고 "RollerBall"이라는 새 프로젝트를 만듭니다.
2. 프로젝트 설정의 스크립팅 런타임 버전이 **.NET 4.x Equivalent**를 사용하도록 되어 있는지 확인합니다. (이 설정은 유니티 2017에서는 실험 옵션이지만 2018.3부터 기본값입니다)
3. 파일 시스템에서 복제된 ML-Agents 리포지토리가 포함된 폴더로 이동합니다.
4. `UnitySDK/Assets`에 있는 `ML-Agents`및 `Gizmos` 폴더를 유니티 에디터의 프로젝트 창으로 드래그합니다.

유니티 **Poject** 창에는 다음과 같이 에셋이 포함되어야 합니다:

![Project window](images/mlagents-NewProject.png)

## 환경 만들기

다음으로 ML-Agents 환경으로 작동하는 매우 간단한 씬을 만듭니다. 환경의 "물리적" 구성 요소에는 에이전트가 움직일 수 있는 바닥 역할을 하는 평면, 에이전트가 추구하는 목표 또는 대상 역할을 하는 큐브 및 에이전트를 나타내는 구가 포함됩니다.

### 바닥 평면 생성

1. 계층 창에서 마우스 오른쪽 버튼을 클릭하고 3D Object > Plane을 선택합니다.
2. GameObject의 이름을 "Floor"로 지정합니다.
3. Floor 평면을 선택해서 인스펙터 창에서 속성을 봅니다.
4. Transform의 Position = (0, 0, 0), Rotation = (0, 0, 0), Scale = (1, 1, 1)로 설정합니다.
5. Mesh Renderer의 Materials 속성을 확장하고 *Default-Material*을 *LightGridFloorSquare* (또는 원하는 적합한 재질)로 변경합니다.

(새 머티리얼을 설정하려면 현재 머티리얼 이름 옆에있는 작은 원 아이콘을 클릭합니다. 그러면 프로젝트에 있는 모든 머티리얼 목록에서 다른 머티리얼을 선택할 수 있는 선택 다이얼로그가 열립니다.)

![The Floor in the Inspector window](images/mlagents-NewTutFloor.png)

### 대상 큐브 추가

1. 계층 창에서 마우스 오른쪽 버튼을 클릭하고 3D Object > Cube를 선택합니다.
2. GameObject 이름을 "Target"으로 지정합니다.
3. Target 큐브를 선택해서 인스펙터 창에서 속성을 봅니다.
4. Transform의 Position = (3, 0.5, 3), Rotation = (0, 0, 0), Scale = (1, 1, 1)로 설정합니다.
5. Mesh Renderer의 Materials 속성을 확장하고 *Default-Material*을 *Block*으로 변경합니다.

![The Target Cube in the Inspector window](images/mlagents-NewTutBlock.png)

### 에이전트 구 추가

1. 계층 창에서 마우스 오른쪽 버튼을 클릭하고 3D Object > Sphere를 선택합니다.
2. GameObject의 이름을 "RollerAgent"로 지정합니다.
3. RollerAgent 구를 선택해서 인스펙터 창에서 속성을 봅니다.
4. Transform의 Position = (0, 0.5, 0), Rotation = (0, 0, 0), Scale = (1, 1, 1)로 설정합니다.
5. Mesh Renderer의 Materials 속성을 확장하고 *Default-Material*을 *CheckerSquare*로 변경합니다.
6. **Add Component**를 클릭합니다.
7. Physics/Rigidbody 컴포넌트를 추가합니다.

![The Agent GameObject in the Inspector window](images/mlagents-NewTutSphere.png)

이 튜토리얼의 뒷부분에서 이 GameObject에 컴포넌트로 추가할 에이전트 서브 클래스를 만들게 됩니다.

### 아카데미용 빈 게임 오브젝트 추가

1. 계층 창에서 마우스 오른쪽 단추를 클릭하고 Create Empty를 선택합니다.
2. GameObject의 이름을 "Academy"로 지정합니다.

![The scene hierarchy](images/mlagents-NewTutHierarchy.png)

런타임시 씬을 더 잘 볼 수 있도록 카메라 각도를 조정할 수 있습니다. 다음 단계는 ML-Agent 컴포넌트를 만들고 추가하는 것입니다.

## 아카데미 구현

아카데미는 씬에서 ML-Agents를 조정하고 반복 시뮬레이션에서 의사 결정 부분을 주도합니다. 모든 ML-Agent 씬에는 하나의 아카데미 인스턴스가 필요합니다. 베이스 Academy 클래스는 추상 클래스이기 때문에 특정 환경에서 어떤 메소드도 사용할 필요가 없는 경우에도 자신만의 서브 클래스를 만들어야 합니다.

먼저 앞에서 만든 Academy GameObject에 새 스크립트 컴포넌트를 추가합니다:

1. Academy GameObject를 선택하여 인스펙터 창에서 봅니다.
2. **Add Component**를 클릭합니다.
3. 컴포넌트 목록에서 맨 아래에 있는 **New Script**를 클릭합니다.
4. 스크립트 이름을 "RollerAcademy"로 지정합니다.
5. **Create and Add**를 클릭합니다.

다음으로 새 `RollerAcademy` 스크립트를 편집합니다.

1. 프로젝트 창에서 `RollerAcademy` 스크립트를 두 번 클릭하여 코드 편집기에서 엽니다. 기본적으로 새 스크립트는 **Assets** 폴더에 배치됩니다.
2. 코드 편집기에서 `using MLAgents;`를 추가합니다.
3. 기본 클래스를 `MonoBehaviour`에서 `Academy`로 변경합니다.
4. 기본으로 추가된 Start() 및 Update() 메소드를 삭제합니다.

이러한 기본 씬에서는 아카데미가 환경에서 오브젝트를 초기화, 재설정 또는 제어할 필요가 없으므로 간단하게 아카데미 구현이 가능합니다:

```csharp
using MLAgents;

public class RollerAcademy : Academy { }
```

아카데미 속성의 기본 설정이 환경에 적합하므로 인스펙터 창에서 RollerAcademy 컴포넌트에 대한 내용을 변경할 필요가 없습니다. 아직 Broadcast Hub에 RollerBrain이 없지만 나중에 추가하게 됩니다.

![The Academy properties](images/mlagents-NewTutAcademy.png)

## 브레인 에셋 추가

브레인은 의사 결정 프로세스를 캡슐화합니다. 에이전트는 자신의 관찰을 브레인에게 보내고 그에 대한 결정을 기대합니다. 브레인 유형 (학습, 휴리스틱 또는 플레이어)에 따라 브레인이 결정을 내리는 방법이 결정됩니다.

브레인을 만들려면:
1. **Assets** > **Create** > **ML-Agents**를 클릭하고 생성하려는 브레인 에셋 유형을 선택합니다. 이 튜토리얼에서는 **Learning Brain** 및 **Player Brain**을 생성합니다.
2. 이름을 각각 `RollerBallBrain`과 `RollerBallPlayer`으로 지정합니다.

![Creating a Brain Asset](images/mlagents-NewTutBrain.png)

지금은 `RollerBallBrains` 의 Model 속성을 그대로 `None`으로 둡니다. **Learning Brain**에 모델을 추가하기 전에 먼저 모델을 훈련시켜야 합니다.

## 에이전트 구현

에이전트를 작성하려면:

1. RollerAgent GameObject를 선택하여 인스펙터 창에서 봅니다.
2. **Add Component**를 클릭합니다.
3. 컴포넌트 목록에서 맨 아래에 있는 **New Script**를 클릭합니다.
4. 스크립트 이름을 "RollerAgent"로 지정합니다.
5. **Create and Add**를 클릭합니다.

그리고나서 새 `RollerAgent` 스크립트를 편집합니다:

1. 프로젝트 창에서 `RollerAgent` 스크립트를 두 번 클릭하여 코드 편집기에서 엽니다.
2. 편집기에서 `using MLAgents;` 문을 추가한 후 베이스 클래스를 `MonoBehaviour`에서 `Agent`로 변경합니다.
3. Update() 메소드는 삭제하지만 Start() 함수는 사용하므로 지금은 그대로 놔둡니다.

지금까지 한 것은 ML-Agents를 유니티 프로젝트에 추가하기 위한 기본적인 단계입니다. 다음으로 에이전트가 강화 학습을 사용하여 대상 큐브를 향해 굴러가는 것을 배우게 하는 로직을 추가합니다.

이 간단한 시나리오에서는 아카데미를 사용하여 환경을 제어하지 않습니다. 시뮬레이션 전이나 도중에 바닥의 크기를 변경하거나, 에이전트 또는 기타 오브젝트를 추가 또는 제거하는 등 환경을 변경하려는 경우, 아카데미에서 적절한 메소드를 구현할 수도 있습니다. 여기서는 아카데미 대신 에이전트가 시도를 성공하거나 실패할 때 자신과 대상을 재설정하는 모든 작업을 수행하게 합니다.

### 에이전트 초기화 및 재설정

에이전트가 목표 대상에 도달하면 에이전트는 완료된 것으로 표시하고 에이전트 재설정 함수는 대상을 임의의 위치로 이동합니다. 또한 에이전트가 바닥 평면 아래로 떨어지면 재설정 함수가 에이전트를 다시 바닥 위에 놓습니다.

대상 게임 오브젝트를 이동하려면 Transform (3D 월드에서 게임 오브젝트의 위치, 방향 및 스케일을 저장)에 대한 참조(reference)가 필요합니다. 이 참조를 얻으려면 RollerAgent 클래스에 Transform 타입의 public 필드를 추가합니다. 유니티에서 컴포넌트의 public 필드는 인스펙터 창에 표시되므로 유니티 에디터에서 대상으로 사용할 게임 오브젝트를 선택할 수 있습니다.

에이전트의 속도를 재설정하고 나중에 에이전트를 이동시키기 위해 힘을 가하려면 Rigidbody 컴포넌트에 대한 참조가 필요합니다. [Rigidbody](https://docs.unity3d.com/ScriptReference/Rigidbody.html)는 물리 시뮬레이션을 위한 유니티의 주요 요소입니다 (유니티 [Physics](https://docs.unity3d.com/Manual/PhysicsSection.html) 참고). Rigidbody 컴포넌트가 같은 게임 오브젝트에 있기 때문에, Rigidbody에 대한 참조를 얻을 수 있는 가장 좋은 방법은 에이전트 스크립트의 `Start()`에서 GameObject.GetComponent<T>()를 사용하는 것입니다.

지금까지 작성한 RollerAgent 스크립트는 다음과 같습니다:

```csharp
using System.Collections.Generic;
using UnityEngine;
using MLAgents;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    void Start () {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public override void AgentReset()
    {
        if (this.transform.position.y < 0)
        {
            // If the Agent fell, zero its momentum
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.position = new Vector3( 0, 0.5f, 0);
        }

        // Move the target to a new spot
        Target.position = new Vector3(Random.value * 8 - 4,
                                      0.5f,
                                      Random.value * 8 - 4);
    }
}
```

다음으로 `Agent.CollectObservations()` 메소드를 구현해 봅니다.

### 환경 관찰하기

에이전트는 우리가 수집한 정보를 브레인에 전송하고 이를 사용하여 동작을 결정합니다. 에이전트를 훈련하거나 훈련된 모델을 사용하면 데이터가 신경망에 특성 벡터(feature vector)로 제공됩니다. 에이전트가 과제를 성공적으로 배우려면 올바른 정보를 제공해야 합니다. 수집할 정보를 결정하는 데 있어 가장 좋은 방법은 문제에 대한 분석적 솔루션을 찾는 데 필요한 사항을 고려하는 것입니다.

이 예제의 경우 에이전트는 다음과 같은 정보를 수집합니다:

* 대상의 위치 

```csharp
AddVectorObs(Target.position);
```

* 에이전트의 위치

```csharp
AddVectorObs(this.transform.position);
```

* 에이전트의 속도. 이렇게 하면 에이전트가 속도를 제어하는 ​​방법을 익히므로 대상을 빗나가거나 바닥에서 떨어지지 않습니다.

```csharp
// Agent velocity
AddVectorObs(rBody.velocity.x);
AddVectorObs(rBody.velocity.z);
```

종합하면 상태 관측에는 8개의 값이 포함되어 있고 브레인의 속성을 설정할 때 연속 상태 공간(continuous state space)을 사용해야 합니다:

```csharp
public override void CollectObservations()
{
    // Target and Agent positions
    AddVectorObs(Target.position);
    AddVectorObs(this.transform.position);

    // Agent velocity
    AddVectorObs(rBody.velocity.x);
    AddVectorObs(rBody.velocity.z);
}
```

에이전트 코드의 마지막 부분은 Agent.AgentAction() 메소드입니다. 브레인으로부터 결정을 받고 보상을 할당합니다.

### 동작

브레인의 결정은 `AgentAction()`함수에 전달되는 action 배열의 형태로 제공됩니다. 이 배열의 크기는 에이전트의 브레인 설정에 있는 `Vector Action`, `Space Type` 및 `Space Size` 에 따라 결정됩니다. RollerAgent는 연속 벡터 액션 스페이스(continuous vector action space)을 사용하며 브레인으로부터 오는 두 가지 연속된 제어 신호를 필요로 합니다. 따라서 브레인의 `Space Size`를 2로 설정합니다. 첫 번째 요소 `action[0]`는 x축을 따라 가해지는 힘을 결정합니다. `action[1]`은 z축을 따라 적용되는 힘을 결정합니다. (에이전트가 3차원으로 이동할 수 있을 경우 `Vector Action Size`을 3으로 설정해야 합니다.) 브레인은 동작 배열의 값이 실제로 무엇을 의미하는지 전혀 알지 못합니다. 훈련 과정은 관측값에 대한 응답으로 동작 값을 조정한 다음, 결과로 어떤 보상이 제공되는지 확인합니다.


RollerAgent는 `action[]` 배열의 값을 `Rigidbody.AddForce` 함수를 사용해서 Rigidbody 컴포넌트인 `rBody`에 적용합니다:

```csharp
Vector3 controlSignal = Vector3.zero;
controlSignal.x = action[0];
controlSignal.z = action[1];
rBody.AddForce(controlSignal * speed);
```

### 보상

강화 학습에는 보상이 필요합니다. `AgentAction()` 함수에서 보상을 할당합니다. 학습 알고리즘은 시뮬레이션 및 학습 프로세스 중 에이전트에 할당된 보상을 통해 에이전트에게 최적의 동작이 제공되는지 여부를 판별합니다. 할당된 작업을 완료한 에이전트에게는 보상을 제공하려고 합니다. 이 경우 에이전트는 대상 큐브에 도달하면 1.0의 보상을 받습니다.

RollerAgent는 대상에 도달했는지 알기 위해 둘 간의 거리를 계산합니다. 이 경우 코드는 `Agent.SetReward()` 메소드를 호출하여 보상 1.0을 지정하고 에이전트에서 `Done()` 메소드를 호출하여 에이전트를 완료된 것으로 표시합니다.

```csharp
float distanceToTarget = Vector3.Distance(this.transform.position,
                                          Target.position);
// Reached target
if (distanceToTarget < 1.42f)
{
    SetReward(1.0f);
    Done();
}
```

**참고:** 에이전트를 완료로 표시하면 재설정 될 때까지 에이전트의 활동이 중지됩니다. 인스펙터 창에서 Agent.ResetOnDone 속성을 true로 설정하여 에이전트를 즉시 재설정하거나 아카데미가 환경을 재설정할 때까지 기다릴 수 있습니다. 이 RollerBall 환경은 `ResetOnDone` 메커니즘을 사용하며 아카데미의 `Max Steps` 설정을 사용하지 않습니다. (따라서, 환경을 재설정하지 않습니다.)

마지막으로 에이전트가 바닥에서 떨어지면 에이전트를 자체적으로 재설정하도록 합니다.

```csharp
// Fell off platform
if (this.transform.position.y < 0)
{
    Done();
}
```

### AgentAction()

위에서 설명한 동작 및 보상 로직을 사용한 `AgentAction()` 함수의 최종 버전은 다음과 같습니다:

```csharp
public float speed = 10;
public override void AgentAction(float[] vectorAction, string textAction)
{
    // Actions, size = 2
    Vector3 controlSignal = Vector3.zero;
    controlSignal.x = vectorAction[0];
    controlSignal.z = vectorAction[1];
    rBody.AddForce(controlSignal * speed);

    // Rewards
    float distanceToTarget = Vector3.Distance(this.transform.position,
                                              Target.position);

    // Reached target
    if (distanceToTarget < 1.42f)
    {
        SetReward(1.0f);
        Done();
    }

    // Fell off platform
    if (this.transform.position.y < 0)
    {
        Done();
    }

}
```

함수 앞에 정의된 `speed` 클래스 변수에 유의하십시오. speed는 public 변수이므로 인스펙터 창에서 값을 설정할 수 있습니다.

## 에디터 설정 마무리

모든 게임 오브젝트와 ML-Agent 컴포넌트가 완성되었으므로 이제 Unity 에디터에서 모든 것을 함께 연결할 차례입니다. 여기에는 브레인 에셋을 에이전트에 할당하고 일부 에이전트 컴포넌트의 속성을 변경하며 에이전트 코드와 호환되도록 브레인 속성을 설정해야합니다.

1. 계층 창에서 Academy를 선택하고 인스펙터에서 **Broadcast Hub**에 `RollerBallBrain`과 `RollerBallPlayer` 브레인을 추가합니다.
2. **RollerAgent** 게임 오브젝트를 선택해서 속성을 인스펙터 창에 표시합니다.
3. 프로젝트 창에서 **RollerBallPlayer** 브레인을 RollerAgent의 **Brain** 필드로 드래그합니다.
4. **Decision Interval**을 1에서 10으로 변경합니다.
5. 계층 창에서 **Target** 게임 오브젝트를 RollerAgent의 **Target** 필드로 드래그합니다.

![Assign the Brain to the RollerAgent](images/mlagents-NewTutAssignBrain.png)

마지막으로 **프로젝트** 창에서 **RollerBallBrain** 에셋을 선택하고 다음과 같이 속성을 설정합니다:

* `Vector Observation` `Space Size` = 8
* `Vector Action` `Space Type` = **Continuous**
* `Vector Action` `Space Size` = 2

**프로젝트** 창에서 **RollerBallPlayer** 에셋을 선택하고 속성값을 동일하게 설정합니다.

이제 훈련을 시작하기 전에 환경을 테스트할 준비가 되었습니다.

## 환경 테스트

훈련을 시작하기 전에 항상 수동으로 환경을 테스트하는 것이 좋습니다. `RollerBallPlayer` 브레인을 만든 이유는 직접 키보드를 사용하여 에이전트를 제어할 수 있기 때문입니다. 먼저 키보드와 동작 간의 매핑을 정의해야 합니다. RollerAgent는 `Action Size`가 두 개이지만 한 키에는 양수 값을 지정하고 한 키에는 음수 값을 지정하는 식으로 하면 총 4 개의 키를 사용할 수 있습니다.

1. RollerBallPlayer 에셋을 선택하여 인스펙터 창에서 속성을 봅니다.
2. **Key Continuous Player Actions**를 확장합니다. (`PlayerBrain`을 사용할 때만 표시됩니다.)
3. **Size**를 4로 설정합니다.
4. 다음과 같이 매핑을 설정합니다.

| Element   | Key | Index | Value |
| :-------- | :-: | :---: | :---: |
| Element 0 | D   | 0     | 1     |
| Element 1 | A   | 0     | -1    |
| Element 2 | W   | 1     | 1     |
| Element 3 | S   | 1     | -1    |

**Index** 값은 `AgentAction()`에 전달되는 action 배열의 인덱스에 대응됩니다. **Key**를 누르면 **Value**가 action[Index]에 할당됩니다.

**Play**를 눌러 씬을 실행하고 WASD 키를 사용하여 에이전트를 플랫폼 주위로 이동해 보십시오. 유니티 에디터 Console 창에 오류가 표시되지 않고 에이전트가 대상에 도달하거나 바닥에서 떨어질 때 에이전트가 재설정되는지 확인합니다. ML-Agents SDK에는 보다 복잡한 디버깅을 위해 게임 창에 에이전트 상태 정보를 쉽게 표시하는데 사용할 수 있는 편리한 Monitor 클래스가 포함되어 있습니다.

추가적으로 `notebooks/getting-started.ipynb` [Jupyter notebook](Background-Jupyter.md)을 사용하여 환경과 Python API가 예상대로 작동하는지 테스트합니다. 주피터 노트북 내 env_name을 이 환경을 빌드할 때 지정한 환경 파일의 이름으로 설정하십시오.

## 환경 훈련하기

이제 에이전트를 훈련시킬 수 있습니다. 학습 준비를 하려면 먼저 `RollerBallBrain` 에셋을 **RollerAgent** 게임 오브젝트의 `Brain` 필드로 드래그해서 학습 브레인으로 변경해야 합니다. 그런 다음 Academy 게임 오브젝트를 선택한 뒤 **Broadcast Hub**의  RollerBallBrain 리스트에서 `Control` 체크 박스를 체크합니다. 이 과정은 [Training ML-Agents](Training-ML-Agents.md)에 설명된 것과 동일합니다. 모델은 원본 ml-agents 프로젝트 폴더인 `ml-agents/models`에 생성됩니다.

훈련용 하이퍼파라미터는 `mlagents-learn` 프로그램에 전달하는 설정 파일에 지정되어 있습니다. 원본 `ml-agents/config/trainer_config.yaml` 파일에 지정된 기본 설정을 사용하여 RollerAgent는 약 30만 번의 스텝 동안 훈련합니다. 다음 하이퍼파라미터를 변경하여 훈련 속도를 높일 수 있습니다 (20,000 스텝 미만):

    batch_size: 10
    buffer_size: 100

이 예제는 입력 및 출력이 거의 없는 매우 간단한 훈련 환경을 만들기 때문에 작은 배치 크기와 버퍼 크기를 사용하면 훈련 속도가 상당히 빨라집니다. 그러나 환경을 더 복잡하게 하거나 보상 또는 관찰 기능을 변경하면 다른 하이퍼파라미터 값으로 훈련이 더 잘 수행될 수도 있습니다.

**참고:** 이러한 하이퍼파라미터 값을 설정하는 것 외에도 에이전트의 **DecisionFrequency** 매개 변수는 훈련 시간 및 성공에 큰 영향을 줍니다. 값이 클수록 훈련 알고리즘이 고려해야 하는 결정의 수가 줄어들어서 간단한 환경에서는 훈련 속도가 빨라집니다.

에디터에서 훈련하려면 플레이를 누르기 전에 터미널 또는 콘솔 창에서 다음 Python 명령을 실행합니다:

    mlagents-learn config/config.yaml --run-id=RollerBall-1 --train

(여기서 `config.yaml`은 수정된 `batch_size` 및 `buffer_size` 하이퍼파라미터를 가지고 있는 `trainer_config.yaml`의 복사본입니다.)`

**참고:** 명령을 실행할 때 `command not found`이 오류가 발생하면, ML-Agents [설치](Installation.md)에 있는 *파이썬과 mlagents 패키지 설치* 과정을 따랐는지 확인합니다.

훈련 중 에이전트의 성능 통계를 모니터링하려면 [텐서보드(TensorBoard)](Using-Tensorboard.md)를 사용하십시오.

![TensorBoard statistics display](images/mlagents-RollerAgentStats.png)

특히 *cumulative_reward* 및 *value_estimate* 통계는 에이전트가 작업을 얼마나 잘 수행하고 있는지 보여줍니다. 이 예에서 에이전트가 받을 수 있는 최대 보상은 1.0이므로 에이전트가 문제를 성공적으로 해결하면 이러한 통계가 해당 값에 근접합니다.

**참고:** 텐서보드를 사용하는 경우 항상 `mlagents-learn` 명령에 전달하는 `run-id` 값을 늘리거나 변경하십시오. 동일한 id 값을 사용하면 여러 실행에 대한 통계가 합쳐져서 해석하기가 어려워집니다.

## 선택 사항 : 같은 씬에서 다수의 훈련 영역 사용

많은 [예시 환경](Learning-Environment-Examples.md)에서 다수의 훈련 영역 사본이 씬에서 인스턴스화됩니다. 이는 일반적으로 훈련 속도를 높여 환경이 여러 경험을 동시에 모을 수 있게합니다. 동일한 브레인을 공유하는 많은 에이전트를 인스턴스화하여 간단하게 할 수 있습니다. 다음 단계를 따라 RollerBall 환경을 병렬화하십시오.

### 다수의 훈련 영역 인스턴스화

1. 계층 창을 마우스 오른쪽 버튼으로 클릭하고 빈 게임 오브젝트를 만들어서 이름을 TrainingArea로 지정합니다.
2. TrainingArea의 Transform에서 Position (0,0,0), Rotation (0, 0, 0), Scale = (1,1,1)로 설정합니다.
3. 계층 창의 Floor, Target 및 RollerAgent 게임 오브젝트를 TrainingArea 게임 오브젝트로 드래그합니다.
4. TrainingArea 게임 오브젝트를 포함된 다른 게임 오브젝트와 함께 프로젝트 창의 Assets로 드래그하여 프리팹으로 바꿉니다.
5. 이제 TrainingArea 프리팹의 사본을 인스턴스화 할 수 있습니다. 프리팹을 씬으로 드래그하여 겹치지 않도록 배치하십시오.

### 스크립트 편집

이전 섹션에서 TrainingArea가 (0,0,0)에 있다고 가정하고 `this.transform.position.y < 0`으로 에이전트가 바닥에서 떨어졌는지 여부를 확인하는 방식으로 수행하는 스크립트를 작성했습니다. 씬에서 여러 TrainingArea를 사용하려면 이를 변경해야 합니다.

현재 코드를 빠르게 변형하는 방법은 position 대신 localPosition을 사용하여 위치 좌표가 월드 좌표가 아닌 TrainingArea 프리팹 내의 위치를 ​​참조하도록 하는 것입니다.

1. RollerAgent.cs에 있는 모든 `this.transform.position` 참조를 `this.transform.localPosition`으로 바꿉니다.
2. RollerAgent.cs에 있는 모든 `Target.position` 참조를 `Target.localPosition`으로 바꿉니다.

이것은 목표를 달성하는 한 가지 방법일 뿐입니다. 상대적 포지셔닝을 사용할 수 있는 다른 방법은 [예시 환경]((Learning-Environment-Examples.md)을 참조하십시오.

## 검토: 씬 레이아웃

이 섹션에서는 유니티 환경에서 에이전트를 사용할 때 씬을 구성하는 방법을 간략하게 검토합니다.

유니티 ML-Agents를 사용하려면 씬에 포함해야 하는 두 가지 종류의 게임 오브젝트가 있습니다: 아카데미와 하나 이상의 에이전트입니다. 또한 브레인 에셋이 에이전트와 아카데미에 적절하게 연결되어 있어야 합니다.

명심하십시오:

한 씬에는 하나의 아카데미 게임 오브젝트만 있을 수 있습니다.
아카데미의 Broadcast Hub 목록에 추가된 학습 브레인만 훈련할 수 있습니다.

## 한글 번역
이 문서의 한글 번역은 [사이안](https://github.com/esselasy)에 의해 진행되었습니다. 내용상 오류나 오탈자가 있는 경우 링크를 통해 연락주시면 감사드리겠습니다.
