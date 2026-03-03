# UE网络复制

## 一、UE网络核心概念（必须掌握）

UE是服务器权威（Server-Authority）架构
所有决定游戏结果的数据，都必须由服务器（Server）来修改
客户端（Client）不能直接改（否则会作弊）
客户端能做的：请求
服务器能做的：决定

## 二、角色移动复制（UE自动处理）

CharacterMovementComponent自带

- 客户端预测

- 服务器纠正

- 平滑差值

- 输入同步

  首先要确保

  ```c++
  bReplicates = true;
  ```


## 三、UE网络中的三个重要概念

### （1）Replication（复制）

服务器执行完客户端自动同步

```c++
UPROPERTY(Replicated)
float Health;
```

### （2）RPC（远程过程调用Remote Procedure Call）

|           类型           |  调用  |     执行     |
| :----------------------: | :----: | :----------: |
|     Server（服务器）     | Client |    Server    |
|     Client（客户端）     | Server | 指定的客户端 |
| NetMulticast（网络多播） | Server |  所有客户端  |

### （3）Owner（拥有者）

某个 PlayerController 拥有某个 Pawn/Actor
可以发 Server RPC

## 四、变量复制（最常用）

### 3.1、声明可复制变量

```c++
//.h文件中
UPROPERTY(Replicated)
float Health;
```

当变量被服务器修改时：

OnRep = 客户端收到服务器给的新信息（是同步的）

```c++
//.h文件中
UPROPERTY(ReplicatedUsing = OnRep_SetHealth)
float Health;
UFUNCTION()
void OnRep_SetHealth();
```

```c++
//.cpp
void AMyActor::OnRep_SetHealth()
{
    //需要同步的信息
}
```

### 3.2、注册复制变量

```c++
//.h文件中
virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;
```

```c++
//.cpp文件中
void AMyActor::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);
    
    DOREPLIFETIME(AMyActor, Health);//网络复制
}
```

### 3.3、Replicated的规则

服务器修改->自动同步->所有客户端**（有效）**

客户端修改**（无效）**

## 五、RPC（远程过程调用）

### 4.1、Server RPC （客户端向服务器发送指令）

用于客户端向服务端发送请求，让服务器执行再同步给所有客户端。

```c++
//.h文件中
UFUNCTION(Server, Reliable)
void ServerSetHealth();
```

```c++
//.Cpp文件中
void AMyActor::ServerSetHealth_Implementation()
{
    //服务器执行代码
}
```

### 4.2、Client RPC（服务器指定一个客户端执行）

用于向单个客户端执行，例如本机玩家自己捡到一个物品，就本机玩家自己知道自己捡到这个物品，其他服务器玩家和其他客户端玩家都同步。

```c++
//.h文件中
UFUNCTION(Client, Reliable)
void ClientSetHealth();
```

### 4.3、NetMulticast（服务器调用->所有客户端执行）

网络多播通常用于表现上（动画、特效、音效等）

执行时通知所有客户端

```c++
//.h文件中
UFUNCTION(NetMulticast, Reliable)
void MulticastSetHealth();
```
