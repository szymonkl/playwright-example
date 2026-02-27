# Copilot Instructions for PlaywrightTests Project

This document provides guidelines for GitHub Copilot when working with this .NET NUnit Playwright testing project.

## Project Overview

- **Framework**: .NET 10
- **Testing Framework**: NUnit
- **Browser Automation**: Playwright for .NET
- **Language**: C#

## Code Style and Best Practices

### 1. Test Structure

- Follow the Arrange-Act-Assert (AAA) pattern for all tests
- Use descriptive test names that clearly indicate what is being tested
- Do not use comments to explain what the test is doing; the code should be self-explanatory
- Do not use Arrange/Act/Assert comments; structure the code to make these phases clear without comments
- Use [SetUp] and [TearDown] methods for test initialization and cleanup

Example structure:
```csharp
[Test]
public async Task Should_Navigate_To_Homepage_And_Verify_Title()
{
    var page = await Browser.NewPageAsync();
    
    await page.GotoAsync("https://example.com");
    
    await Assertions.Expect(page).ToHaveTitleAsync(new Regex("Expected Title"));
     
}
```

### 2. Playwright Usage

- Run tests in headed mode for better debugging and visibility
- Use async/await patterns consistently throughout all Playwright calls
- Prefer implicit waits over explicit Thread.Sleep()
- Use built-in Playwright locator automatic waiting and retry mechanisms instead of manual waits
- Always close browsers and pages in [TearDown] or using statements
- Leverage Page Objects or Helper methods for reusable page interactions

Example with proper async handling:
```csharp
await page.Locator("selector").FillAsync("text");
await page.Locator("button").ClickAsync();
```

### 3. Locators
- Prefer user-centric locators: `GetByRole()`, `GetByLabel()`, `GetByPlaceholder()`, `GetByText()`
- Use data-testid attributes when UI locators are not suitable
- Avoid complex CSS/XPath selectors when simpler options exist

Example:
```csharp
var button = page.GetByRole(AriaRole.Button, new() { Name = "Submit" });
var input = page.GetByPlaceholder("Enter your name");
var link = page.GetByText("Click here", new() { Exact = true });
```

### 4. Error Handling and Assertions

- Use Playwright built-in assertions when possible
Example:
```csharp
await Assertions.Expect(page.GetByRole(AriaRole.Heading)).ToHaveTextAsync("Welcome");
```

### 5. Configuration and Setup

- Store configuration, i.e timeouts in class level constants
- Use [OneTimeSetUp] for expensive operations like browser launch

Example:
```csharp
[OneTimeSetUp]
public async Task OneTimeSetUp()
{
    _playwright = await Playwright.CreateAsync();
    _browser = await _playwright.Chromium.LaunchAsync(new() { Headless = false });
}

[SetUp]
public async Task SetUp()
{
    _context = await _browser.NewContextAsync();
    _page = await _context.NewPageAsync();
}

[TearDown]
public async Task TearDown()
{
    await _context?.CloseAsync();
}
```

### 6. Test Data and Fixtures
- Use NUnit [TestCase] and [TestCaseSource] for parameterized tests
- Keep test data separate from test logic
- Avoid hardcoded values; use constants or configuration

Example:
```csharp
[TestCase("admin", "password123")]
[TestCase("user", "pass456")]
public async Task Should_Login_With_Valid_Credentials(string username, string password)
{
    // Test implementation
}
```

### 7. Page Object Pattern

The Page Object Model (POM) is a design pattern that encapsulates page interactions and element locators into dedicated classes. This improves maintainability, readability, and reduces code duplication.

#### Structure and Principles

- **One page object per page/component**: Create a separate class for each page or logical component
- **Encapsulate locators**: Keep all selectors in the page object, not in tests
- **Provide action methods**: Expose high-level methods for user interactions (not raw element interactions)
- **Return page objects or data**: Methods should return the next page object or assertion-ready data
- **Keep page objects independent**: Each page object should be self-contained and reusable

#### Basic Page Object Example

```csharp
public class LoginPage
{
    private readonly IPage _page;
    
    private ILocator UsernameInput => _page.GetByPlaceholder("Username");
    private ILocator PasswordInput => _page.GetByPlaceholder("Password");
    private ILocator LoginButton => _page.GetByRole(AriaRole.Button, new() { Name = "Login" });
    private ILocator ErrorMessage => _page.Locator("[data-testid='error-message']");
    
    public LoginPage(IPage page)
    {
        _page = page;
    }
    
    public async Task NavigateAsync(string url = "https://example.com/login")
    {
        await _page.GotoAsync(url, new() { WaitUntil = WaitUntilState.NetworkIdle });
    }
    
    public async Task<HomePage> LoginAsync(string username, string password)
    {
        await UsernameInput.FillAsync(username);
        await PasswordInput.FillAsync(password);
        await LoginButton.ClickAsync();
        await _page.WaitForLoadStateAsync(LoadState.NetworkIdle);
        
        return new HomePage(_page);
    }
    
    public async Task<bool> IsErrorMessageVisibleAsync()
    {
        return await ErrorMessage.IsVisibleAsync();
    }
    
    public async Task<string> GetErrorMessageAsync()
    {
        return await ErrorMessage.TextContentAsync() ?? string.Empty;
    }
}
```

#### Usage in Tests

```csharp
[Test]
public async Task Should_Login_Successfully_With_Valid_Credentials()
{
    
    var loginPage = new LoginPage(_page);
    
    await loginPage.NavigateAsync();
    var homePage = await loginPage.LoginAsync("user@example.com", "password123");
    
    await Assertions.Expect(homePage.WelcomeHeading).ToContainTextAsync("Welcome");
}
```


## File Organization
- **Test Classes**: `Tests/` folder with `*Tests.cs` naming
- **Page Objects**: `Pages/` folder with `*Page.cs` naming
- **Fixtures**: `Fixtures/` folder for test data and setup
- **Utilities**: `Helpers/` folder for common functions


## Security

- Never commit actual credentials; use environment variables or secure vaults


## Resources

- [Playwright .NET Documentation](https://playwright.dev/dotnet/)
- [NUnit Documentation](https://docs.nunit.org/)
- [Best Practices for Playwright Testing](https://playwright.dev/dotnet/docs/best-practices)