

# UEC++射线检测、Timeline（时间轴）和Timer（定时器）

## 射线检测

射线：在3D空间中生成一根由用户设置长度的线，由用户设置的起点和终点来检测。

玩家可设置射线可以碰到哪些碰撞类型，这个由玩家决定。

### 定义射线示例代码：

```c++
//.cpp文件中
UWorld* World = GetWorld();
if(!World) return;

FVector Start = GetActorLocation();//设置射线开始位置
FVector End = Start + (GetActorForwardVector + 1000.0f);//设置射线结束位置 开始位置+（Actor的向前向量+Value）

FHitResult HitResult;//定义命中结果，用于存储检测到的信息

//单对象射线
bool bDidHit = World->LineTraceSingleByChannel(
        HitResult,      // 存储命中结果的引用
        Start,          // 射线起点
        End,            // 射线终点
        ECC_Visibility, // 碰撞通道
    );

if (bDidHit)
    {
        AActor* HitActor = HitResult.GetActor();//获取命中到的Actor
        UPrimitiveComponent* HitComponent = HitResult.GetComponent();//实际被射线击中的那个具体组件
        FVector ImpactPoint = HitResult.ImpactPoint;// 命中点
        FVector Normal = HitResult.Normal;// 命中点法线

        if (HitActor)
        {
            //如果命中到了Actor就在日志打印被命中的Actor的名字
            UE_LOG(LogTemp, Warning, TEXT("Hit Actor: %s"), *HitActor->GetName());
        }
	
        //在编辑器中绘制调试射线（红色表示命中）
        DrawDebugLine(World, Start, ImpactPoint, FColor::Red, false, 5.0f, 0, 2.0f);//绘制射线的颜色
        DrawDebugPoint(World, ImpactPoint, 10.0f, FColor::Red, false, 5.0f);//绘制命中点的颜色
    }
    else
    {
        // 没有命中任何物体
        // 可选：在编辑器中绘制调试射线（绿色表示未命中）
        DrawDebugLine(World, Start, End, FColor::Green, false, 5.0f, 0, 2.0f);
    }
```

### 常用的检测方法：

```c++
LineTraceSingleByChannel();//单对象检测
LineTraceMultiByChannel();//多对象检测
LineTraceSingleByObjectType();//碰撞通道检测
SweepSingleByChannel();//形状扫描检测
```

------



## Timeline（时间轴）

用于Actor的状态平滑的过渡到Actor的另一个状态 。

三个参数：StartValue开始值；TargetValue目标值；TransitionPeriod过渡时间；

例如：一个Actor的初始大小为1，想在1秒内大小变成2。

开始值 = 1；目标值 = 2；过渡时间 = 1；

平滑的从1到2。

示例代码：

```c++
//.h文件中
#pragma once
#include "CoreMinimal.h"
#include "Components/TimelineComponent.h"
#include "GameFramework/Actor.h"
#include "MyActor.generated.h"

class UStaticMeshComponent;

UCALSS()
class DEMO_API AMyActor : public AActor
{
    GENERATED_BODY();
public:
    AMyActor();
protected:
    virtual void BeginPlay() override;
public:
    virtual void Tick(float DeltaTime);
protected:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Timeline")
    UStaticMeshComponent* StaticMesh;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Timeline")
    UTimelineComponent* ActorSizeTimeLine;//时间轴
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Timeline")
    UCurveFloat* TransitionPeriod;//过渡时间的曲线、
    
    FOnTimelineFloat OnActorSizeTimeline;//时间轴更新代理

    UFUNCTION()
    void OnTimelineUpdate(float Alpha);
}

```

```c++
//.cpp文件中
#include "Actor/MyActor.h"
#include "Components/StaticMeshComponent.h"

void AMyActor::AMyActor()
{
    PrimaryActorTick.bCanEverTick = flase;
    
    ActorSizeTimeLine = CreateDefultSubObject<UTimelineComponent>(TEXT("ActorSizeTimeLine"));
    
}

void AMyActor::BeginPlay()
{
    Super::BeginPlay();
    
    OnActorSizeTimeline.BindUFunction("OnTimelineUpdate");//绑定执行函数
    ActorSizeTimeline->AddInterpFloat(TransitionPeriod, OnActorSizeTimeline);//添加浮点数插值轨道
}

void AMyActor::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);
}

void AMyActor::OnTimelineUpdate(float Alpha)
{
    FVector TempVector = FMath::Lerp(1, 2, Alpha);
    StaticMesh->SetActorRelativeScale3D(TempVector);
}
```

## 定时器

用于延迟执行操作，定时器适合简单的延迟和周期性任务。

有两种方法：延迟定时器和循环定时器

```c++
//.cpp文件
#include "TimrManager.h"
FTimerManager& TimerManager = GetWorld()->GetTimeManager();

UWorld* World = GetWorld();
if(World)
{
    FTimerManager& TimerManager = World->GetTimeManager();
}
```

### 延迟定时器使用：

延迟执行绑定的函数，可以设置变量，什么时候停止。

```c++
FTimerHandle MyTimerHandle;
GetWorld()->GetTimeManager().SetTimer(
    MyTimerHandle,          //定时器句柄，用于后续控制
    this,                   //调用对象
    &AMyActor::MyDelayedFunction, //要执行的函数
    2.0f,                   //延迟时间（秒）
    false  //不循环
)
```

### 循环定时器使用：

循环执行绑定的函数，可以设置变量，什么时候停止循环。

```c++
FTimerHandle MyTimerHandle;
GetWorld()->GetTimeManager().SetTimer(
    MyTimerHandle,          // 定时器句柄，用于后续控制
    this,                   // 调用对象
    &AMyActor::MyDelayedFunction, // 要执行的函数
    1.0f,                   //每1秒执行一次
    true  //是否循环
)
```

### 带参数的Lambda：

```c++
FTimerHandle MyTimerHandle;
GetWorld()->GetTimeManager().SetTimer(
    MyTimerHandle,          // 定时器句柄，用于后续控制
[this]()//Lambda捕获列表
{
    //执行代码
    UE_LOG(LogTemp, Warning, TEXT("Lambda定时器"));
}
    1.0f,                   //每1秒执行一次
    true  //是否循环
)
```

### 定时器的其他属性方法

```c++
//暂停定时器
TimerManager.PauseTimer(MyTimerHandle);
//恢复定时器
TimerManager.UnPauseTimer(MyTimerHandle);
//清除指定定时器（注：可以析构或摧毁时清楚定时器）
TimerManager.ClearTimer(MyTimerHandle);
//清除所有定时器（谨慎使用）
TimerManager.ClearAllTimersForObject(this)
```

### 定时器避免问题：

避免在 Tick 中频繁设置/清除定时器。

使用成员变量存储 TimerHandle，避免重复创建。

对于高频操作，考虑使用自定义的计时逻辑。