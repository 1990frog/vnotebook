In this Java exceptions tutorial, learn what an exception is in Java, what is a checked exception and how it is different from a unchecked exception. We will also learn some best practices around Java checked exceptions.

[TOC]

# 1. What is an exception in Java?
```
“An exception is an unexpected event that occurs during the execution of a program that disrupts the normal flow of instructions.”
```

In Java, all errors and exceptions are represented with Throwable class. When an error occurs within a method, the method creates an object (of any subtype of Throwable) and hands it off to the runtime system. The object, called an exception object.

Exception object contains information about the error, including its type and the state of the program when the error occurred. Creating an exception object and handing it to the runtime system is called throwing an exception.

## 1.1. Exception handling
We have two choices when an exception object is created in our application.

+ Either we will handle it within method
+ Or we can pass it to the caller method to let it handle.

This is very important decision to be made while setting the responsibilities of a method. A method should clearly indicate that what all exceptional scenarios it will handle and which it will not. It is defined in method syntax using throws clause.

To handle the exception, We must catch the exception in catch section of try-catch block.

```
If an exception is not handled in the application, then it will propagate to JVM and JVM will usually terminate the program itself.
```

# 2. Checked vs unchecked exceptions in Java
## 2.1. Exception Hierarchy
In Java, exceptions are broadly categorized into two sections: checked exceptions and unchecked exceptions.
![ExceptionHierarchyJava](_v_images/20191105172906005_58041845.png)

## 2.2. Checked Exceptions
Java forces you to handle these error scenarios in some manner in your application code. They will come immediately into your face, once you start compiling your program. You can definitely ignore them and let them pass to JVM, but it’s bad habit. Ideally, you must handle these exceptions at suitable level inside your application so that you can inform the user about failure and ask him to retry/ come later.

Generally, checked exceptions denote error scenarios which are outside the immediate control of the program. They occur usually interacting with outside resources/network resources e.g. database problems, network connection errors, missing files etc.

```
Checked exceptions are subclasses of Exception class.
```

Example of checked exceptions are : ClassNotFoundException, IOException, SQLException and so on.

Checked Exception Example
FileNotFoundException is a checked exception in Java. Anytime, we want to read a file from filesystem, Java forces us to handle error situation where file may not be present in place.

Try to read file with handle FileNotFoundException
```java
public static void main(String[] args) 
{
    FileReader file = new FileReader("somefile.txt");
}
```
In above case, you will get compile time error with message – Unhandled exception type FileNotFoundException.

To make program able to compile, you must handle this error situation in try-catch block. Below given code will compile absolutely fine.

Try to read file with exception handling
```java
public static void main(String[] args) 
{
    try
    {
        FileReader file = new FileReader("somefile.txt");
    } 
    catch (FileNotFoundException e) 
    {
        //Alternate logic
        e.printStackTrace();
    }
}
```

## 2.3. Unchecked Exceptions
Java also provides UncheckedExceptions, the occurrences of which are not checked by the compiler. They will come into life/occur into your program, once any buggy code is executed.

A method is not forced by compiler to declare the unchecked exceptions thrown by its implementation. Generally, such methods almost always do not declare them, as well.

Unchecked Exceptions are subclasses of RuntimeException. Example of unchecked exceptions are : ArithmeticException, ArrayStoreException, ClassCastException and so on.

“The strange thing is that RuntimeException is itself subclass of Exception i.e. all unchecked exception classes should have been checked exceptions implicitly, BUT they are not.”

### Unchecked Exception Example
Checkout the given code below. Above code does not give any compile time error. But when you the example, it throws NullPointerException. NullPointerException is unchecked exception in Java.

JVM not forcing you to check NullPointerException
```java
public static void main(String[] args) 
{
    try
    {
        FileReader file = new FileReader("pom.xml");
         
        file = null;
         
        file.read();
    } 
    catch (IOException e) 
    {
        //Alternate logic
        e.printStackTrace();
    }
}
```

Remember the biggest difference between checked and unchecked exceptions is that checked exceptions are forced by compiler and used to indicate exceptional conditions that are out of the control of the program (for example, I/O errors), while unchecked exceptions are occurred during runtime and used to indicate programming errors (for example, a null pointer).

# 3. Java exception handling best practices
1. Checked exceptions can be used when a method cannot do what its name says it does. e.g. A method named prepareSystem() which pre-populate configuration files and do some configuration using them, can declare throwing              FileNotFoundException which implies that method uses configuration files from file system.
2. Checked exceptions ideally should never be used for programming errors, but absolutely should be used for resource errors and for flow control in such cases.
3. Throw only those exceptions which a method can not handle by any mean. Method should first try to handle it as soon as it encounters. Throw the exception only if it is not possible to handle inside method.
4. A good way to define method signatures is to declare exceptions close to method name. If your method is named openFile, then it is expected to throw FileNotFoundException?. If your method is named findProvider, then it is expected to throw NoSuchProviderException.
Also, these type of exceptions should be made checked exceptions as it forces the caller to deal with the problems that are inherent to the semantic of your methods.
5. Rule is if a client can reasonably be expected to recover from an exception, make it a checked exception. If a client cannot do anything to recover from the exception, make it an unchecked exception.
In reality, most applications will have to recover from pretty much all exceptions including NullPointerException, IllegalArgumentExceptions and many other unchecked exceptions. The action / transaction that failed will be aborted but the application has to stay alive and be ready to serve the next action / transaction.
The only time it is normally legal to shut down an application is during startup. For example, if a configuration file is missing and the application cannot do anything sensible without it, then it is legal to shut down the application.

# 4. Conclusion
In this post, we learned the difference between checked vs unchecked exceptions in Java, along with how to handle unchecked exceptions, exception hierarchy in Java with examples.
