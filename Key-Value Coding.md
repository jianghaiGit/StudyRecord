# Key-Value Coding

key-valueCoding 是一种间接访问对象属性的途径。它使用字符串来辨识属性，而不是通过存取方法或者实例变量来访问。key-valueCoding需要的方法定义在__NSKeyValueCoding.h__,默认由__NSObject__实现。

和基本类型、结构体作为属性一样，key-valueCoding也支持对象作为属性。非对象类型的参数和返回类型会自动进行装箱和拆箱操作。
  
  key-value coding的API提供了通过key查询对象和设置对象值的通用方法。

在你设计应用中的对象的时候你应该定义一个包含你模型所有property key的集合
，同时实现相符合的存取方法。

# 使用Key_Value Coding让代码更简洁

通过使标识符和你想要显示的属性键相同，你可以明显的简化你的代码。 

***未使用key-value coding***

```
-(id)tableView:(NSTableView *)tableview 
objectValueForTableColumn:(id)column row:(NSInteger)row
{

	ChildObject *child = [childrenArray objectAtIndex:row];
	
	if ([[column identifier] isEqualToString:@"name"]) {
        return [child name];
    }
    if ([[column identifier] isEqualToString:@"age"]) {
        return [child age];
    }
    if ([[column identifier] isEqualToString:@"favoriteColor"]) {
        return [child favoriteColor];
    }
}
```
 ***使用key-value coding***

```
- (id)tableView:(NSTableView *)tableview
      objectValueForTableColumn:(id)column row:(NSInteger)row {
 
    ChildObject *child = [childrenArray objectAtIndex:row];
    return [child valueForKey:[column identifier]];
}
```

# Terminology

key-value coding可以用来存取三种不同的对形象类型：属性、一对一的关系、一对多的关系，属性可以是以上的三种类型。

* attributes:包含NSString、BOOL等基础类型，值对象，例如NSNumber还有其他不可变的对象类型(NSColor等)
* 一对一关系：属性拥有自己的属性，自己的属性的改变不会影响自身，比如UIView实例的父view就是一个一对一的关系
* 一对多的关系: 由对象的集合类型构成，例如NSArray、NSSet。key-value coding 允许使用元素为自定义类的集合。

# Key-Value Coding的原理
##键和键路径
###key
一个键为一个字符串明确的标志了一个对象的某个属性。典型的就是一个键对应接受对象的存取方法或者实例变量的名字。__key必须是ASCII编码，使用小写字母开头，不能包含空格。__(例如：payee, openingBalance, transactions and amount)
###key path
keypath是一个由点和多个key值组成的字符串。其中的第一个key属于接受者，后面的每一个key属于前面的keypath取出的值。例如，keypath address.street会从接收者中取出key为address的值，然后street属性会确定属于address对象

#使用Key-Value Coding获取属性的值
方法` valueForKey: `返回接收者中key对应的值。如果存取器或者实例变量与key对应，` valueForKey: `的接收者会向自己发送一个`valueForUndefinedKey: `消息，`valueForUndefinedKey: `的默认实现会抛出一个NSUndefinedKeyException,子类可以重写这个方法。

同样的`valueForKeyPath: `返回消息接收者中keypath对应的值，如果keypath中任意一个对应key的对象不符合key-value coding，接收者会向自身发送`valueForUndefinedKey:`消息。

方法`dictionaryWithValuesForKeys:`会查找消息接收者中所有的和数组中key对应的value，返回的NSDictionary包含了所有的key对应的value和key本身。


```
	集合对象，NSArray、NSSet和NSDictionary，不能包含nil值，不过可以用NSNull代替。
dictionaryWithValuesForKeys:和setValuesForKeysWithDictionary:会自动在nil和
NSNull两者中转换。
```

如果一个keypath中包含一个key对应一对多的属性，同时这个key不是keypath中的最后一个key，那么返回值会是一个集合类型，包含了这个key对应的值。例如：accounts.transactions.payee ,accounts.transactions的value是一个NSArray，那么accounts.transactions.payee将返回一个数组，包含所有transactions中每一个对象对应的payee的值。
#使用Key-Value Coding设置属性的值
`setValue:forKey:`使用传入的值设置消息接收者中对应key的值。默认的`setValue:forKey:`会自动将NSValue对象解包为数值类型或者结构体并分配给对应的属性。

如果key不存在receiver会被发送一个`setValue:forUndefinedKey: `消息，默认的`setValue:forUndefinedKey: `会抛出一个NSUndefinedKeyException，子类可以重写这个方法操作这个请求。`setValue:forKeyPath:`相似，可以handle keypath。

`setValuesForKeysWithDictionary:`使用传入的字典设置消息接收者的值，使用传入的dictionary中的key去识别接收者中的属性。默认的实现会对应每一个键值对调用`setValue:forKey:`,同时调用中会将value为nil的值转换成NSNull。

当设置一个非对象类型的值为nil的时候，接收者会给自己发送`setNilValueForKey:`消息,`setNilValueForKey:`默认实现会抛出NSInvalidArgumentException。有特殊需求的话可以重写`setValuesForKeysWithDictionary:`，替换value,然后调用`setValue:forKey:`

#点语法与Key-Value Coding
kvc和点语法是正交的，使用kvc不一定要使用点语法，也可以在kvc之外使用点语法。在kvc中点语法用来在keypath中界定节点。如果你使用点语法访问了一个属性，那么你就调用了接收者的标准存取方法。

###example:
```
@interface MyClass
@property NSString *stringProperty;
@property NSInteger integerProperty;
@property MyClass *linkedInstance;
@end
```
###在KVC中你可以像下面这样使用

```
MyClass *myInstance = [[MyClass alloc] init];
NSString *string = [myInstance valueForKey:@"stringProperty"];
[myInstance setValue:@2 forKey:@"integerProperty"];

```

###点语法

```
MyClass *anotherInstance = [[MyClass alloc] init];
myInstance.linkedInstance = anotherInstance;
myInstance.linkedInstance.integerProperty = 2;
```
###与上面的结果相同
```
MyClass *anotherInstance = [[MyClass alloc] init];
myInstance.linkedInstance = anotherInstance;
[myInstance setValue:@2 forKeyPath:@"linkedInstance.integerProperty"];
```

#KVC存取方法
为了让KVC能找出对应的存取方法供`valueForKey:`、`setValue:forKey:`、`mutableArrayValueForKey:`、
`mutableSetValueForKey:`调用，你需要实现kvc的存取方法。格式为`-set<Key>/-<key>`（key是占位符，代表属性的名称）。
example:属性name，存取方法应该为`-setName:和-name.`
###存取方法的通用模式
通常的获取方法是-<key>,返回值为对象、数值或者数据结构，对BOOL类型的属性来说-is<Key>也是支持的。

@property BOOL hidden;
`- (BOOL)hidden {
   return ...;
}`
`- (BOOL)isHidden {
   return ...;
}`这两个getter方法都可是合法的。

为了让属性和一对一的关系支持`setValue:forKey:`方法，`-set<key>:`必须实现：`- (void)setHidden:(BOOL)flag {
   return;
}`

