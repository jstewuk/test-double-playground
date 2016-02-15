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

