# Integrating Razor Components into an ASP.NET Core MVC Project

This guide walks through setting up, configuring, and rendering interactive Razor Components inside an existing ASP.NET Core MVC application.

## 1. Adding the Components Folder and Files

Create a `Components` folder in your MVC project root to organize your Razor files:

```text
YourMvcProject/
  Components/
    _Imports.razor
    App.razor
    Counter.razor
```

* **`_Imports.razor`**: Shares common `@using` directives across all Razor component files in the folder.
* **`App.razor`**: Acts as the root component marker required by `MapRazorComponents<App>()`.
* **`Counter.razor`**: A reusable, interactive UI component that can be hosted inside MVC views.

## 2. Creating a Razor Component

Create a component named `Counter.razor` inside the `Components` folder:

```razor
<h3>Counter</h3>
<p>Current count: @count</p>
<button @onclick="Increment">
    Increment
</button>

@code {
    private int count = 0;

    void Increment()
    {
        count++;
    }
}
```

## 3. Add Directives to `_Imports.razor`

Add the required component namespaces to `Components/_Imports.razor`:

```razor
@using System.Net.Http
@using Microsoft.AspNetCore.Authorization
@using Microsoft.AspNetCore.Components.Authorization
@using Microsoft.AspNetCore.Components.Forms
@using Microsoft.AspNetCore.Components.Routing
@using Microsoft.AspNetCore.Components.Web
@using Microsoft.AspNetCore.Components.Web.Virtualization
@using Microsoft.JSInterop
@using test_razor
```

## 4. Update `App.razor`

Add the root component markup to `Components/App.razor`:

```razor
@* App.razor - required only as a root marker for MapRazorComponents<App>() *@
<Counter />
```

## 5. Enabling Interactive Components

Configure Razor Components and interactive server support in `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();
builder.Services.AddRazorComponents()
    .AddInteractiveServerComponents();

var app = builder.Build();

app.UseStaticFiles();
app.UseRouting();
app.UseAntiforgery();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.MapRazorComponents<App>()
    .AddInteractiveServerRenderMode();

app.Run();
```

* Uses SignalR behind the scenes to manage UI updates over a real-time connection.
* Enables interactive C# event handlers such as `@onclick`.

## 6. Configuring View Imports & Layout

### Enable MVC Tag Helpers

Ensure `Views/_ViewImports.cshtml` includes the MVC Tag Helpers directive:

```cshtml
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

> **Note:** Without this directive, the `<component>` tag helper is unavailable in your `.cshtml` views.

### Include Blazor Script & Base Path

In `Views/Shared/_Layout.cshtml`, ensure `<base href="~/" />` is in `<head>` and include the Blazor script before the closing `</body>` tag:

```html
<head>
    <base href="~/" />
    <!-- other head elements -->
</head>
<body>
    @RenderBody()
    <script src="_framework/blazor.web.js"></script>
</body>
```

## 7. Rendering a Component in an MVC View

Use the `<component>` tag helper inside any `.cshtml` view (for example, `Views/Home/Index.cshtml`):

```cshtml
@using test_razor.Components

<component type="typeof(Counter)" render-mode="ServerPrerendered" />
```

* Component output becomes part of the rendered MVC view.
* Supports both static and interactive rendering modes.

### Rendering Modes Reference

| Mode | Description |
| :--- | :--- |
| `Static` | Server-rendered HTML only (no interactivity). |
| `ServerPrerendered` | Prerendered on the server, then interactive via SignalR. |
| `WebAssemblyPrerendered` | Prerendered on the server, then runs in the browser via WebAssembly. |

