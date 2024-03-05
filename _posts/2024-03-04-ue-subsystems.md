---
title: "Unreal Engine - Programming Subsystems"
date: 2024-03-04T16:00:00+02:00
author: Jan Mesarč
categories:
  - unrealengine
tags:
  - ue
  - subsystems
  - unrealengine
  - gamedev
toc: true
---

If you are a programmer working with Unreal Engine, you might have heard of the concept of subsystems. Subsystems are a powerful feature that allows you to create custom classes that can be accessed from anywhere in your project, without having to modify or inherit from the engine classes. In this blog post, I will explain what subsystems are, how they work, and how you can use them to enhance your game development workflow. Don't forget to check out documentation [Programming Subsystems](https://docs.unrealengine.com/5.3/en-US/programming-subsystems-in-unreal-engine/).

![Divert all power to subsystems!](https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExanJnMXF4M2dncDQxNzNjcWhzcjZpemd3cm9weHJlNHkycXRyb2szdCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/l0HlvFC13t3H3r1YY/giphy.gif)

## What are subsystems?

Subsystems are a cool way to create your own custom functionality in Unreal Engine. They are classes that implement the `USubsystem` interface, and the engine takes care of creating and destroying them for you. You can specify the scope of each subsystem, such as the game instance, the world, the local player, or the application. Subsystems can expose properties and functions to Blueprint, making them ideal for creating game-specific logic and systems that can be easily accessed and modified by designers and scripters. Subsystems are great for enhance your game with more features, options and customizable.

Here are examples of subsystems that you can use or create in your own projects:

- `UGameInstanceSubsystem`: This subsystem is perfect for storing and managing data that is relevant to the entire game session, such as player progress, save data, options, etc. It is created for each game instance and persists across level transitions, so you don't have to worry about losing your data when switching levels.
- `UWorldSubsystem`: This subsystem is ideal for implementing logic and systems that are specific to a certain level, such as environmental effects, level streaming, AI spawning, etc. It is created for each world (or level) and is destroyed when the world is unloaded, so you can keep your code clean and organized for each level.
- `ULocalPlayerSubsystem`: This subsystem is great for handling input, UI, camera, sound, etc. for each local player. It is created for each local player controller and is destroyed when the player controller is destroyed, so you can customize your gameplay experience for each player.
- `UEngineSubsystem`: This subsystem is beneficial for implementing logic and systems that are tied to the Engine’s lifecycle, such as game mechanics, physics simulations, and rendering processes. It is an instance subsystem that is created and destroyed with the Engine, so its accessibility in your code is bound to the Engine’s existence.

## What are benefits of subsystem?

Benefits:

- Time-Saving: Easy to use extension, automatically manage lifetimes, allowing developers to focus on functionality
- Avoiding Overrides: help avoid overriding engine classes, instead of modifying existing classes, you can create subsystem to extend functionality
- Blueprint exposure: Subsystems offer Blueprint exposure ouf of the box
- Modularity and consistency: provide a consistent interface for non-actor objects, similar to components, making code clearer and more maintainable

Cons:

- Learning Curve: Understanding subsystems and their proper usage may require some initial effort. Developers need to grasp the concept and decide when to employ them effectively.
- Complexity: While subsystems simplify certain tasks, they can add complexity if not used judiciously. Overusing subsystems might lead to unnecessary layers of abstraction.
- Limited Use Cases: Subsystems are most beneficial for non-actor objects. For certain scenarios, traditional approaches (such as direct class modification) may still be more straightforward.

In summary, Unreal Engine subsystems enhance code organization, maintainability, and extensibility, but developers should weigh their benefits against potential complexities.


## Subsystems lifetime overview

Let’s explore the lifecycle of different subsystem types and provide concise examples for each:

1. `UGameInstanceSubsystem`:
    - Associated with the game instance.
    - Initialization:
        - Called when the game instance is created.
        - Set up global state or resources.
    - Deinitialization:
        - Called when the game instance is destroyed.
        - Clean up resources.
    - These subsystems can be accessed through UGameInstance

    ```cpp
    UGameInstance* GameInstance = ...;
    UMyGameSubsystem* MySubsystem = GameInstance->GetSubsystem<UMyGameSubsystem>();
    ```

2. `UWorldSubsystem`:
    - Associated with the world.
    - Initialization:
      - Called when a new level is loaded.
      - Set up level-specific resources.
    - Deinitialization:
      - Called when a level is unloaded.
      - Clean up level-specific resources.

3. `ULocalPlayerSubsystem`:
     Tied to local players.
     Initialization:
     - Called when a local player joins.
     - Set up player-specific resources.
     Deinitialization:
     - Called when a local player leaves.
     - Clean up player-specific resources.
     These subsystems can be accessed through ULocalPlayer:

    ```cpp
    LocalPlayer* LocalPlayer = ...;
    MyPlayerSubsystem* MySubsystem = LocalPlayer->GetSubsystem<UMyPlayerSubsystem>();
    ```

4. `UEngineSubsystem`:

    - Tied to the Engine’s lifecycle.
    - Initialization:
      - Called during engine startup.
      - Set up global resources.
    - Deinitialization:
      - Called during engine shutdown.
      - Clean up global resources.
    - These subsystems are accessed through GEngine:

    ```cpp
    UMyEngineSubsystem* MySubsystem = GEngine->GetEngineSubsystem<UMyEngineSubsystem>();
    ```

## How do subsystems work?

To create a subsystem, you need to create a new C++ class that inherits from `USubsystem` or any of its derived classes (e.g., `UGameInstanceSubsystem`, `UWorldSubsystem`, etc.). For example, to create a game instance subsystem called `UMyGameInstanceSubsystem`, you would write something like this:

```cpp
#include "Subsystems/GameInstanceSubsystem.h"
#include "MyGameInstanceSubsystem.generated.h"

UCLASS()
class UMyGameInstanceSubsystem : public UGameInstanceSubsystem
{
    GENERATED_BODY()

    // Declare your properties and functions here
public:
    UMyGameInstanceSubsystem();

    // Override the virtual functions of USubsystem
    virtual bool ShouldCreateSubsystem(UObject* Outer) const override;
    virtual void Initialize(FSubsystemCollectionBase& Collection) override;
    virtual void Deinitialize() override;
};
```

The `ShouldCreateSubsystem` function determines whether the subsystem should be created for the given outer. The default implementation returns true, but you can override it to add some custom logic. The `Initialize` and `Deinitialize` functions are called when the subsystem is created and destroyed, respectively. You can use them to perform any initialization or cleanup tasks for your subsystem.

To access your subsystem from anywhere in your code, you can use the function `GetSubsystem<T>` of the corresponding outer class. For example, to get a reference to your game instance subsystem from a game mode class, you would write something like this:

```cpp
UGameInstanceSubsystem* MySubsystem = GetGameInstance()->GetSubsystem<UGameInstanceSubsystem>();
```

In this line of code, `GetGameInstance()` is a function that returns a pointer to the current game instance. `GetSubsystem<UGameInstanceSubsystem>()` is a template function that returns a pointer to the subsystem of the specified type. If the subsystem does not exist, it returns `nullptr`.

This way, you can access your subsystems from anywhere in your code, as long as you have access to the corresponding outer object (game instance, world, local player, or application). This makes subsystems a powerful tool for organizing your game code and making it more modular and reusable. Remember to always check if the returned pointer is not nullptr before using it.

### UGameInstanceSubsystem

First example of a custom subsystem is `UGameInstanceSubsystem`. Let's create subsystem called `UAnalyticsSubsystem` that inherits from `UGameInstanceSubsystem`. Our focus will be on handling game analytics and tracking important metrics.

First, let’s set up the basic structure for our `UAnalyticsSubsystem`. Create the following files:

`AnalyticsSubsystem.h`

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Subsystems/GameInstanceSubsystem.h"
#include "AnalyticsSubsystem.generated.h"

UCLASS()
class SAVECRAFT_API UAnalyticsSubsystem : public UGameInstanceSubsystem
{
    GENERATED_BODY()

public:
    // Constructor
    UAnalyticsSubsystem();

    // Initialize the subsystem
    virtual bool ShouldCreateSubsystem(UObject* Outer) const override;
    virtual void Initialize(FSubsystemCollectionBase& Collection) override;
    virtual void Deinitialize() override;

    // Custom methods for tracking events
    void TrackEvent(FString EventName, TMap<FString, FString> EventProperties);

private:
    // Internal data structures for analytics
    // You can use third-party analytics services or custom implementations here
};

```

`AnalyticsSubsystem.cpp`

```cpp
#include "AnalyticsSubsystem.h"

UAnalyticsSubsystem::UAnalyticsSubsystem()
{
    // Initialize any internal data structures here
}

bool UAnalyticsSubsystem::ShouldCreateSubsystem(UObject* Outer)
{
    return true;
}

void UAnalyticsSubsystem::Initialize(FSubsystemCollectionBase& Collection)
{
    Super::Initialize(Collection);
    // Set up any global state related to analytics
}

void UAnalyticsSubsystem::Initialize(FSubsystemCollectionBase& Collection)
{
    Super::Initialize(Collection);
    // Set up any global state related to analytics
}

void UAnalyticsSubsystem::Deinitialize()
{
    // Clean up resources
    Super::Deinitialize();
}

void UAnalyticsSubsystem::TrackEvent(FString EventName, TMap<FString, FString> EventProperties)
{
    // Implement your analytics tracking logic here
    // Send event data to your analytics service or handle it locally
}
```

Usage example of `UAnalyticsSubsystem`:

- When a player completes a level, call `TrackEvent("LevelCompleted", EventProperties)` to track the event.
- Customize the `EventProperties` map with relevant data (e.g., level name, score, time taken).

### UWorldSubsystem example

Let’s create an example of a custom subsystem called `UEnemySpawnerSubsystem` that inherits from `UWorldSubsystem`. Our goal is to handle the spawning and despawning of enemies in your game.

Possible implementation of header file:

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Subsystems/WorldSubsystem.h"
#include "EnemySpawnerSubsystem.generated.h"

UCLASS()
class SAVECRAFT_API UEnemySpawnerSubsystem : public UWorldSubsystem
{
    GENERATED_BODY()

public:
    // Constructor
    UEnemySpawnerSubsystem();

    // Initialize the subsystem
    virtual void Initialize(FSubsystemCollectionBase& Collection) override;
    virtual void Deinitialize() override;

    // Custom methods for enemy spawning and despawning
    void SpawnEnemy(TSubclassOf<AActor> EnemyClass, FVector SpawnLocation);
    void DespawnEnemy(AActor* EnemyActor);

private:
    // Internal data structures for enemy management
    UPROPERTY()
    TArray<AActor*> ActiveEnemies;
};
```

Possible implementation of cpp file:

```cpp
#include "EnemySpawnerSubsystem.h"

UEnemySpawnerSubsystem::UEnemySpawnerSubsystem()
{
    // Initialize any internal data structures here
}

void UEnemySpawnerSubsystem::Initialize(FSubsystemCollectionBase& Collection)
{
    Super::Initialize(Collection);
    // Set up any global state related to enemy spawning
}

void UEnemySpawnerSubsystem::Deinitialize()
{
    // Clean up resources
    Super::Deinitialize();
}

void UEnemySpawnerSubsystem::SpawnEnemy(TSubclassOf<AActor> EnemyClass, FVector SpawnLocation)
{
    if (EnemyClass)
    {
        // Spawn an enemy actor at the specified location
        AActor* NewEnemy = GetWorld()->SpawnActor(EnemyClass, &SpawnLocation);
        if (NewEnemy)
        {
            ActiveEnemies.Add(NewEnemy);
        }
    }
}

void UEnemySpawnerSubsystem::DespawnEnemy(AActor* EnemyActor)
{
    if (IsValid(EnemyActor))
    {
        // Despawn an enemy actor
        EnemyActor->Destroy();
        ActiveEnemies.Remove(EnemyActor);
    }
}
```

Now you can use `UEnemySpawnerSubsystem` in your game code. For instance:

- To spawn an enemy, call SpawnEnemy(EnemyClass, SpawnLocation).
- To despawn an enemy, call DespawnEnemy(EnemyActor).

Another example can be subsytem that manages the health and damage of all characters in the game. We can create a new C++ class called `UHealthSubsystem` that inherits from `UWorldSubsystem` and add some logic to handle health changes and damage events. Here is a possible implementation of the header file:

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Subsystems/WorldSubsystem.h"
#include "HealthSubsystem.generated.h"

UCLASS()
class SAVECRAFT_API UHealthSubsystem : public UWorldSubsystem
{
 GENERATED_BODY()

public:

    // Constructor
    UHealthSubsystem();

    // Initialize the subsystem
    virtual bool ShouldCreateSubsystem(UObject* Outer) const override;
    virtual void Initialize(FSubsystemCollectionBase& Collection) override;
    virtual void Deinitialize() override;

    // Get the health of a character
    float GetHealth(ACharacter* Character) const;

    // Set the health of a character
    void SetHealth(ACharacter* Character, float NewHealth);

    // Apply damage to a character
    void ApplyDamage(ACharacter* Character, float Damage, AController* InstigatedBy, AActor* DamageCauser);

    // Delegate for health changed event
    DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(FOnHealthChanged, ACharacter*, Character, float, OldHealth, float, NewHealth, AController*, InstigatedBy);
    FOnHealthChanged OnHealthChanged;

private:
    // A map of character to health
    UPROPERTY()
    TMap<ACharacter*, float> HealthMap;
};
```

Here is a possible implementation of the cpp file:

```cpp
#include "HealthSubsystem.h"
#include "GameFramework/Character.h"

UHealthSubsystem::UHealthSubsystem()
{
    // Initialize any internal data structures here
}

bool UHealthSubsystem::ShouldCreateSubsystem(UObject* Outer) const
{
    return true;
}

void UHealthSubsystem::Initialize(FSubsystemCollectionBase& Collection)
{
    Super::Initialize(Collection);
    // Set up any global health-related state
}

void UHealthSubsystem::Deinitialize()
{
    // Clean up resources
    Super::Deinitialize();
}

void UHealthSubsystem::ApplyDamage(ACharacter* Character, float Damage, AController* InstigatedBy, AActor* DamageCauser)
{
    if (IsValid(Character))
    {
        // Apply damage logic (e.g., reduce health)
        HealthMap.FindOrAdd(Character) -= Damage;
    }
}

void UHealthSubsystem::SetHealth(ACharacter* Character, float NewHealth)
{
    if (IsValid(Character))
    {
        // Set new health for selected character
        HealthMap.FindOrAdd(Character) = NewHealth;
    }
}

float UHealthSubsystem::GetHealth(ACharacter* Character) const
{
    return HealthMap.Contains(Character) ? HealthMap[Character] : 0.0f;
}
```

Now you can use `UHealthSubsystem` in your game code. For instance, when an enemy takes damage, call `ApplyDamage(EnemyActor, DamageAmount)` to update its health. To retrieve an actor’s current health, use `GetHealth(EnemyActor)`.

### ULocalPlayerSubsystem

One of the benefits of using subsystems in Unreal Engine is that they allow you to create modular and reusable functionality that can be attached to different types of actors. 

Let’s create an example of a custom subsystem called `UPlayerUIHandlerSubsystem` that inherits from `ULocalPlayerSubsystem`. Our focus will be on handling UI-related logic for local players.

First, let’s set up local player. We need to create new class for local player, that derived from`ULocalPlayer`. For this purpose I create class `UMyLocalPlayer` that only implements two functions: `ShowHUD()` and `HideHUD()`. 

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Engine/LocalPlayer.h"
#include "MyLocalPlayer.generated.h"

/**
 * UMyLocalPlayer - Custom Local Player Class
 * This class extends ULocalPlayer and provides additional methods for managing the HUD.
 */
UCLASS()
class SAVECRAFT_API UMyLocalPlayer : public ULocalPlayer
{
	GENERATED_BODY()

public:

    // Show the HUD elements
	void ShowHUD();
    
    // Hide the HUD elements
	void HideHUD();	
};
```

Now, let’s set up the basic structure for our `UPlayerUIHandlerSubsystem`. Create the following files:

`PlayerUIHandlerSubsystem.h`

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Subsystems/LocalPlayerSubsystem.h"
#include "LocalPlayerSubsystem.generated.h"

UCLASS()
class SAVECRAFT_API UPlayerUIHandlerSubsystem : public ULocalPlayerSubsystem
{
    GENERATED_BODY()

public:
    // Constructor
    UPlayerUIHandlerSubsystem();

    // Initialize the subsystem
    virtual void Initialize(FSubsystemCollectionBase& Collection) override;
    virtual void Deinitialize() override;

    // Show the HUD for a specific player
    UFUNCTION(BlueprintCallable, Category = "UI")
    void ShowHUDForPlayer();
    
    // Hide the HUD for a specific player
    UFUNCTION(BlueprintCallable, Category = "UI")
    void HideHUDForPlayer();
}
```

`PlayerUIHandlerSubsystem.cpp`

```cpp
#include "PlayerUIHandlerSubsystem.h"

UPlayerUIHandlerSubsystem::UPlayerUIHandlerSubsystem()
{
    // Initialize any internal data structures here
}

void UPlayerUIHandlerSubsystem::Initialize(FSubsystemCollectionBase& Collection)
{
    Super::Initialize(Collection);
    // Set up any global state related to UI
}

void UPlayerUIHandlerSubsystem::Deinitialize()
{
    // Clean up resources
    Super::Deinitialize();
}

void UPlayerUIHandlerSubsystem::ShowHUDForPlayer()
{
    if (IsValid(GetLocalPlayer<UMyLocalPlayer>()))
    {        
        // Show the HUD for the specified player
        GetLocalPlayer<UMyLocalPlayer>()->ShowHUD();
    }
}

void UPlayerUIHandlerSubsystem::HideHUDForPlayer()
{
    if (IsValid(GetLocalPlayer()))
    {
        // Hide the HUD for the specified player
        GetLocalPlayer<UMyLocalPlayer>()->HideHUD();
    }
}

```

Now you can use `UPlayerUIHandlerSubsystem` in your game code. For instance:

- When a player logs in, call `ShowHUDForPlayer()` to display the HUD.
- When a player logs out, call `HideHUDForPlayer()` to hide the HUD.

### UEngineSubsystem example

This custom `UEngineSubsystem` efficiently tracks and logs the performance of objects with ticking enabled, providing valuable insights into their runtime behavior. By utilizing this subsystem, developers can easily start and stop logging, gaining a comprehensive overview of how long ticking actors have been active in the game world.

`PerformanceLoggerSubsystem.h`

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Subsystems/EngineSubsystem.h"
#include "PerformanceLoggerSubsystem.generated.h"

/**
 * Custom Engine Subsystem for Performance Logging
 */
UCLASS()
class SAVECRAFT_API UPerformanceLoggerSubsystem : public UEngineSubsystem
{
	GENERATED_BODY()

public:

	/** Log ticking actors information */
	UFUNCTION(BlueprintCallable, Category = "Performance Logging")
	void LogTickingActors(UWorld* World);

private:

	/** Log performance information for an object */
	void LogPerformanceInfo(AActor* Object);

    /** Write message to log file */
	void WriteToLog(FString Message);
};
```

`PerformanceLoggerSubsystem.cpp`

```cpp
#include "PerformanceLoggerSubsystem.h"
#include "Engine/World.h"
#include "HAL/PlatformFilemanager.h"
#include "Misc/FileHelper.h"
#include "GameFramework/Actor.h"
#include "EngineUtils.h"

void UPerformanceLoggerSubsystem::LogTickingActors(UWorld* World)
{
    if (!IsValid(World) || !World->IsGameWorld())
    {
        // Only log performance information in the game world
        return;
    }        

    int32 ActorsCount = 0;
    int32 TickingActorsCount = 0;

    // Iterate through all actors in the world
    for (TActorIterator<AActor> It(World); It; ++It)
    {
        AActor* Actor = *It;

        // Check if the actor has tick enabled
        if (Actor->PrimaryActorTick.bCanEverTick && Actor->IsActorTickEnabled())
        {
            // Log performance information for the ticking actor
            LogPerformanceInfo(Actor);
            TickingActorsCount++;
        }

        ActorsCount++;
    }

    // Write to log total count of actors and count of ticking actors
    FString LogMessage = FString::Printf(TEXT("Total actors count %d, Actor ticking count %d\n"), ActorsCount, TickingActorsCount);
    WriteToLog(LogMessage);
}

void UPerformanceLoggerSubsystem::LogPerformanceInfo(AActor* Actor)
{
    // Get the object's name and log some performance information
    FString ActorName = Actor->GetName();
    float TimeSinceCreation = Actor->GetWorld()->GetTimeSeconds() - Actor->CreationTime;

    FString LogMessage = FString::Printf(TEXT("Actor %s has been alive for %.2f seconds.\n"), *ActorName, TimeSinceCreation);
    WriteToLog(LogMessage);
}

void UPerformanceLoggerSubsystem::WriteToLog(FString LogMessage)
{
    // You can customize how you log this information (e.g., write to a file, print to console, etc.)
    // For this example, we'll write to a file.
    FFileHelper::SaveStringToFile(
        LogMessage,
        *(FPaths::ProjectLogDir() / TEXT("PerformanceLog.txt")),
        FFileHelper::EEncodingOptions::AutoDetect,
        &IFileManager::Get(),
        EFileWrite::FILEWRITE_Append);
}
```

This code is a custom Unreal Engine subsystem for performance logging, specifically for logging information about ticking actors in a game world.

The header file `PerformanceLoggerSubsystem.h` declares a class `UPerformanceLoggerSubsystem` that inherits from `UEngineSubsystem`. It has a public method `LogTickingActors(UWorld* World)` for logging ticking actors information, and two private methods `LogPerformanceInfo(AActor* Object)` and `WriteToLog(FString Message)` for logging performance information for an object and writing a message to a log file, respectively.

The source file `PerformanceLoggerSubsystem.cpp` implements these methods. `LogTickingActors(UWorld* World)` iterates through all actors in the game world, checks if the actor has tick enabled, and if so, logs performance information for the ticking actor and increments the ticking actors count. It then writes to the log the total count of actors and the count of ticking actors. `LogPerformanceInfo(AActor* Actor)` gets the actor’s name and logs some performance information such as the time since its creation. `WriteToLog(FString LogMessage)` writes a log message to a file in the project’s log directory.

This subsystem can be useful for monitoring and optimizing the performance of a game, especially in terms of the number and behavior of ticking actors. Ticking actors are actors that have their Tick function called every frame, and they can have a significant impact on game performance. By logging this information, developers can gain insights into how these actors are affecting the game’s performance and make necessary adjustments. This can be particularly useful in complex games with a large number of actors and complex behaviors.

For test perfomance logger subsystem, I called that on begin play of player character with timer for update every 5 seconds.
Example of call:

```cpp
UPerformanceLoggerSubsystem* PerfSubsystem = GEngine->GetEngineSubsystem<UPerformanceLoggerSubsystem>();
if (IsValid(PerfSubsystem))
{
    FTimerHandle Handle;
    GetWorld()->GetTimerManager().SetTimer(Handle, FTimerDelegate::CreateLambda([&] { PerfSubsystem->LogTickingActors(GetWorld()); }), 5.0f, true);
}
```

Example of output:

```cpp
Actor VolumetricCloud_1 has been alive for 15.01 seconds.
Actor SkyAtmosphere_1 has been alive for 15.01 seconds.
Actor BP_Door_C_UAID_E89C25424BAA1CDC01_1831171407 has been alive for 15.01 seconds.
Actor GameplayDebuggerPlayerManager_0 has been alive for 15.01 seconds.
Actor AIController_0 has been alive for 15.01 seconds.
Actor AIController_1 has been alive for 15.01 seconds.
Actor BP_ThirdPersonCharacter_C_UAID_E89C25424BAA0CDD01_1546070648 has been alive for 15.01 seconds.
Actor BP_ThirdPersonCharacter_C_UAID_E89C25424BAA0CDD01_1540088647 has been alive for 15.01 seconds.
Actor AbstractNavData-Default has been alive for 15.01 seconds.
Actor BP_ThirdPersonController_C_0 has been alive for 15.01 seconds.
Actor HUD_0 has been alive for 15.01 seconds.
Actor BP_ThirdPersonCharacter_C_0 has been alive for 15.01 seconds.
Total actors count 73, Actor ticking count 12
```

## Conclusion

![Sweet Conclusion](https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExZDkxeWpnY2V0Z3lwbXpsbzhmZmhwZ2F1NGpydjV5Zm40NnVuZjllMCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/06gVN8Aj6HGAqezXmq/giphy-downsized.gif)

In this article, we explored the fascinating world of Unreal Engine subsystems—powerful tools that enhance modularity and organization in your game development process. Let’s recap what we’ve learned:

1. What Are Subsystems?
    - Subsystems are specialized components that encapsulate specific functionalities within your game.
    - They provide a clean separation of concerns, making your codebase more maintainable and extensible.

2. What are benefits of subsystem?
    - Unreal Engine subsystems offer streamlined development, modularity, and consistent interfaces for non-actor objects.
    - Enhance code organization and extensibility, developers should be mindful of the learning curve and potential complexities when deciding to use them.
    - Weighing their benefits against traditional approaches is essential for effective implementation in game development projects.

3. Lifecycle of Subsystems:
    - UGameInstanceSubsystem: Associated with the game instance, it shares its lifetime. Use it for global game state management.
    - UWorldSubsystem: Tied to the world, it initializes with each level and handles level-specific logic (e.g., enemy spawning).
    - ULocalPlayerSubsystem: Associated with local players, it manages player-specific resources and state.
    - UEngineSubsystem: Tied to engine lifetime, it initializes during engine startup and handles global functionalities.

4. Creating Subsystems:
    - Define your custom subsystem class inheriting from the appropriate base class (e.g., UWorldSubsystem).
    - Implement the Initialize and Deinitialize methods for setup and cleanup.
    - Add custom methods to handle specific functionality (e.g., health management, UI, analytics).

5. Examples:
    - AnalyticsSubsystem: Tracks game metrics and events.
    - HealthSubsystem: Handles health changes and damage events for game entities.
    - EnemySpawnerSubsystem: Manages enemy spawning and despawning.
    - PlayerUIHandlerSubsystem: Manages UI-related logic for local player (e.g., UI visibility).
    - PerformanceLoggerSubsystem: This example demonstrates a basic usage of UEngineSubsystem for performance logging. It logs information about objects with enabled ticking and saves the log to a file

Remember that subsystems empower you to create modular, efficient, and well-organized game systems. Whether you’re building a small indie project or a massive AAA title, mastering subsystems will elevate your Unreal Engine experience. But as with any tool, it’s the community of users that truly brings it to life.

So, now it’s your turn. Are you using Unreal Engine and its subsystems in your projects? Have you encountered any challenges or discovered any innovative solutions? We want to hear about your experiences.

Join the conversation and share your insights with us. Tweet us at [@flyingratstudio](https://twitter.com/flyingratstudio) with your thoughts, screenshots, or even videos of your projects.

Happy coding, and may your games thrive!
