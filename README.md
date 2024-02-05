# 05-Garbage-Collection
<div align="center">
  <a href="Images\Main.png" target="_blank">
    <img src="Images\Main.png" style="height:400px;"/>
  </a>
</div>

## Introduction
Unity utilises a managed memory system (also called the scripting memory system), which is a controlled/monitored memory environment. In other words, Unity keeps track of all of the memory allocations your game makes.

Of this managed memory, the most significant part to be aware of in the context of optimisation is the [managed heap](https://docs.unity3d.com/Manual/performance-memory-overview.html#managed-memory). The managed heap is a non-linear allocation of memory that is useful for storing data types of dynamic sizes.

A 32-bit float contains 4 bytes and will always be 4 bytes no matter the value it stores, so it is not stored on the managed heap as its size isn't dynamic. Lists, like arrays, hold multiple elements of the same data type in a sequence. Unlike arrays however, lists are dynamic and can shrink or grow in size, so lists always exist on the managed heap.

The managed heap is automatically controlled by a garbage collector (GC). And any memory allocated to the managed heap is referred to as GC Allocation, which can be recorded using the Profiler. You can do as much GC Allocation as you want and may find minimal issues in your game, but that's just because it isn't a problem until the managed heap needs to be cleared from memory. This is called garbage removal.

Garbage removal automatically occurs when the managed heap reaches a threshold predefined by Unity (just like emptying your rubbish bin when it is looking close to full) or when the game is trying to store more data in the managed heap but there isn't enough space for it. It also checks for references to the data on the managed heap, and if there is data on there that no longer has any references (unused data), then it is removed.

Unity's built-in garbage collector comes from an open-source project called the [Boehm-Demers-Weiser garbage collector](https://www.hboehm.info/gc/). This collector actually stops running your code to remove any garbage that has been collected and will only resume your code once it has completed garbage removal. This is why you may find spikes in the profiler that are related to garbage removal. In other words, that GC Allocation needs to be cleared from memory at some point, and when it does it is taxing on performance.

You have little choice over when garbage collection occours. Although you can trigger it additionally at times of your choosing using [System.GC.Collect](https://learn.microsoft.com/en-us/dotnet/api/system.gc.collect?view=net-5.0). You'd probably want to trigger it when the user won't notice, such as when loading a new level or opening a menu. This is something developer Ruben Bonet did while working on Star Trek Bridge Crew for Oculus Quest (see [here](https://thegamedev.guru/unity-performance/garbage-collection-manually/) to learn more about his implementation).

Data fragmentation can be an issue with the managed heap, as the managed heap can be far from full and have enough free space for new GC allocation, but is unable to fit it due to past allocations. Imagine a game that allocates the position of every new object that is instantiated to a list, meaning the list keeps getting larger. A list is actually a disguised array; whenever the list needs to be larger, the array is copied into a larger array and this becomes the list. All of these arrays are stored in the managed heap. Imagine this is the case in the image below; in this simple example we have a managed heap of size 32 and we started by allocating a list of size two, then three, then four, then five, then six as the list kept expanding and its contents were copied to larger arrays. Like previously, another object is instantiated so its position needs to be added to the the list, so we now have an upcoming GC allocation of size 7. Although the total free space is of size 12, we can't fit this GC allocation, so we need to run garbage collection. All of the GC allocation is unused, the game only references the list of size 7, so it's likely all of the GC allocated will be collected, leaving the managed heap empty. This is one of the things that happens in today's Unity tutorial.

<div align="center">
  <a href="Images\Memory Fragmentation.png" target="_blank">
    <img src="Images\Memory Fragmentation.png" style="height:100px;"/>
  </a>
</div>
<br>

If you find garbage collection is causing an irregular frame rate because of long interruptions during the running of your game, you can turn on Unity's incremental garbage collection. This distributes the workload over multiple frames for multiple but shorter interruptions. This concept might sound familiar, it is simple time-slicing like discussed in [week 1](https://github.com/danmilneusw/01-Measuring-Game-Engine-Performance). This can be enabled at Project Settings > Player > Use Incremental GC.

Leave it off for our project today, because in reality, it is best to minimise GC Allocation as much as possible by redesigning your code, which as a result minimises garbage collection.

## Summary
- Understand garbage collection
- See the extent of garbage collection using the profiler
- Use object pools to reduce garbage collection

## Tutorial
We'll soon be moving away from covering Unity optimisations and instead work primarily in PyGame. So I'm introducing you to a tutorial to follow from [Catlike Coding](https://catlikecoding.com/). I'm doing this because on this site you'll find a collection of what is probably the best Unity tutorials on the internet, and the site is supported by a Patreon so they're all completely free too. I highly recommend you look at this site if you are interested in learning more about Unity in your free-time and you might get some ideas of your own for the first assessment from the site also.

To get you up to speed: the Unity project I have provided is from the [Catlike Coding tutorial series on Object Management](https://catlikecoding.com/unity/tutorials/object-management/). Part one involved creating a project that instantiated a new cube from a prefab whenever the user pressed the C key. They could instantiate as many cubes as they wanted and they would appear in a random position and rotation within the scene. This data is stored in a list. They could save this list to a file with another key press, destroy all objects with another, and then load the data with another (as a side note, [this tutorial I am talking about](https://catlikecoding.com/unity/tutorials/object-management/persisting-objects/) is a great resouce for understanding the saving of game states).

<div align="center">
  <a href="Images\Persisting Objects.jpg" target="_blank">
    <img src="Images\Persisting Objects.jpg" style="height:200px;"/>
  </a>
</div>
<div align="center">
  <a href="https://catlikecoding.com/unity/tutorials/object-management/persisting-objects/">
  source
  </a>
</div>
<br>

In [part two of the tutorial](https://catlikecoding.com/unity/tutorials/object-management/object-variety/), the range of shapes were increased using different prefabs and randomising their colours. This information is stored in the list also and can be saved. This tutorial also discusses GPU Instancing to improve performance, like we discussed [last week](https://github.com/danmilneusw/04-Draw-Calls/blob/main/README.md)).

<div align="center">
  <a href="Images\Object Variety.jpg" target="_blank">
    <img src="Images\Object Variety.jpg" style="height:200px;"/>
  </a>
</div>
<div align="center">
  <a href="https://catlikecoding.com/unity/tutorials/object-management/object-variety/">
  source
  </a>
</div>
<br>

1. Finally, we are at the stage where you can begin working on the project, so open **Unity Project - Catlike Coding - Object Management - Reusing Objects**.

2. Also open [the Catlike Coding tutorial](https://catlikecoding.com/unity/tutorials/object-management/reusing-objects/). I have completed the tutorial up to '3 Object Pools', to get you up to the stage of monitoring for garbage. Give the contents a quick skim read up to this header (some of the content is more useful than other content). In short, the project has introduced sliders that allow for automatic instantiation and destruction of the shapes from the list. Continue the tutorial from this point on. You will see how to look for garbage collection and learn an example of reducing the amount of garbage using an optimisation technique called Object Pooling (the author sometimes calls it 'recycling').

## Extra Resources
- [The Game Dev Guru - Manual Garbage Collection for Star Trek Bridge Crew](https://thegamedev.guru/unity-performance/garbage-collection-manually/)
- [Unity Docs - Garbage Collection Best Practices](https://docs.unity3d.com/Manual/performance-garbage-collection-best-practices.html#reusablepools)
- [Unity Docs - Garbage Collection Best Practices: Reusable Pools (Object Pooling)](https://docs.unity3d.com/Manual/performance-garbage-collection-best-practices.html#reusablepools) ⭐
- [Unity Docs - Memory Overview](https://docs.unity3d.com/Manual/performance-memory-overview.html)
- [Unity Docs - Managed Memory (including Managed Heap](https://docs.unity3d.com/Manual/performance-managed-memory.html)
- [Unity Docs - Garbage Collector Overview](https://docs.unity3d.com/Manual/performance-garbage-collector.html)
