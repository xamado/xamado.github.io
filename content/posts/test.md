+++
author = "Xabier Amado"
title = "UE5: Accessing BP components from CDO"
date = "2019-03-11"
description = "Sample article showcasing basic Markdown syntax and formatting for HTML elements."
tags = [
    "unreal",
    "ue5",
]
categories = [
    "unreal",
]
series = ["UE5"]
+++

I was recently trying to create an actor during runtime, and populate it with the components from another actor, specified as the "template" through it's class. This other actor wasn't being Spawned, but rather used as a reference.
<!--more-->

#### Accessing the Blueprint generated class

Every actor in Unreal has a default internally created instance called the CDO (Constructed Default Object) which is a very special instance of the actor, which doesn't exist in the World, and for all intents and purposes it's not running.

Blueprint defined classes inherit from the UBlueprintGeneratedClass (which in turn is a UClass). The UBlueprintGeneratedClass has information related to the class, among it, the "Simple Construction Script", which is a graph of all the components to be created for such an actor.

```cpp
// Assuming you have a TSubclassOf<AActor> pointer to a BP class
TSubclassOf<class AActor> ConstructableActor;

// We can cast it to a UBlueprintGeneratedClass to get access to the BP specific information
const UBlueprintGeneratedClass* ActorBlueprintGeneratedClass = Cast<UBlueprintGeneratedClass>(ActorClass);
```

Optionally you could fetch a linear list of the hierarchy of classes, this way you could iterate and access the components provided by the parents as well.

```cpp
TArray<const UBlueprintGeneratedClass*> BlueprintClasses;
UBlueprintGeneratedClass::GetGeneratedClassesHierarchy(ActorBlueprintGeneratedClass, BlueprintClasses);
```

#### Fetching nodes

Now you can iterate the nodes of the Simple Construction Script to fetch the data for each component. Each node is represented by the USCS_Node class.

```cpp
const TArray<USCS_Node*>& ActorBlueprintNodes = ActorBlueprintGeneratedClass->SimpleConstructionScript->GetAllNodes();

for (const USCS_Node* Node : ActorBlueprintNodes)
{
    // do stuff
}
```

Now you can query Node->ComponentClass to know what type of component you are dealing with, and find the ones that interest you. You can also use Node->ComponentTemplate to access the CDO's instance of the component. You can use it to query default values, or to instance a new component, which was my case, so here's a working example of duplicating static meshes from an Actor class:

```cpp
const UBlueprintGeneratedClass* ActorBlueprintGeneratedClass = Cast<UBlueprintGeneratedClass>(ConstructableData->ConstructableActor);
const TArray<USCS_Node*>& ActorBlueprintNodes = ActorBlueprintGeneratedClass->SimpleConstructionScript->GetAllNodes();

TArray<UActorComponent*> AddedComponents;

for (const USCS_Node* Node : ActorBlueprintNodes)
{
    if (Node->ComponentClass == UStaticMeshComponent::StaticClass())
    {
        UStaticMeshComponent* NewComponent = NewObject<UStaticMeshComponent>(this, UStaticMeshComponent::StaticClass(), Node->GetVariableName(), RF_NoFlags, Node->ComponentTemplate);

        NewComponent->AttachToComponent(this->GetRootComponent(), FAttachmentTransformRules::KeepRelativeTransform);
        this->AddInstanceComponent(NewComponent);
        NewComponent->RegisterComponent();

        AddedComponents.Add(NewComponent);
    }
}
```

