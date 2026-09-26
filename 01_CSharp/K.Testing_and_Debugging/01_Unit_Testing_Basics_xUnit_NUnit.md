
# Unit Testing Basics: xUnit & NUnit in C#

## 📌 What is it?

Unit testing is writing small, automated tests that verify a single "unit" of code (typically a method) behaves correctly in isolation. **xUnit** and **NUnit** are the two most popular .NET testing frameworks that provide the attributes, assertions, and test runner infrastructure to write and execute these tests.

## 🤔 Why do we need it?

- Catches regressions automatically — a test breaks the moment behavior changes unexpectedly.
- Documents intended behavior — tests act as executable specifications.
- Enables safe refactoring — you can change implementation details confidently if tests still pass.
- Essential for CI/CD pipelines — tests run automatically before code is merged/deployed.

## 🧠 Intuition

A unit test follows the **Arrange-Act-Assert (AAA)** pattern: set up inputs (Arrange), call the method under test (Act), then verify the result matches expectations (Assert). Each test should verify **one specific behavior**, be independent of other tests, and be fast/repeatable.

## 🌍 Real-world analogy

Like a quality control checkpoint on a factory assembly line testing individual components before they go into the final product — not the whole car at once, just "does this single bolt meet spec?" Each checkpoint (test) is isolated, fast, and tells you exactly which part failed if something's wrong.

## 📊 Comparison Table — xUnit vs NUnit

| Aspect                    | xUnit                                                       | NUnit                                 |
| ------------------------- | ----------------------------------------------------------- | ------------------------------------- |
| Test method attribute     | `[Fact]` (no params) / `[Theory]` (parameterized)       | `[Test]` (both cases)               |
| Setup before each test    | Constructor                                                 | `[SetUp]` method                    |
| Teardown after each test  | `IDisposable.Dispose()`                                   | `[TearDown]` method                 |
| Parameterized tests       | `[Theory]` + `[InlineData]`/`[MemberData]`            | `[TestCase]` / `[TestCaseSource]` |
| Ignore/skip a test        | `[Fact(Skip = "reason")]`                                 | `[Ignore("reason")]`                |
| Class-level setup (once)  | `IClassFixture<T>`                                        | `[OneTimeSetUp]`                    |
| Popularity in modern .NET | Very high (default in many templates)                       | Long-established, still widely used   |
| Test instance lifecycle   | New instance**per test method** (isolation by design) | Configurable, often shared instance   |

## 💻 Code Examples

### Basic xUnit test

```csharp
public class CalculatorTests
{
    [Fact]
    public void Add_TwoPositiveNumbers_ReturnsSum()
    {
        // Arrange
        var calculator = new Calculator();

        // Act
        int result = calculator.Add(2, 3);

        // Assert
        Assert.Equal(5, result);
    }
}
```

### Basic NUnit test (equivalent)

```csharp
[TestFixture]
public class CalculatorTests
{
    [Test]
    public void Add_TwoPositiveNumbers_ReturnsSum()
    {
        var calculator = new Calculator();
        int result = calculator.Add(2, 3);
        Assert.That(result, Is.EqualTo(5));
    }
}
```

### xUnit — parameterized tests with `[Theory]`

```csharp
public class CalculatorTests
{
    [Theory]
    [InlineData(2, 3, 5)]
    [InlineData(-1, 1, 0)]
    [InlineData(0, 0, 0)]
    public void Add_VariousInputs_ReturnsExpectedSum(int a, int b, int expected)
    {
        var calculator = new Calculator();
        int result = calculator.Add(a, b);
        Assert.Equal(expected, result);
    }
}
```

### NUnit — parameterized tests with `[TestCase]`

```csharp
[TestFixture]
public class CalculatorTests
{
    [TestCase(2, 3, 5)]
    [TestCase(-1, 1, 0)]
    [TestCase(0, 0, 0)]
    public void Add_VariousInputs_ReturnsExpectedSum(int a, int b, int expected)
    {
        var calculator = new Calculator();
        Assert.That(calculator.Add(a, b), Is.EqualTo(expected));
    }
}
```

### Setup/teardown per test — xUnit (constructor + Dispose)

```csharp
public class OrderServiceTests : IDisposable
{
    private readonly OrderService _service;

    public OrderServiceTests() // runs before EVERY test (constructor = setup)
    {
        _service = new OrderService(new InMemoryOrderRepository());
    }

    public void Dispose() // runs after EVERY test (teardown)
    {
        // cleanup if needed
    }

    [Fact]
    public void CreateOrder_ValidInput_ReturnsOrderWithId()
    {
        var order = _service.CreateOrder("Widget", 3);
        Assert.True(order.Id > 0);
    }
}
```

### Setup/teardown per test — NUnit

```csharp
[TestFixture]
public class OrderServiceTests
{
    private OrderService _service;

    [SetUp]
    public void Setup() => _service = new OrderService(new InMemoryOrderRepository());

    [TearDown]
    public void TearDown() { /* cleanup if needed */ }

    [Test]
    public void CreateOrder_ValidInput_ReturnsOrderWithId()
    {
        var order = _service.CreateOrder("Widget", 3);
        Assert.That(order.Id, Is.GreaterThan(0));
    }
}
```

### Testing exceptions

```csharp
// xUnit
[Fact]
public void Withdraw_InsufficientFunds_ThrowsException()
{
    var account = new Account(balance: 100);
    Assert.Throws<InsufficientFundsException>(() => account.Withdraw(500));
}

// NUnit
[Test]
public void Withdraw_InsufficientFunds_ThrowsException()
{
    var account = new Account(balance: 100);
    Assert.Throws<InsufficientFundsException>(() => account.Withdraw(500));
}
```

## ⚙️ AAA Pattern in practice

```
Arrange → set up objects, inputs, mocks
Act     → call the single method/behavior under test
Assert  → verify the outcome matches expectations
```

A good unit test name reads like a sentence: `MethodName_Scenario_ExpectedBehavior` (e.g., `Withdraw_InsufficientFunds_ThrowsException`) — instantly tells you what broke when it fails.

## ⚡ Performance considerations

- Unit tests should be **fast** (milliseconds each) — no real DB calls, no real network I/O, no `Thread.Sleep`. Use fakes/mocks (see `02_Mocking_with_Moq.md`) for dependencies.
- xUnit creates a **new test class instance per test method** by design — encourages stateless, isolated tests (no accidental shared state between tests), though this means shared expensive setup needs `IClassFixture<T>` instead of the constructor.
- Slow test suites discourage developers from running them often — keep the unit test layer lean and fast; push slower integration tests to a separate suite.

## 🚨 Common mistakes

- ❌ Testing multiple unrelated behaviors in one test method — makes failures ambiguous.
- ❌ Tests that depend on execution order or shared mutable state between tests — breaks isolation, causes flaky failures.
- ❌ Testing implementation details instead of observable behavior — makes tests brittle to harmless refactors.
- ❌ Not naming tests descriptively — `Test1()` tells you nothing when it fails.
- ❌ Hitting real databases/APIs/file systems in "unit" tests — that's integration testing, not unit testing; keep them separate.
- ❌ Ignoring/skipping failing tests instead of fixing or deleting them — accumulates technical debt and erodes trust in the suite.

## 💡 Best practices

- One logical assertion focus per test (can be multiple `Assert` calls if they verify the same single behavior/outcome).
- Follow AAA and a clear naming convention consistently across the whole test suite.
- Keep unit tests isolated — no shared mutable state, no reliance on execution order, no real external dependencies.
- Use parameterized tests (`[Theory]`/`[TestCase]`) to cover multiple input scenarios without duplicating test method bodies.
- Pair unit tests with mocking (see `02_Mocking_with_Moq.md`) to isolate the unit under test from its dependencies.

## 🎤 Interview Questions

1. **What's the key structural difference between xUnit and NUnit for test setup?**
   → xUnit uses the constructor (+`IDisposable.Dispose()`) for per-test setup/teardown, creating a fresh instance per test; NUnit uses explicit `[SetUp]`/`[TearDown]` attributes.
2. **What is the AAA pattern?**
   → Arrange (set up inputs/dependencies), Act (invoke the method under test), Assert (verify the result) — a standard structure for readable, focused unit tests.
3. **Why does xUnit create a new test class instance per test method?**
   → To enforce test isolation by default — prevents accidental state leakage between tests without requiring developers to remember manual resets.
4. **What's the difference between a unit test and an integration test?**
   → A unit test verifies a single unit of code in isolation (dependencies mocked/faked); an integration test verifies multiple components working together, often including real infrastructure (DB, network).
5. **How do you test that a method throws a specific exception?**
   → `Assert.Throws<TException>(() => methodCall())` in both xUnit and NUnit — verifies the exact exception type is thrown.

## 📝 30-second Revision Cheat Sheet

| Concept       | xUnit                                 | NUnit                    |
| ------------- | ------------------------------------- | ------------------------ |
| Test method   | `[Fact]` / `[Theory]`             | `[Test]`               |
| Setup         | Constructor                           | `[SetUp]`              |
| Teardown      | `Dispose()`                         | `[TearDown]`           |
| Parameterized | `[InlineData]`                      | `[TestCase]`           |
| Pattern       | Arrange → Act → Assert              | Arrange → Act → Assert |
| Golden rule   | Fast, isolated, one behavior per test | Same                     |
