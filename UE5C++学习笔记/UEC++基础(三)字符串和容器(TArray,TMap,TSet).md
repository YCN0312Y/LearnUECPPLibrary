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

