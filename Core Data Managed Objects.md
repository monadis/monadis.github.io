## Core Data Managed Objects



#### transient property

```swift
var fullnameChanged = false

extension Employee {


var firstname: String? {
  get {
    willAccessValueForKey("firstname")
    let firstname = primitiveValueForKey("firstname") as? String
    didAccessValueForKey("firstname")
    return firstname
  }

  set {
    willChangeValueForKey("firstname")
    setPrimitiveValue(newValue, forKey: "firstname")
    didChangeValueForKey("firstname")
    fullnameChanged = true
  }
}

	@NSManaged var primitiveFirstname: String?
	@NSManaged var lastname: String?
	var fullname: String? {
		get {
			if fullnameChanged {
				let firstname = primitiveValueForKey("firstname") as? String
				let lastname = primitiveValueForKey("lastname") as? String
				let newFullname = firstname! + " " + lastname!
				setPrimitiveValue(newFullname, forKey: "fullname")
				fullnameChanged = false
				return newFullname
			} else {
				willAccessValueForKey("fullname")
				let fullname = primitiveValueForKey("fullname") as? String
				didAccessValueForKey("fullname")
				return fullname
			}
		}
	}
	@NSManaged var date: NSDate?

}

```



#### KVO

NSOperation은 isExecuting등의 상태변화를 표현하기 위해 willChangeValueForKey...등의 메소드들을 이용한다.

```objective-c
// NSOperation

- (void) start {
    Log(@"Starting %@",self);
    [self willChangeValueForKey: @"isExecuting"];
    _isExecuting = YES;
    [self didChangeValueForKey: @"isExecuting"];
}

- (void) finish {
    [self willChangeValueForKey: @"isExecuting"];
    [self willChangeValueForKey: @"isFinished"];
    _isExecuting = NO;
    _isFinished = YES;
    [self didChangeValueForKey: @"isFinished"];
    [self didChangeValueForKey: @"isExecuting"];
}
```



## 참고

http://petersteinberger.com/blog/2012/dont-call-willchangevalueforkey/

