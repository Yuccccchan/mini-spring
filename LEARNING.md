# mini-spring 学习记录

## 学习方式

- 核心学习目标：理解 Spring 为什么这样设计，并逐步连接 Spring Boot 如何使用这些机制；mini-spring 是理解机制的教学模型，不要求先读完整仓库再接触 Boot。
- 后续每节围绕四个问题消化：原有方案有什么具体问题；新增设计如何解决；插入现有执行流程哪里；在真实 Spring/Boot 中对应什么场景。真实实现的对应需核实，不能把 mini-spring 简化代码当成真实源码。
- 每节结束使用用户自己的简短解释和一个有区分度的预测确认理解；不以测试通过、回复 ok 或记住类名代替理解，不要求反复逐字段观察已掌握的相同机制。
- 设计说明区分功能必须的部分与为扩展/复用作出的选择，比较合并或直接调用的替代方式及其代价。优先消化主流程和对象关系，继承细节按需定位。
- 每次只解决一个问题，通常用 15～25 分钟。
- 按“预测 → 运行验证 → 自己解释”的顺序推进。
- 按理解程度推进；中断后从本文件的下一步恢复。
- 引导时一次只给一个小任务，不提前展开全部分支。
- 新增 XML 配置先追踪属性文本如何读取、如何存入 BeanDefinition，再看工厂如何使用；不直接从配置跳到反射调用。用户明确要求补齐 init-method 的解析步骤。
- 进入新阶段先明确要切换的分支和测试入口，再引导用户操作；不在未切换、未定位代码时连续提出新阶段概念题。用户再次明确要求一步一步学习。
- 引入新模块前先说明：已知代码的哪个问题需要它、它接在现有流程的哪里，再解释新名词，最后安排实验。用户明确反馈资源加载阶段缺少这一过渡，需减小概念跨度。
- 用户再次明确要求每节都先讲为什么要做这个功能：现有问题、新功能解决什么、接在已学流程哪里；解释后再安排一个操作。
- 不要求逐个掌握所有继承类和接口；先沿实际调用理解职责。解释继承时明确同一个工厂对象包含各层状态，父类 private 字段通过其方法访问。

## 当前阶段

- 当前本地分支：`package-scan`。
- 入口：PackageScanTest.testScanPackage()。
- AOP 与属性占位符核心小节完成；当前学习自动发现组件并注册定义。
- 已完成第一轮：理解简单容器 → 理解 BeanDefinition → 看懂容器创建对象。
- 已完成第二轮：实例化策略 → 属性赋值 → Bean 依赖注入。
- 第三轮目标：资源加载 → XML 定义读取 → 容器扩展机制 → ApplicationContext。

## 当前进度

### 2026-09-30：对象引用与创建时机

- 已正确预测：同一名称连续获取两次，`a == b` 为 `true`。
- 已识别：此阶段由调用方创建 HelloService，容器保存并返回它。
- 需要精确表述：引用类型的 `==` 判断是否指向同一个对象，不是比较可观察的物理内存地址。
- 创建时机：`new HelloService()` 作为实参先执行，随后才进入 `registerBean()`。
- 运行验证：用户已确认本轮验证测试通过。

### 2026-09-30：同名注册

- 已正确预测：第二次同名注册后，`result == first` 为 `false`，`result == second` 为 `true`。
- 已解释原因：`Map.put()` 在键相同时替换该键对应的值。
- 精确表述：BeanFactory 内部使用 HashMap；替换的是容器中保存的引用，first 对象不会因此立即消失，已有引用仍指向它。
- 运行验证：用户已确认三个断言通过，简单容器阶段完成。

### 2026-09-30：类型信息与实例

- 用户已正确判断：注册 `HelloService.class` 后，当前 getBean() 返回 Class 对象，而非 HelloService 实例。
- 用户主动提出：为什么要注册类型信息，以及 Class 对象到底是什么。
- 本轮解释：Class 对象由 JVM 管理，表示运行时类型，可获取类型名称、方法、字段和构造方法等信息；传入类型信息为容器通过反射创建实例提供依据。
- 下一分支用 BeanDefinition 保存 beanClass，再在 doCreateBean() 中创建实例。

### 2026-10-01：实例与类型的比较

- 用户给出正确结果：`first == second` 为 false，`first.getClass() == second.getClass()` 为 true。
- 原因：两次 new 创建不同实例；这两个实例的运行时类型相同，对应同一个 Class 对象。
- 当前可进入 BeanDefinition 阶段。

### 2026-10-01：BeanDefinition 分支测试

- 已确认当前分支为 `bean-definition-and-bean-definition-registry`。
- 用户确认 `BeanDefinitionAndBeanDefinitionRegistryTest` 测试通过。
- 实例创建流程尚待用户调试观察，不能仅根据测试通过视为已理解。

### 2026-10-01：定位反射创建

- 用户正确指出：beanClass 表示 HelloService 类型，`bean = beanClass.newInstance()` 执行后 bean 成为实例。
- 精确表述：beanClass 是表示 HelloService 类型的 Class 对象；bean 是 HelloService 实例。
- 本轮解释：Class.newInstance() 通过反射调用所表示类型的无参构造方法，创建并返回新实例；此分支 HelloService 未显式声明构造方法，编译器提供默认无参构造方法。
- 当前已定位对象创建位置；用户询问原理，继续结合反射和构造方法解释。

### 2026-10-04：单例缓存预测

- 用户正确判断：去掉 addSingleton 后，两次 getBean() 不会返回同一对象。
- 用户解释：缓存查询得到 null，因此每次都进入 createBean()。
- 补充前提：使用新建容器，且没有其他代码预先注册该单例。
- 已理解单例复用依赖容器缓存。

### 2026-10-04：单例缓存验证与引用类型问题

- 用户确认实验结果符合 true → false → true；已检查 addSingleton 在当前源码中恢复。
- 用户提出：second 声明为 Object，helloService 声明为 HelloService 且经过强转，为何 == 仍为 true？
- 本轮解释：声明类型影响编译期可访问成员；两个引用指向的实际对象都是同一个 HelloService。引用类型强转检查兼容性，不创建或复制对象；错误强转可能抛出 ClassCastException。
- getBean() 返回 Object 是为了容纳不同类型的 Bean；调用方强转后可使用 HelloService 的方法。

### 2026-10-04：声明类型与强转理解

- 用户正确解释：Object 是类的共同父类，因此 Object 引用可指向 HelloService 实例；通过 Object 引用不能直接调用 sayHello()。
- 已解释强转使该表达式按 HelloService 类型参与编译检查，不会创建新对象。

### 2026-10-04：定义注册表与实例缓存

- 用户正确判断：注册定义后、首次 getBean 前，beanDefinitionMap 保存 HelloService 的 BeanDefinition，singletonObjects 为空。
- 已区分创建说明与业务实例；注册 BeanDefinition 不创建 HelloService 实例。
- 解释：首次创建成功后，BeanDefinition 仍在定义表中，HelloService 实例被加入单例缓存。

### 2026-10-04：单例按名称缓存

- 用户正确预测 helloA 与 helloB 的实例比较为 false。
- 用户正确解释：缓存使用 Bean 名称作为键，helloB 查询不到 helloA 名下的实例，因此创建并缓存自己的实例。
- 精确表述：helloB 是首次创建，不替换 helloA 的实例；两个名称各自保存一个实例。
- 用户确认边界实验输出 false，BeanDefinition 阶段完成。

### 2026-10-04：实例化策略代码已就绪

- 用户确认测试通过；检查当前工作区已包含 InstantiationStrategy 与 SimpleInstantiationStrategy。
- 当前实际分支仍叫 instantiation-strategy，按已核对的代码继续学习。
- 本轮说明：工厂通过 InstantiationStrategy 接口委托对象创建，默认实现是 SimpleInstantiationStrategy；缓存仍由工厂管理。

### 2026-10-04：策略替换与多态

- 用户正确判断：setter 换入新实现后，会执行新实现的方法，并识别出这是多态。
- 用户指出 getter 返回 instantiationStrategy；补充关键原因是 setter 替换字段引用，运行时根据引用所指实际对象分派方法。

### 2026-10-04：自定义策略实验通过

- 用户确认：自定义策略日志打印一次，两个引用 == 为 true。
- 已验证策略替换生效，缓存命中时不重复实例化。
- 用户表示不熟悉 lambda，先补充 Java 知识再继续容器学习。

### 2026-10-04：lambda 参数名与 getter

- 用户正确解释：definition 是 lambda 参数名，同步改名不会改变功能。
- 用户提出：为什么通过 getInstantiationStrategy() 而非直接读取 instantiationStrategy 字段？
- 已核对 getter 仅返回字段；在当前实现且未重写 getter 时，两种调用效果相同，多态不依赖 getter。
- getter 提供统一访问入口，并允许子类重写获取策略的行为；这是扩展空间，不是当前实例化的必要步骤。

### 2026-10-04：private 字段与重写 getter

- 用户询问：父类策略字段是 private，子类如何使用另一种策略？
- 解释：父类私有状态存在于子类实例中，但子类代码不能直接访问；可调用继承的 public getter/setter，由父类方法读写字段。
- 另一条路径：子类声明自己的 ownStrategy 并重写 public getter；父类 createBeanInstance() 调用 this.getInstantiationStrategy() 时通过动态分派执行子类 getter。
- 示例继承 DefaultListableBeanFactory（其父类是 AbstractAutowireCapableBeanFactory），以复用已实现的定义注册逻辑。
- 对照：父类方法直接读取私有字段时，使用父类字段；通过可重写 getter 获取时，可返回子类自己的策略。
- 之前直接字段访问实验尚未确认，不要求在未理解此问题时继续。

### 2026-10-04：Java 补充理解确认

- 用户确认理解 private 字段、公开访问方法和子类重写 getter 的区别。
- 子类 getter 示例为解释性示例，未确认运行，不作为阶段必需实验。

### 2026-10-04：CGLIB 实验通过，实例化策略阶段完成

- 用户提供实际输出：HelloService$$EnhancerByCGLIB$$41dffb47；父类 HelloService；instanceof 为 true；同名获取 == 为 true；sayHello() 输出 hello。
- 已验证：CGLIB 创建 HelloService 的子类实例，继承关系允许按 HelloService 使用；缓存与调用流程继续复用。
- 实例化策略阶段完成：理解接口委托、多态、策略替换，以及普通反射创建与动态子类创建的区别。

### 2026-10-04：属性赋值测试通过

- 当前已在 populate-bean-with-property-values 分支，用户确认新测试通过。
- 已核对 Person 没有显式构造方法或字段初始值，创建后 name 为 null、age 为 0；属性赋值由后续 applyPropertyValues() 完成。

### 2026-10-04：属性赋值调试完成

- 用户观察：赋值前 name=null、age=0，赋值后为配置值 derek、18。
- 用户正确定位：BeanUtil.setFieldValue(bean, name, value) 写入字段。
- 已区分实例化与配置属性赋值；PropertyValues 保存多个字段名和值，applyPropertyValues 逐项应用。

### 2026-10-04：private 字段反射实验完成

- 用户确认：开启反射访问后输出 ivy；注释 setAccessible(true) 后出现 private 成员访问错误。
- 已验证：获取 Field 描述不等于获得字段读写权限；setAccessible(true) 使该 Field 的反射操作跳过访问检查，字段声明本身仍是 private。
- 属性赋值阶段核心内容完成；用户随后确认恢复后的测试通过。

### 2026-10-04：属性赋值阶段完成

- 用户确认恢复后测试通过，当前工作区仅 LEARNING.md 尚未跟踪。
- 已掌握：属性配置、创建后填充、反射字段写入及 private 访问检查。

### 2026-10-04：Bean 依赖注入测试通过

- 当前已在 populate-bean-with-bean 分支；用户确认 testPopulateBeanWithBean() 通过。
- 测试配置用 BeanReference("car") 描述 Person.car 的依赖，实际解析过程待观察。

### 2026-10-04：依赖解析观察

- 用户观察：解析前 value 指向 BeanReference，执行嵌套 getBean 后指向 Car，包含配置的品牌值（测试配置为 "porsche"）。
- 解释：这里是变量重新赋值，并非强转或 BeanReference 对象自身变成 Car。
- 后续 BeanUtil.setFieldValue(bean, name, value) 将 Car 引用写入 Person.car。

### 2026-10-04：创建顺序需要进一步区分

- 用户回答“先创建 car 的 bean 之后再进行装配依赖”，正确指出 Car 需准备好才能注入，但尚未明确 Person 已先实例化。
- 当前测试从 getBean("person") 开始；代码预期顺序为 A-person → A-car → B-car → B-person。
- 说明：Person 先创建实例，填充 car 属性时才嵌套创建 Car；Car 先完成并缓存，然后回到 Person 填充依赖并缓存。

### 2026-10-04：Bean 依赖注入阶段完成

- 用户确认第一次实例化断点的 beanName 为 person，并表示理解先实例化 Person、先完成 Car 装配的区别。
- Bean 依赖注入阶段完成；第二轮三项目标全部完成。

### 2026-10-04：classpath 资源读取通过

- 用户确认 testClassPathResource() 达成预期，输出 hello world 并通过断言。
- 当前实际分支为 resource-and-resource-loader，已核对实现与本阶段一致。

### 2026-10-04：资源加载引导需要补足过渡

- 用户反馈 Resource 和 classpath 的引入跨度过大，尚未理解为什么进入资源加载模块。
- 读取测试虽通过，不能视为已理解资源抽象；先补学习动机和概念连接。
- 暂停此前的文件不存在实验，不要求用户继续验证异常。

### 2026-10-04：配置文件与 BeanDefinition 的连接已理解

- 用户正确回答：先把配置内容整理成 BeanDefinition；读取配置不等于创建 Person 实例。
- 继续遵循每次一个概念，先解释 classpath，再讨论 Resource 抽象。

### 2026-10-04：classpath 仍需具体解释

- 用户表示仍不清楚为何 test-classes 是根目录，以及查找位置的集合在当前项目中是什么。
- 已核实实际文件：target/classes 下有 ClassPathResource.class；target/test-classes 下有测试类 .class 和 hello.txt。
- 需要明确：test-classes 名称没有 JVM 特殊含义，是 Maven 默认测试输出目录；测试启动器将输出目录与依赖 JAR 加入 classpath 才使其成为查找起点。
- 当前测试进程的确切 classpath 尚未读取，不把惯常目录列表当作已观测的启动配置。

### 2026-10-04：区分主代码编译与测试准备，降低讲解负担

- 用户反馈内容复杂，并观察到 compile 后只有 target/classes，运行测试后才出现 target/test-classes。
- 用户观察符合当前流程：Maven compile 处理主代码；运行测试前还会准备测试资源并编译测试代码，生成测试输出目录。
- 明确此前“构建时复制”的说法过于笼统，需要说明是测试准备阶段，不能暗示 compile 必然生成 test-classes。
- 取消打印 classpath 的任务，不要求用户继续分析启动配置。

### 2026-10-04：测试准备阶段说明已接受

- 用户回复 ok，继续以小步方式推进，不要求展开 classpath 启动列表。

### 2026-10-04：资源构造方法说明已接受

- 用户回复 ok；已说明构造方法只保存路径，不读取文本内容。

### 2026-10-04：输入流观察已完成

- 用户确认：停在读取内容语句前，stream 为非 null。
- 当前已观察到资源描述对象到输入流的步骤；下一步读取文本。

### 2026-10-04：字符串读取已观察

- 用户反馈 content 是字符串，写作 "hello,world"；已核对源文件与测试输出目录文件均为 "hello world"，断言也使用空格版本，需按实际字符区分。
- 已观察 Resource → InputStream → String 的读取链。

### 2026-10-04：不同资源实现读取验证通过

- 用户确认：磁盘路径返回 FileSystemResource，读取内容保持不变。
- 已实际比较 classpath 与磁盘两种定位方式；两种实现都返回 InputStream，读取代码保持相同。
- 继续以此前的多态知识解释 Resource 统一接口；不追加 URL 实验。

### 2026-10-04：资源加载阶段完成

- 用户确认 XML 读取代码可以通过 Resource 统一接口复用，并指出 loader 选择类型、多态执行对应实现。
- 精确表述：loader 根据位置选择并创建 Resource，多态体现在 getInputStream() 调用各自实现。
- 已掌握资源描述、打开流、读取文本和不同资源来源的统一访问；资源加载阶段完成。

### 2026-10-04：XML 定义测试通过

- 已确认当前分支 xml-file-define-bean；用户确认 testXmlFile() 通过。
- 先解释配置映射：bean.id 对应注册名，bean.class 对应类型信息，property.value 对应普通值，property.ref 对应 BeanReference。

### 2026-10-04：value 与 ref 的区别待巩固

- 用户正确区分 value="car" 是字符串，但同时认为该字符串用于查询 Bean。
- 已核对解析代码：普通 value 原样成为 PropertyValue 的字符串值；仅 ref 被包装为 BeanReference，之后填充属性时才按名称调用 getBean。
- 需要纠正普通字符串不会因为内容是 car 就自动查询容器。

### 2026-10-04：ref 含义理解正确，普通值需对照具体配置

- 用户正确说明只有 ref="car" 才按名称查找依赖，但对假设 value="car" 的例子回答返回 derek。
- 需区分当前文件 value="derek" 与判断题假设 value="car"：普通值按该配置中的文本保存，不固定为 derek。
- 当前用两个配置和各自结果对照解释，不增加新名词或立即安排文件修改。

### 2026-10-04：普通值文本映射说明已接受

- 用户回复 ok；接下来通过观察解析结果确认概念。

### 2026-10-04：属性转换理解正确，XML 对象结构需补充

- 用户正确解释普通 value 与 ref 包装为 BeanReference 的区别。
- 明确 nameAttribute 来自 property 标签的 name 属性（表示业务对象字段名）。
- 用户表示不清楚 Document、Element、NodeList；先补 XML 的对象表示结构，不继续要求断点结果。

### 2026-10-04：XML 对象结构说明已接受

- 用户回复 ok；已说明 Document、Element、NodeList 与当前 XML 层次的对应关系。

### 2026-10-04：bean 属性字符串观察完成

- 用户确认 id=person，className=org.springframework.test.ioc.bean.Person。
- 已观察 Element.getAttribute() 返回 XML 属性文本。

### 2026-10-04：Class.forName 结果观察完成

- 用户确认 clazz 表示 Person 类型。
- 已连接 XML 中类名字符串与 BeanDefinition 所需的 Class 对象。

### 2026-10-04：XML Bean 定义阶段完成

- 用户确认加载后定义表有 car、person，单例表为空，并正确解释容器得到的是定义。
- XML 阶段完成：理解文件读取、XML 对象结构、属性文本、类名转 Class、普通值与 BeanReference、定义注册与实例创建的分离。

### 2026-10-04：修改定义的需求理解正确，整理工厂职责

- 用户正确选择修改 BeanDefinition 的属性配置。
- 用户询问两张 Map 的归属和是否需要逐个掌握所有类接口。
- 已核对继承链：DefaultListableBeanFactory → AbstractAutowireCapableBeanFactory → AbstractBeanFactory → DefaultSingletonBeanRegistry。
- 职责：定义表和注册；实例化与填充；getBean 获取流程；单例缓存。
- 两张 Map 分别为 DefaultListableBeanFactory 的 private beanDefinitionMap，以及 DefaultSingletonBeanRegistry 的 private singletonObjects；后者经 getSingleton/addSingleton 访问。
- 同一个 DefaultListableBeanFactory 实例承载这些状态和方法，不是创建四个独立工厂对象。

### 2026-10-04：准备引入 BeanFactoryPostProcessor

- 用户回复 ok，继续按实际需求学习一个扩展入口。
- 已检查对应远程分支：接口提供 postProcessBeanFactory(beanFactory)，测试在 loadBeanDefinitions 后、getBean 前手动调用。
- 示例处理器取得 person 的 BeanDefinition，并调用 addPropertyValue(new PropertyValue("name", "ivy"))。
- 后续已核对本分支 PropertyValues：同名项通过 List.set 替换，未重名才追加；此前未检查该分支具体实现而记为追加的描述已纠正。

### 2026-10-04：定义处理器测试通过

- 用户确认最终 Person.name=ivy；当前实际分支为 bean-factory-post-processor-and-bean-post-processor。
- 已核对当前同名属性处理是替换配置对象，而不是保存两份 name。

### 2026-10-04：BeanFactoryPostProcessor 小节完成

- 用户观察：定义中 name 修改为 ivy，singletonObjects 仍为空。
- 已确认 BeanFactoryPostProcessor 修改定义，并未创建 Person，定义处理小节完成。

### 2026-10-04：BeanPostProcessor 实例处理测试通过

- 用户确认实例处理测试与预期一致，Car.brand=lamborghini。
- 示例处理器直接调用 Car.setBrand()，后续比较定义配置是否保持不变。

### 2026-10-04：定义与实例处理的区别确认

- 用户确认 Car.brand 已改变，定义中的品牌配置仍保持原值。
- 已理解 BeanFactoryPostProcessor 修改定义、BeanPostProcessor 处理实际实例的区别。

### 2026-10-04：初始化前后回调顺序已验证

- 用户提供 testBeanPostProcessor() 输出：前置处理 → car 初始化占位日志 → 后置处理，最后 Car.brand=lamborghini。
- 已验证当前回调执行顺序；invokeInitMethods() 仍只打印占位日志。
- 下一步用一个判断确认：前置处理收到的 Car 已完成实例化和属性填充；BeforeInitialization 不表示对象创建前。

### 2026-10-04：初始化回调位置理解确认，容器扩展小节完成

- 用户确认前置处理执行前，XML 中的属性已经填入实例。
- 已理解 BeforeInitialization 是初始化前；当前流程为实例化 → 属性填充 → 前置处理 → 初始化 → 后置处理 → 缓存。
- BeanFactoryPostProcessor 与 BeanPostProcessor 小节完成；当前初始化仍是占位实现。

### 2026-10-04：手动注册处理器的作用理解正确

- 用户正确预测：新建工厂中省略 addBeanPostProcessor()，Car.brand 保持 XML 配置的 porsche。
- 已理解创建处理器对象与把它加入工厂处理器列表是两步；未注册就不会参与该工厂的回调。
- 已读取本地 application-context 分支：XML 声明两类处理器，refresh() 负责加载定义、调用定义处理器、注册实例处理器、提前实例化单例。
- 当前未切换分支，未修改用户实验代码。

### 2026-10-04：ApplicationContext 引入需放慢，动机说明已接受

- 用户反馈突然引入 ApplicationContext 不易理解；暂停此前关于注册时机的判断题，不能视为已回答。
- 已从当前测试的手动准备步骤解释：工厂负责创建 Bean，ApplicationContext 负责组织读取定义、调用或注册处理器等启动准备，内部继续使用已有工厂。
- 用户回复 ok；接下来先用 XML 中声明处理器的具体配置衔接，不展开新类的继承结构。

### 2026-10-04：处理器的 XML 定义理解正确

- 用户正确判断：XML 读取器解析 CustomerBeanPostProcessor 的 bean 标签后，得到处理器的 BeanDefinition，而非处理器实例。
- 下一步连接两种注册：定义进入 beanDefinitionMap；取得处理器实例后，addBeanPostProcessor() 将其加入回调列表。

### 2026-10-04：恢复分支与测试入口优先的小步引导

- 用户指出讲解跳过了分支切换和代码定位，不接受连续概念题的推进方式。
- 之前“Car 缓存后注册处理器是否补执行”的问题未回答，暂停，不作为进入下一阶段的前置任务。
- 已检查：当前分支仍为 bean-factory-post-processor-and-bean-post-processor；工作区实验修改只有 PopulateBeanWithPropertyValuesTest.java 的一处注释，该文件在当前分支与本地 application-context 分支内容相同。LEARNING.md 未跟踪。
- 恢复一步一个操作：先让用户切换到 application-context，确认后再定位和运行 ApplicationContextTest.testApplicationContext()。

### 2026-10-04：ApplicationContext 分支切换成功，先复习

- 用户确认切换成功，已核对当前分支为 application-context。
- 用户要求先梳理之前内容，再继续；沿 XML → BeanDefinition → 实例创建 → 属性填充与依赖 → 初始化回调 → 单例缓存的流程复习。
- 已核对新分支测试预期：Person.name=ivy，Car.brand=lamborghini；尚未收到用户运行结果。

### 2026-10-04：ApplicationContext 测试通过

- 用户确认 testApplicationContext() 通过；测试验证 Person.name=ivy、Car.brand=lamborghini。
- 测试通过不等于理解启动过程；下一步从构造方法定位 refresh() 的调用。

### 2026-10-04：构造方法保存配置路径已观察

- 用户在 refresh() 调用前观察到 this.configLocations 数组包含 classpath:spring.xml。
- 已连接测试传入的路径与上下文保存的配置；接着只观察 refreshBeanFactory() 执行后的定义表。

### 2026-10-04：上下文加载四个定义已观察

- 用户观察 beanDefinitionMap 中有 car、person、customBeanFactoryPostProcessor、customerBeanPostProcessor，值均为 BeanDefinition。
- 已确认 refreshBeanFactory() 将业务 Bean 与处理器的定义一并加载。
- 下一步仅观察按 BeanFactoryPostProcessor 类型取得的处理器实例，不展开实例处理器注册。

### 2026-10-04：按类型取得定义处理器实例已观察

- 用户观察 beanFactoryPostProcessorMap 的键为 customBeanFactoryPostProcessor，值实际类型为 CustomBeanFactoryPostProcessor。
- 已观察处理器由 BeanDefinition 经工厂创建为实例，并按 BeanFactoryPostProcessor 类型选出。
- 下一步观察上下文自动调用 postProcessBeanFactory()，修改 Person 定义中的 name。

### 2026-10-04：上下文自动调用定义处理器已观察

- 用户单步执行 CustomBeanFactoryPostProcessor 的属性替换后，确认 Person 定义中的 name 变成 ivy。
- 已沿实际调用观察：加载处理器定义 → 按类型取得实例 → 上下文调用 postProcessBeanFactory() → 修改 Person 定义。
- 下一步只观察 BeanPostProcessor 实例加入工厂处理器列表。

### 2026-10-04：上下文自动注册实例处理器已观察

- 用户观察工厂 beanPostProcessors 列表已加入 CustomerBeanPostProcessor 实例。
- 已连接旧测试的手动 addBeanPostProcessor 与上下文的自动注册步骤。
- 下一步观察 refresh() 最后的 preInstantiateSingletons() 是否在测试显式 getBean 前创建并缓存业务 Bean。

### 2026-10-04：启动时提前创建单例已观察

- 用户在测试第一次显式 getBean("person", Person.class) 执行前，确认工厂 singletonObjects 已包含 person 和 car。
- 已观察 ClassPathXmlApplicationContext 构造期间调用 refresh()，最后通过 preInstantiateSingletons() 提前创建并缓存业务 Bean。
- 下一步仅确认：测试随后 getBean("person") 会命中缓存，不重新创建 Person。

### 2026-10-04：ApplicationContext 核心流程小节完成

- 用户正确判断：构造完成后 getBean("person", Person.class) 返回缓存中已实例化的 Person。
- 已逐步观察构造保存路径、refreshBeanFactory 加载四个定义、自动调用定义处理器、注册实例处理器、提前创建单例，以及后续缓存获取。
- 第三轮资源加载、XML 定义、容器扩展、ApplicationContext 的核心流程已完成。
- 下一阶段从 invokeInitMethods() 的占位日志衔接真实初始化与销毁方法，分支为 init-and-destroy-method，测试入口为 InitAndDestoryMethodTest.testInitAndDestroyMethod()（文件名实际拼写为 Destory）。
- 切换前已检查新增工作区修改：AbstractApplicationContext.java 的注释 /** 改为 /**v；目标分支同一文件新增关闭流程，普通 git switch 可能拒绝切换。不得丢弃用户修改或强制切换。

### 2026-10-04：初始化与销毁方法分支测试通过

- 用户确认切换并运行测试成功，已核对当前分支为 init-and-destroy-method。
- 测试加载 init-and-destroy-method.xml，其中 person 配置 init-method="customInitMethod" 和 destroy-method="customDestroyMethod"。
- 先只学习初始化：把此前占位初始化阶段连接到 Person 的实际 customInitMethod()，暂不展开销毁和 InitializingBean 接口。

### 2026-10-04：自定义初始化方法中的属性已填充

- 用户在 Person.customInitMethod() 中观察 name=derek，car 的品牌为 porsche。
- 精确表述：this.car 是 Car 对象引用，不是品牌字符串；其 brand 属性为配置文本 porsche。
- 已验证真实初始化方法执行前，普通属性与依赖均已填充。

### 2026-10-04：初始化方法解析需要补齐，暂停反射调用

- 用户反馈讲解混乱，指出 init-method 如何从 XML 进入 BeanDefinition 也是新增内容，应先逐步观察。
- 暂停 initMethod.invoke(bean) 断点任务，未收到其观察结果。
- 改从 XmlBeanDefinitionReader 中 getAttribute(INIT_METHOD_ATTRIBUTE) 开始，一次只观察读取属性文本这一步，再追踪 setInitMethodName()。

### 2026-10-05：init-method 属性读取已验证，区分测试入口

- 用户最初观察 id=person 但 initMethodName 为空，随后自行确认运行的是 ApplicationContextTest，加载 spring.xml，其中 person 没有 init-method，空字符串是正常结果。
- 用户改为调试 InitAndDestoryMethodTest.testInitAndDestroyMethod()，确认属性读取观察通过，得到 customInitMethod。
- 下一步只观察 setInitMethodName() 把读取到的字符串存入 BeanDefinition，不跳到反射调用。

### 2026-10-05：初始化方法名存入定义已观察

- 用户在 XML 读取器执行 setInitMethodName() 后确认 beanDefinition.initMethodName=customInitMethod。
- 已逐步连接 XML 属性文本 → 局部变量 → BeanDefinition 字段；尚未追踪工厂使用该字段的过程。

### 2026-10-05：用户询问接口初始化入口及取名后的流程

- 用户询问 invokeInitMethods() 中读取方法名之前的 if，以及读取后如何继续；未明确反馈 getter 断点值，不记为已验证。
- 按用户问题先补 InitializingBean：接口约定 afterPropertiesSet()；Person 实现该接口，instanceof 判断成立，强转后调用其方法。
- 区分接口约定的 afterPropertiesSet() 与 XML 指定的 customInitMethod()；本例两者都会执行，前者在先。
- 读取 XML 方法名后，工厂按名称查找 Method，再在已有 Person 实例上调用；反射调用尚未观察。

### 2026-10-05：接口初始化入口说明已接受

- 用户回复 ok，已说明 InitializingBean 约定 afterPropertiesSet()，当前 Person 实现该接口，因此先调用接口初始化方法。
- 当前 XML 另配置 customInitMethod，属于不同的方法；不把两个入口混为同一次调用。
- 接着先解释读取方法名后的条件判断，避免跳过新代码直接进入反射。

### 2026-10-05：自定义初始化条件分支已观察

- 用户确认调试停在 AbstractAutowireCapableBeanFactory.java 第 157 行，当前 Person 的 customInitMethod 进入自定义初始化分支。
- 下一步仅单步执行方法查找，观察返回的 Method；还不执行 invoke。

### 2026-10-05：按名称查找到 Method 已观察

- 用户提供调试截图：initMethod 为 Method 对象，显示 public void org.springframework.test.ioc.bean.Person.customInitMethod()；clazz=Person，name=customInitMethod，returnType=void。
- 已确认按定义中的类型与方法名找到了目标方法；查找后尚未确认执行。

### 2026-10-05：两种初始化入口的调用顺序理解正确

- 用户表示理解，并正确说明 Person 先打印 afterPropertiesSet 的日志，再打印 customInitMethod 的日志。
- 已理解接口初始化入口在前、XML 指定方法在后，以及 Method.invoke(bean) 对已有实例执行方法；用户此次是理解说明，未明确报告实际运行日志或方法体断点结果。
- 自定义初始化链路已逐步连接：XML 属性 → BeanDefinition.initMethodName → 工厂 getter → 查找 Method → 调用已有实例的方法。

### 2026-10-05：初始化小节完成

- 用户确认实际控制台初始化日志顺序与预测一致：afterPropertiesSet → customInitMethod。
- 已掌握 XML 方法名读取与定义保存、工厂读取、接口初始化入口、避免重复调用的条件、Method 查找与实例方法调用。
- 下一步仍在 init-and-destroy-method 分支学习销毁，先解释用途，再沿 XML 配置进入定义，不直接跳到关闭流程。

### 2026-10-05：销毁方法名存入定义已观察

- 用户确认 Person 的 beanDefinition.destroyMethodName=customDestroyMethod。
- 下一步先用显式 close() 观察触发时机；原测试 registerShutdownHook() 在 JVM 正常退出时触发清理，为便于后续顺序调试先改为手动关闭。

### 2026-10-05：显式关闭的销毁日志已验证

- 用户按实验反馈销毁日志顺序：I died in the method named destroy → I died in the method named customDestroyMethod。
- 已观察关闭触发结果，尚未追踪销毁注册表和适配器调用机制。

### 2026-10-05：用户自主追踪销毁注册与遍历，需补适配器

- 用户自主调试并概述：创建 Bean 时 registerDisposableBeanIfNecessary 判断 DisposableBean 或配置 destroy-method，close 后遍历 disposableBeans 并调用 destroy。
- 用户尚不清楚 XML 指定的方法在哪里执行。关键补充：Map 值为 DisposableBeanAdapter 实例，不是直接存 Person；适配器保存业务实例及定义中的销毁方法名。
- DisposableBeanAdapter.destroy() 先调用业务实例的 DisposableBean.destroy()，再按 destroyMethodName 查找 Method 并 invoke(bean)，执行 XML 指定方法。
- 精确用语：业务类实现 DisposableBean 接口，注册时保存实例及其适配器，不是注册类。

### 2026-10-06：销毁适配器理解确认，转为梳理类拆分的理由

- 用户确认理解 DisposableBeanAdapter 封装销毁逻辑；已纠正为实现 DisposableBean 接口，而非继承类。
- 用户认为继承层次混乱，主动询问为何拆成多个类、为何需要 ClassPathXmlApplicationContext。
- 已说明 ClassPathXmlApplicationContext 提供具体配置入口、保存路径并调用 refresh；AbstractApplicationContext 组织启动顺序，配置来源可变化，拆分是复用设计选择而非功能上必须。
- 用户回复 ok，接着比较 AbstractRefreshableApplicationContext 与 AbstractXmlApplicationContext；尚未确认这一比较的理解。

### 2026-10-06：类拆分说明已接受，准备学习 Aware

- 用户确认理解工厂管理与 XML 加载的职责拆分，并要求继续主线。
- 初始化与销毁阶段完成，接着学习 aware-interface：如果 Bean 本身需要所在工厂或上下文引用，容器如何交给它。
- 已读取本地 aware-interface 的 AwareInterfaceTest.test()，测试获取 HelloService 后断言其 ApplicationContext 和 BeanFactory 引用非空。
- 当前仍为 init-and-destroy-method；待用户切换。已有 PopulateBeanWithPropertyValuesTest 注释修改，该文件在两分支间一致。

### 2026-10-06：Aware 分支已切换，测试结果待确认

- 用户确认 aware-interface 分支切换成功，要求以后每节先说明原因。
- 已说明动机：Bean 不会自动持有所在工厂和上下文，需要时可通过 Aware 回调获取引用。
- 已安排运行 AwareInterfaceTest.test()；用户回复 ok，尚未明确反馈测试通过，不视为已验证。

### 2026-10-06：Aware 测试通过

- 用户明确确认 AwareInterfaceTest.test() 通过，HelloService 的 ApplicationContext 与 BeanFactory 引用均非空。
- 先只学习 BeanFactoryAware，不同时展开 ApplicationContextAware。

### 2026-10-06：BeanFactoryAware 接收工厂类型已观察

- 用户在 HelloService.setBeanFactory() 断点观察参数实际类型为 DefaultListableBeanFactory。
- 下一步追踪调用来源：工厂 initializeBean() 判断 BeanFactoryAware，再调用 setBeanFactory(this)。

### 2026-10-06：BeanFactoryAware 的 this 引用理解已验证

- 用户在上一层 initializeBean() 调用栈观察 this 也是 DefaultListableBeanFactory。
- 已连接工厂 setBeanFactory(this) → HelloService 保存参数；父类方法中的 this 是正在创建 Bean 的同一工厂对象。
- 下一步只观察 ApplicationContextAwareProcessor 构造时保存的上下文引用；先解释工厂和上下文是两个对象，需要把上下文交给处理器，再追踪前置回调。

### 2026-10-06：上下文与 Aware 动机补充已接受

- 用户在处理器构造断点任务前反馈仍不理解上下文，暂停原任务。
- 已从手动准备工厂的测试解释：上下文封装创建工厂、XML 解析并注册定义、组织处理器、提前创建单例和关闭流程，内部持有工厂并委托 getBean。
- 用户正确概括上下文是对这些流程的封装。
- 已澄清关闭时机：close() 手动立即触发；registerShutdownHook() 注册 JVM 正常退出时的清理，创建上下文不等于自动注册关闭钩子。
- 用户指出只查 Bean 时 BeanFactory 已够用；已确认，Aware 用于传递容器引用，HelloService 同时实现两个接口仅为演示，sayHello 本身不需要容器引用。普通依赖可继续 ref 注入。
- 用户回复 ok 并要求继续，恢复处理器构造参数的观察，未把此前暂停的任务记为已完成。

### 2026-10-06：处理器接收上下文实例已观察

- 用户观察 ApplicationContextAwareProcessor 构造参数实际类型为 ClassPathXmlApplicationContext，并询问是否由于其非抽象。
- 已说明实际类型来自测试最初 new 的具体对象；父类 refresh() 中的 this 指向同一实例，非抽象只是可实例化的条件。用户回复 ok。
- 下一步仅观察前置处理回调把处理器保存的上下文引用传给 HelloService。

### 2026-10-06：Aware 核心流程小节完成

- 用户确认 HelloService.setApplicationContext() 的调用栈上一层为 ApplicationContextAwareProcessor.postProcessBeforeInitialization()，并正确概括在前置处理回调中传入上下文。
- 精确表述：HelloService 预先在类声明中实现 ApplicationContextAware；前置处理器运行时调用其 setter 并赋值，不是在运行时实现接口。
- 两条路径已理解：工厂 initializeBean() 直接 setBeanFactory(this)；上下文先把自身引用交给 ApplicationContextAwareProcessor，处理器通过前置回调 setApplicationContext()。
- 已验证测试与相关断点；Aware 核心流程小节完成。

### 2026-10-06：明确设计理解与 Spring Boot 对照的学习目标

- 用户说明学习初衷是理解为何如此设计，并理解 Spring Boot 如何使用这些底层机制，询问如何消化。
- 后续按“具体问题 → 可替代方案及取舍 → 主流程验证 → 真实 Spring/Boot 场景对照”推进，保持一个任务一步。
- 建议每节只留下四句个人解释（问题、设计、插入位置、实际用途），复习时先凭记忆说明、再按需查代码；不要求记录所有父类接口。
- 已核对 Spring Boot 官方自动配置说明：根据依赖等条件配置应用，用户配置可替代部分默认配置。后续对照不把 Boot 自动配置等同于 mini-spring 的 XML 读取器。

### 2026-10-06：启动流程自主复述，两个时间点需巩固

- 用户正确复述：上下文创建工厂、解析 XML 并注册定义，定义包括处理器；获取流程查缓存，缺失才创建，创建流程包含实例化、处理器与初始化、最终缓存。
- 待纠正一：本项目 ClassPathXmlApplicationContext 构造期间 refresh() 最后调用 preInstantiateSingletons()，内部调用 getBean 创建 HelloService，因此测试显式调用 getBean 前已有缓存实例。
- 待纠正二：BeanFactoryPostProcessor 在 refresh() 加载定义后统一取得并调用，不是在每次 getBean/createBean 时自动执行。
- 补齐被省略的属性填充，以及本分支 BeanFactoryAware 在前置处理之前、ApplicationContextAware 通过前置处理器调用。
- 当前仅确认上述时间点，不据此重复完整断点实验或进入下一分支。

### 2026-10-06：refresh 与 getBean 边界已理解，准备作用域

- 用户正确判断后续 getBean 不会重新执行 BeanFactoryPostProcessor；已精确说明本项目每次 refresh 调用一次，而不是对象获取时调用。
- 用户询问下一阶段；已核对 changelog 顺序：Aware → prototype-bean → factory-bean。
- 下一阶段 prototype 作用域，动机为当前同名单例复用，部分对象需要每次获取独立实例。
- 本地尚无 prototype-bean 分支，已有 origin/prototype-bean；现有测试注释修改所在文件在两分支一致。

### 2026-10-06：prototype 分支已切换

- 用户确认切换成功，已核对当前分支为 prototype-bean。
- 新测试连续按同名 car 获取两次，断言 car1 != car2；XML 配置 scope="prototype"。
- 先只运行测试确认结果，再解释 scope 解析与创建流程的改变。

### 2026-10-06：prototype 测试通过，主动追踪创建流程

- 用户确认 PrototypeBeanTest.testPrototype() 通过，并主动指出 refresh 不提前实例化 prototype、getBean 缓存未命中会重新 doCreateBean，因此两次实例不同。
- 需补全因果：doCreateBean 只有 isSingleton() 时 addSingleton；prototype 首次创建后不加入单例缓存，才保证后续获取仍未命中。不提前创建只影响首次创建时机，不独自决定是否复用。
- 当前 getBean 对 prototype 仍执行 getSingleton 查询，并非按 scope 跳过查询；正常流程下没有该名称的缓存实例。
- 下一步用一个预测巩固“创建时机”和“实例复用”两个不同设计问题。

### 2026-10-06：prototype 核心机制理解确认

- 用户正确预测保留单例缓存时两次获取仍返回同一对象，明确缓存复用与提前创建是不同问题。
- 用户主动指出 scope 解析进入 BeanDefinition 与 init-method 基本一致，不重复逐字段实验。
- 已核对新增差异：setScope() 保存 scope 字符串，同时更新 singleton/prototype 布尔字段；未配置非空 scope 时沿用默认 singleton=true。
- 当前 XML 读取器使用 attributeValue("scope") 读取，再在非空时 setScope()；不继续引用旧分支 getAttribute 的实现细节。
- prototype 测试与核心创建/缓存机制已理解，完成本节核心流程。

### 2026-10-06：准备进入 FactoryBean

- 用户回复 ok，继续后续阶段。
- 已核对 origin/factory-bean 的 CarFactoryBean：getObject() 手动创建 Car 并设置 brand，isSingleton() 返回 true；测试入口为 FactoryBeanTest.testFactoryBean()。
- 引入动机：当目标对象的创建需要自定义逻辑时，可通过 FactoryBean.getObject() 封装，让调用方继续按名称从容器获取对象；本例创建逻辑简单，仅演示接口。

### 2026-10-06：FactoryBean 测试通过

- 用户确认 FactoryBeanTest.testFactoryBean() 通过。
- 下一步先区分 XML 注册的 CarFactoryBean 与 getBean 返回的 Car，不直接展开工厂缓存分支。

### 2026-10-06：FactoryBean 与实际注解配置的对照

- 用户主动询问 FactoryBean 是否类似 Spring Boot 中 @Component/@Bean。
- 已核对官方文档：三者属于 Spring Framework 能力，Boot 应用也使用；@Component 标记扫描注册的组件类，@Bean 方法提供返回对象，FactoryBean 是特殊 Bean，按其名称通常取得 getObject() 的产品。
- 在自定义创建逻辑方面 FactoryBean.getObject 与 @Bean 方法相似，但机制不同；不能把 @Bean 理解为自动转换成 FactoryBean。
- 之前 this/car 类型观察任务尚未反馈，不视为已完成。

### 2026-10-06：FactoryBean 与产品对象及缓存已识别

- 用户观察 this 实际类型为 CarFactoryBean，局部 car 为 Car，并自主调试发现 getBean 判别 FactoryBean 后使用另一份缓存。
- 已核对本分支 factoryBeanObjectCache 直接位于 AbstractBeanFactory，未拆出 FactoryBeanRegistrySupport。
- 当前 XML 默认 singleton，因此 singletonObjects[car] 保存 CarFactoryBean；其 isSingleton() 返回 true，因此 factoryBeanObjectCache[car] 保存产品 Car。
- 需区分两种复用控制：BeanDefinition.scope 控制工厂 Bean 实例，FactoryBean.isSingleton() 控制产品缓存；后者返回 false 不等于工厂 Bean 变为 prototype。

### 2026-10-06：产品缓存预测正确，补充泛型获取方法

- 用户正确预测 CarFactoryBean.isSingleton() 返回 false 时两次获取 Car 不同；尚未明确回答工厂自身是否复用，解释中补充 XML 默认 singleton 会复用 CarFactoryBean。
- 用户询问 getBean(String name, Class<T> requiredType) 中 T 与 requiredType 的关系。
- 当前实现 return ((T) getBean(name))，requiredType 在方法体内未使用；Class<T> 参数为编译器关联参数类型与返回类型提供信息，不能把它当成当前实现已有运行时类型校验。

### 2026-10-06：泛型获取说明已接受，FactoryBean 核心小节完成

- 用户回复理解泛型 getBean：Class<T> 参数关联编译期返回类型 T，requiredType 为参数名，当前方法体未使用它进行类型校验。
- FactoryBean 核心流程已掌握：XML 定义工厂 Bean → 创建并填充工厂 Bean → getBean 识别 FactoryBean → getObject 创建产品 → 按 isSingleton 决定产品缓存。
- 已通过测试、this/car 类型观察及两份缓存的自主调试，产品不缓存时不同实例的预测正确；未声称已实际运行 isSingleton=false 实验。
- 本节完成，不重复已掌握的解析、缓存或泛型实验。

### 2026-10-06：用户询问 prototype FactoryBean 的用途与实现边界

- 用户询问工厂 Bean 本身为 prototype 的价值；解释为隔离创建者实例及其状态，而仅需要新产品时 singleton 工厂加 isSingleton=false 已足够（当前 CarFactoryBean.getObject 每次 new Car）。
- 已核对重要教学简化：本项目产品缓存只判断 FactoryBean.isSingleton，未要求工厂自身在 singletonObjects 中；若 XML scope=prototype 且 isSingleton=true，会新建多个工厂却复用同名产品缓存。
- 已核对真实 Spring FactoryBeanRegistrySupport.getObjectFromFactoryBean：产品全局缓存分支要求 factory.isSingleton() && containsSingleton(beanName)。prototype 工厂不走该分支；产品是否实际独立仍取决于 getObject 实现。
- 不把 prototype FactoryBean 必然产生新产品泛化到任意 getObject 实现；当前 Car 示例返回新产品时可如此推断。

### 2026-10-06：FactoryBean 补充说明已接受，准备容器事件

- 用户回复 ok，接受 prototype 工厂用于隔离创建者状态，以及 mini-spring 与真实 Spring 产品缓存条件的区别。
- 已核对项目下一节为 event-and-event-listener，入口 EventAndEventListenerTest。
- 新阶段动机：启动或关闭时需要通知多个关注者，避免把每个业务动作直接写入上下文流程；通过事件发布与监听器扩展。
- 当前仍在 factory-bean，本地尚无 event-and-event-listener，已有 origin 同名分支；现有测试注释修改所在文件在两分支一致。

### 2026-10-06：事件监听分支已切换

- 用户回复 ok，已实际核对当前分支为 event-and-event-listener。
- 测试构造上下文后发布 CustomEvent，再注册关闭钩子；先运行确认输出，不提前展开监听器泛型及分发实现。

### 2026-10-06：事件监听日志已观察

- 用户提供三条监听日志，顺序为 ContextRefreshedEventListener → CustomEventListener → ContextClosedEventListener。
- 先追踪启动完成事件，不同时展开三种事件及分发器继承。
- 新概念区分：事件是表达发生何事的对象，监听器是收到事件后执行动作的对象；日志打印的是监听器类名。

### 2026-10-06：启动完成事件类型已观察

- 用户在 publishEvent 的第一次断点观察参数实际类型为 ContextRefreshedEvent（输入中前缀 z 按笔误处理）。
- 下一步先确认分发器已登记的监听器，不跳过注册准备直接展开泛型反射筛选。

### 2026-10-06：分发器登记三个监听器已观察

- 用户观察 applicationListeners 集合有 ContextRefreshedEventListener、CustomEventListener、ContextClosedEventListener。
- 接着解释筛选需求：三个已登记的监听器不一定接收同一个事件；泛型声明 ApplicationListener<ContextRefreshedEvent> 表达关注的类型。
- supportsEvent 使用反射读取监听器类的泛型接口，下一步先只观察这一读取结果，暂不展开后续类型匹配。

### 2026-10-06：监听器泛型接口已观察

- 用户先报告 CustomEvent 类型名，随后确认 type 完整值为 org.springframework.context.ApplicationListener<org.springframework.test.ioc.common.event.CustomEvent>。
- 本次沿 CustomEventListener 的声明继续，不强求切换到 ContextRefreshedEventListener；当前完整类型说明监听器关注 CustomEvent。
- 下一步连接 ParameterizedType 取第一个实际类型参数，类型名经 Class.forName 转为 Class，再按 isAssignableFrom 判断能否接收当前事件。

### 2026-10-06：事件类型匹配需解释设计理由

- 用户正确预测 CustomEventListener 不匹配 ContextRefreshedEvent，但不熟悉 isAssignableFrom。
- 已解释 A.isAssignableFrom(B)：B 对象能否赋给 A 变量，左侧是接收类型，不是比较对象是否同一个。
- 用户进一步询问为何不直接比较同种类型；当前重点解释父类事件监听器可接收子类，精确 Class 相等会排除子类。
- 本项目 ApplicationEvent 为公共父类，ContextRefreshedEvent、ContextClosedEvent、CustomEvent 都经 ApplicationContextEvent 继承它。

### 2026-10-06：首次分发配对已观察，事件对象细节待补

- 用户接受父类事件匹配的用途；反馈本节改动多、代码复杂。
- 已用登记 → 发布 → 筛选 → 调用四步收拢主流程。
- 用户在 onApplicationEvent(event) 调用处首次观察 ContextRefreshedEventListener 与 ContextRefreshedEvent；已说明构造期间先发布启动完成事件，首次停住是首个匹配回调，不是 HashSet 的第一个元素。
- 用户理解大致方向，但事件与监听器的具体代码仍不清楚；不能将事件阶段记为完成。
- 接下来先只解释事件对象：类型表达发生何事，source 保存来源上下文；监听器回调负责响应动作，分发器负责选择和调用。

### 2026-10-06：事件 source 的上下文类型已观察

- 用户确认 ContextRefreshedEvent.source 实际类型为 ClassPathXmlApplicationContext。
- 已连接事件类型表达发生何事，source 保存发生于哪个上下文；事件对象不等于上下文，也不直接执行响应动作。
- 下一步只看 ContextRefreshedEventListener 的回调实现和其中 this 的含义。

### 2026-10-06：监听器回调中的 this 已观察

- 用户确认 ContextRefreshedEventListener.onApplicationEvent() 中 this 为 ContextRefreshedEventListener。
- 事件对象、事件 source（ClassPathXmlApplicationContext）、响应监听器三个角色的具体对象已分别观察。
- 下一步将登记、发布、类型筛选、回调串起来，并通过新增同类型监听器是否需要修改 refresh 来确认解耦设计的用途；不重复逐类型断点。

### 2026-10-06：新增监听器的扩展方式理解正确

- 用户正确说明新增监听器无需修改发布代码，监听事件后提供具体响应动作。
- 用户询问 refresh 中“初始化事件发布者”的作用；需澄清注释对应 initApplicationEventMulticaster，实际创建并保存 SimpleApplicationEventMulticaster 分发器。
- 区分发布入口与通知执行：ApplicationContext.publishEvent 接收事件并委托分发器；分发器登记、筛选并调用监听器。BeanFactory.addSingleton 把同一分发器对象登记为基础设施单例。

### 2026-10-06：分发器为何注册到工厂的设计取舍

- 用户追问分发器已有上下文字段，为何还 addSingleton 到工厂。
- 已搜索项目：事件发布和监听器注册均通过上下文字段使用分发器，没有其他代码按保留名称查询；当前正常通知流程不依赖此注册行。
- 注册提供按名称 getBean("applicationEventMulticaster") 获取同一基础设施实例的入口，字段与缓存保存同一引用；直接 addSingleton 不等于经过普通 Bean 创建/初始化/销毁注册流程。
- 已核对真实 Spring：initApplicationEventMulticaster 先检查同名用户 Bean，有则使用，否则创建默认分发器并注册单例。mini-spring 此分支仅实现默认创建与注册，不具备对应的自定义替换选择逻辑。

### 2026-10-06：事件与监听器核心小节完成

- 用户确认理解分发器登记到工厂的用途、当前流程中的非必需性，以及真实 Spring 同名 Bean 自定义入口与教学简化的区别。
- 已实际运行并观察启动、自定义、关闭三条监听日志，追踪事件对象、source、监听器回调、登记集合与类型筛选；能解释新增监听器无需改发布代码。
- 本节核心流程完成，不要求掌握所有继承类或复写完整泛型反射实现；自定义/关闭事件的发布位置可按需要补查，不声称逐个完成了相同断点。
- 主设计结论：事件表达发生何事，监听器提供响应，分发器筛选与通知；发布者不写死所有业务响应。

### 2026-10-06：回看 supportsEvent 逐行解析

- 用户明确要求解析事件与监听器匹配代码，继续本节答疑，不进入下一阶段。
- 解释目标：getClass 取监听器类；getGenericInterfaces()[0] 取第一个带泛型接口；ParameterizedType.getActualTypeArguments()[0] 取事件类型参数；getTypeName 与 Class.forName 得到事件 Class；isAssignableFrom 判断实际事件是否可作为声明类型传入。
- 当前简化实现假设直接实现的第一个接口就是 ApplicationListener<具体事件类型>，不把这种写法泛化为所有监听器继承结构都适用。

### 2026-10-06：XML 顺序与按类型获取顺序的区别

- 用户观察 event-and-event-listener.xml 中标签顺序与 getBeansOfType 的结果顺序不同，询问原因。
- 当前定义表 beanDefinitionMap 和 getBeansOfType 返回 result 都是 HashMap，不承诺按插入顺序遍历；XML 按标签顺序注册不等于按该顺序返回。
- 分发器另用 HashSet 保存监听器，通知遍历也不承诺登记顺序。不能把未保证顺序说成每次随机变化。
- 按类型筛选只是判断并选择对象，不会恢复 XML 顺序；若要保留登记顺序，需一并考虑定义表、返回集合和分发器集合，不只是替换一处 Map。

### 2026-10-06：事件继承答疑完成，准备切点表达式

- 用户确认事件子类通过 extends（直接或间接）继承事件类，监听器通过 ApplicationListener<事件类型> 声明接收范围；要求继续。
- 已核对后续顺序：pointcut-expression → jdk-dynamic-proxy；先学习切点规则匹配，不能跳过直接进入代理。
- 下一入口 PointcutExpressionTest.testPointcutExpression()，先理解如何选出需要附加动作的类与方法。
- 当前 PopulateBeanWithPropertyValuesTest 的未提交修改是一条学习注释；目标分支同一文件变更了 import 路径，普通分支切换可能拒绝。先定向 stash 保存该文件，LEARNING.md 保留不动；之后切换并将注释恢复到新分支。

### 2026-10-06：切点分支已切换，学习注释恢复

- 用户回复 ok 后核对发现已在 pointcut-expression，stash 列表为空，无需再切换。
- 新分支测试文件中的原学习注释不存在，按此前已核对的 git diff 恢复同一条注释，未修改实验逻辑及目标分支 import。
- 下一步只运行切点表达式测试，验证规则匹配 HelloService 与 sayHello。

### 2026-10-06：切点测试通过，测试代码含义已补充

- 用户确认测试通过，但不理解测试代码；已解释 expression 规则、Class 与无参 Method 获取，以及两条匹配断言。
- 用户正确预测指定 sayGoodbye 后 sayHello 不匹配；询问 assertThat，已解释为 AssertJ 验证实际匹配结果，不属于 AOP 实现。
- 用户要求继续；先说明构造时借用 AspectJ 将表达式字符串解析成 PointcutExpression，再追踪类与方法匹配，不从构造跳过直接展开库内部。

### 2026-10-06：切点规则解析对象已观察

- 用户在解析后观察 pointcutExpression 实际类型为 PointcutExpressionImpl。
- 已连接 PointcutExpression 接口字段与 AspectJ 返回的具体实现对象；后续匹配委托给此规则对象。
- 下一步只追踪类筛选 matches(Class)；couldMatchJoinPointsInType 表达该类型中是否可能存在匹配连接点，不等于类内所有方法都已匹配。

### 2026-10-06：类匹配参数已观察

- 用户确认 matches(Class) 的 clazz 表示 HelloService 类型。
- 测试已验证类匹配为 true；下一步只看具体 Method 如何交给规则匹配，不重做类筛选断点。

### 2026-10-06：方法匹配需要具体解释，补充 ShadowMatch

- 用户确认 method 为 sayHello，指出 matchesMethodExecution 与 alwaysMatches 的说明太笼统。
- 已核对 AspectJ 官方接口文档：matchesMethodExecution(Method) 返回 ShadowMatch，表达 always/sometimes/never 的匹配结论；alwaysMatches 判断是否确定匹配，不是所有方法都匹配，也不执行目标方法。
- 后续解释须先按当前 expression 与 public String HelloService.sayHello() 逐项对照所属类、返回类型、方法名、参数；再说明结果对象为何需要多于 boolean 的结论，避免只翻译 API 名称。

### 2026-10-06：ShadowMatch 的具体说明已接受

- 用户回复 ok，已说明当前规则与 sayHello 方法信息逐项匹配，matchesMethodExecution 返回结果对象、alwaysMatches 判断确定匹配，不执行目标方法。
- 下一步补齐本节接口设计：ClassFilter 管类初筛，MethodMatcher 管具体方法，Pointcut 提供两种能力的获取入口。
- AspectJExpressionPointcut 同时实现三个接口，因此 getClassFilter/getMethodMatcher 返回 this 可提供不同接口视角，不创建两个新对象。

### 2026-10-06：切点接口的设计动机需按调用方解释

- 用户不理解三个接口/getter 的设计，助理误以为引用基础困难，用户明确纠正：已懂引用，疑问是直接 AspectJExpressionPointcut.matches 已足够，为什么还返回筛选器。
- 后续不要重复 Java 引用基础或布置 this 相等判断；聚焦真实调用者需求与设计取舍。
- 已核对当前测试直接调用 matches，两 getter 不是本分支测试所需；后续 auto-proxy 的 DefaultAdvisorAutoProxyCreator 通过 advisor.getPointcut().getClassFilter 获取初筛能力，并把 getMethodMatcher 交给代理配置。
- 说明动机：通用代理代码依赖 Pointcut/匹配接口而非写死 AspectJ 实现，允许替换规则，并允许类筛选和方法匹配由不同对象组成；当前实现用同一对象承担两职责所以 return this。合并接口方法也可实现，这是一种扩展设计选择，非功能必需。

### 2026-10-06：Pointcut 接口与 return this 的用途已理解

- 已补清对象职责：PointcutParser 解析表达式字符串并返回 PointcutExpression；后者提供匹配操作，AspectJExpressionPointcut 保存并委托该规则对象。
- 用户正确解释：新增 Pointcut 实现可复用外部调用代码；已补充创建或配置位置仍需提供新实现，匹配流程无需依赖具体实现类。
- getClassFilter/getMethodMatcher 提供两种匹配能力；当前对象自己实现两接口，因此返回 this，其他实现也可返回内部独立对象。不重新解析表达式。
- 下一步用一个有区分度的预测确认类初筛与具体方法匹配的区别，再进入 jdk-dynamic-proxy；不重复引用基础。

### 2026-10-07：切点表达式核心小节完成

- 用户正确预测限定 sayHello 后：类筛选 true、sayHello 方法匹配 true、getBeanFactory 方法匹配 false；解释类筛选通过不代表所有方法都符合规则。
- 已确认构造时解析一次表达式，之后使用规则对象匹配并对外返回 boolean；本节不执行目标方法、不附加动作、不创建代理。
- 切点表达式核心已理解，准备进入 jdk-dynamic-proxy；尚未切换分支或运行新测试。

### 2026-10-07：JDK 动态代理分支与测试已就绪

- 已核对当前分支 jdk-dynamic-proxy；用户确认 testJdkDynamicProxy 测试通过。
- 用户要求先解释什么是动态代理再推进；先区分业务目标对象、运行时生成的代理对象与 InvocationHandler 调用处理对象。
- 下一步在 JdkDynamicAopProxy.invoke 首行设断点，从 proxy.explode() 观察 method 和目标对象；先不展开所有配置类。

### 2026-10-09：代理创建入口与设计职责已梳理

- 用户已在 invoke 观察 method.getName() 为 explode，在 TargetSource.getTargetClass() 观察接口数组包含 WorldService。
- 已用静态代理与动态代理例子说明：JDK 生成实现业务接口的代理类，代理调用转交给 InvocationHandler；生成类示例为近似代码，不是项目源码。
- 已区分目标对象、生成的代理对象、JdkDynamicAopProxy handler；当前类同时负责 getProxy 创建入口和 invoke 调用处理，可拆成两个对象。
- 已解释 AdvisedSupport 保存目标、匹配器和拦截器配置，AopProxy 统一创建入口；均属组织与扩展选择，不是 JDK 动态代理必需。
- 用户回看并确认 Pointcut 的统一使用入口用途，要求继续当前分支。
- 下一步只追踪 invoke 中 matches 判断；随后解释拦截器和 proceed 如何执行目标方法，不重复创建代理的断点。

### 2026-10-09：匹配分支与拦截器分工已学习，回顾完整流程

- 用户观察 invoke 的匹配判断进入 if 内部。
- 已解释没有匹配判断时，经过代理转交到 handler 的调用都会执行所写动作；原始目标对象的直接调用不会经过代理。
- 已解释统一 invoke 返回 Object：引用返回原对象、基本类型经包装、void 反射调用返回 null，代理对外接口签名不变。
- 已解释拦截器分离的用途：代理负责分派，MethodInterceptor 决定附加动作；ReflectiveMethodInvocation 保存 target/method/arguments，proceed 通过反射执行目标方法。直接在 handler 写日志也能工作，拆分为复用和更换动作。
- 已核对 MethodInterceptor 与 MethodInvocation 来自 AOP Alliance 依赖，InvocationHandler 来自 JDK；具体实现由项目提供。
- 用户理解 ReflectiveMethodInvocation 构造参数由执行需要设计，并非接口规定。
- 用户要求细致总结，按当前测试串联完整流程；尚不将整节视为独立解释与边界预测均已完成。

### 2026-10-11：目标对象、反射与 Class 信息答疑已完成

- 用户正确预测去掉 invocation.proceed 后业务日志不会出现。
- 已说明 TargetSource 封装目标对象与接口信息，当前可直接保存 target 替代；method.invoke(target, arguments) 是反射调用，统一执行本次 Method 指定的业务方法。
- 已区分 getClass 返回实际运行时类型的 Class 对象，getInterfaces 返回直接实现接口的 Class 对象数组；接口、普通类、数组类型均有 Class 对象，但 getInterfaces 只返回接口类型。
- 用户表示可以继续；下一步验证切点不匹配时仍调用业务、跳过拦截器的路径，作为当前分支收尾实验。

### 2026-10-11：不匹配路径已验证

- 用户最初预测不匹配时日志都不出现；已纠正：只跳过拦截器，if 后的 method.invoke 仍执行目标业务方法。
- 用户运行修改表达式的实验并确认仅输出业务日志，验证不匹配分支。
- 核心结论：切点决定是否执行附加动作，不匹配不等于禁止业务调用。
- 下一步先恢复测试表达式中的 explode，保留 proxy.explode()；再继续后续阶段。

### 2026-10-11：JDK 动态代理核心小节完成，准备 CGLIB

- 已检查 DynamicProxyTest 的不匹配表达式恢复，文件无当前未提交差异；JdkDynamicAopProxy 仅保留用户的拦截器设计学习注释。
- 已完成测试、创建代理的接口来源、匹配分支观察、proceed 作用预测与不匹配路径运行验证；本节核心完成。
- 用户要求总结 AOP 的目的与解决方案后继续；下一阶段为 cglib-dynamic-proxy，入口 DynamicProxyTest.testCglibDynamicProxy()。
- 已核对目标分支中 JdkDynamicAopProxy 和保留学习注释的 PopulateBeanWithPropertyValuesTest 与当前分支无差异，可保留修改切换。
- 下一动机：JDK 生成实现接口的代理；没有业务接口时需要基于目标类生成子类的代理方式。复用当前目标、规则与附加动作配置，不重新展开全流程。

### 2026-10-11：CGLIB 分支已切换

- 用户确认切换，已实际核对分支为 cglib-dynamic-proxy。
- 新测试将共用配置移到 @Before setup；先只运行 testCglibDynamicProxy，观察是否仍有前置、业务、后置三条日志。
- 先说明基于接口的 JDK 代理与基于目标类子类的 CGLIB 代理差别，不提前展开两个内部类与同名 MethodInterceptor。

### 2026-10-11：CGLIB 测试通过，创建工具与内部类职责已说明

- 用户确认 CGLIB 测试按前置、业务、后置顺序输出日志。
- 已说明生成类继承 WorldServiceImpl，通过重写方法将调用交给 callback；示例为近似生成代码，不是实际源码。
- 当前父类已实现 WorldService，setInterfaces 传入已有接口对本测试类型关系是冗余配置，不是 CGLIB 必需。
- 已解释 Enhancer 配置与 create 对应 JDK Proxy.newProxyInstance 的创建职责。
- 已区分 CGLIB MethodInterceptor.intercept 接收代理调用与 AOP Alliance MethodInterceptor.invoke 安排附加动作；内部类组织为当前实现选择，static 不表示静态代理。
- 下一步在 DynamicAdvisedInterceptor.intercept 首行断点，确认 proxy.explode 到达 callback，并观察 this 类型；暂不展开 MethodProxy。

### 2026-10-12：callback 与代理对象已在断点中区分

- 用户截图确认 intercept 中 this 是 CglibAopProxy$DynamicAdvisedInterceptor，o 是 WorldServiceImpl$$EnhancerByCGLIB 生成的代理对象。
- 已通过具体保存引用和转交调用的示例解释 setCallback，不再仅用“交给 CGLIB”抽象表述。
- 下一步连接 intercept 创建 CglibMethodInvocation、复用 WorldServiceInterceptor，以及 proceed 的重写；先观察实际进入哪个 proceed，不展开 FastClass 内部。

### 2026-10-12：CGLIB 的 proceed 与原始目标对象已确认

- 已解释 MethodProxy 是 CGLIB 提供的具体方法调用工具，当前项目用它替代 Method.invoke；Method 仍用于规则匹配。Method.invoke 在当前原始目标对象上也可工作，不把 MethodProxy 当成必需或必然更快。
- 用户预测并在 CglibMethodInvocation.proceed 断点确认 this.target 实际类型为 WorldServiceImpl，而不是生成的代理子类。
- 用户发现两种回调的代理对象参数均未用于当前业务调用，继续解释通用回调签名与当前实现只使用原始目标的区别。

### 2026-10-12：CGLIB 动态代理核心小节完成

- 用户确认理解回调参数 proxy/o 是代理对象实例；当前业务调用使用配置保存的原始目标对象，所以参数可以不使用。通用回调接口提供上下文，不要求实现使用全部参数。
- 已区别 methodProxy.invoke(target,args) 在独立目标对象上执行与 invokeSuper(o,args) 在代理对象上执行父类实现；当前项目采用前者。
- 已完成测试日志观察、callback 的 this 与代理参数 o 区分、原始目标类型预测与断点核对；CGLIB 核心调用链完成。
- 后续先核对下一分支与测试入口，再解释新模块接入现有流程的位置；不重复本节相同断点。

### 2026-10-12：准备代理工厂小节

- 用户要求下一节；已核对 origin/proxy-factory 中 ProxyFactory 与 DynamicProxyTest.testProxyFactory。
- 新功能动机：当前调用方直接 new JdkDynamicAopProxy/CglibAopProxy；ProxyFactory 集中选择创建方式，调用方统一用 getProxy 获取代理。
- 当前简化选择规则只看 AdvisedSupport.proxyTargetClass：false 选择 JDK，true 选择 CGLIB；不自动按是否存在接口推断。
- 已核对两个保留学习注释的文件在目标分支无差异，当前仍为 cglib-dynamic-proxy，先引导切换 proxy-factory，再运行新测试。

### 2026-10-12：代理工厂的选择逻辑已理解

- 用户正确概括：AdvisedSupport 新增 boolean 配置，ProxyFactory 通过 if 选择代理方式。
- 已补清 createAopProxy 返回 JDK/CGLIB 代理创建者，getProxy 再调用创建者的 getProxy 返回实际业务代理对象。
- 用户确认理解统一创建入口及 AopProxy 的用途；尚未收到 testProxyFactory 的运行确认。

### 2026-10-12：代理工厂核心小节完成

- 已实际核对当前分支 proxy-factory；用户确认 testProxyFactory 的两组前置、业务、后置日志符合预期。
- 已掌握配置 boolean 选择创建者、AopProxy 统一创建入口、创建者 getProxy 返回实际代理对象。
- 本节核心完成，后续按仓库顺序核对 pointcut-advisor 的入口与动机后推进。

### 2026-10-12：PointcutAdvisor 分支已切换，需补前置动作适配

- 已核对实际分支 pointcut-advisor，入口 testAdvisor；此测试使用 MethodBeforeAdviceInterceptor 与 WorldServiceBeforeAdvice，用户此前尚未学习 common-advice 的适配内容。
- 前面直接引导 pointcut-advisor，需在当前分支用已有 testBeforeAdvice 先补齐：BeforeAdvice 只写 before，适配器将 before 与 invocation.proceed 接到原 MethodInterceptor 入口。
- 不要求切回前一分支，先补一个小环节再学 Advisor；不把新的适配器与 Advisor 同时展开。

### 2026-10-13：前置动作适配验证完成，进入 Advisor

- 用户正确识别适配后可以像 WorldServiceInterceptor 一样使用；已区分 WorldServiceBeforeAdvice 实现 MethodBeforeAdvice，与外层 MethodBeforeAdviceInterceptor 实现 MethodInterceptor。
- 用户确认 testBeforeAdvice 输出前置与业务两行日志，适配小节完成。
- 接下来解释 Advisor 保存规则与对应动作，testAdvisor 从它取得 ClassFilter、MethodMatcher 和 Advice，仍填入 AdvisedSupport 创建代理；Advisor 自己不创建代理或执行业务。

### 2026-10-13：Advisor 配置接入代理已验证

- 用户正确解释 Advisor 将切点与 advice 封装到同一个对象。
- 已用本地 aopalliance jar 核对接口关系：MethodBeforeAdviceInterceptor implements MethodInterceptor，后者 extends Interceptor extends Advice；Advice 是无方法的标记接口。
- 用户断点确认 testAdvisor 的 ClassFilter 判断进入 if；已区分类初筛在创建代理前，MethodMatcher 在调用代理时判断具体方法。
- 已解释从 advisor.getAdvice 强转 MethodInterceptor，与从 getPointcut 获取 MethodMatcher 后填入 AdvisedSupport；用户确认测试最终输出前置与业务两行。
- 下一步只做一个规则与动作分离的预测：保持 expression，只把 advice 换成 WorldServiceInterceptor，确认匹配范围不变、附加动作改变；随后收尾。

### 2026-10-13：PointcutAdvisor 核心小节完成

- 用户正确预测：保持表达式不变，只将 advice 换成 WorldServiceInterceptor，匹配范围不变，输出变为前置、业务、后置。
- 已完成用户自己的组合职责解释、类初筛断点、测试运行以及有区分度的更换动作预测，本节核心完成。
- 后续准备 auto-proxy：当前仍由测试手动读取 Advisor、填充配置和创建代理，下一阶段看容器如何接管这些操作。先核对下一分支和测试入口再切换。

### 2026-10-13：AOP 学习路径回顾，准备自动代理入口

- 用户要求串联一路的动机和流程后进入下一节。
- 已核对 origin/auto-proxy：入口 AutoProxyTest，配置 auto-proxy.xml，DefaultAdvisorAutoProxyCreator 在 postProcessBeforeInstantiation 中取 Advisor、类筛选并配置 ProxyFactory。
- 当前教学实现的接入点是 createBean 内 resolveBeforeInstantiation，发生在普通 doCreateBean 之前；匹配则处理器自行实例化目标并返回代理，不能说成普通 Bean 初始化完成后包装，也不能等同真实 Spring 的完整常规流程。
- 已核对两个保留学习注释的文件在目标分支无差异，当前尚在 pointcut-advisor；先引导切换 auto-proxy，再定位测试运行。

### 2026-10-13：自动代理分支已切换

- 用户确认切换，已实际核对分支 auto-proxy。
- 先运行 AutoProxyTest.testAutoProxy 观察行为；此测试 expected RuntimeException，第二个业务对象会抛异常，JUnit 预期该异常。
- 当前 XML 带组合通知与环绕通知，不提前展开全部类型；后续先按 XML 文本读取、BeanDefinition 保存、对象创建和处理器注册追踪自动代理创建器。

### 2026-10-13：自动代理日志已观察

- 用户提供正常与异常两组环绕日志：正常业务输出前、业务、成功返回、最终结束；异常业务输出前、业务、异常、最终结束。
- 异常日志的 null 来自无消息 RuntimeException 的 getMessage，不代表业务目标为空；未将日志输出单独当作 JUnit 通过确认。
- 下一步从 auto-proxy.xml 中无 id 的 DefaultAdvisorAutoProxyCreator 定义追踪 class 文本、BeanDefinition 与名称注册，随后再看处理器对象如何登记到工厂；不直接跳到回调实现。

### 2026-10-13：自动代理创建器的定义名称已观察

- 用户在 XML 定义注册处确认 beanName 为 defaultAdvisorAutoProxyCreator；已区分定义注册与处理器实例创建/登记。
- 下一步沿 refresh 的 registerBeanPostProcessors：getBeansOfType 创建/取得处理器对象，addBeanPostProcessor 将对象登记到工厂处理器列表；不将定义表注册等同处理器列表登记。

### 2026-10-13：处理器登记与业务检查入口已观察

- 用户确认 registerBeanPostProcessors 中创建/取得 DefaultAdvisorAutoProxyCreator 实例并登记为处理器。
- postProcessBeforeInstantiation 首次停在 methodThrowsAdviceInterceptor，已解释辅助对象会被 Advice/Pointcut/Advisor 类型检查排除。
- 用户反馈推进太快、缺少动机；已补清本节把手动读取 Advisor、填充配置和创建代理交给容器，处理器接入普通 Bean 创建之前的检查点。
- 已区别辅助类型直接排除、普通 Bean 的 Advisor 类初筛、具体调用的方法匹配；不匹配返回 null 表示工厂继续普通创建，并不是不创建 Bean。
- 用户设置 worldService 条件断点，确认 beanClass 为 WorldServiceImpl、beanName 为 worldService。
- 下一步只追踪从工厂取得 Advisor 对象，观察其 expression，连接 XML 配置；不一次展开剩余代理创建代码。

### 2026-10-13：自动代理创建与工厂返回链已观察

- 用户观察查询 Advisor 时嵌套创建 Advisor 及依赖；已核对 getBeansOfType 按定义筛选后调用 getBean，区分外层 worldService 检查等待与内层依赖创建。
- 用户确认返回 for 循环并通过 ClassFilter 判断，随后执行 ProxyFactory 返回代理。
- 在工厂 createBean 的 bean 非 null 判断处，截图显示 bean 实际类型为 $Proxy2，内部 h 为 JdkDynamicAopProxy；字符串展示的 WorldServiceImpl 不是实际代理类型。
- 已解释处理器非 null 返回导致工厂直接返回代理、跳过 doCreateBean；不声称该路径完成普通属性填充、初始化或单例缓存。
- 下一步用移除自动代理创建器但保留 Advisor 的预测确认：Advisor 保存配置，处理器才负责读取配置并自动创建代理；不重复已掌握的对象类型断点。

### 2026-10-13：自动代理创建核心小节完成

- 用户正确预测并解释：移除 DefaultAdvisorAutoProxyCreator 定义、保留其他配置时，不出现环绕日志，因为 getBean 返回原始业务对象，不是代理。
- 已掌握 XML 定义注册、处理器实例登记、业务创建前检查、Advisor 嵌套获取、类筛选、目标实例化、代理组装与工厂非 null 返回；核心小节完成。
- 此预测为概念确认，未要求或声称运行移除定义的实验；日志输出不单独等同 JUnit 状态确认。
- 下一阶段先核对仓库顺序与入口，并围绕代理路径跳过 doCreateBean 后目标属性填充与生命周期的差异说明动机；不把当前教学模型当作完整 Spring 实现。

### 2026-10-13：用户发现代理路径未写入单例缓存

- 用户指出代理路径跳过 doCreateBean 的 addSingleton，询问下次是否重新创建；已核对当前 getBean 在 createBean 返回后也没有补充缓存，自动代理创建器亦未缓存目标或代理。
- 因此当前 worldService 即使定义为 singleton，连续 getBean 仍重新创建目标与代理；这是当前分支实现的缺口，不是代理对象无法缓存，也不是单例应有行为。
- 不替用户修改实现；可在同一上下文连续获取 worldService 并以 == 比较，预期 false。尚未运行该边界实验。
- 后续应先处理或说明缓存与普通生命周期路径缺口，再推进后续阶段；不声称已修复。

### 2026-10-13：Advice、适配器与组合拦截器的层次已理解

- 用户反馈跟流程能懂，但接口继承与同名拦截器容易混淆；后续按配置、动作执行、代理入口三个职责解释，不要求记全继承树。
- 已区分 Advice 是公共标记、PointcutAdvisor 组合规则与动作、AOP Alliance MethodInterceptor 是执行入口、MethodInvocation 保存本次调用并提供 proceed。
- MethodInterceptor 接口不含 advice 属性；具体适配器保存动作对象并安排 before/after 与 proceed 的位置。
- 用户正确概括 CombineAdviceInterceptor 编排其他拦截器，并确认理解；它与适配器同实现 MethodInterceptor，类型上同级，对象上通过字段组合。
- 当前组合实现配置 around 时直接委托并返回，不是所有通知自动组成连续链；没有要求用户删除 around 或声称无 around 的组合路径正确。
- 代理单例缓存缺口仍未修复或运行两次获取实验，不能因本次答疑完成而视为已解决。

### 2026-10-13：按用户要求放下缓存缺口，准备代理目标属性填充

- 用户明确代理未缓存只是发现的教学实现疏漏，无需处理；后续不以此阻碍推进，不要求缓存修复或重复验证。
- 用户要求继续，已核对 origin/populate-proxy-bean-with-property-values。
- 新动机：上一分支在普通创建前实例化目标并返回代理，跳过 applyPropertyValues；需要把代理创建移到 postProcessAfterInitialization，先完成原始目标的属性填充与初始化，再包装代理。
- 关键工厂修改为 bean = initializeBean(...)，保留后置处理返回的代理；此变化将代理接入普通创建链，不是把字段直接写到代理对象上。
- 已核对当前两个学习注释文件在目标分支无差异；尚未切换，下一步只引导切换并定位测试。

### 2026-10-13：代理目标属性填充分支已切换

- 用户确认切换，已核对分支 populate-proxy-bean-with-property-values。
- 测试配置给 worldService.name 设置 earth，调用代理 explode 后断言代理 getName 返回 earth。
- 下一步只运行 testPopulateProxyBeanWithPropertyValues 确认行为，随后按 XML 属性文本读取、BeanDefinition 保存、applyPropertyValues 到后置包装顺序追踪，不跳过配置读取过程。

### 2026-10-13：带属性的代理测试通过

- 用户确认 testPopulateProxyBeanWithPropertyValues 测试通过，断言代理 getName 返回 earth。
- 下一步先连接 XML name/value 文本到 PropertyValue、BeanDefinition.PropertyValues，再在 doCreateBean 的 applyPropertyValues 调用处观察原始业务对象与赋值时机。
- 尚未确认后置包装的断点，不把测试通过等同已掌握流程改变。

### 2026-10-13：代理接入时机与工厂检查点已解释

- 用户发现 createBean 仍有 resolveBeforeInstantiation，已核对当前处理器的 postProcessBeforeInstantiation 返回 null，实际继续 doCreateBean。
- 已解释保留通用工厂检查点，由处理器选择参与时机；本节代理创建移至 postProcessAfterInitialization，工厂改为 bean = initializeBean(...) 接收替代对象。
- 已解释实例化后 boolean 检查点：当前自动代理处理器返回 true，允许继续；当前 false 分支直接 return 会跳过整个剩余创建流程，不只属性填充。
- 用户要求继续，下一步只在后置回调入口观察 worldService 原始对象与 name=earth，确认准备目标后再包装；此前赋值前后断点尚未收到观察确认。

### 2026-10-13：属性填充与后置代理包装顺序已理解

- 用户指出实例化后检查点之前 name 尚未赋值，已区分 postProcessAfterInstantiation（填充前）与 postProcessAfterInitialization（填充与初始化后），不混用名称。
- 用户正确解释当前前置处理保持原对象、后置处理返回代理的主线；已补初始化方法位于两者之间，工厂通过 bean = initializeBean(...) 接收代理。
- 用户询问原对象未单独缓存与局部变量作用域；已解释代理→处理对象→AdvisedSupport→TargetSource→原始目标的引用链，局部变量结束不意味着被引用对象消失。用户确认理解。
- 测试通过已确认，但没有声称每个属性填充前后断点都实际核对；以已确认理解和运行结果推进。
- 下一步预测把 bean = initializeBean(...) 改为忽略返回值时的效果，确认接收替代对象的重要性，再收尾。

### 2026-10-13：代理目标属性填充核心小节完成

- 用户正确预测忽略 initializeBean 返回值后：getBean 得到原始对象，name 仍为 earth，不出现前置日志，因为代理未被工厂接收。
- 已完成测试通过确认、创建检查点动机答疑、实例化后与初始化后时机区分、对象引用链解释，以及返回值的有区分度预测；本节核心完成。
- 后续先核对仓库下一阶段、分支与测试入口，再说明它解决已有流程的什么问题，不自行修改实验代码。

### 2026-10-13：准备属性占位符小节

- 用户要求下一节；按仓库扩展篇核对 property-placeholder-configurer，入口 org.springframework.test.expanding.PropertyPlaceholderConfigurerTest。
- 动机：XML 中写死字段值，改配置需改 Bean 定义文件；使用 ${key} 引用独立 properties 配置，复用既有 BeanFactoryPostProcessor 在实例创建前修改 BeanDefinition 的能力。
- 新配置先追踪 XML value 文本如何保存 PropertyValue，再看处理器读取属性文件并替换定义里的占位符，最后看工厂写入对象；不从占位符直接跳到字段赋值。
- 已核对保留学习注释的两个文件在目标分支无差异；当前未切换。

### 2026-10-13：属性占位符分支已切换，代码基线差异已核实

- 当前实际分支 property-placeholder-configurer；用户正确预测将占位符替换为配置值后复用正常 Bean 创建流程。已补清 XML 先原样存储 ${brand}，BeanFactoryPostProcessor 后续替换定义值。
- 已核对官方 Boot 文档说明外部配置与 @Value/@ConfigurationProperties，区分教学处理器与 Environment 体系；properties 与 YAML 均被支持。
- 用户发现上一节后置包装修改消失，已核对：本分支仍是旧 auto-proxy 的实例化前代理基线，不包含 populate-proxy-bean-with-property-values 的生命周期调整；代理功能本身仍存在。
- 切换引导时漏说明基线差异，后续不能默认教学分支按学习顺序累积所有变更，应提前核对并点明相关差别。
- 本节 XML 只定义配置处理器与 Car，不登记自动代理创建器，不影响本节占位符学习；不擅自合并上一节代码。

### 2026-10-13：属性占位符测试通过

- 用户确认 PropertyPlaceholderConfigurerTest.test 通过，Car.brand 最终为 lamborghini。
- 下一步先在 XML 读取保存属性处观察 value 为原始 ${brand}，不把测试通过等同已观察替换时机。

### 2026-10-13：占位符原始文本与配置读取已观察

- 用户确认 XML 读取器将 ${brand} 原样保存到定义。
- 用户确认 loadProperties 后 properties.getProperty("brand") 为 lamborghini；已区分配置集合读取完成与 BeanDefinition 替换完成，不把二者当作同一步。
- 已解释 resolvePropertyValues 先检查 String，再按 ${ 与 } 标记判断是否替换；无占位符字符串保持原样。
- 用户确认理解普通字符串安全保留，下一步观察 addPropertyValue 替换同名定义值；尚未收到写回定义断点结果。

### 2026-10-13：属性占位符核心小节完成

- 用户确认 addPropertyValue 后 BeanDefinition 中 brand 的配置值已替换；同名属性项由 PropertyValues.set 覆盖，不是重复追加。
- 用户正确预测移除 PropertyPlaceholderConfigurer 定义后，Car.brand 返回字面量 ${brand}；普通字段填充不会自动解析占位符。
- 已完成测试、XML 原始文本观察、Properties 读取观察、定义替换确认、用户自己的机制预测及边界预测，本节核心完成。
- 下一阶段按扩展篇继续 package-scan；先核对代码与测试入口，再说明从手写 XML 定义到扫描登记定义的动机，不提前展开 Value/Autowired。

### 2026-10-13：准备包扫描分支与入口

- 已核对 origin/package-scan，入口 org.springframework.test.ioc.PackageScanTest.testScanPackage()；配置 package-scan.xml 的 context:component-scan base-package 指定 org.springframework.test.bean。
- 已核对核心工厂、自动代理创建器和两个保留学习注释文件与当前 property-placeholder-configurer 无差异，延续相同代码基线。
- 扫描实现按 Component 注解找候选类型、构造 BeanDefinition 并注册；业务对象仍由已有工厂流程创建，不把扫描登记当成直接实例化。
- 已向用户结合官方 Boot 文档说明 ComponentScan 与 SpringBootApplication 的关系；本节是教学扫描模型，不等同完整 Boot 启动实现。
- 当前未切换，先引导切换后运行测试，不提前同时展开 Value/Autowired。

### 2026-10-14：包扫描测试通过，准备追踪扫描范围

- 用户已确认切换 package-scan，并确认 PackageScanTest.testScanPackage 测试通过。
- 用户曾不清楚 IoC 与 AOP 的界限，已解释 IoC 负责对象准备/装配/管理，AOP 负责方法附加行为；包扫描接在 BeanDefinition 登记阶段，不等于创建代理或直接实例化。
- 下一步先在 XmlBeanDefinitionReader 中观察 component-scan 的 base-package 文本读取，再进入扫描器；一次一个观察，不提前展开所有扫描类与作用域。

## 下一步（读取包扫描范围）

- 当前实际分支为 package-scan；测试已通过，先观察 XML 读取的扫描包名，再追踪发现 Component 候选类与定义注册。
- 新阶段动机：多个业务方法需要公共日志/耗时统计等附加动作，避免逐个修改业务方法；第一节只解决“哪些方法需要这些动作”，后续代理解决“如何插入动作”。
- 继续一次一个操作，不提前展开自动代理、Advisor 或全部代理实现。

## 工作区备注

- 当前分支名为 package-scan。
- 学习记录保留在工作区；不代替用户修改实验代码。
