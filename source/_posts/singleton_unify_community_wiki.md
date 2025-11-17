---
title: Singleton - Unify Community Wiki
date: 2023-01-05 14:11:02
tags:
- Unity
- Singleton
categories:
- [Unify Community Wiki,C Sharp]
- [Unify Community Wiki,MonoBehaviour]
- [Unify Community Wiki,Design Patterns]
---
# Singleton

> From [Singleton - Unify Community Wiki (archive.org)](http://web.archive.org/web/20201112042127/http://wiki.unity3d.com/index.php/Singleton)

[TOC]

## Alternatives

#### Scriptable Objects

One excellent alternative to the singleton pattern in Unity is the use of ScriptableObjects as a type of global variable. Ryan Hipple from Schell Games gave a presentation at Unite Austin 2017 titled [Game Architecture with Scriptable Objects](https://youtu.be/raQ3iHhE_Kk) that explains how to implement them and the many advantages over singletons.

#### Toolbox

The "[Toolbox](http://wiki.unity3d.com/index.php/Toolbox "Toolbox")" concept improves upon the singleton pattern further by offering a different approach that makes them more modular and improves testability.

<!--more-->

## Introduction

The [singleton](http://en.wikipedia.org/wiki/Singleton_pattern) pattern is a way to ensure a class has only a single globally accessible instance available at all times. Behaving much like a regular static class but with some advantages. This is very useful for making global manager type classes that hold global variables and functions that many other classes need to access. However, the convenience of the pattern can easily lead to misuse and abuse and this has made it [somewhat controversial](https://softwareengineering.stackexchange.com/questions/40373/so-singletons-are-bad-then-what/218322#218322) with many critics considered it an anti-pattern that should be avoided. But like any design pattern, singletons can be very useful in certain situations and it's ultimately up to the developer to decide if it's right for them.

#### Advantages

* Globally accessible. No need to search for or maintain a reference to the class.
* Persistent data. Can be used to maintain data across scenes.
* Supports interfaces. Static classes can not implement interfaces.
* Supports inheritance. Static classes can not inherent for another class.

The advantage of using singletons in Unity, rather than static parameters and methods, is that static classes are lazy-loaded when they are first referenced, but must have an empty static constructor (or one is generated for you). This means it's easier to mess up and break code if you're not careful and know what you're doing. As for using the Singleton Pattern, you automatically already do lots of neat stuff, such as creating them with a static initialization method and making them immutable.

#### Disadvantages

* Must use the ***Instance*** keyword (e.g. <ClassName>.Instance) to access the singleton class.
* There can only ever be one instance of the class active at a time.
* Tight connections. Modifying the singleton can easily break all other code that depends on it. Requiring a lot of refactoring.
* No polymorphism.
* Not very testable.

## Implementation

The singleton pattern is commonly applied to multiple classes but the implementation is always the same. So creating a singleton base class others can inherit from is ideal because it eliminates the need to recreate the same code over and over again for each class that needs to be a singleton.

#### Notes:

* This script will not prevent non singleton constructors from being used in your derived classes. To prevent this, add a protected constructor to each derived class.
* When Unity quits it destroys objects in a random order and this can create issues for singletons. So we prevent access to the singleton instance when the application quits to prevent problems.

### Singleton.cs

```C#
using UnityEngine;

/// <summary>
/// Inherit from this base class to create a singleton.
/// e.g. public class MyClassName : Singleton<MyClassName> {}
/// </summary>
public class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    // Check to see if we're about to be destroyed.
    private static bool m_ShuttingDown = false;
    private static object m_Lock = new object();
    private static T m_Instance;

    /// <summary>
    /// Access singleton instance through this propriety.
    /// </summary>
    public static T Instance
    {
        get
        {
            if (m_ShuttingDown)
            {
                Debug.LogWarning("[Singleton] Instance '" + typeof(T) +
                    "' already destroyed. Returning null.");
                return null;
            }

            lock (m_Lock)
            {
                if (m_Instance == null)
                {
                    // Search for existing instance.
                    m_Instance = (T)FindObjectOfType(typeof(T));

                    // Create new instance if one doesn't already exist.
                    if (m_Instance == null)
                    {
                        // Need to create a new GameObject to attach the singleton to.
                        var singletonObject = new GameObject();
                        m_Instance = singletonObject.AddComponent<T>();
                        singletonObject.name = typeof(T).ToString() + " (Singleton)";

                        // Make instance persistent.
                        DontDestroyOnLoad(singletonObject);
                    }
                }

                return m_Instance;
            }
        }
    }


    private void OnApplicationQuit()
    {
        m_ShuttingDown = true;
    }


    private void OnDestroy()
    {
        m_ShuttingDown = true;
    }
}
```

## Requirements

The above script makes use of the custom extension method [GetOrAddComponent](http://wiki.unity3d.com/index.php/GetOrAddComponent "GetOrAddComponent") which is not a part of Unity.

## Usage

To make any class a singleton, simply inherit from the Singleton base class instead of MonoBehaviour, like so:

```C#
public class MySingleton : Singleton<MySingleton>
{
    // (Optional) Prevent non-singleton constructor use.
    protected MySingleton() { }

    // Then add whatever code to the class you need as you normally would.
    public string MyTestString = "Hello world!";
}
```

Now you can access all public fields, properties and methods from the class anywhere using **&lt;ClassName&gt;.Instance**:

```C#
public class MyClass : MonoBehaviour
{
    private void OnEnable()
    {
        Debug.Log(MySingleton.Instance.MyTestString);
    }
}
```

Categories: 
* [C Sharp](http://wiki.unity3d.com/index.php/Category:C_Sharp "Category:C Sharp")
* [MonoBehaviour](http://wiki.unity3d.com/index.php/Category:MonoBehaviour "Category:MonoBehaviour")
* [Design Patterns](http://wiki.unity3d.com/index.php/Category:Design_Patterns "Category:Design Patterns")
