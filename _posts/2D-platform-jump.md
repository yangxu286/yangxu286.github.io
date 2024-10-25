---
title: 2D平台跳跃笔记
date: 2024-10-21 13:47:20
tags: [Unity]
categories: [Unity]
---

#### 第一部分：Input System
Input System的使用可以大大简化输入操作相关的编程。目前我用到的部分中，它的功能相当于把键鼠、手柄等的输入整理归纳成”一个操作“，之后在脚本编写中只要提及这个操作就可以。此外，它的Action Maps功能允许在不同的界面（比如游戏界面和暂停界面）编写不同的操作，也十分方便清晰。
Input System需要通过Package Manager安装。安装之后，使用它的脚本需要```using UnityEngine.InputSystem;``` 然而，此时vs可能报错。一个可能的解决方案：点击Edit -> Preferences...然后点External Tools。勾选框Generate .csproj files for: Registry packages，然后单击Regenerate project files。
##### 开始使用
要开始使用Input System，在project view中的合适的文件夹（我放在Assets/Settings）中右键Create→Input Actions。双击新建出的内容，弹出窗口。左边为Action Maps，中间为Actions，右边则是供我们录入输入信息的。
首先新建一个Action Map。
此处仅讲解我们这次要用到的上下左右和跳跃控制：
上下左右：＋号新建一个Action，删除默认的那个No binding。由于上下左右控制是一个二维向量信息，将Action Type改为”Value“，Control Type改为”Vector 2“。右边有一个带小三角形的＋号，点选”Add...Composite"，会自动生成一个含有上下左右的Binding。选中它（左边标签是蓝色），右边Composite Type为2D Vector，Mode为Digital，这样x与y只会在-1，0，1之间变化。
首先绑定WASD键：如绑定W和Up，就点选Up，在Path中选W[Keyboard]。也可以用Listen功能，这样按一下键盘就可以出来对应的按键。
接下来，我们仍需要绑定上下左右键。在同一个Action下建立一个新的同类型的Binding绑定不同的按键即可。
跳跃：较为简单，新建一个类型为Button的Action，选择最简单的Add Binding即可。可以添加多个Binding来绑定更多按键。

##### 后续操作
完成编辑后，记得上方Save Asset。勾选Inspector中的Generate C# Class，下面会出来文件路径、文件名等信息，按需编辑即可（namespace一般不用动）。
之后就会生成一个脚本。我的脚本名为PlayerInputActions，上面那些Action所属的Action Map名为GamePlay。打开脚本，发现看不懂，但不用管。以后在别的脚本中，可以通过以下代码获取操作信息：
```c#
PlayerInputActions playerInputActions;
Vector2 axes => playerInputActions.Gameplay.Axes.ReadValue<Vector2>();
public bool Jump => playerInputActions.Gameplay.Jump.WasPressedThisFrame();
public bool StopJump => playerInputActions.Gameplay.Jump.WasReleasedThisFrame();
private void Awake()
{
    playerInputActions = new PlayerInputActions();
}
```
我们往往会对这些最原始的信息进行一些处理，再拿给接下来要讲到的状态机系统中去使用。

### 第二部分：状态机系统的两个基类
实际上，在实现玩家操控的过程中我们做的是玩家自己的状态机系统，之后还会有敌人、机关等的状态机系统。但是，这些系统都将会建立在两个基类上。
这两个基类分别为：IState，作为所有状态的基类；StateMachine，作为所有状态机的基类。状态不必赘述，而此处的状态机就是负责状态更新、开关与切换等的一个类。
一个状态需要具备四个基本的方法：**进入时操作Enter()，退出时操作Exit()，逻辑更新操作LogicUpdate()，物理更新操作PhysicUpdate()**。在IState中，显然这些都为空，然而它的派生类必须具备这四个方法（即使里面是空的，因为状态机里会用到它们）。所以，IState在实现上实际上是一个接口。
*接口：简而言之，接口里的东西，它的派生类必须实现。接口定义了所有类继承接口时应遵循的语法合同。接口定义了语法合同 "是什么"部分，派生类定义了语法合同 "怎么做" 部分。显然，接口是不可以实例化的。*
IState接口如下：
```c#
public interface IState
{
    void Enter();
    void Exit();
    void LogicUpdate();
    void PhysicUpdate();
}
```
StateMachine继承自MonoBehaviour。最最首先，它有一个当前状态```IState currentState;```。
最简单的是负责在Update()、FixedUpdate()中分别进行逻辑更新和物理更新（以防你不知道，物理相关操作都要在FixedUpdate中进行）。此外，它还负责开启某状态和切换某状态（区别在于有没有前一个状态），看起来像是这样：
```c#
public void SwitchOn(IState newState)
{
	currentState = newState;
	currentState.Enter();
}
public void SwitchState(IState newState)
{
	currentState.Exit();
	SwitchOn(newState);
}
```
此外，为了方便管理多个状态，StateMachine中存储了一个字典“状态表”，实现稍后再说：
```protected Dictionary<System.Type, IState> stateTable;```
*System.Type是Typeof()或GetType()的返回值类型。*

### 第三部分：PlayerObject挂载的三个脚本
#### 所有玩家状态的基类PlayerState（并不是挂载的脚本之一）
可以先思考一下玩家所具备的状态：Idle，Run，Jump，Fall，AirJump（二段跳）。每一个状态都将有一个自己的类和相应的脚本。但是，我们并不直接让这些类从IState接口派生，因为同为玩家的状态，它们会有一些共同的部分。最容易想到的，自然就是（稍后将要细说的）需要存储的玩家状态机PlayerStateMachine，以及输入系统、Animator等。总之，需要一个PlayerState作为所有玩家状态的基类。
这个类不仅继承自IState，还要继承自ScriptableObject。后续，将会对玩家的每一个状态都生成一个资产文件，大大节省我们的内存。（这边其实还没有完全理解。之后会在状态机里开一个数组，放入每一个状态的资源文件，或许只有ScriptableObject可以这么搞）
前面四个基础方法都不需要改动。以上这些共同的部分还需要进行初始化，所以需要一个Initialize()，长这样：
```c#
public void Initialize(Animator animator, PlayerController player,
    PlayerInput input,PlayerStateMachine stateMachine)
{
    this.animator = animator;
    this.player = player;
    this.input = input;
    this.stateMachine = stateMachine;
}
```
若还有其他部分，将在完善各个状态的过程中添加。

#### 玩家状态机 PlayerStateMachine
直接上代码：
```c#
public class PlayerStateMachine : StateMachine
{
    [SerializeField] PlayerState[] states;//序列化后便可以在Inspector中方便地放入资产文件
    Animator animator;
    PlayerController player;
    PlayerInput input;
    private void Awake()
    {
        animator = GetComponentInChildren<Animator>();
        player = GetComponent<PlayerController>();
        input = GetComponent<PlayerInput>();
        stateTable = new Dictionary<System.Type, IState>(states.Length);
        foreach (var state in states) //更新每一个状态的信息
        {
            state.Initialize(animator, player, input, this);
            stateTable.Add(state.GetType(), state);//存入状态表
        }
    }
    private void Start()
    {
        SwitchOn(stateTable[typeof(Idle_PlayerState)]);//默认开始状态为Idle
    }
}
```

#### 玩家输入处理器PlayerInput
对来自Input System的信息进行一些“翻译”。
```c#
public class PlayerInput : MonoBehaviour
{
    PlayerInputActions playerInputActions;
    Vector2 axes => playerInputActions.Gameplay.Axes.ReadValue<Vector2>();
    public bool Jump => playerInputActions.Gameplay.Jump.WasPressedThisFrame();
    public bool StopJump => playerInputActions.Gameplay.Jump.WasReleasedThisFrame();
    public bool Move => AxisX != 0f;
    public float AxisX => axes.x;
    
    private void Awake()
    {
        playerInputActions = new PlayerInputActions();
    }
    public void EnableGameplayInputs()
    {
        playerInputActions.Gameplay.Enable();//启用玩家控制Action Map（以后可能有UI等其他Action Map)
        Cursor.lockState = CursorLockMode.Locked;//游戏中常见的锁定光标
    }
}
```

#### 玩家控制器PlayerController
封装一些控制玩家运动的指令，简化后续状态的编写。此外，也进行一些不属于状态机系统的控制。
```c#
public class PlayerController : MonoBehaviour
{
    PlayerInput input;
    Rigidbody2D rb2D;
    public float MoveSpeed => Mathf.Abs(rb2D.velocity.x);//lambda表达式
    private void Awake()
    {
        input = GetComponent<PlayerInput>();
        rb2D = GetComponent<Rigidbody2D>();
    }
    private void Start()
    {
        input.EnableGameplayInputs();//启用玩家操控
    }
    public void Move(float speed, float acceleration)
    {
        if (input.Move)//根据移动方向翻转
        {
            transform.localScale = new Vector3(input.AxisX * 6.25f, 6.25f, 1f);
        }
        SetVelocityX(Mathf.MoveTowards(rb2D.velocity.x, speed * input.AxisX, acceleration * Time.deltaTime));
    }
    public void SetVelocity(Vector2 velocity)
    {
        rb2D.velocity = velocity;
    }
    public void SetVelocityX(float velocityX)
    {
        rb2D.velocity = new Vector2(velocityX, rb2D.velocity.y);
    }
    public void SetVelocityY(float velocityY)
    {
        rb2D.velocity = new Vector2(rb2D.velocity.x, velocityY);
    }
    public void SetUseGravity(bool useGravity)
    {
        rb2D.gravityScale = useGravity ? 1f : 0f;
    }
}
```

### 第四部分：状态编写范例（Idle和Run状态）
（懒了）作为状态编写的样例，之后其他的状态就不会细说了，不然就不是笔记而是教程了。
```c#
[CreateAssetMenu(menuName ="Data/StateMachine/PlayerState/Idle",fileName = "Idle_PlayerState")]
//在project view右键create资产文件
public class Idle_PlayerState : PlayerState
{
    [SerializeField] float deceleration = 5f;
    public override void Enter()
    {
        base.Enter();
    }
    public override void LogicUpdate()
    {
        if(input.Move)
        {
            stateMachine.SwitchState(typeof(Run_PlayerState));
        }
    }
    public override void PhysicUpdate()
    {
        player.Move(0f, deceleration);
    }
}

```
```c#
[CreateAssetMenu(menuName = "Data/StateMachine/PlayerState/Run", fileName = "Run_PlayerState")]
public class Run_PlayerState : PlayerState
{
    [SerializeField] float runSpeed = 3f;
    [SerializeField] float acceleration = 5f;
    public override void Enter()
    {
        base.Enter();
    }
    public override void LogicUpdate()
    {
        if (!input.Move)
        {
            stateMachine.SwitchState(typeof(Idle_PlayerState));
        }
    }
    public override void PhysicUpdate()
    {
        player.Move(runSpeed, acceleration);
    }
}
```

### 第五部分：动画相关
动画操作在PlayerState类的Enter()里统一进行，使用下面这个语句：
```animator.CrossFade(stateHash, transitionDuration);```
其中，stateHash是Animator里状态的名字转换成的哈希值，可以通过函数Animator.StringToHash()获取。所以，和动画播放有关的语句就这些：
```c#
[SerializeField] string stateName;
[SerializeField, Range(0f, 1f)] float transitionDuration = 0.1f;//可以使用滑块调整数据了
...
void OnEnable()
{
    stateHash = Animator.StringToHash(stateName);
}
public virtual void Enter()
{
    animator.CrossFade(stateHash, transitionDuration);
}
```
之后，需要手动在资产文件那边输入stateName。

### 第六部分：地面检测
本项目中地面检测的方法是在玩家脚底做一个很薄的矩形判定，我把这个功能放到玩家的一个新建子物体下（好像无关紧要）：
```c#
public class PlayerGroundDetector : MonoBehaviour
{
    [SerializeField] Vector2 detectionSize = new Vector2(0.1f, 0.0001f);
    [SerializeField] float detectionAngle = 0f;//矩形旋转角度
    [SerializeField] LayerMask groundLayer;//筛选groundLayer层的碰撞体

    Collider2D[] colliders = new Collider2D[1];
    public bool IsGrounded
    {
        get//属性
        {
            return Physics2D.OverlapBoxNonAlloc(transform.position, detectionSize,
                detectionAngle, colliders, groundLayer) != 0;
        }
    }//NonAlloc：不分配内存的检测，若频繁进行推荐使用，会把碰撞体存进colliders，返回的是碰撞体的数量
}
```
判断出玩家是否在地面上后，就可以进行Jump、Fall等状态的切换。矩形的宽度最好和玩家碰撞体的宽度一样，不然会出现以Fall的状态卡在空中的bug。
*属性：属性由两个访问器组成：`get` 访问器和 `set` 访问器。`get` 访问器用于返回属性的值，`set` 访问器用于设置属性的值。属性可以是只读的（只有 `get` 访问器）、只写的（只有 `set` 访问器）或读写的（同时有 `get` 和 `set` 访问器）。*
有一些有落地动画的项目可能会采取更高的矩形判断或者球体判断以提前播放动画。

### 第七部分：优化操作体验
#### 土狼时间
在玩家跑着离开平台的一定时间内暂时不下落，放宽输入的判定。
把土狼时间状态写成一个新的状态，名为CoyoteTime。若玩家处于Run状态且不在地上，进入CoyoteTime。
进入CoyoteTime后，关闭重力；在PlayerState中添加一个float stateStartTime，用以检测是否过了时间，若超出时间则进入Fall状态（退出时开启重力）。
#### 跳跃输入缓冲
防止玩家在还没落地时就按下跳跃导致没有跳跃成功，产生挫败感，本质上也是放宽判定。若玩家进入Fall状态后且不能触发AirJump，再次按下跳跃键时使用bool HasJumpInputBuffer保留一段时间的输入缓冲。若掉到地面进入Idle状态后有输入缓冲，则立刻跳起。
输入缓冲属于输入处理器，所以放在PlayerInput里面定义。然后是理清逻辑：
* 进入Jump时，清除输入缓冲。
* Fall且不能AirJump时，进行输入缓冲。
* Idle且有输入缓冲时，跳。
* 在PlayerInput里弄一个协程JumpInputBufferCoroutine()，这个协程会进行输入缓冲后等待给定时间，然后清除输入缓冲。
* 为了避免同时有多个协程，封装一个新的方法如下：
```c#
    public void SetJumpInputBufferTimer()
    {
        StopCoroutine(nameof(JumpInputBufferCoroutine));
        StartCoroutine(nameof(JumpInputBufferCoroutine));
    }
```

用调用这个方法代替前面说的“进行输入缓冲”操作。
* 另外，本项目为了实现小跳大跳功能，在玩家松开跳跃键时会立刻进入Fall，所以松开跳跃键时自然也应该清除输入缓冲。采用委托的写法：
```c#
private void OnEnable()
{
    playerInputActions.Gameplay.Jump.canceled += delegate
    {
        HasJumpInputBuffer = false;
    };
}
```
*`+= delegate { ... }` 语法用于将一个匿名方法（delegate）添加到 `canceled` 事件的事件处理程序列表中。这段代码的作用是在脚本启用时（即 `OnEnable` 方法调用时），将一个匿名方法注册到 `playerInputActions.Gameplay.Jump.canceled` 事件。当跳跃动作取消时（例如，玩家松开跳跃键），该匿名方法会被调用，从而将 `HasJumpInputBuffer` 设置为 `false`。*

### 第八部分：视差效果+无限循环背景
无限循环背景：开了个数组存背景图，检测到相机边越界就瞬移然后轮换数组位置。
视差效果：给背景图存一个父物体，直接控制父物体，移动量是相机移动量的一半。
比较大的坑点在于父物体的坐标会对子物体造成影响。要把坐标和scale都调好。
相机真正宽度的计算法：
```c#
var camera = mainCameraObject.GetComponent<Camera>();
float orthographicSize = camera.orthographicSize;
float aspectRatio = Screen.width * 1.0f / Screen.height; 
float cameraHeight = orthographicSize * 2;
mapWidth = fbgs[0].GetComponent<SpriteRenderer>().sprite.bounds.size.x
    *fbgs[0].transform.localScale.x;
cameraWidth = cameraHeight * aspectRatio;
```