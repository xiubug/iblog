---
title: 征战Objective-C
tags:
  - note
categories:
  - Objective-C
date: 2019-04-07 09:40:48
---

## Oc语言语法快速预览

### 源代码文件扩展名对比：
|            | 头文件        | 实现文件        | 
| ------     | ------       | ------     | 
| c语言       | .h           | .c      |
| c++语言     | .h           |.cpp     |
| oc语言      | .h           |.m      |
| oc&c++     | .h           |.mm     |

### 类定义
``` Objective-C
// 类名SimpleClass 继承 NSObject 类
@interface SimpleClass : NSObject

@end
```

### 类的属性申明
``` Objective-C
// 类名Person 继承 NSObject 类
@interface Person : NSObject

// NSString 类型的对象，因为前面有个 *（表示指针，指针指向一块堆内存）
@property NSString *firstName;

@property NSNumber *yearOfBirth;

// 基础类型
@property int yearOfBirth;

// 只读属性
@property (readonly) NSString * fristName;

@property NSString *lastName;

@end
```

### 方法
#### 减号方法（普通方法又称对象方法）申明
``` Objective-C
@interface Person: NSObject

- (void)someMethod;

- (void)someMethodWithValue:(SomeType)value;

- (void)someMethodWithFirstValue:(SomeType)info1 secondValue:(AnotherType)info2;

@end
```

#### 加号方法（类方法，又称静态方法）申明
``` Objective-C
@interface NSString: NSObject

+ (id)string;

+ (id)stringWithString:(NSString *)aString;

+ (id)stringWithFormat:(NSString *)format;

+ (id)stringWithContentsOfFile:(NSString *)path encoding:(NSStringEncoding)enc error:(NSError **)error;

+ (id)stringWithCString:(const char *)cString encoding:(NSStringEncoding)enc;

@end
```

### 类的实现
``` Objective-C
#import "XYZPerson.h"

@implementation XYZPersion

@end
```

### 完整的例子
``` Objective-C
// XYZPerson.h文件
@interface XYZPerson : NSObject

- (void)sayHello;

@end

// XYZPerson.m文件
#import "XYZPerson.h"

@implementation XYZPerson

- (void)sayHello {
  // @ 表示这是一个 Oc 类型字符串，不加 @，表示是一个纯 C 语言的字符串
  NSLog(@"Hello, World!");
}

@end
```


