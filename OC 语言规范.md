# ****代码风格规范****



\### A. 代码风格

\> * [*Apple代码风格规范*](  https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)

\> * [*Google代码风格规范*](https://google.github.io/styleguide/objcguide.xml)



\1. 缩进与换行

​    \* 代码缩进4个空格，不要使用Tab(Xcode可以设置)

​    \* 大括号跟在行末 (大括号前加一个空格)

​    \* 只有一行的逻辑也用{}包起来

​    \* 多行判断逻辑的 &&、||放在行首，与第一行判断条件对齐

​    \```objc

​    if (aaaaa

​        && bbb

​        && ccc)

​    \```

\2. 每行最长100个字符(Xcode设置Preferences > Text Editing > Page guide at column: 100)

\3. 方法声明、定义中-/+后面跟一个空格，后面除参数之间外没有其他空格

​    \```objc

​    \- (void)aaa:(NSString *)str param:(int)x {

​        int *a, b;

​    }

​    \```

\4. 成员变量时@public @private不缩进(Xcode默认)

\5. protocol放在<>中时与前面的类型之间不要有空格

​    \```objc

​    @interface MyProtocoledClass : NSObject<NSWindowDelegate> {

​    @private

​        id<MyFancyDelegate> _delegate;

​    }

​    \- (void)setDelegate:(id<MyFancyDelegate>)aDelegate;

​    @end

​    \```

\6. 较大的block应该独立出来成为一个局部变量，而不是直接inline写在调用方法参数中，如20行以上

\7. NSArray/NSDictionary的literal前后加空格（整体内容前后各有一个空格）：

​    \```objc

​    NSArray *array = @[ [foo description], @"Another String", [bar description] ];

​    NSDictionary *dict = @{ NSForegroundColorAttributeName : [NSColor redColor] };

​    NSArray *array = @[

​        @"This",

​        @"is",

​        @"an",

​        @"array"

​    ];

​    \```

​    多行情况下冒号前面只有一个空格，冒号后面可选择一个或多个空格(为了多行对齐)

​    \```objc

​    NSDictionary* option1 = @{

​        NSFontAttributeName : [NSFont fontWithName:@"Helvetica-Bold" size:12],

​        NSForegroundColorAttributeName : fontColor

​    };

​    NSDictionary* option2 = @{

​        NSFontAttributeName :            [NSFont fontWithName:@"Arial" size:12],

​        NSForegroundColorAttributeName : fontColor

​    };

​    \```

\8. 文件扩展名

​    | 文件后缀名 | 文件类型 |

​    |:---|---|

​    | .h   | C/C++/Objective-C header file     |

​    | .m   | Objective-C implementation file   |

​    | .mm  | Objective-C++ implementation file |

​    | .cpp | Pure C++ implementation file      |

​    | .c   | C implementation file             |

\9. 空行：同一代码块内为保持逻辑分开可在中间空一行分隔，不要空多行，方法末尾不要空行

\10. 代码注释：

​    \* 尽量解释清楚某段代码为什么要这么做，特别是在有tricky或复杂的判断逻辑的地方

​    \* 头文件中声明的方法、接口强制加注释

​        \* 接口定义应该明确说明输入、输出及功能，复杂的接口组合应有相应的调用代码示例

​        \* 实现文件中有必要的地方加注释

​        \* 功能相对独立、提供服务给别的模块的类需要加更丰富的注释

​    \* 用#pragma mark组织类的内容，方便其他人阅读

​    \* 多行注释格式不作明确要求

​    \* 同行注释建议在代码后面空两格

​    \```objc

​        @property (nonatomic, strong) UIView *view;  //注释"//"前空两空格

​    \```

\11. 避免类的头文件中引入其他头文件，可以用前向声明的使用前向声明

​    \* 注：实现文件里系统头文件与SGI头文件分开先后顺序，可按模块组织

\12. 尽量使用static const来定义常量而不是#define，对于需要对外公开名字而隐藏具体值的应使用extern的方式定义

​    \* 例1：.m中定义 `static const NSTimeInterval kAnimationDuration = 0.3`

​    \* 例2：.h中定义 `extern NSString *const EOCStringConstant;`

​    \* 例3：.m中定义 `NSString *const EOCStringConstant = @"VALUE";`

\13. 尽量解决新代码的编译warning（现有的warning在4.5版本统一清一部分）	

​    \* 例：int与unsigned int的类型转换等

​    \* 可以使用AppCode等工具来查更多的warning和使用其refactoring的功能

​        \* 如：找未使用方法等

​    \* 方法名、属性名等注意不要跟基类重名（override除外）

\14. 成员变量一般应在.m中声明或由属性自动生成，若要在.h中声明，应加@private，

​    \* 新加代码.h里就不要定义成员变量了 

​    \* 自定义成员变量加”_”

​    \* 自定义方法前不加”_”

​    \* 局部变量、成员变量名、方法名首字母小写，类名首字母大写

​    \* Objective-C代码中的变量名一般不要使用”_”分隔

\15. Designated initializer: 用注释或其他方式明确本类的designated initializer，方便别人重载

​    \* 在写子类的init...方法时，一定要重载父类的designated initializer

\16. 重载NSObject方法时应将实现写到@implementation的上部，包括但不限于init..., copyWithZone:, dealloc

​    \* Singleton的初始化方法也放@implementation的上部

\17. 不用显示地将类成员变量初始化为nil或0，alloc方法除了分配内存，还自动初始化了所有成员变量，此条同时适用于ARC和MRC

​    \* BOOL型变量应该显式初始化

​    \* 局部变量应该显式初始化

\18. 避免使用+new方法，使用alloc&init

\19. Objective-C/Objective-C++的头文件使用#import来包含，C/C++的使用#include

\20. init和dealloc方法中使用成员变量而不是属性，避免在init里加太复杂逻辑

​    \```objc

​    \- (instancetype)init {

​        self = [super init];

​        if (self) {

​            _bar = [[NSMutableString alloc] init];  // good

​        }

​        return self;

​    }



​    \- (void)dealloc {

​        [_bar release];                           // good

​        [super dealloc];

​    }

​    \```

​    \* 对于属性和成员变量的选择可依以下原则：

​        \* 只有类自己内部使用的情况：若后续可能有延迟加载等逻辑，优先选用属性；否则可使用成员变量或属性，不强制统一

​        \* 公开给别的模块调用的数据应使用属性

\21. dealloc中的dealloc的顺序与声明顺序保持一致

\22. 接受NSString类型参数的Setters应该copy这个字符串，防止被别的模块修改

​    \* 可以用copy属性的地方优先使用copy属性

​    \```objc

​    \- (void)setFoo:(NSString *)aFoo {

​        [_foo autorelease];

​        _foo = [aFoo copy];

​    }

​    \```

\23. 根据实际需求，读取CGRect相关属性应该使用CGRectGetMinX(frame)等方法，而不是直接使用frame.origin.x

​    \* 注：在CGGeometry文档中所有的函数，接受CGRect结构体作为输入，在计算它们结果时隐式地标准化这些rectangles

\24. BOOL变量：

​    \* 不要直接比较一个BOOL变量是否等于YES或NO

​        \* 因为YES被定义为1，而`BOOL`是一个`char`型，所以 `11111110 != YES`

​    \* BOOL变量不要判断相等，

​        \* `aFlag != bFlag` 应写成 `(!!aFlag ^ !!bFlag)`, 定义一个工具宏

​        \* 或者使用 `if ((aFlag && !bFlag) || (!aFlag && bFlag)) `

\25. `init`方法应该返回`instancetype`

\26. ARC VS. MRC

​    \* 新加的类应使用ARC，若有性能优化等其他原因需要使用MRC的应先做review

\27. 异常/错误：

​    \* 尽量少使用异常，ARC不是异常安全的，只在严重错误的情况下才使用异常

​        \* 注：对于第三方c++代码的exception，64位平台上的进程可以使用c++风格的catch或oc的`@catch(...)`来catch

​    \* 合理使用NSError *类，填入适当的Domain, code, user info等，对可能产生错误的方法返回BOOL值，error用`NSError **`类型参数传出           

​        \* 调用方调用之后应判断是否成功，按业务逻辑决定是否展示error信息

​            \* 反例：`+[SGISynchedLexiconHelper _exportLexicons:withFileName:]`返回`void`

\28. 宏定义：

​    \* `#define`常量和局部常量(const local variable)等建议以小写字母k开头，enum可按业务逻辑及相关上下文自行决定是否以k开头

​    \* `#define`的宏定义在其值为表达式的情况下要求用小括号包围，如：`#define kAAA (123+234）`

​        \* 非表达式的情况不作强制要求，建议在符合需求的情况下加小括号

\### B. 键盘常见问题

\1. 内存相关：

​    1) 频繁申请临时对象的地方使用`@autoreleasepool`，减少内存峰值

​    2) 避免循环引用，block中注意使用weak reference，理清业务逻辑上的reference关系，显式定义retain关系

​    3) 键盘中避免使用`[UIImage imageNamed:]`，`NSDateFormatter`等已知性能有问题的方法

​    4) 较大的图片或数据文件考虑使用文件映射的方式来读写

​    5) 新功能实现完成后应使用instrument验证没有内存泄露，且峰值内存保持在合理区间

​    6) NSTimer会retain其target，确保timer在合适的地方被停止并置空

​    7) 后台线程在申请较大块内存（或频繁申请内存）时可以先向主线程发个消息，等消息返回后再进行分配，防止主线程正在处理memory warning时后台还在申请内存

\2. 多线程相关：

​    1) 后台任务尽量避免直接dispatch_async到global_queue上或者新建NSThread，统一使用SGILaunchCenter来进行管理

​    2) 尽量避免使用无锁同步机制，多线程共享同一内存数据的情况下优先考虑使用锁进行保护

​        \* 无锁同步机制涉及volatile+memory_barrier的使用，不易保证正确性且给其他人阅读代码带来困难

​    3) 禁止使用OSSpinlock锁，会导致优先级反转问题

​    4) 使用synchronized时应注意其对象不能为nil，否则将不能起到锁的作用

​    5) Singleton初始化应使用dispatch_once来保证线程安全

\3. 键盘调起速度相关：

​    1) 尽量做到类和数据的懒加载，用的时候再加载所需资源

​    2) 尽量不要阻塞主线程

​    3) 慎用Autolayout，性能问题

​    4) 慎用Swift，性能问题，而且导致包大小增加

​    5) 慎用[NSString sizeWithAttributes:]，性能问题

​    6) 避免使用category，过多的category会使程序加载变慢

​        \* 考虑把一些影响范围小的category去掉

​            \* 例：`NSString(CrashReport), NSMutableString(Helper), UILabel (dynamicSizeMe), UIView (DPUtils)`等

​    7) 尽量避免使用`+load`方法和static构造函数，若需使用，也要注意与其他类之间的先后顺序，不要有依赖

​    8) 严格控制在调起过程中增加复杂、耗时逻辑，包括文件I/O、网络请求等，调起过程主要包括：

​        \* `KeyboardViewController`的`init`方法到`viewWillAppear:`之间

​            \* `viewWillAppear:` & `viewDidAppear:`之间也不要有太多复杂逻辑，可能会导致键盘高度变化卡顿

​        \* 键盘调起时即显示的各级view的`layout`及`drawRect`等布局和渲染方法

\4. 自测：

​    1) 测试模块性能或测试比较tricky的实现时应设置工程的`Optimization Level`，使之与Release配置一致

​    2) 尽量为代码中较复杂的逻辑写单元测试，一些边界条件的bug也可以用单元测试的test case来记录、验证

​    3）新功能使用`ThreadSanitizer`, `AddressSanitizer`等工具来进行自测以帮助发现潜在问题

\5. 网络请求：

​    1) 网络请求尽量使用异步请求，同步请求会阻塞线程，即使在后台任务调度系统中，也会对后续任务造成时序上的影响

​        \* 使用NSURLSession

​            \* 现存NSURLConnection修改为session //看4.6排期

\6. 单例：

​    1) 避免单例初始化时的循环引用（1初始化时引用2，2初始化时又引用1）情况，把相关逻辑抽出来，必要时使用caller/callee.py工具来验证

\### C. 设计原则

\1. SOLID原则:

​    \* Single Responsibility Principle

​        \* A module should have a single responsibility, and that responsibility should be entirely encapsulated by the module

​        \* 复杂的类尽量拆分，相对独立的功能分到别的类里去做，每个类有比较明确的职责定义

​            \* 例：KeyboardViewController、SGIMainView、SGInputManager等类都承担了太多的责任

​            \* 例：SGISetting类中的设置项太多，而且平铺，考虑按层级、模块拆分，将设置项与其存储实现分离，将相同的实现合并

​    \* Open Closed Principle

​        \* A module should be open for extension but closed for modification

​            \* 例：皮肤模块新增图片时得改多处代码

​            \* 例：SGISetting增加设置也得修改

​    \* Liskov's Substitution Principle

​        \* Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.

​    \* Interface Segregation Principle

​        \* Many client-specific interfaces are better than one general-purpose interface.

​        \* 按不同的功能分出不同的接口

​            \* 例：SGInputManager/KeyboardViewController

​    \* Dependency Inversion Principle

​        \* Abstractions should not depend on details. Details should depend on abstractions.

​        \* 用方依赖于特定的接口而不是直接引用具体的万能类

​            \* 例：KeyboardViewController, SGInputManager等

\2. DRY原则: Don't Repeat Yourself

​    \* 避免拷贝源代码，有可以复用的地方应该先做等价重构再进行复用

​        \* 例：Autopingback VS. UnitedPingback代码基本相同

​    \* 避免重复实现已有的功能

​        \* 例：从hex得到UIColor的实现有多处

\### D. 代码结构

\1. MVC

​    \* 模型、视图、控制器逻辑分开，尽量保证各自的独立性

​        \* 模型和视图分开：尽量避免在模型中引入对现有视图的假设，模型与视图之间尽量保持一个抽象的接口，模型不应该知道视图

​        \* 控制器与模型、视图分开：业务逻辑不要写在视图中，写在控制器中；控制器不要知道太多视图的东西

​    \*  用`@protocol`来定义回调API

​        \* 不是必须实现的方法使用@optional定义

\2. Delegate模式

​    \* _delegate应该是weak的，避免retain cycle

\3. 模块化+分层设计

​    \* 尽量避免层级间的反向依赖，适当的情况下考虑使用delegate+protocol或notification等方式来解耦

​        \* 例：太多地方知道KeyboardViewController了

\---

\> #### ****提交代码之前开发者应该保证代码符合以上原则，代码的review者若发现不符合某条标准的情况，应向提交者提出并跟进修改。****