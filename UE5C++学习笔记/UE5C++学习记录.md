# **UEC++学习记录**

## 引擎编译器版本

Unreal Engine版本：Unreal Engine5.4.4

Visual Studio版本：Visual Studio 2022

## UEC++基础(一)入门介绍

### 创建项目

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

## UEC++基础(三)字符串和容器(TArray,TMap,TSet)

### 字符串之间的转换

FString：适合常规字符串操作

FName：适合标识符、资源名称等无需修改的场景

FText：专注于本地化与用户界面文本显示

```C++
void ADemoGameMode::BeginPlay()
{
	Super::BeginPlay();
    
    //FString
	FString MyStr = TEXT("Hello Unreal Engine !");
	//FString->FName
	FName String_Name = FName(*MyStr);
	//FString->FText
	FText String_Text = FText::FromString(MyStr);

	//FName
	FName MyName = TEXT("Hello Unreal Engine !");
	//FName->FString
	FString Name_String = MyName.ToString();
	//FName->FText
	FText Text_Name = FText::FromName(MyName);

	//FText
	FText MyText = FText::FromString(TEXT("Hello Unreal Engine !"));
	//FText->FString
	FString Text_String = MyText.ToString();
	//FText->FName(先转化成FString 再转化成FName)
	FName Text_Name = FName(MyText.ToString());

}
```

### 其他类型与FString之间的转换

```C++
void ADemoGameMode::BeginPlay()
{
	Super::BeginPlay();
    
    //-----其他类型转换字符串-----

	//float->FString
	float MyFloat = 3.14f;
	FString Float_String = FString::SanitizeFloat(MyFloat);
    
	//int32->FString
	int32 MyInt = 100;
	FString Int_String = FString::FromInt(MyInt);
    
	//bool->FString
	bool MyBool = true;
	FString Bool_String = MyBool ? (TEXT("true")) : (TEXT("flase"));
    
	//FVector->FString
	FVector MyVector = FVector(10, 10, 10);
	FString Vector_String = MyVector.ToString();
    
	//FVector2D->FString
	FVector2D MyVector2D = FVector2D(20, 20);
	FString Vector2D_String = MyVector2D.ToString();
    
	//FRotator->FString
	FRotator MyRotation = FRotator(45, 45, 45);
	FString Rotation_String = MyRotation.ToString();
    
	//FLinearColor->FString
	FLinearColor MyColor = FColor::Blue;
	FString Color_String = MyColor.ToString();
    
	//UObject->FString
	AActor* MyActor = this;
	FString Actor_String = (MyActor != nullptr) ? MyActor->GetName() : FString(TEXT("None"));

	//FString->float
	float String_Float = FCString::Atof(*Float_String);

	//FString->int32
	int32 String_Int = FCString::Atoi(*Int_String);

	//FString->bool
	bool String_Bool = Bool_String.ToBool();

}
```

### TArray

TArray是虚幻C++中的动态数组 

特点：速度快 内存消耗小 安全性高. 并且TArray所有元素均完全为相同类型 不能进行不同元素类型的混合

YCNTArray.h

```c++
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "YCNTArray.generated.h"

UCLASS()
class DEMO_API AYCNTArray : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	AYCNTArray();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

public:

	//声明一个Fruit
	TArray<FString>Fruit;
	
};
```

YCNTArray.cpp

```c++
#include "Container/YCNTArray.h"

// Sets default values
AYCNTArray::AYCNTArray()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

}

// Called when the game starts or when spawned
void AYCNTArray::BeginPlay()
{
	Super::BeginPlay();

	//添加
    Fruit.Add("Apple");//添加一个元素
	Fruit.Init("Orange", 5);//添加数量元素
	Fruit.Emplace("Banana");//添加一个元素
	Fruit.AddUnique("Watermelon");//添加元素 数组中有该元素则添加失败 数组中无该元素则添加成功
	Fruit.SetNum(10);//添加索引数量
	Fruit.Insert("Pear", 9);//在对应的索引位置添加元素("元素",索引)

	TArray<FString> Name;//声明一个新的数组用于
	Name.Add("ZhangSan");

	Fruit.Append(Name);//追加

	//排序
	Fruit.Sort();//按元素的类型排序(如字母：元素首字母按照26个英文字母排序)

	Fruit.Sort([](const FString& A, const FString& B) {
		//按照数组长度排序
		return A.Len() < B.Len();

		});

	Fruit.HeapSort();//按元素的类型排序(如字母：元素首字母按照26个英文字母排序)

	Fruit.HeapSort([](const FString& A, const FString& B) {
		//按照数组长度排序
		return A.Len() < B.Len();

		});

	Fruit.StableSort();//按字母表排序出现字母大小写情况根据先后顺序排

	Fruit.StableSort([](const FString& A, const FString& B) {
		//按照数组长度排序
		return A.Len() < B.Len();

		});

	bool bValid1 = Fruit.IsValidIndex(3);//查询该索引是否有效

	bool bValid2 = Fruit.IsValidIndex(6);

	bool bValid3 = Fruit.IsValidIndex(9);

	bool bValid4 = Fruit.IsValidIndex(15);

	FString NewSte = Fruit[5].ToUpper();//将所有元素内容变为大写



	//查询
	int32 FruitFind = Fruit.Find(TEXT("Orange"));//从前往后查找 所查的元素
	int32 FruitFindLast = Fruit.FindLast(TEXT("Watermelon"));//从后往前查找 所查的元素
	FString FruitLast = Fruit.Last(6);//从后往前查找 该索引的元素
	FString FruitTop = Fruit.Top();//查找最后一位索引的元素
	bool bFruitContains = Fruit.Contains(TEXT("Apple"));//查找所输入的元素是否在数组中包含 有则返回true 没有则返回false

	int32 PlayerHealth = 0;

	//查询对应的数据类型
	//遍历&PlayerHealth
	bool bLen5 = Fruit.ContainsByPredicate([&PlayerHealth](const FString& Str) {

		++PlayerHealth;

		UE_LOG(LogTemp, Warning, TEXT("bLen5,[%d]"), PlayerHealth);
		//遍历数组中是否有一个字符长度为5的
		return Str.Len() == 5;
		}
	);

	//移除
	Fruit.Remove(TEXT("Banana"));//移除元素
	Fruit.RemoveSingle(TEXT("Orange"));//移除数组中包含该元素的第一个元素(两个元素重复的情况下)
	Fruit.RemoveAt(9);//移除索引位置的元素
	Fruit.Shrink();//移除所有无效元素的索引
	Fruit.RemoveAtSwap(6);//移除在指定索引的元素
	Fruit.RemoveSwap(TEXT("Orange"));//移除数组中所有输入的元素
	Fruit.Empty();//清空数组(也可以在参数列表里写上数量保留几个索引)
	Fruit.Empty(5);//也可以在参数列表里写上数量保留几个索引

	TArray<FString>Fruit2;
	Fruit2 = { TEXT("Apple"), TEXT("Orange"), TEXT("Banana"), TEXT("Watermelon"), TEXT("Pear") };

	Fruit2.Reset();//移除所有元素 保留索引

	//运算符
	TArray<FString>NewFruit { TEXT("Apple"), TEXT("Orange"), TEXT("Banana"), TEXT("Watermelon"), TEXT("Pear") };

	TArray<FString>NewFruit2 { TEXT("A"), TEXT("B"), TEXT("C"), TEXT("D"), TEXT("E") };

	NewFruit = MoveTemp(NewFruit2);//搬运将输入的元素覆盖当前数组的元素

	Exchange(NewFruit, NewFruit2);//交换两个数组的元素

	//原始内存
	TArray<FString>MyFruit;
	MyFruit.AddUninitialized(1);//添加指定数量的索引
	MyFruit.SetNumUninitialized(2);//添加指定数量的索引

	TArray<FString>MyFruit2;
	MyFruit2.SetNumZeroed(3);//设置对应数量的元素为空的索引

	TArray<FString>MyFruit3;
	MyFruit3.Reserve(4);//添加指定空间

	TArray<FString>MyFruit4;
	MyFruit4.Emplace(TEXT("Apple"));//索引为0
	MyFruit4.Emplace(TEXT("Orange"));//索引为1
	MyFruit4.InsertUninitialized(1, 2);//在索引1前面插入两个空索引 (索引,数量)
	MyFruit4.AddZeroed();//添加对应数量的元素为空的索引
	MyFruit4.AddZeroed(5);//添加对应数量的元素为空的索引

}

// Called every frame
void AYCNTArray::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

```

### TMap

TMap：是一种键值对容器 里面的数据都是成对出现的(Key,Value) Value通过Key值来获取 且Key值不能重复 key值唯一

YCNTMap.h

```c++

```

YCNTMap.cpp

```c++

```

### TSet

TSet是一种快速容器类(通常)用于在排序不重要的情况下存储唯一元素。
TSet类似于TMap和TMultiMap 但有一个重要区别：TSet是通过对元素求值的可覆盖函数 使用数据值本身作为键 而不是将数据值与独立的键相关联。
TSet可以非常快速地添加 查找和删除元素(恒定时间)
TSet也是值类型 支持常规复制 赋值和析构函数操作 以及其元素较强的所有权

YCNTSet.h

```c++

```

YCNTSet.cpp

```c++

```

