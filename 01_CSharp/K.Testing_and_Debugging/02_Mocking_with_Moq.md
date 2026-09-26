
# Mocking with Moq in C#

## 📌 What is it?

Moq is a popular .NET mocking library that lets you create **fake implementations of interfaces/abstract classes** at test time — objects that mimic real dependencies (DB repositories, HTTP clients, email services) but return pre-programmed responses and let you verify how they were called, without touching any real infrastructure.

## 🤔 Why do we need it?

- Unit tests must isolate the "unit under test" from its dependencies — you don't want a test for `OrderService` to actually hit a real database.
- Mocks let you simulate scenarios that are hard to trigger for real (network failures, specific edge-case data) in a controlled, repeatable way.
- Enables fast, deterministic tests — no flaky network calls, no test data pollution in a real DB.

## 🧠 Intuition

A mock is a **stunt double** for a real dependency. It looks like the real thing (implements the same interface) but does exactly what you tell it to for the test — return this value, throw that exception, do nothing — and it remembers every interaction so you can verify "was this method actually called, and with what?"

## 🌍 Real-world analogy

Like a flight simulator for pilot training: it looks and responds like a real cockpit (same interface), but instead of a real engine, it's a controlled program that reacts exactly how the trainer configures it — simulate engine failure at any moment, verify the trainee pressed the right buttons — all without risking an actual plane.

## ⚙️ Core Moq concepts

| Term                                     | Meaning                                                                |
| ---------------------------------------- | ---------------------------------------------------------------------- |
| `Mock<T>`                              | Creates a mock object implementing interface/abstract class`T`       |
| `.Setup(...)`                          | Configures what a method/property should return when called            |
| `.Returns(...)`                        | Specifies the return value for a setup call                            |
| `.ThrowsAsync(...)` / `.Throws(...)` | Makes a mocked call throw an exception                                 |
| `.Verify(...)`                         | Asserts that a specific method was called (optionally, how many times) |
| `.Object`                              | The actual fake instance you inject into the class under test          |
| `It.IsAny<T>()`                        | Matches any argument of type`T` in a setup/verify                    |

## 💻 Code Examples

### Basic — mocking a repository dependency

```csharp
public interface IOrderRepository
{
    Order? GetById(int id);
    void Save(Order order);
}

public class OrderService
{
    private readonly IOrderRepository _repository;
    public OrderService(IOrderRepository repository) => _repository = repository;

    public Order GetOrder(int id)
    {
        var order = _repository.GetById(id);
        if (order is null) throw new OrderNotFoundException(id);
        return order;
    }
}

// Test
[Fact]
public void GetOrder_ExistingId_ReturnsOrder()
{
    // Arrange
    var mockRepo = new Mock<IOrderRepository>();
    var expectedOrder = new Order { Id = 1, ProductName = "Widget" };
    mockRepo.Setup(r => r.GetById(1)).Returns(expectedOrder);

    var service = new OrderService(mockRepo.Object); // inject the FAKE, not a real repo

    // Act
    var result = service.GetOrder(1);

    // Assert
    Assert.Equal("Widget", result.ProductName);
}
```

### Testing exception scenarios

```csharp
[Fact]
public void GetOrder_NonExistentId_ThrowsOrderNotFoundException()
{
    var mockRepo = new Mock<IOrderRepository>();
    mockRepo.Setup(r => r.GetById(It.IsAny<int>())).Returns((Order?)null); // simulate "not found"

    var service = new OrderService(mockRepo.Object);

    Assert.Throws<OrderNotFoundException>(() => service.GetOrder(999));
}
```

### Verifying interactions — did the method actually get called?

```csharp
[Fact]
public void CreateOrder_ValidInput_CallsSaveExactlyOnce()
{
    var mockRepo = new Mock<IOrderRepository>();
    var service = new OrderService(mockRepo.Object);

    service.CreateOrder("Widget", 3);

    // Verify Save was called exactly once with any Order argument
    mockRepo.Verify(r => r.Save(It.IsAny<Order>()), Times.Once);
}
```

### Mocking async methods

```csharp
public interface IPaymentGateway
{
    Task<bool> ChargeAsync(decimal amount);
}

[Fact]
public async Task ProcessPayment_GatewaySucceeds_ReturnsTrue()
{
    var mockGateway = new Mock<IPaymentGateway>();
    mockGateway
        .Setup(g => g.ChargeAsync(It.IsAny<decimal>()))
        .ReturnsAsync(true); // async-specific setup

    var result = await mockGateway.Object.ChargeAsync(100m);

    Assert.True(result);
}
```

### Simulating a failure (throwing from a mock)

```csharp
[Fact]
public async Task ProcessPayment_GatewayThrows_HandlesGracefully()
{
    var mockGateway = new Mock<IPaymentGateway>();
    mockGateway
        .Setup(g => g.ChargeAsync(It.IsAny<decimal>()))
        .ThrowsAsync(new HttpRequestException("Network error"));

    var service = new PaymentService(mockGateway.Object);

    var result = await service.TryProcessAsync(100m);

    Assert.False(result.Success); // service should catch and handle gracefully
}
```

### Argument matching with conditions

```csharp
mockRepo.Setup(r => r.GetById(It.Is<int>(id => id > 0))).Returns(validOrder);
mockRepo.Setup(r => r.GetById(It.Is<int>(id => id <= 0))).Returns((Order?)null);
```

### Mocking properties

```csharp
public interface IConfig
{
    string ConnectionString { get; }
}

var mockConfig = new Mock<IConfig>();
mockConfig.Setup(c => c.ConnectionString).Returns("Server=test;Database=test;");
```

## 📊 Comparison Table — Mock vs Stub vs Fake (test double terminology)

| Term           | Purpose                                                   | Verifies behavior?      |
| -------------- | --------------------------------------------------------- | ----------------------- |
| **Stub** | Provides canned answers to calls                          | No — just returns data |
| **Mock** | Provides canned answers AND lets you verify interactions  | Yes —`.Verify(...)`  |
| **Fake** | A working, simplified implementation (e.g., in-memory DB) | Depends on usage        |

> In casual usage, "mock" is often used loosely for all of the above — Moq itself can act as either a stub (just `.Setup`/`.Returns`) or a true mock (adding `.Verify`).

## ⚡ Performance considerations

- Mocking has negligible runtime overhead compared to real dependencies (DB, network) — this is exactly why it makes unit tests fast.
- Overly complex mock setups (deeply chained, many conditional matchers) can slow down test **authoring** and readability, even if runtime cost stays low.

## 🚨 Common mistakes

- ❌ Mocking types you don't own carelessly (e.g., mocking `HttpClient` directly is awkward — wrap it in your own interface like `IHttpClientWrapper` instead).
- ❌ Over-verifying — asserting on every single call instead of the behaviors that actually matter, making tests brittle to harmless refactors.
- ❌ Forgetting `It.IsAny<T>()` and being overly strict on setups, causing tests to fail on trivial argument variations unrelated to what's being tested.
- ❌ Mocking concrete classes without virtual members — Moq can only mock interfaces, abstract members, or virtual methods (it needs something to override).
- ❌ Using mocks for simple value objects/DTOs that don't need faking — just construct real instances.

## 💡 Best practices

- Mock at architectural **seams** — interfaces representing external dependencies (repositories, gateways, clients), not every single class.
- Use `.Setup` for controlling behavior (stubbing), `.Verify` only for interactions that are actually meaningful to the test's purpose.
- Prefer `It.IsAny<T>()` unless the specific argument value matters to the scenario being tested.
- Keep mock setups close to the Arrange section of each test — don't scatter shared, hard-to-trace mock configuration across helper classes unless truly reused.
- Design classes to depend on interfaces (constructor injection) specifically to make them mockable — a core reason dependency injection and testability go hand in hand.

## 🎤 Interview Questions

1. **What's the difference between a mock, a stub, and a fake?**
   → A stub returns canned data without verification; a mock does the same but also lets you verify interactions occurred; a fake is a real, simplified working implementation (e.g., an in-memory repository).
2. **Why can't Moq mock a concrete class with non-virtual methods?**
   → Moq works by generating a dynamic proxy that overrides members at runtime — it needs an interface or `virtual`/`abstract` members to override; non-virtual methods can't be intercepted.
3. **What does `It.IsAny<T>()` do, and when should you avoid it?**
   → It matches any argument of type `T` in a setup or verify call; avoid it when the specific argument value is actually relevant to what the test is verifying — being too lenient can hide bugs.
4. **Why is it usually better to mock an interface you own (like `IHttpClientWrapper`) rather than `HttpClient` directly?**
   → `HttpClient`'s key methods aren't easily mockable/virtual in a clean way; wrapping it in your own interface gives you a clean seam that's trivially mockable and decouples your code from the concrete HTTP implementation.
5. **What's the risk of over-verifying mock interactions in tests?**
   → Tests become brittle — tightly coupled to *how* the code achieves its result rather than *what* result it produces, so harmless refactors break tests unnecessarily.

## 📝 30-second Revision Cheat Sheet

| Concept                | Key Point                                                |
| ---------------------- | -------------------------------------------------------- |
| `Mock<T>`            | Creates a fake implementation of`T`                    |
| `.Setup().Returns()` | Configure canned responses                               |
| `.Verify()`          | Assert a method was actually called                      |
| `.Object`            | The fake instance to inject into the class under test    |
| `It.IsAny<T>()`      | Matches any argument — use judiciously                  |
| Requirement            | Moq needs interfaces or virtual/abstract members to mock |
