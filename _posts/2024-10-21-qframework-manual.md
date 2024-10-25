---
title: 面向我自己的QFramework快速入门（1）
date: 2024-10-21 11:34:35 +0800
tags: [Unity, QFramework]
categories: [Unity]
---

#### 一些废话

QFramework有作者本人编写的教程。此前经常阻止我写笔记的一个原因是我学习东西的途径绝大多数时候也是网络教程，且相比视频更偏好文档，那么有教程的存在我的笔记又有什么意义呢？不过阅读了这篇教程（并且经历了一个昏昏欲睡十分痛苦的下午）后，我感觉这篇教程虽然使用了一个案例覆盖了基本全部元素，但并没有特别详细，也没有让我体会到QFramework这些元素的设计特色与本质，而这是我比较喜欢在最开始就了解到的。此外也有一些前置知识我自己不太了解。因此我打算参考原教程给自己重新整理一个教程。

#### QFramework的基础：MVC架构

MVC架构将应用程序分为了**Model、View和Controller**三部分。Model简单来说就是管数据和与紧密相关的一些逻辑的；View是用户可交互的内容，不负责处理逻辑，只负责根据Model展示内容和接收用户的输入；Controller负责处理用户的输入，并引起Model和View的一系列改变。

在这里可能比较难以区分Model和Controller各自负责的逻辑部分，但介绍了Controller负责的两部分逻辑后这一点可能会更加明白：

* 交互逻辑：根据用户的输入对Model进行操作；

* 表现逻辑：根据Model对View进行操作。

而其他如Model自身的一些处理就是Model自己负责的。

如教程里举的这个例子，一个简单的点击计数器：

![img](https://file.liangxiegame.com/902ef543-64b1-45ad-ae45-ea180ebec133.png)

Model包含一个存储当前数值的整型变量；View包含两个按钮和一个数字文本；Controller则负责监听按钮被按下的输入，并且完成两个逻辑：

* 在按钮被点击后，改变数值（交互逻辑）

* 在数值改变后，改变数字文本（表现逻辑）

暂时不引入QFramework，我们用脚本实现这一系列内容：

```c#
using UnityEngine;
using UnityEngine.UI;

namespace QFramework.Example
{
    // Controller
    public class CounterAppController : MonoBehaviour
    {
        // View
        private Button mBtnAdd;
        private Button mBtnSub;
        private Text mCountText;

        // Model
        private int mCount = 0;

        void Start()
        {
            // View 组件获取
            mBtnAdd = transform.Find("BtnAdd").GetComponent<Button>();
            mBtnSub = transform.Find("BtnSub").GetComponent<Button>();
            mCountText = transform.Find("CountText").GetComponent<Text>();


            // 监听输入
            mBtnAdd.onClick.AddListener(() =>
            {
                // 交互逻辑
                mCount++;
                // 表现逻辑
                UpdateView();        
            });

            mBtnSub.onClick.AddListener(() =>
            {
                // 交互逻辑
                mCount--;
                // 表现逻辑
                UpdateView();
            });

            UpdateView();
        }

        void UpdateView()
        {
            mCountText.text = mCount.ToString();
        }
    }
}
```

这其中比较容易引起疑惑的是`AddListener(...)`那一部分。这就需要引入**回调函数**（Callback Function）的概念：简单地说，就是作为参数传给另一个函数，并且在满足特定条件的时候会被自动调用的函数。在QFramework的编写中会经常用到回调函数，尤其是用Lambda表达式编写的回调函数，所以如果之前不太习惯这种风格，那么需要注意。

#### QFramework的Model

不难发现在上面这段代码中，Model只属于`CounterAppController`，不便于全局共享。若想让它变成共享的，可以将其变成静态变量或单例；但QFramework提供了一个更美的方案。

首先我们引入一个继承自`AbstractModel`类的具体类来存储这个整型数值，这个类是QFramework提供给所有Model的基类：

```c#
public class CounterAppModel : AbstractModel
    {
        public int Count;

        protected override void OnInit()
        {
            Count = 0;
        }
    }
```

然后，我们引入一个继承自`Architecture`的整体架构类，并且在里面注册好需要的所有Model、System和Utility（后面会讲）等：

```c#
public class CounterApp : Architecture<CounterApp>
    {
        protected override void Init()
        {
            // 注册 Model
            this.RegisterModel(new CounterAppModel());
        }
    }
```

这个`Architecture`类提供 MVC、分层、模块管理等多种功能，其中一个体现便是Model等一经注册，就可以在任何地方访问了，如Controller。改后完整代码如下：

```c#
using UnityEngine;
using UnityEngine.UI;

namespace QFramework.Example
{

    // 1. 定义一个 Model 对象
    public class CounterAppModel : AbstractModel
    {
        public int Count;

        protected override void OnInit()
        {
            Count = 0;
        }
    }


    // 2.定义一个架构（提供 MVC、分层、模块管理等）
    public class CounterApp : Architecture<CounterApp>
    {
        protected override void Init()
        {
            // 注册 Model
            this.RegisterModel(new CounterAppModel());
        }
    }

    // Controller
    public class CounterAppController : MonoBehaviour , IController 
    /* 3.实现 IController 接口 */
    {
        // View
        private Button mBtnAdd;
        private Button mBtnSub;
        private Text mCountText;

        // 4. Model
        private CounterAppModel mModel;

        void Start()
        {
            // 5. 获取模型
            mModel = this.GetModel<CounterAppModel>();

            // View 组件获取
            mBtnAdd = transform.Find("BtnAdd").GetComponent<Button>();
            mBtnSub = transform.Find("BtnSub").GetComponent<Button>();
            mCountText = transform.Find("CountText").GetComponent<Text>();


            // 监听输入
            mBtnAdd.onClick.AddListener(() =>
            {
                // 6. 交互逻辑
                mModel.Count++;
                // 表现逻辑
                UpdateView();        
            });

            mBtnSub.onClick.AddListener(() =>
            {
                // 7. 交互逻辑
                mModel.Count--;
                // 表现逻辑
                UpdateView();
            });

            UpdateView();
        }

        void UpdateView()
        {
            mCountText.text = mModel.Count.ToString();
        }

        // 3.指定架构
        public IArchitecture GetArchitecture()
        {
            return CounterApp.Interface;
        }

        private void OnDestroy()
        {
            // 8. 将 Model 设置为空
            mModel = null;
        }
    }
}
```

其中有几个疑问点，一一解答：

##### 1.IController

我们发现`CounterAppController`类继承多了一个`IController`，而它以I开头，说明它是一个接口。和C++不同，C#只支持同时继承一个类，但可以同时继承（或者更应该说实现）多个接口。这里**类继承**是一种“**是**（is-a）”的关系，**接口实现** 则是一种“**能做**（can-do）”的关系。

> *类声明关键字`Interface`*：表明声明一个接口。接口不能被直接实例化；继承自接口的类必须实现接口内的所有方法。

`IController`是QFramework提供的controller接口，不过它并没有任何方法要实现。

##### 2.为什么this.GetModel<>()可以获取Model

你可以认为这是一个类似Unity的`GetComponent`功能，它的作用就是在*它所属的架构*（所以有个this）中寻找一个后面填入的类型的Model。而后面的这个函数就是为了配合实现这个功能：

```c#
public IArchitecture GetArchitecture()
        {
            return CounterApp.Interface;
        }
```

所谓“指定架构”就是让`GetArchitecture`函数可以返回它所在的架构。这里我们可以发现，所有的东西都必须依附在一个架构上，只不过有一些是用`Register`，Controller（由于是最上层的东西）则需要自己来指定。`Interface`可以视作返回`CounterApp`本身。至于具体如何配合，我们可以认为是这样的流程：

> 调用`this.GetModel`→调用`GetArchitecture()`→调用架构中的`GetModel`

然而深挖源代码后我发现里面有些对我来说的新知识，所以我打算在下面的折叠里详细介绍：

<details><summary>详细介绍</summary>

首先，`IController`继承自一系列其他的接口，包括但不限于`IBelongToArchitecture, ICanGetSystem, ICanGetModel, ICanGetUtility`等。而`GetModel`这个方法，是以**扩展方法**的形式实现的：


```
    
    public static class CanGetModelExtension
    {
        public static T GetModel(this ICanGetModel self) where T : class, IModel =>
            self.GetArchitecture().GetModel();
    }
    
    
```

· `CanGetModelExtension`是专门用于定义这个扩展方法的静态类，这是因为C#中扩展方法必须定义在静态类中。  
· `this ICanGetModel self`：扩展方法的第一个参数表示这个方法是针对 `ICanGetModel`类型的对象进行扩展的，`self`是方法的调用者在这个方法中的变量名。这意味着，任何实现了`ICanGetModel`接口的对象都可以调用这个扩展方法，如同它是类本身的方法一样。扩展方法还可以有第二个第三个等更多参数，它们的性质就和普通的参数一样了。  
· `where T : class, IModel`是一个**泛型约束**，它指定了T必须是一个引用类型（`class`）（与“值类型”相对），并且必须实现`IModel`接口。这里T是一个Model的类，而QFramework也有一个给Model的接口IModel，这么做就可以理解了。  

以上这些足够解释，不过我发现泛型约束还出现在另外一个地方。首先，`Architecture`也有一个接口`IArchitecture`,而`Architecture`实际上是一个继承自这个接口的抽象类，而这个抽象类的声明第一行是这样的：

```

public abstract class Architecture : IArchitecture where T : Architecture, new()

```

这也是个泛型约束。首先，`Architecture<T>`表明T本身必须是一个继承了`Architecture`的类。这种模式被称为递归泛型。  
`new()`则要求T类型必须有一个无参构造函数（即T可以通过`new T()`创建）。  
不难发现`CounterApp`符合这个条件。  

值得一提的是，`CounterApp.Interface`实际上是在`Architecture<T>`中这么干的：

```
    
     protected static T mArchitecture;
     public static IArchitecture Interface
    {
        get
        {
            if (mArchitecture == null) MakeSureArchitecture();
            return mArchitecture;
        }
    }

```

所以返回的东西实际上只有`IArchitecture`的性质，也就是一个接口。
</details>

#### Command

Command是用来简化Controller中的交互逻辑的。

可以把它直接看成一个更规范整洁的、经常被使用所以需要整合的函数或者方法。比如计数器中的Command可以像是这样：

```c#
	public class IncreaseCountCommand : AbstractCommand
    {
        protected override void OnExecute()
        {
            this.GetModel<CounterAppModel>().Count++;
        }
    }

    public class DecreaseCountCommand : AbstractCommand
    {
        protected override void OnExecute()
        {
            this.GetModel<CounterAppModel>().Count--;
        }
    }
```

使用则像：` this.SendCommand<DecreaseCountCommand>();` 它也可以在许多地方调用。*如果你读过详细介绍，我可以告诉你它也是一个扩展方法。*

Command也可以在别处复用，当然还有其他的一些性质。实际上，你肯定有过把经常复用的代码整合到一个函数中的经验，但是Command是将所有的这些函数加上了“是一个Command”这样的性质。一来再乱也乱不出Command类，二来可以通过这个共同点实现更多Command的独特功能。

#### Event

Event是用来简化Controller中的表现逻辑的。

我个人认为Event非常像Qt的signal和slot（如果你并不了解，不用在意这句话，但如果你了解则可以帮助理解）。非常明显的一点在于，Event的声明只需要一个类型，且为了让garbage collection最少，往往使用结构体：

```c#
public struct CountChangeEvent
{
}
```

这样来执行一个事件，非常像发射signal：

```c#
this.SendEvent<CountChangeEvent>();
```

在Controller（或者其他地方）中，可以完善具体的Event内容：

```c#
this.RegisterEvent<CountChangeEvent>(e =>
{
	UpdateView();
}).UnRegisterWhenGameObjectDestroyed(gameObject);
```

这里传入的也是一个有参数e的回调函数（我们也经常写Lambda表达式形式的slot函数，不是吗？），不过这个参数没有被使用。当gameObject被销毁后，这个Event内容便会被清除。

*是的，`SendEvent`和`RegisterEvent`都是扩展方法*。

---

在这里我们这么做的初衷是什么？说回简化表现逻辑的事——在计数器的例子中，我们希望Model每次被改变就更新显示，那么我们就可以手动定义更新显示事件（上面已经有了）并且在修改Model后执行：

```c#
    public class IncreaseCountCommand : AbstractCommand 
    {
        protected override void OnExecute()
        {
            this.GetModel<CounterAppModel>().Count++;
            this.SendEvent<CountChangeEvent>(); // ++
        }
    }

    public class DecreaseCountCommand : AbstractCommand
    {
        protected override void OnExecute()
        {
            this.GetModel<CounterAppModel>().Count--;
            this.SendEvent<CountChangeEvent>(); // ++
        }
    }
```

也就是说，我们不再需要手动调用表现逻辑，而只需要在执行交互逻辑的Command里紧接着发送事件。而Controller原本表现逻辑的部分只需要把事件订阅好就可以了。**绝大多数情况下，Command里内容执行完后必须发送事件，除非View不用更新！**

*不过我个人认为，Event这个设计更吸引我的点是它可以进行架构内的通讯。实际上我们在很多地方都可以响应一个`SendEvent`，不仅限于Command里面。*

> Command和Event是基于**CQRS**（Command Query Responsibility Segregation）原则的。什么违反了这个原则呢？若一个函数同时进行了写入（改动）和读取（返回）功能，则它违反了这个原则，比如`stack`或者`queue`的`pop()`等。

#### Utility

Utility没有自带的功能。它用来写一些我们自定义的基础设施，比如存储方法、序列化方法、网络连接方法、蓝牙方法、SDK、框架继承等。以这个计数器中的存储方法为例，我们可以定义存储的Utility，这样就可以在需要修改的时候只修改Utility中的内容：

```c#
	public class Storage : IUtility
    {
        public void SaveInt(string key, int value)
        {
            PlayerPrefs.SetInt(key,value);
        }

        public int LoadInt(string key, int defaultValue = 0)
        {
            return PlayerPrefs.GetInt(key, defaultValue);
        }
    }

	public class CounterApp : Architecture<CounterApp>
    {
        protected override void Init()
        {
            // 注册 Model
            this.RegisterModel(new CounterAppModel());

            // 注册存储工具的对象  !!!!
            this.RegisterUtility(new Storage());
        }
    }
```

使用Utility be like：`this.GetUtility<Storage>().SaveInt(nameof(Count), Count);`*是的，`GetUtility`也是……*

#### System

System用于集成一些事件（如相似性质的规则触发的事件），帮助Controller承担一部分逻辑，尤其是在多个表现层共享的逻辑，比如计时系统、商城系统、成就系统等。

比如计数器的成就系统：

```c#
	public class AchievementSystem : AbstractSystem // +
    {
        protected override void OnInit()
        {
            var model = this.GetModel<CounterAppModel>();

            this.RegisterEvent<CountChangeEvent>(e =>
            {
                if (model.Count == 10)
                {
                    Debug.Log("触发 点击达人 成就");
                }
                else if (model.Count == 20)
                {
                    Debug.Log("触发 点击专家 成就");
                } else if (model.Count == -10)
                {
                    Debug.Log("触发 点击菜鸟 成就");
                }
            });
        }
    }
```

System同样需要注册：`this.RegisterSystem(new AchievementSystem());`

#### Query

Query是CQRS原则的另一块拼图。它可以集成查找操作，和Command用法几乎一样，把它们分为两种类型大抵是为了将这两种操作分类。

Query继承自`AbstractQuery`，不过需要写明查询结果类型：

```c#
	public class SchoolAllPersonCountQuery : AbstractQuery<int>
	{
		protected override int OnDo()
            {
                return this.GetModel<StudentModel>().StudentNames.Count +
                       this.GetModel<TeacherModel>().TeacherNames.Count;
            }
        }

```

这样使用：

```c#
	if (GUILayout.Button("查询学校总人数"))
	{
		mAllPersonCount = this.SendQuery(new SchoolAllPersonCountQuery());
	}
```

#### 架构的规则

虽然上面尤其提到了在许多地方都可以使用扩展方法，但具体哪里可以使用肯定也是有明确的规定的。只要在源码里查看对应的接口所继承的接口就可以了，比如：

```c#
public interface IController : IBelongToArchitecture, ICanSendCommand, ICanGetSystem, ICanGetModel,ICanRegisterEvent, ICanSendQuery
{
}
```

意味着它必须指定所属架构；可以发送Command和Query；可以获取System和Model；可以监听Event。

其他的可以自己去查看源码，但为了保持这篇教程的完整性，转载一些原教程中的规则：

---

QFramework 架构提供了四个层级：

- 表现层：IController
- 系统层：ISystem
- 数据层：IModel
- 工具层：IUtility

除了四个层级，还提供了 Command、Query、Event、BindableProperty 等概念和工具。

这里有一套层级的规则，如下：

- 表现层：ViewController 层。IController接口，负责接收输入和状态变化时的表现，一般情况下，MonoBehaviour 均为表现层
  - 可以获取 System、Model
  - 可以发送 Command、Query
  - 可以监听 Event

- 系统层：System层。ISystem接口，帮助IController承担一部分逻辑，在多个表现层共享的逻辑，比如计时系统、商城系统、成就系统等
  - 可以获取 System、Model、Utility
  - 可以监听Event
  - 可以发送Event

- 数据层：Model层。IModel接口，负责数据的定义、数据的增删查改方法的提供
  - 可以获取 Utility
  - 可以发送 Event

- 工具层：Utility层。IUtility接口，负责提供基础设施，比如存储方法、序列化方法、网络连接方法、蓝牙方法、SDK、框架继承等。啥都干不了，可以集成第三方库，或者封装API

- Command：命令，负责数据的增删改。
  - 可以获取 System、Model、Utility
  - 可以发送 Event、Command、Query

- Query：查询、负责数据的查询
  - 可以获取 System、Model
  - 可以发送 Query

或者列个表格：

|            | 获取System | 获取Model | 获取Utility | 监听Event | 发送Event | 发送Command | 发送Query |
| ---------- | ---------- | --------- | ----------- | --------- | --------- | ----------- | --------- |
| Controller | √          | √         | -           | √         | -         | √           | √         |
| System     | √          | √         | √           | √         | √         | -           | -         |
| Model      | -          | -         | √           | -         | √         | -           | -         |
| Utility    | -          | -         | -           | -         | -         | -           | -         |
| Command    | √          | √         | √           | -         | √         | √           | √         |
| Query      | √          | √         | -           | -         | -         | -           | √         |

- 通用规则：
  - IController 更改 ISystem、IModel 的状态必须用Command
  - ISystem、IModel 状态发生变更后通知 IController 必须用事件或BindableProperty（还没讲到的工具）
  - IController可以获取ISystem、IModel对象来进行数据查询
  - ICommand、IQuery 不能有状态
  - 上层可以直接获取下层，下层不能获取上层对象
  - 下层向上层通信用事件
  - 上层向下层通信用方法调用（只是做查询，状态变更用 Command），IController 的交互逻辑为特别情况，只能用 Command

此外如果想改变以上规则也很容易，只要在源码加上对应接口即可。