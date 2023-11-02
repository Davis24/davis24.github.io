---
layout: post
title: Setting Game Mode and PlayerController with C++
description: In this walkthrough we’re going to be setting up a new GameMode with a new PlayerController using C++. Since this is a simple example the only functionality we’re going to code for is a simple keypress to test....
author: Megan Davis
devto_link: 
github_link: 
tags: unrealengine c++ walkthrough
---

Most tutorials and forum posts focus on how to setup new Game Modes and PlayerControllers using Unreal Engine's blueprinting system. However in this walkthrough we’re going to be setting up a new GameMode with a new PlayerController using C++ only. Since this is a simple example the only functionality we’re going to code for is a simple keypress to test. 


## Walkthrough 

To start off we're going to go ahead and create a new project (or open an existing project you wish to modify). Once your project is created we're going to create two new C++ classes. 

First let's create a new Game Mode, which will have a parent class of GameModeBase. 

![Game Mode Creation](/assets/images/posts/game_mode_through_cpp/game_mode_base_choose.png)

I stuck with default name, but you can change it to anything. 

![Game Mode Creation Naming](/assets/images/posts/game_mode_through_cpp/game_mode_base_name.png)

Next we're going to create a new PlayerController, which will have a parent class of PlayerController. 

![Player Controller Creation](/assets/images/posts/game_mode_through_cpp/player_controller_base_choose.png)

Same thing again, I stuck with the default naming convention.

![Game Mode Creation Naming](/assets/images/posts/game_mode_through_cpp/player_controller_base_name.png)

Now with our two new classes created let's go ahead and make some updates to them.

### MyPlayerController Code

We'll start with `MyPlayerController.h`. We need to account for two things in the new PlayerController, the first being `BeginPlay()`, where we'll have our input rebound, the second being a method `TestInput()`, which is what will get called when the key is pressed. In this instance we're going to have it print out a test message into log.


`MyPlayerController.h`
~~~ cpp
#include "CoreMinimal.h"
#include "GameFramework/PlayerController.h"
#include "MyPlayerController.generated.h"

UCLASS()
class MYDEMOPROJECT_API AMyPlayerController : public APlayerController
{
	GENERATED_BODY()

public:
	virtual void BeginPlay() override;

	void TestInput();	
};
~~~

Inside `MyPlayerController.cpp`, let's go ahead and build out those methods. 

```cpp
#include "MyPlayerController.h"


void AMyPlayerController::BeginPlay() {
	Super::BeginPlay();

	InputComponent->BindAction("TestInput", IE_Pressed, this, &AMyPlayerController::TestInput);
	UE_LOG(LogTemp, Warning, TEXT("Begin Play called."));
}

void AMyPlayerController::TestInput() {
	UE_LOG(LogTemp, Warning, TEXT("KEY PRESSED"));
}
```

Note: `BindAction()` is on its way to being deprecated in 5.0+, personally I didn't want to mess with EnhancedInputs yet so keep that in mind as we'll have to change some project settings later on.

### MyGameModeBase Code

In `MyGameModeBase.h`, we need a new initializer `AMyGameModeBase()`.

```cpp
#include "CoreMinimal.h"
#include "GameFramework/GameModeBase.h"
#include "MyGameModeBase.generated.h"


UCLASS()
class MYDEMOPROJECT_API AMyGameModeBase : public AGameModeBase
{
	GENERATED_BODY()
	
	AMyGameModeBase();
};
```

Then in `MyGameModeBase.cpp` we need to include `MyPlayerController.h` and set the PlayerControllerClass to the class we created in `MyPlayerController.h`.

```cpp
#include "MyGameModeBase.h"
#include "MyPlayerController.h"

AMyGameModeBase::AMyGameModeBase() {

	PlayerControllerClass = AMyPlayerController::StaticClass();
}
```

Save your code and rebuild the project. 

### Fixing Project Settings

First let's go ahead and change our Project's GameMode to `MyGameModeBase`, this can be done the Project - Maps & Modes section, under the Default Modes -> Default GameMode. You'll notice that the Selected GameMode suboptions are greyedout. The only time these are changeable in the settings is if you use a blueprint. 

![Change Project GameMode](/assets/images/posts/game_mode_through_cpp/change_default_game_mode.png)

Now we need to make a quick change to ensure `BindAction` works correctly, since in 5.0+ BindAction is no longer default, and we need to add a keybind.

Inside Project Settings -> Input, at the top there is a section called Bindings and nested in there is ActionMappings. We want to add a new ActionMapping, its name should correspond to the Method we created that should fire on press. And what key we want to bind it to. In this instance I picked E. 

![Project Settings Input](/assets/images/posts/game_mode_through_cpp/project_settings_input.png)

![Project Settings Input Keybind](/assets/images/posts/game_mode_through_cpp/project_settings_input_keybind.png)

Then farther down in the settings, in Default Classes we want to change those back to: PlayerInput and InputComponent. 

![Project Settings Input Keybind](/assets/images/posts/game_mode_through_cpp/project_settings_default_input.png)

Now run the game, when you press E you'll find that inside your OutputLog (found in the dock at the bottom), you'll see two orange messages. 

![Output](/assets/images/posts/game_mode_through_cpp/output_log.png)


## After Thoughts / Rants 

I come from a technical background and absolutely adore coding. So I decided that when I started learning Unreal Engine I wanted to stick to C++ as much as possible. 

Simple enough? 

No.

Finding tutorials or discussion points that fully stick to C++ is near impossible. There is a heavy emphasis on blueprints, even in the [technical documentation](https://docs.unrealengine.com/5.0/en-US/setting-up-a-game-mode-in-unreal-engine/). I haven’t fully wrapped my head around the reasons why. Is it because the Blueprints are easier? Is the Unreal Community less technical? A bit of both? This remains to be seen but hopefully in the future when I have questions like this I won’t spend hours searching for a solution. 
