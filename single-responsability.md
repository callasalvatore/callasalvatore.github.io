# Elevating Code Quality: Separating Concerns with Dependency Injection and Interfaces in C#

In the realm of software development, the pursuit of clean, maintainable, and efficient code stands as a testament to the craftsmanship of a skilled 
developer. One of the guiding principles in this pursuit is the Single Responsibility Principle (SRP), which posits that a class should have a 
singular reason to change. In this article, we embark on a journey to explore this principle and other best practices, 
using Dependency Injection in C# to finely disentangle concerns within our code.

## The initial scenario - A testing challenge

Our journey commences with an examination of the initial scenario. In this scenario, we are presented with a ShoppingCart class. 
While fully functional, it grapples with a dual role—managing the shopping cart and processing orders. 
This dual responsibility, as we shall see, poses a significant challenge when it comes to testing.

```csharp
using System;
using System.Collections.Generic;

public class ShoppingCart
{
    private List<string> items = new List<string>();

    public void AddItem(string item)
    {
        items.Add(item);
        Console.WriteLine($"Added {item} to the cart.");
    }

    public void Checkout()
    {
        Console.WriteLine("Processing the order...");
        // Logic for processing the order
        Console.WriteLine("Order processed successfully.");
    }
}

```

To underscore this challenge, let's delve into a practical test. Here is a simple test using the initial version of the ShoppingCart class. 
We aim to verify the functionality of the Checkout process, but we swiftly encounter a roadblock:

```csharp
using Xunit;
using System;

public class ShoppingCartTests
{
    [Fact]
    public void Checkout_ShouldProcessOrder()
    {
        // Arrange
        var shoppingCart = new ShoppingCart();

        // Act
        shoppingCart.AddItem("Product A");
        shoppingCart.AddItem("Product B");
        shoppingCart.Checkout();

       // Assert
        // Our goal is to verify that the order is processed successfully during the checkout.
        // However, due to the intertwined nature of the code, we face significant testing challenges:

        // Challenge 1: Lack of Isolation
        // In this scenario, the order processing logic is deeply entwined with the shopping cart management within the Checkout method.
        // This means we can't test the order processing in isolation. We're essentially testing both shopping cart management and order
        // processing simultaneously, making it impossible to isolate issues or verify the order processing logic independently.

        // Challenge 2: Mocking Complexity
        // To test the checkout process thoroughly, we would ideally want to create unit tests. However, without separation of concerns,
        // achieving this requires complex mocking frameworks to simulate the dependencies of the Checkout method. This complexity not only
        // makes tests harder to write but also less reliable and harder to maintain.

        // Challenge 3: Limited Test Scenarios
        // Furthermore, because we can't easily isolate the order processing logic, testing different scenarios (e.g., edge cases, errors)
        // becomes cumbersome. We are limited in our ability to create comprehensive tests that ensure the Checkout method behaves correctly
        // under various conditions.

        // In essence, the intertwined nature of the code makes writing effective, focused tests for the checkout process a daunting task.
        // This challenge emphasizes the need for better code organization and separation of concerns, which we will address in the next steps
        // of our journey.

        // As software developers, we aim to not only ensure the functionality of our code but also its testability.
        // Robust, testable code allows us to have confidence in our software's behaviour and its ability to adapt to future changes. Our journey towards
        // improved code quality through Dependency Injection and Interfaces begins here.
    }
}

```

## Separating concerns with different classes and interfaces

In recognition of the testing challenge and with a commitment to heighten code quality, we embark on the pivotal step of concern separation. 
We introduce a novel class, OrderProcessor, designed exclusively to manage order processing. Simultaneously, we reshape the ShoppingCart 
class to concentrate solely on shopping cart management.

`IShoppingCart` and `IOrderProcessor` Interfaces:

```csharp
public interface IShoppingCart
{
    void AddItem(string item);
    void Checkout();
}

public interface IOrderProcessor
{
    void ProcessOrder();
}
```

ShoppingCart class implementing IShoppingCart:
```csharp
using System;
using System.Collections.Generic;

public class ShoppingCart : IShoppingCart
{
    private List<string> items = new List<string>();
    private readonly IOrderProcessor orderProcessor;

    public ShoppingCart(IOrderProcessor orderProcessor)
    {
        this.orderProcessor = orderProcessor;
    }

    public void AddItem(string item)
    {
        // Simulate adding an item to the shopping cart.
        items.Add(item);

        // Provide feedback in the console.
        Console.WriteLine($"Added {item} to the cart.");
    }

    public void Checkout()
    {
        // Simulate the checkout process.
        Console.WriteLine("Processing the order...");

        // Delegate the order processing to the OrderProcessor class.
        orderProcessor.ProcessOrder(items);

        // Simulate the completion of the order processing.
        Console.WriteLine("Order processed successfully.");
    }
}
```

OrderProcessor class implementing IOrderProcessor:
```csharp
public class OrderProcessor : IOrderProcessor
{
    public void ProcessOrder(List<string> items)
    {
        // Actual order processing logic goes here:
        // - Calculate the total cost.
        // - Check item availability.
        // - Handle payment processing.
        // - Update order records in a database.

        Console.WriteLine("Order processing logic is executed here.");
    }
}
```

With the integration of interfaces, we redefine the contracts for our classes. The `ShoppingCart` and `OrderProcessor` classes 
now implement these interfaces, IShoppingCart and IOrderProcessor, respectively. We have not only enhanced the modularity and 
organization of the code but also profoundly improved its testability. Through Dependency Injection, the ShoppingCart class receives 
an IOrderProcessor through its constructor, which positions our code for seamless adaptation to future requirements and changes.

## Tests - The improved way

Now that we have a well-structured code, let's see how easy is to write tests and mocking

```csharp
using Xunit;
using Moq;
using System.Collections.Generic;

public class ShoppingCartTests
{
    [Fact]
    public void Checkout_ShouldProcessOrderWithItems()
    {
        // Arrange
        var orderProcessorMock = new Mock<IOrderProcessor>();
        var shoppingCart = new ShoppingCart(orderProcessorMock.Object);

        // Act
        shoppingCart.AddItem("Product A");
        shoppingCart.AddItem("Product B");
        shoppingCart.Checkout();

        // Assert
        // Verify that the ProcessOrder method was called with items.
        orderProcessorMock.Verify(op => op.ProcessOrder(It.IsAny<List<string>>()), Times.Once);
    }

    [Fact]
    public void Checkout_EmptyCart_ShouldNotProcessOrder()
    {
        // Arrange
        var orderProcessorMock = new Mock<IOrderProcessor>();
        var shoppingCart = new ShoppingCart(orderProcessorMock.Object);

        // Act
        shoppingCart.Checkout();

        // Assert
        // Verify that the ProcessOrder method was not called for an empty cart.
        orderProcessorMock.Verify(op => op.ProcessOrder(It.IsAny<List<string>>()), Times.Never);
    }

    [Fact]
    public void Checkout_SingleItem_ShouldProcessOrder()
    {
        // Arrange
        var orderProcessorMock = new Mock<IOrderProcessor>();
        var shoppingCart = new ShoppingCart(orderProcessorMock.Object);

        // Act
        shoppingCart.AddItem("Product A");
        shoppingCart.Checkout();

        // Assert
        // Verify that the ProcessOrder method was called with a single item.
        orderProcessorMock.Verify(op => op.ProcessOrder(It.Is<List<string>>(items => items.Count == 1)), Times.Once);
    }

    [Fact]
    public void Checkout_MultipleItems_ShouldProcessOrder()
    {
        // Arrange
        var orderProcessorMock = new Mock<IOrderProcessor>();
        var shoppingCart = new ShoppingCart(orderProcessorMock.Object);

        // Act
        shoppingCart.AddItem("Product A");
        shoppingCart.AddItem("Product B");
        shoppingCart.Checkout();

        // Assert
        // Verify that the ProcessOrder method was called with multiple items.
        orderProcessorMock.Verify(op => op.ProcessOrder(It.Is<List<string>>(items => items.Count == 2)), Times.Once);
    }
}
```
These test scenarios cover various cases, including a non-empty cart, an empty cart, a cart with a single item, and a cart 
with multiple items. Each test verifies whether the ProcessOrder method is called with the correct number of items. 
The use of Moq makes it easy to set up expectations and validate interactions. With the separation of concerns and the use of interfaces, 
testing different scenarios becomes straightforward.

## Conclusion

By adhering to principles such as the Single Responsibility Principle, employing Dependency Injection, and embracing Interfaces, 
we advance towards code that is cleaner, more maintainable, and exceptionally testable. This approach not only elevates the quality of 
our existing code but also positions it for seamless adaptation to future requirements and changes.

