# swift 4.2 代码规范

## 列表

* [正确性](#正确性)
* [命名](#命名)
  * [单调](#单调)
  * [类前缀](#类前缀)
  * [委托](#委托)
  * [类型推导](#类型推导)
  * [语言](#语言)
* [代码组织](#代码组织)
  * [协议一致性](#协议一致性)
  * [无用代码](#无用代码)
  * [最小导入](#最小导入)
* [间隔](#间隔)
* [注释](#注释)
* [类和结构体](#类和结构体)
  * [使用那一个](#使用那一个)
  * [类定义](#类定义)
  * [self使用](#self使用)
  * [计算属性](#计算属性)
  * [Final](#Final)
* [函数声明](#函数声明)
* [函数调用](#函数调用)
* [类型](#类型)
  * [常量](#常量)
  * [静态方法和变量类型属性](#静态方法和变量类型属性)
  * [可选类型](#可选类型)
  * [懒加载](#懒加载)
  * [类型推断](#类型推断)
  * [语法糖](#语法糖)
* [函数VS方法](#函数VS方法)
* [内存管理](#内存管理)
  * [延长对象生命周期](#延长对象生命周期)
* [访问控制](#访问控制)
* [控制流](#控制流)
  * [三元操作符](#三元操作符)
* [黄金路径](#黄金路径)
  * [Guard失败](#Guard失败)
* [分号](#分号)
* [圆括号](#圆括号)
* [多行字符串字面量](#多行字符串字面量)
* [不要使用Emoji](#不要使用Emoji)
* [版权声明](#版权声明)
* [引用](#引用)
 

## 正确性

力求使代码编译时没有警告。这条规则从根本禁止了一些文法使用，如推荐使用#selector文而不是用字符串

## 命名

描述性和一致性的命名能使代码更容易阅读和理解，可以参考[API Design Guidelines](https://swift.org/documentation/api-design-guidelines/) 命名规范, 一些重要点如下：

* 调用时力求清晰
* 清晰优于简洁
* 使用驼峰
* 对于类型（包括协议）首字母大写，其他小写
* 包含必要单词省略不需要的单词
* 利用变量、参数和关联类型的角色命名，而非类型
* 为弱类型（Any, AnyObject和NSObject）添加信息，使其表达更加明确
* 力求流畅
* 工厂的方法开头用`make`
* 根据方法具体行为为其命名
* 动词可变方法以-ed结尾，非可变则以-ing结尾
* 名词可变方法遵循formX规则
* 布尔值类型读起来应该是一个断言
* 描述事物的协议，读起来应该是一个名词
* 描述能力的协议，应该以-able或者-ible结尾
* 慎用术语
* 避免缩写
* 遵从先例
* 方法和属性比自由函数更好
* 首字母缩写遵循要都是大写或者小写
* 同一个含义的方法共享一个基础方法名
* 避免返回值重载
* 取一个好的参数名称，启到文档作用
* 命名第一个参数比放在函数名中更好，delegate除外
* 标注闭包和元组
* 利用好默认参数值

### 单调

引用的方法名没有歧义是最重要的，尽可能保持简单

* 方法名没有参数， 你调用 ‘addTarget’
* 方法名有参数标注，你调用 'addTarget(_:action)'
* 方法名有参数标注和类型，你调用 ‘addTarget(_: Any?, action: Selector?)’


### 类前缀

Swift中类别(类，结构体)在编译时会把模块设置为默认的命名空间，所以不用为了区分类别而添加前缀，比如RW。如果担心来自不同模块的两个名称发生冲突，可以在使用时添加模块名称来区分，注意不要滥用模块名称，仅在有可能发生冲突或疑惑的场景下使用。

```
import SomeModule

let myClass = MyModule.UsefulClass()
```

### 委托

当创建自定义委托方法时，第一个未命名参数应该是委托数据源。（UIKit里面有很多例子）

**推荐**

```
func namePickerView(_ namePickerView: NamePickerView, didSelectName name: String)
func namePickerViewShouldReload(_ namePickerView: NamePickerView) -> Bool
```

**不推荐**

```
func didSelectName(namePicker: NamePickerViewController, name: String)
func namePickerShouldReload() -> Bool
```

### 类型推导

利用编译器类型推导去写短小，清晰的代码

**推荐**

```
let selector = #selector(viewDidLoad)
view.backgroundColor = .red
let toView = context.view(forKey: .to)
let view = UIView(frame: .zero)
```

**不推荐**

```
let selector = #selector(ViewController.viewDidLoad)
view.backgroundColor = UIColor.red
let toView = context.view(forKey: UITransitionContextViewKey.to)
let view = UIView(frame: CGRect.zero)
```

### 泛型

泛型类型参数应具有可以描述的，遵守“大驼峰命名”法则，当类型名没有一个明确含义的时候，用大写字母，如 T，U或者V

**推荐**

```
struct Stack<Element> { ... }
func write<Target: OutputStream>(to target: inout Target)
func swap<T>(_ a: inout T, _ b: inout T)
```

**不推荐**

```
struct Stack<T> { ... }
func write<target: OutputStream>(to target: inout target)
func swap<Thing>(_ a: inout Thing, _ b: inout Thing)
```

### 语言

尽量美式英语拼写并且Apple's API保持一致

**推荐**

```
let color = "red"
```

**不推荐**

```
let colour = "red"
```

## 代码组织结构

利用扩展去组织逻辑代码块，每个扩展必须以 `//MARK:-`加上注释

### 协议一致性

尤其当添加一个协议给类型时，推荐添加一个扩展给协议方法，这样保证相关代码集中在一起，从而简化给类型添加协议

**推荐**

```
class MyViewController: UIViewController {
  // class stuff here
}

// MARK: - UITableViewDataSource
extension MyViewController: UITableViewDataSource {
  // table view data source methods
}

// MARK: - UIScrollViewDelegate
extension MyViewController: UIScrollViewDelegate {
  // scroll view delegate methods
}
```

**不推荐**

```
class MyViewController: UIViewController, UITableViewDataSource, UIScrollViewDelegate {
  // all methods
}
```

对于UIkit viewController，利用扩展对声明周期、自定义存取器和IBAction进行分组。

### 无用代码

无用的代码，包括Xcode生成模板代码和占位符应该被删除，除非辅导性说明引导用户去使用注释的代码，
一些方法简单调用父类方法也应该删除。

**推荐**

```
override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
  return Database.contacts.count
}
```

**不推荐**

```
override func didReceiveMemoryWarning() {
super.didReceiveMemoryWarning()
  // Dispose of any resources that can be recreated.
}

override func numberOfSections(in tableView: UITableView) -> Int {
  // #warning Incomplete implementation, return the number of sections
  return 1
}

override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
  // #warning Incomplete implementation, return the number of rows
  return Database.contacts.count
}
```

### 最小化导入

只导入源文件需要module， 例如Foundation够用情况下不需要导入UIKit，导入UIKit就不需要导入Foundation。

**推荐**

```
import UIKit
var view: UIView
var deviceModels: [String]
```

```
import Foundation
var deviceModels: [String]
```

**不推荐**

```
import UIKit
import Foundation
var view: UIView
var deviceModels: [String]
```

```
import UIKit
var deviceModels: [String]
```

### 间隔
* 方法和其他（`if`/`else`/`switch`/`while` 等）首括号应该与首行语句同一行。

**推荐**

```
if user.isHappy {
  // Do something
} else {
  // Do something else
}
```

**不推荐**

```
if user.isHappy
{
  // Do something
}
else {
  // Do something else
}
```

* 方法之间只有一个空行，有助于清晰和组织，当有一个方法有多个段落时意味需要重构成多个函数。
* 在大括号开始和结束之前不需要空行。
* 冒号总是左边没有空格，右边需要保留空格，例外三元操作符、空字典和`#selector`语法 `(_:action:)`。


**推荐**

```
class TestDatabase: Database {
  var data: [String: CGFloat] = ["A": 1.2, "B": 3.2]
}
```

**不推荐**

```
class TestDatabase : Database {
  var data :[String:CGFloat] = ["A" : 1.2, "B":3.2]
}
```

* 一行代码最多70个字符换行。
*  避免行结尾添加空格。
*  每个文件结尾添加一个新行	。


## 注释
当必要的时候，用注释去阐明代码块用途，并且要保持更新或者及时删除，避免代码中出现注释块，因为代码本身起到文档说明作用，如果注释为了生成文档则是例外，避免C方式注释（`/* ... */`）,推荐使用`//` 或者 `///`。


## 类和结构体
### 使用哪一个
结构体是值类型。结构体在使用中没有标识。一个数组包含[a, b, c]和另外一个数组包含[a, b, c]是完全一样的，它们完全可以互相替换，使用第一个还是使用第二个都一样，因为它们代表的是同一个东西。这就是为什么数组是结构体。

类是引用类型。类使用的场景是需要一个标识或者需要一个特定的生命周期。假设你需要对人抽象为一个类，因为两个人，是两个不同的东西。即使两个人有同样的名字和生日，也不能确定这两个人是一样的。但是人的生日是一个结构体，因为日期1950/03/03和另外一个日期1950/03/03是相同的,日期是结构体没有唯一标示。

有时，一些事物应该定义为结构体，但是还需要兼容AnyObject或者已经在以前的历史版本中定义为类（NSDate，NSSet），所以尽可能的注意类和结构体之间的区别。

### 类定义
一个良好设计类的定义：

```
class Circle: Shape {
  var x: Int, y: Int
  var radius: Double
  var diameter: Double {
      get {
        return radius * 2
      }
      set {
        radius = newValue / 2
      }
  }

  init(x: Int, y: Int, radius: Double) {
    self.x = x
    self.y = y
    self.radius = radius
  }

  convenience init(x: Int, y: Int, diameter: Double) {
    self.init(x: x, y: y, radius: diameter / 2)
  }

  override func area() -> Double {
    return Double.pi * radius * radius
  }
}

extension Circle: CustomStringConvertible {
  var description: String {
    return "center = \(centerString) area = \(area())"
  }
  private var centerString: String {
    return "(\(x),\(y))"
  }
}
```

上面的例子展示了下面的设计准则：

* 属性，变量，常量和参数等在声明定义时，其中: 符号后有空格，而: 符号前没有空格，比如x: Int, 和Circle: Shape。
* 如果定义多个变量和数据结构时出于相同的目的和上下文，可以定义在同一行。
* 缩进getter，setter的定义和属性观察器的定义。
* 不需要添加internal这样的默认的修饰符。也不需要在重写一个方法时添加访问修饰符。
* 在扩展里面添加额外功能。
* 通过private修饰符隐藏不共享实现，例如centerString。

### self使用

从简洁角度来看，应该避免使用self，唯一使用地方 in `@escaping`闭包和构造器里面，其他都可以忽略。

### 计算属性
从简洁角度来看，如果只有读，get应该忽略，只有set语句时候get语句才需要。

**推荐**

```
var diameter: Double {
  return radius * 2
}
```

**不推荐**

```
var diameter: Double {
  get {
    return radius * 2
  }
}

```

### Final
正常是不需要将类和成员标记为final，然而有时是值得利用final能说明你的意图。下面例子，box不需要继承，final表达更清晰。

```
// Turn any generic type into a reference type using this Box class.
final class Box<T> {
  let value: T
  init(_ value: T) {
    self.value = value
  }
}
```

## 函数声明
函数声明要尽可能短且在一行， 行内包括头括号：

```
func reticulateSplines(spline: [Double]) -> Bool {
  // reticulate code goes here
}
```

对于有较多参数的函数，每个参数占单独一行且同样缩进：

```
func reticulateSplines(
  spline: [Double], 
  adjustmentFactor: Double,
  translateConstant: Int, comment: String
  ) -> Bool {
    // reticulate code goes here
  }
```

不要用`(Void)`做为input参数，用`()`即可，函数输出和闭包可以用`Void`来代替`()`。

**推荐**

```
func updateConstraints() -> Void {
  // magic happens here
}

typealias CompletionHandler = (result) -> Void
```
**不推荐**

```
func updateConstraints() -> () {
  // magic happens here
}

typealias CompletionHandler = (result) -> ()
```

## 函数调用
单行函数调用应该写成这样：

```
let success = reticulateSplines(splines)
```

如果是多行，每个参数一行且同样缩进：

```
let success = reticulateSplines(
spline: splines,
adjustmentFactor: 1.3,
translateConstant: 2,
comment: "normalize the display")

```

## 闭包表达式
使用尾随闭包仅在闭包表达式在所有参数列表最后一个时，给闭包参数一个可描述性名字

**推荐**

```
UIView.animate(withDuration: 1.0) {
  self.myView.alpha = 0
}

UIView.animate(withDuration: 1.0, animations: {
  self.myView.alpha = 0
}, completion: { finished in
  self.myView.removeFromSuperview()
})
```

**不推荐**

```
UIView.animate(withDuration: 1.0, animations: {
  self.myView.alpha = 0
})

UIView.animate(withDuration: 1.0, animations: {
  self.myView.alpha = 0
}) { f in
  self.myView.removeFromSuperview()
}
```

对于上下文很清晰的单闭包表达式，用隐式的返回值：

```
attendeeList.sort { a, b in
  a > b
}
```

链式调用在上下文中使用尾随闭包会更清晰和易读，至于空格，换行和匿名参数不做具体要求。

```
let value = numbers.map { $0 * 2 }.filter { $0 % 3 == 0 }.index(of: 90)

let value = numbers
.map {$0 * 2}
.filter {$0 > 50}
.map {$0 + 10}
```

## 类型
尽可能优先使用swift原生类型和表达式，objective-c方法仍可以使用，swift提供到objective-c的桥接功能。

**推荐**

```
let width = 120.0                                    // Double
let widthString = "\(width)"                         // String
```

**少推荐**

```
let width = 120.0                                    // Double
let widthString = (width as NSNumber).stringValue    // String
```

**不推荐**

```
let width: NSNumber = 120.0                          // NSNumber
let widthString: NSString = width.stringValue        // NSString
```

在绘制代码里面，用CGFloat能够减少没有必要转换，让代码更加简洁。

### 常量
常量定义用`let`,变量定义用`var`, 如果变量值不变化时用`let`替代`var`，

**提示**： 一个好的技巧是定义任何东西都用`let`， 只有在编译器警告的时候用`var`，

你可以使用类型属性来定义类型常量而不是实例常量，使用`static let` 可以定义类型属性常量。 这样方式定义类型属性整体上优于全局常量，因为更容易区分于实例属性. 比如:

```
enum Math {
  static let e = 2.718281828459045235360287
  static let root2 = 1.41421356237309504880168872
}

let hypotenuse = side * Math.root2
```

**提示**
用枚举的好处是变量不会被意外初始化，且在一个独立命名空间。

**不推荐**

```
let e = 2.718281828459045235360287  // pollutes global namespace
let root2 = 1.41421356237309504880168872

let hypotenuse = side * root2 // what is root2?
```

### 静态方法和变量类型属性
静态方法和类型属性的类似于全局方法和全局属性，应该克制使用，它们的使用场景在于如果某些功能局限于特别的类型或和Objective-C 互相调用。

### 可选类型
可以变量和函数返回值声明为可选类型(?)，如果nil值可以接受。

当你确认实例变量会稍晚在使用前初始化，可以在声明时使用!来隐式的拆包类型，比如在viewDidLoad中会初始化的子视图。当你访问一个可选类型值时，如果只需要访问一次或者在可选值链中有多个可选类型值时，请使用可选类型值链：

```
textContainer?.textLabel?.setNeedsDisplay()
```

当命名可选类型变量，避免使用`optionalString `或者`maybeView `命名，因为在`option-ness`已经在类型声明中。
在可选值绑定时，直接映射原始的命名比使用诸如unwrappedView 或 actualLabel更好。

**推荐**

```
var subview: UIView?
var volume: Double?

// later on...
if let subview = subview, let volume = volume {
  // do something with unwrapped subview and volume
}

// another example
UIView.animate(withDuration: 2.0) { [weak self] in
  guard let self = self else { return }
  self.alpha = 1.0
}
```

**不推荐**

```
var optionalSubview: UIView?
var volume: Double?

if let unwrappedSubview = optionalSubview {
  if let realVolume = volume {
    // do something with unwrappedSubview and realVolume
  }
}

// another example
UIView.animate(withDuration: 2.0) { [weak self] in
  guard let self = self else { return }
  self.alpha = 1.0
}
```

### 懒加载
考虑使用懒加载可以更为细粒度的控制对象生命周期，尤其是对`UIViewController`视图的懒加载，也可以调用`{}()`的闭包或者一个私有的工厂方法。
如下：

```
lazy var locationManager = makeLocationManager()

lazy var locationManager = makeLocationManager()
  private func makeLocationManager() -> CLLocationManager {
  let manager = CLLocationManager()
  manager.desiredAccuracy = kCLLocationAccuracyBest
  manager.delegate = self
  manager.requestAlwaysAuthorization()
  return manager
}
```

**提示：**

* `Location manager`的负作用会弹出对话框要求用户提供权限，这就是细粒度控制意义所在。

### 类型推断
推荐紧凑的代码，让编译器推断出每个实例的常量或变量的类型。类型推断也适用于小的非空数组和字典。必要时，指定特定类型，如`CGFloat`或`Int16`。

**推荐**

```
let message = "Click the button"
let currentBounds = computeViewBounds()
var names = ["Mic", "Sam", "Christine"]
let maximumWidth: CGFloat = 106.5
```

**不推荐**

```
let message: String = "Click the button"
let currentBounds: CGRect = computeViewBounds()
var names = [String]()
```

#### 为空数组和字典键入注解

对于空数组和字典，请使用类型注解。（对赋予一个大的、多行文字的数组或者字典，使用类型注解）。

**推荐**

```
var names: [String] = []
var lookup: [String: Int] = [:]
```

**不推荐**

```
var names = [String]()
var lookup = [String: Int]()
```

**注意:**
遵循规范意味着选择描述性名称比任何时候更加重要。

### 语法糖
使用简短类型声明语法，而不是全称语法。

**推荐**

```
var deviceModels: [String]
var employees: [Int: String]
var faxNumber: Int?
```

**不推荐**

```
var deviceModels: Array<String>
var employees: Dictionary<Int, String>
var faxNumber: Optional<Int>
```

## 函数 VS 方法

应克制使用没有依附类或类型的自由函数，尽可能优先使用类或类型的方法而不是自由函数，这样有助于代码的可读性和易发现性。

自由函数使用场景是跟任何特定类型或实例无关联。

**推荐**

```
let sorted = items.mergeSorted()  // 很容易被发现
rocket.launch()  // 作用于模型
```

**不推荐**

```
let sorted = mergeSort(items)  // 更难发现
launch(&rocket)
```

**自由函数例外**

```
let tuples = zip(a, b)  // 天然充当自由函数（对称)
let value = max(x, y, z)  // 另一个自然函数
```

## 内存管理
代码应避免循环引用，分析对象图谱，使用`weak`和`unowned`阻止强引用，另一种选择是使用值类型（`struct`，`enum`）来完全防止循环引用。

### 延长对象的生命周期
使用`[weak self]`和`guard let self = self else { return }`惯用语法延长对象的生命周期，`[weak self]`更加优于`[unowned self]`，`self`的生命周期会超出闭包。显式延长生命周期更优于可选链。

**推荐**

```
resource.request().onComplete { [weak self] response in
  guard let self = self else {
    return
  }
  let model = self.updateModel(response)
  self.updateUI(model)
}
```

**不推荐**

```
// 如果在响应返回之前释放self，则可能会崩溃
resource.request().onComplete { [unowned self] response in
  let model = self.updateModel(response)
  self.updateUI(model)
}
```

**不推荐**

```
// 在更新模型和更新UI之间可能会发生deallocate
resource.request().onComplete { [weak self] response in
  let model = self?.updateModel(response)
  self?.updateUI(model)
}
```

## 访问控制

使用`private`和`fileprivate`会增加清晰度并提升封装，`private`优于`fileprivate`，除非在编译需要的时候才使用`fileprivate`。

只有完全访问控制时才显式使用`open`，`public`和`internal`。
访问控制符一般放在属性修饰符的最前面， 例外`static`修饰符 ,`@IBAction`, `@IBOutlet`和`@discardableResult`等属性。

**推荐**

```
private let message = "Great Scott!"
class TimeMachine {  
  private dynamic lazy var fluxCapacitor = FluxCapacitor()
}
```

**不推荐**

```
fileprivate let message = "Great Scott!"
class TimeMachine {  
  lazy dynamic private var fluxCapacitor = FluxCapacitor()
}
```

## 控制流
循环`for-in`优于`while`。

**推荐**

```
for _ in 0..<3 {
  print("Hello three times")
}

for (index, person) in attendeeList.enumerated() {
  print("\(person) is at position #\(index)")
}

for index in stride(from: 0, to: items.count, by: 2) {
  print(index)
}

for index in (0...3).reversed() {
  print(index)
}
```

**不推荐**

```
var i = 0
while i < 3 {
  print("Hello three times")
  i += 1
}


var i = 0
while i < attendeeList.count {
  let person = attendeeList[i]
  print("\(person) is at position #\(i)")
  i += 1
}
```

### 三元操作符
三元操作符`?:`在于使代码更清楚和整洁，单个判断条件的时候可以考虑使用三元操作符，多个判断条件的时候，使用if会使代码更加具有可读性，或者将中间结果使用变量存储代替。一般而言，三元操作符最好的使用场景是给一个变量赋值或者选择哪一个值应该被使用。

**推荐**

```
let value = 5
result = value != 0 ? x : y

let isHorizontal = true
result = isHorizontal ? x : y
```

**不推荐**

```
result = a > b ? x = c > d ? c : d : y
```

## 黄金路径
当编写带有条件语句时，左边的距离是黄金路径，换句话说，不要写很多if嵌套语句，应该多个return，guard就为此而生的。

**推荐**

```
func computeFFT(context: Context?, inputData: InputData?) throws -> Frequencies {
  guard let context = context else {
    throw FFTError.noContext
  }
  guard let inputData = inputData else {
    throw FFTError.noInputData
  }

  // use context and input to compute the frequencies
  return frequencies
}
```

**不推荐**

```
func computeFFT(context: Context?, inputData: InputData?) throws -> Frequencies {
  if let context = context {
    if let inputData = inputData {
      // use context and input to compute the frequencies
      return frequencies
    } else {
      throw FFTError.noInputData
    }
  } else {
    throw FFTError.noContext
  }
}
```

当有多个条件需要用`guard`或`if let`解包，可用复合语句避免嵌套, 如下：

**推荐**

```
guard 
  let number1 = number1,
  let number2 = number2,
  let number3 = number3 
  else {
    fatalError("impossible")
  }
  // do something with numbers
```

**不推荐**

```
if let number1 = number1 {
  if let number2 = number2 {
    if let number3 = number3 {
      // do something with numbers
    } else {
      fatalError("impossible")
    }
  } else {
    fatalError("impossible")
  }
} else {
  fatalError("impossible")
}
```

### Guard失败

guard用于退出场景，如return, throw, break, continue, 和fatalError（）等，避免出现大的代码块， 如果存在多个退出点有清理代码，使用`defer`块。

## 分号

Swift不强制每条语句后面有分号，只在一行有多个语句的时候需要，不过不推荐。

**推荐**

```
let swift = "not a scripting language"
```

**不推荐**

```
let swift = "not a scripting language";
```
**注意：**swift与js是非常不一样的，js删掉分号一般认为是不太安全的。

## 圆括号
条件判断时圆括号不是必须的，建议省略。

**推荐**

```
if name == "Hello" {
  print("World")
}
```

**不推荐**

```
if (name == "Hello") {
  print("World")
}
```
不过，当多表达式，圆括号可以使得代码读起来更清晰, 如下：

```
let playerMark = (player == current ? "X" : "O")
```

## 多行字符串字面量

当编写长的字符串字面量，应该使用多行字符串字面量，开始处不包含文本，后续文本保持同样缩进。

**推荐**

```
let message = """
You cannot charge the flux \
capacitor with a 9V battery.
You must use a super-charger \
which costs 10 credits. You currently \
have \(credits) credits available.
"""
```

**不推荐**

```
let message = """You cannot charge the flux \
capacitor with a 9V battery.
You must use a super-charger \
which costs 10 credits. You currently \
have \(credits) credits available.
"""
```

**不推荐**

```
let message = "You cannot charge the flux " +
"capacitor with a 9V battery.\n" +
"You must use a super-charger " +
"which costs 10 credits. You currently " +
"have \(credits) credits available."
```
## 不要使用Emoji

在你的工程中不要使用Emoji，emoji看上去可爱，影响阅读代码的流畅性。


## 版权声明
下面版权声明应该放在每个源文件开头：

```
/*
* Copyright (c) 2017 Baidu, Inc. All Rights Reserved.
*
*  Licensed under the Apache License, Version 2.0 (the "License");
*  you may not use this file except in compliance with the License.
*  You may obtain a copy of the License at
*
*     http://www.apache.org/licenses/LICENSE-2.0
*
*  Unless required by applicable law or agreed to in writing, software
*  distributed under the License is distributed on an "AS IS" BASIS,
*  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
*  See the License for the specific language governing permissions and
*  limitations under the License.
*/
```

## 引用

* [API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)
* [Swift Style Guide](https://github.com/raywenderlich/swift-style-guide)
* [The Swift Programming Language 中文版](http://special.csdncms.csdn.net/the-swift-programming-language-in-chinese/index.shtml)



















































































