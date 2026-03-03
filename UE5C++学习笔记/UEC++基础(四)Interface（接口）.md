# UEC++Interface

接口类有助于确保一组不相关的类实现一组通用函数，在某些游戏功能可能被大量复杂而不同的类共享的情况下，可以使用接口来让任意类共用这个功能。

## 1.1接口声明

### 1.1.1在C++实现接口

在声明接口的时候所有的方法都需要在public下声明，一遍对外部的类可见

YCNInterface.h

```c++
#pragma once

#include "CoreMinimal.h"
#include "UObject/Interface.h"
#include "YCNInterface.generated.h"

UINTERFACE(MinimalAPI)
class UYCNInterface : public UInterface
{
	GENERATED_BODY()
};

class DEMO_API IYCNInterface
{
	GENERATED_BODY()

public:

};
```

### 1.1.2仅在C++中使用的接口函数

```c++
public:
//虚函数
virtual void PureFunction() {};
UFUNCTION(BlueprintCallable, BlueprintNativeEvent)
void BlueprintFunction();
```

只在C++使用的虚函数需要在.h或者.cpp文件中定义这个函数。

如果这个函数想在蓝图中使用或者覆写那就需要给函数添加UFUNCTION宏再加上说明符，且该函数就不需要再声明virtual了。

BlueprintCallable：蓝图可调用。

BlueprintNativeEvent：蓝图原生事件，可以在蓝图中覆写。

### 1.1.3继承该接口

在接口中声明好函数后需要让使用接口的类继承该接口。

```c++
class DEMO_API AYCNActor : public AActor, public IYCNInterface
```

当在一个Actor类中继承接口后，可以创建并定义这个函数。

YCNActor.h

```c++
public:
virtual void PureFunction() override;
virtual void BlueprintFunction_Implementation() override;
```

在接口中添加了说明符的函数就不需要在使用该方法的Actor中再添加相对应的说明符了，但是需要函数的末端添加_Implementation。

注意：该函数可以在蓝图中覆写。

YCNActor.cpp

```c++
void YCNCharacter::PureFunction()
{
	//函数内容...
}
void YCNCharacter::BlueprintFunction_Implementation()
{
	//函数内容...
}
```

### 1.1.4蓝图可实现类

如果你想要蓝图能够实现此接口，则必须使用"Blueprintable"元数据说明符。蓝图类要覆盖的每个接口函数都必须是"BlueprintNativeEvent"或"BlueprintImplementableEvent"。标记为"BlueprintCallable"的函数仍然可以被调用，但不能被覆盖。你将无法从蓝图访问所有其他函数。

## 1.2确定类是否继承了接口

为了与实现接口的C++和蓝图类兼容，可以使用以下任意函数：

```c++
bool bIsImplemented = YCNObject->GetClass()->ImplementsInterface(UYCNInterface::StaticClass()); 
//如果YCNObject继承了UYCNInterface，则bIsImplemented将为true。
bIsImplemented = YCNObject->Implements<UYCNInterface>(); 
//如果YCNObject继承了UYCNInterface，则bIsImplemented将为true。
IYCNInterface* YCNInterfaceObject = Cast<IYCNInterface>(YCNObject); 
//如果OriginalObject实现了UYCNInterface，则YCNObject将为非空。
 
```

## 1.3转换到其他虚幻类型

虚幻引擎的转换系统支持从一个接口转换到另一个接口，或者在适当的情况下，从一个接口转换到一个虚幻类型。

```c++
IReactToTriggerInterface* ReactingObject = Cast<IReactToTriggerInterface>(OriginalObject);
//如果接口被实现，则ReactingObject将为非空。 	
ISomeOtherInterface* DifferentInterface = Cast<ISomeOtherInterface>(ReactingObject);
//如果ReactingObject为非空而且还实现了ISomeOtherInterface，则DifferentInterface将为非空。
AActor* Actor = Cast<AActor>(ReactingObject);
//如果ReactingObject为非空且OriginalObject为AActor或AActor派生的类，则Actor将为非空。
```

