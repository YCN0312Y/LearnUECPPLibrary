# **UEC++学习记录**

## 环境配置设置

UE源码可以自行在网络上查找下载因为我这边一进源码版网址就404无法提供下载方式

Unreal Engine版本：Unreal Engine5.4.4源码版

Visual Studio版本：Visual Studio 2022

![UE5Visual Studio界面介绍1](C:\UE\UE5C++学习笔记\Picture\UEC++Base_1\UE5Visual Studio界面介绍1.png)

调试模式：DebugGame Editor

UE5：UE官方开发的包

Programs：UE官方提供的一些独立程序可提供给开发者自行使用

## UEC++基础(一)入门介绍

### 创建项目

点击上面的Local Windows Debugger启动UE5

![UE5启动界面介绍](C:\UE\UE5C++学习笔记\Picture\UEC++Base_1\UE5启动界面介绍.png)

左边的五大块是不同类型的项目创建(游戏、电影/视频&直播活动、建筑、汽车产品设计与制造、模拟)

中间的八小块是八中不同类型的项目案例 (空白、第一人称、拼图、第三人称、俯视角、AR、虚拟现实、赛车)

这里我选择的是Blank空白案例

Project Location：可以自己设置项目存放路径(不能包含中文路径)

Project Name：是项目名称(尽量用英文名称)

![UE5界面介绍](C:\UE\UE5C++学习笔记\Picture\UEC++Base_1\UE5界面介绍.png)

会自动生成并打开打开VS文件

![UE5Visual Studio界面介绍2](C:\UE\UE5C++学习笔记\Picture\UEC++Base_1\UE5Visual Studio界面介绍2.png)

### UnrealBuildTool（UBT）

UE的自定义工具，来编译UE4的逐个模块并处理依赖等。我们编写的Target.cs，Build.cs都是为这个工具服务的。

表示项目环境依赖信息，用到哪些模块（“Core”,“CoreUObject”,“Engine”,“InputCore”，这些是初始内置模块）

```C++
// Copyright Epic Games, Inc. All Rights Reserved.

using UnrealBuildTool;

public class Demo : ModuleRules
{
	public Demo(ReadOnlyTargetRules Target) : base(Target)
	{
		PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
	
		PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore" });

		PrivateDependencyModuleNames.AddRange(new string[] {  });

		// Uncomment if you are using Slate UI
		// PrivateDependencyModuleNames.AddRange(new string[] { "Slate", "SlateCore" });
		
		// Uncomment if you are using online features
		// PrivateDependencyModuleNames.Add("OnlineSubsystem");

		// To include OnlineSubsystemSteam, add it to the plugins section in your uproject file with the Enabled attribute set to true
	}
}

```

一般来说，UBT会先调用UHT会先负责解析一遍C++代码，生成相应其他代码。然后开始调用平台特定的编译工具(VisualStudio,LLVM)来编译各个模块。最后启动Editor或者是Game。
![解决方案资源管理器1](C:\UE\UE5C++学习笔记\Picture\UEC++Base_1\解决方案资源管理器1.png)

LearnUE.Build.cs依赖的模块

LearnUE.Target.cs为目标平台设置

### UnrealHeadTool（UHT）

UE的C++代码解析生成工具，我们在代码里写的那些宏UCLASS等和#include "*.generated.h"都为UHT提供了信息来生成相应的C++反射代码。

用于收集头文件生成反射信息 头文件发生变化时 UE引擎会调用UHT生成新的反射信息

UHT在工作的时候需要你提供正确的前缀，所以虽然说是约定，但你也得必须遵守。

![游戏模式头文件界面介绍](C:\UE\UE5C++学习笔记\Picture\UEC++Base_1\游戏模式头文件界面介绍.png)

```c++
#pragma once//表示头文件只会编译一次

#include "CoreMinimal.h"//包含基本的头文件 还有数据类型、宏、函数、接口等头文件
#include "GameFramework/GameMode.h"//包含游戏模式类头文件
#include "DemoGameMode.generated.h"//生成的反射数据头文件
```



### UEC++主要的头文件介绍

UCLASS()告知虚幻引擎生成类的反射数据 类必须派生自UObject

GENERATED_BODY()表示我们不直接使用父类的构造函数 如果我们要在我们自定义的类中做一些初始化操作 需要我们自己在.h头文件中声明构造函数 然后在.cpp文件中 实现该构造函数 默认它之后的成员函数 成员变量是private

GENERATED_UCLASS_BODY()表示我们使用父类的构造 如果我们在自定义的类中做一些初始化操作 可以直接在.cpp文件中实现构造函数 而不需要在.h头文件中去声明 这个宏会自动生成带有特定参数的构造函数 它之后的成员是public

UPROPERTY()叫做属性声明宏 虚幻C++在标准基础之上实现了一套反射系统（Reflection System） 反射系统负责垃圾回收 引用更新 编译器集成等一系列高级且有用的功能 而UPROPERTY的作用就是声明该属性在反射系统的行为

UFUNCTION()函数声明宏 反射系统可识别的C++函数

USTRUCT()结构体声明宏 反射系统可识别的C++结构体UENUM() 枚举声明宏 反射系统可识别的C++枚举

------



## UEC++基础(二)基础知识

![02_世界划分编辑](C:\UE\UE5C++学习笔记\Picture\UEC++Base_2\02_世界划分编辑.png)

![01_GameMode构架关系](C:\UE\UE5C++学习笔记\Picture\UEC++Base_2\01_GameMode构架关系.png)

### AGameMode

AGameMode是AGameModeBase 的子类，拥有一些额外的功能支持多人游戏和旧行为
所有新建项目默认使用AGameModeBase
如果需要此额外行为，可切换到从AGameMode进行继承. 如从 AGameMode 进行继承，也可从 AGameState 继承游戏状态

![03_GameMode头文件和CPP文件](C:\UE\UE5C++学习笔记\Picture\UEC++Base_2\03_GameMode头文件和CPP文件.png)

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

![04_WorldSettings设置GameMode](C:\UE\UE5C++学习笔记\Picture\UEC++Base_2\04_WorldSettings设置GameMode.png)

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

![05_打印日志](C:\UE\UE5C++学习笔记\Picture\UEC++Base_2\05_打印日志.png)

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
	//游戏开始时执行
	virtual void BeginPlay() override;
	//游戏开始时每一帧都调用
	virtual void Tick(float DeltaTime) override;
	//游戏结束时或者切换关卡时执行
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

