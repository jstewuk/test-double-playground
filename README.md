# Compare struct and class options for Swift test doubles

A friend and I were discussing swift and testing. He wanted to implement his TDD methodology in Swift.  In particular he was interested in ways to create test doubles in swift. He wasn't sure whether he had to use a class or if using a struct might be an option.

 The intent is captured in this sudocode:

     setup:
     init collaborator()
     init sut(collaborator)
 
     test:
     mutate collaborator
     call sut.someMethod()
 
 In order to mutate the collaborator, it has to be a reference type.  However, it doesn't seem right to force the actual collaborator to be a class based on testing requirements, so I looked at options for using protocols and generics to create a type that could be either a class or a struct.  
 
 I'll go through this in several steps, first I'll create a struct test double and try to follow the sudocode.


##### Create a simple test double that can mutate some state
```swift
struct TestDouble {
    private var testString = "Original Test Double"
    
    mutating func updateString(string: String) {
        testString = string
    }
    
    func description() -> String {
        return testString
    }
}
```

##### Create a class to test that uses the struct, this will be the system under test (SUT)
```swift
class Bar {
    var testDouble: TestDouble
    
    init(testDouble: TestDouble) {
        self.testDouble = testDouble
    }
    
    func baz() -> String {
        return testDouble.description()
    }
}
```

##### Test it..
*Setup:*
```swift
var testDouble = TestDouble()                           // init collaborator()
testDouble.description()
var bar = Bar(testDouble: testDouble)                   // init sut(collaborator)
bar.baz()
```
`>>>"Original Test Double"`  
*Test:*
```swift
testDouble.updateString("This is the mutated string")   // mutate the collaborator
testDouble.description()
bar.baz()                                               // call sut.someMethod()
```
`>>>"Original Test Double"`
 
Oops, we wanted the mutated string.  It looks like the struct isn't going to work.
Perhaps creating a protocol with a default extension and making the SUT generic so it can work with either a struct or a class that conforms to the protocol.

##### Try mocking the double with a protocol
```swift
protocol PDouble {
    init(testString: String)
    var testString: String {get set}
    mutating func updateString(string: String)
    func description() -> String
}
```

##### Create a default extension
```swift
extension PDouble {
    mutating func updateString(string: String) {
        self.testString = string
    }
    
    func description() -> String {
        return testString
    }
}
```
##### Implement concrete class and struct types to use in the test
```swift
struct StructDouble: PDouble {
    var testString: String
}

final class ClassDouble: PDouble {
    var testString: String = ""
    required init(testString: String) {
        self.testString = testString
    }
}
```
##### Test it
```swift
var clDouble = ClassDouble(testString: "Class double test string")
var stDouble = StructDouble(testString: "Struct double test string")
clDouble.description()
```
`>>>"Class double test string"` 
```swift
stDouble.description() 
```
`>>>"Struct double test string"`  
##### Make the SUT class generic, so it can use either the struct or the class
```swift
class Bar2<DoubleType: PDouble > {
    var testDouble: DoubleType
    
    init(testDouble: DoubleType) {
        self.testDouble = testDouble
    }
    
    func baz() -> String {
        return testDouble.description()
    }
}
```
##### Create the SUT
```swift
let barClass = Bar2(testDouble: clDouble)
let barStruct = Bar2(testDouble: stDouble)

barClass.baz()
barStruct.baz()
```
##### Mutate the test double
```swift
clDouble.updateString("Mutated class double string")
stDouble.updateString("Mutated struct double string")

barClass.baz()
```
`>>>"Mutated class double string"` 
```swift
barStruct.baz()
```
`>>>"Struct double test string"` 

`PDouble` is a protocol that captures the testDouble's behavior and can be implemented either as a class or a struct.
