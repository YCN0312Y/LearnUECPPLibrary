## UEC++基础(二)基础知识

![02_世界划分编辑](E:\YU\UE5C++学习笔记\Picture\UEC++Base_2\02_世界划分编辑.png)

![01_GameMode构架关系](E:\YU\UE5C++学习笔记\Picture\UEC++Base_2\01_GameMode构架关系.png)

### AGameMode

AGameMode是AGameModeBase 的子类，拥有一些额外的功能支持多人游戏和旧行为
所有新建项目默认使用AGameModeBase
如果需要此额外行为，可切换到从AGameMode进行继承. 如从 AGameMode 进行继承，也可从 AGameState 继承游戏状态

![03_GameMode头文件和CPP文件](E:\YU\UE5C++学习笔记\Picture\UEC++Base_2\03_GameMode头文件和CPP文件.png)

官方文档：https://dev.epicgames.com/documentation/zh-cn/unreal-engine/game-mode-and-game-state-in-unreal-engine?application_version=5.4

### Game Mode 蓝图

可创建派生自GameMode类的蓝图，并将它们用作项目或关卡的默认GameMode
派生自 GameMode的蓝图可进行以下默认设置：

默认Pawn类
HUD类
玩家控制器类
Spectator类
GameState类
PlayerState类
此外，GameMode的蓝图十分实用，因为它们无需调整代码即可启用变量调整. 因此可用于使单一 GameMode适用到多个不同关卡，无需使用硬编码资源引用或为每次调整请求工程支持和代码修改

设置GameMode

![04_WorldSettings设置GameMode](E:\YU\UE5C++学习笔记\Picture\UEC++Base_2\04_WorldSettings设置GameMode.png)

### 生命周期

Actor生命周期官方文档有详细介绍，这里只介绍常用

- BeginPlay() ：游戏开始时执行一次
- Tick(float DeltaTime) ：游戏运行时每帧执行一次
- EndPlay(const EEndPlayReason::Type EndPlayReason) ：游戏结束或者切换关卡时执行一次

继承AGameMode实现ADemoGameMode

```c++
#pragma once

#include "CoreMinimal.h"
#include "DemoPawn.h"
#include "DemoHUD.h"
#include "DemoPlayerController.h"
#include "DemoGameState.h"
#include "DemoPlayerState.h"
#include "GameFramework/GameMode.h"
#include "DemoGameMode.generated.h"

UCLASS()
class DEMO_API ADemoGameMode : public AGameMode
{
	GENERATED_BODY()

public:
	ADemoGameMode();

public:
	//游戏开始时执行一次
	virtual void BeginPlay() override;
	//游戏运行时每帧执行一次
	virtual void Tick(float DeltaTime) override;
	//游戏结束时或者切换关卡时执行一次
	virtual void EndPlay(const EEndPlayReason::Type EndPlayReason) override;

};

```

```c++
#include "GameMode/DemoGameMode.h"

ADemoGameMode::ADemoGameMode()
{
	DefaultPawnClass = ADemoPawn::StaticClass();
	HUDClass = ADemoHUD::StaticClass();
	PlayerControllerClass = ADemoPlayerController::StaticClass();
	GameStateClass = ADemoGameState::StaticClass();
	PlayerStateClass = ADemoPlayerState::StaticClass();
}

void ADemoGameMode::BeginPlay()
{
	Super::BeginPlay();
}

void ADemoGameMode::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
}

void ADemoGameMode::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
	Super::EndPlay(EndPlayReason);
}

```

### 日志打印

第一个输入参数LogTemp是提供给DEFINE_LOG_CATEGORY宏的类别名称。你可以在位于CoreGlobals.h的引擎中找到这些类别。要自行创建自定义日志记录类别。

第二个输入参数Warning是一个日志详细级别，用于将警告打印到控制台和日志文件中。你可以设置不同的日志详细程度，调整日志的换行模式，或者设置日志的文本颜色。

第三个输入参数Text是C语言库函数printf样式中字符串文字的格式。

```c++
void ADemoGameMode::BeginPlay()
{
	Super::BeginPlay();

	//LogTemp : 临时日志，不会保存到文件中
    //Warning : 警告日志，黄色 记录级别
    //TEXT : 打印内容
    //日志级别常用三种：显示(Display)、警告(Warning)、错误(Error)

	UE_LOG(LogTemp, Display, TEXT("Hello Unreal Engine!"));
	UE_LOG(LogTemp, Warning, TEXT("Hello Unreal Engine!"));
	UE_LOG(LogTemp, Error, TEXT("Hello Unreal Engine!");
}
```

![05_打印日志](E:\YU\UE5C++学习笔记\Picture\UEC++Base_2\05_打印日志.png)

```c++
void ADemoGameMode::BeginPlay()
{
	Super::BeginPlay();
    
    //将文本打印到屏幕上(和蓝图里的打印字符串一样)
    UEngine->AddOnScreenDebugMessage(-1,5.0f,FColor::Blue,TEXT("Hello Unreal Engine！"));

}
```

### 变量类型

```c++
#pragma once

#include "CoreMinimal.h"
#include "DemoPawn.h"
#include "DemoHUD.h"
#include "DemoPlayerController.h"
#include "DemoGameState.h"
#include "DemoPlayerState.h"
#include "GameFramework/GameMode.h"
#include "DemoGameMode.generated.h"

UCLASS()
class DEMO_API ADemoGameMode : public AGameMode
{
	GENERATED_BODY()

public:
	ADemoGameMode();

public:
	//运行时执行
	virtual void BeginPlay() override;
	//运行时每一帧都调用
	virtual void Tick(float DeltaTime) override;
	//运行结束时或者切换关卡时执行
	virtual void EndPlay(const EEndPlayReason::Type EndPlayReason) override;

    //变量类型
	//布尔
	bool MyBool;
	//字节
	BYTE MyByte;
	//整形32位
	int32 Myint32;
	//整形64位
	int64 MyInt64;
	//浮点
	float MyFloat;
	//FName字符串
	FName MyName;
	//FString字符串
	FString MyString;
	//FText字符串
	FText MyText;
	//向量FVector(x,y,z)
	FVector MyVector;
	//旋转FRotator(x,y,z) x轴旋转Roll y轴旋转Pitch z轴旋转Yaw
	FRotator MyRotation;
	//旋转FTransform(P,R,S)位置(Position)+旋转(Rotation)+缩放(Scale)
	FTransform MyTransform;

};
```
