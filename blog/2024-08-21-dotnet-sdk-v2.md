---
slug: "dotnet-sdk-v2"
title: "OpenFeature .NET SDK 2.0 Release"
date: 2024-08-21
authors: [toddbaert]
description: "Announcing the 2.0 Release of the OpenFeature SDK for .NET"
tags: [.NET, dotnet, c#, csharp, async, v2.0, v2]
draft: false
---

Today we're announcing the release of our first 2.0 SDK; the OpenFeature SDK for .NET, v2.0!
This release contains a number of ergonomic improvements to the SDK which .NET developers will appreciate.
It also includes some performance optimizations brought to you by the latest .NET primitives.

<!--truncate-->

## Notable Improvements

### Support for Cancellation Tokens, Idiomatic Method Names

Methods on some key interfaces (such as provider resolvers) have been updated to indicate that they work asynchronously, by appending the `Async` suffix in accordance with .NET conventions:

```diff
+ await Api.Instance.SetProviderAsync(myProviderInstance);
- await Api.Instance.SetProvider(myProviderInstance);
```

Additionally, optional cancellation tokens can now be passed to applicable asynchronous methods.
This allows for the cancellation of async tasks, and is consistent with .NET norms:

```diff
+ await client.GetBooleanValueAsync("my-flag", false, cancellationToken);
- await client.GetBooleanValue("my-flag", false);
```

### ValueTasks for Hooks

The return types for stages within [hooks](docs/reference/concepts/hooks) have been updated to take advantage of the performance benefit provided by [ValueTasks](https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/).
The vast majority of hook stages run synchronously, and they are awaited internally by the SDK; so we can avoid the additional allocations associated with `Tasks` by using `ValueTasks`:

```diff
public class MyBeforeHook : Hook
{
-  public Task<EvaluationContext> Before<T>(HookContext<T> context,
-      IReadOnlyDictionary<string, object> hints = null)
-  {
-    // code to run before flag evaluation
-  }
+  public ValueTask<EvaluationContext> BeforeAsync<T>(HookContext<T> context,
+      IReadOnlyDictionary<string, object> hints = null)
+  {
+    // code to run before flag evaluation
+  }
}
```

## Migration Steps

### Application Authors

#### Async suffixes and Cancellation Tokens

Generally, application authors won't have to make many changes.

As mentioned above, some async methods have been changed to confirm to .NET conventions.
Evaluation methods now have the "Async" suffix, and accept an optional cancellationToken:

```diff
+ await client.GetBooleanValueAsync("my-flag", false, cancellationToken);
- await client.GetBooleanValue("my-flag", false);
```

The method for setting a provider has been updated similarly:

```diff
+ await Api.Instance.SetProviderAsync(myProviderInstance);
- await Api.Instance.SetProvider(myProviderInstance);
```

#### "Client Name" has Changed to "Domain"

Parameters previously named "client name" are now named "domain".
"[Domains](https://openfeature.dev/specification/glossary/#domain)" are a more powerful concept than "named clients", but in instances where named clients were previously used, function identically.
No changes to consuming code are necessary these cases.

### Provider Authors

#### Provider Method Name Changes

Provider authors must change their implementations in accordance with new methods names on the provider interface.
Optionally, you can make use of the new cancellation token:

```diff
+ public override Task<ResolutionDetails<bool>> ResolveBooleanValue(
+             string flagKey,
+             bool defaultValue,
+             EvaluationContext? context = null)
+         {
+             // return boolean flag details
+         }
- public override Task<ResolutionDetails<bool>> ResolveBooleanValueAsync(
-             string flagKey,
-             bool defaultValue,
-             EvaluationContext? context = null,
-             CancellationToken cancellationToken = default)
-         {
-             // return boolean flag details
-         }
```

#### No Need for Provider Status

It's no longer necessary to define, update or expose provider status.
If your provider requires initialization, simply define the optional `InitializeAsync` method:

```diff
public class MyProvider : FeatureProvider
{
-   private ProviderStatus _status = ProviderStatus.NotReady;        
-
-   public override ProviderStatus GetStatus()
-   {
-       return _status;
-   }

-   public override Task Initialize(EvaluationContext context)
+   public override Task InitializeAsync(EvaluationContext context)
    {
        // some async initialization
    }
}
```

For more details about this change, see our [previous blog post](https://openfeature.dev/blog/reconciling-with-state).

### Hook Authors

#### Hooks Must Return ValueTask

Hooks must now return `ValueTask` instead of `Task`:

```diff
public class MyBeforeHook : Hook
{
-  public Task<EvaluationContext> Before<T>(HookContext<T> context,
-      IReadOnlyDictionary<string, object> hints = null)
-  {
-    // code to run before flag evaluation
-  }
+  public ValueTask<EvaluationContext> BeforeAsync<T>(HookContext<T> context,
+      IReadOnlyDictionary<string, object> hints = null)
+  {
+    // code to run before flag evaluation
+  }
}
```

## Full Changelog

<!-- TODO: UPDATE THIS LINK WHEN WE HAVE A 2.0! -->
See [here](https://github.com/open-feature/dotnet-sdk/blob/main/CHANGELOG.md#200-2024-03-12) for the full changelog.