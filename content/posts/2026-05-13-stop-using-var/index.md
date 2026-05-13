+++
title = 'Stop Using "var"'
date = "2026-05-13T13:20:05+01:00"
author = "Aleksander Reinhardt"
tags = ["c#", "code review", "code quality", "readability"]
keywords = ["stop", "using", "var"]
description = 'Why Implicit Typing Makes Code Review Harder, And When "var" Actually Earns Its Keep'
showFullContent = false
hideComments = false
+++

`var` is sold as ergonomics. You type less, the compiler infers the type, the reader gets less visual noise. All true, when the type is obvious from the right-hand side. The trouble is that "obvious" usually means "obvious in my IDE, with my cursor on the line, with IntelliSense ready to fire". None of that is available in a pull-request diff, on a screenshot pasted into chat, or in your colleague's head at 21:00 on a Friday.

This isn't a call to ban `var`. It's a call to recognise that every implicit type is a small bet that the next reader can reconstruct what the right-hand side returns without leaving the page they're looking at. Sometimes that bet pays off. Often, it doesn't.

---

{{<toc>}}

---

## Code Review Is The Honest Test

The argument for `var` ("the type is obvious") rests on a context that often doesn't exist when the code is read. In a pull request, you don't have hover tooltips. You don't have "Go To Definition". You have a diff and a comment box. The minute a reviewer has to open another file to work out what `var x = Something()` actually means, you've stolen attention away from the change itself.

That cost compounds. Reviewers will usually do one of three things:

1. Click through to read the called method's signature.
2. Trust the variable name and move on.
3. Ask the author what the type is.

Most pick option 2, which is exactly when `var` starts to bite. The bugs `var` enables are precisely the ones a reviewer cannot catch by reading: deferred execution, integer division, swallowed `Task<T>`, hidden nullability. The compiler is happy. Tests pass on the local data set. Production finds the rest.

---

## What "var" Hides

Each of the following is something a reviewer cannot see without leaving the diff.

**Deferred Execution (`IQueryable` vs Concrete Collection)**

```csharp
var clients = db.Clients.Where(c => c.IsActive);
foreach (var c in clients) { /* DB query executes */ }
foreach (var c in clients) { /* DB query executes again */ }
```

`var` makes both `foreach` blocks look equally cheap. With `IQueryable<Client> clients = ...`, the second loop immediately reads as another database round-trip. If you cannot be sure the source is materialised, force it: `var clients = (await GetClients()).ToList()`.

**Async Result Vs `Task<T>`**

```csharp
var text = File.ReadAllTextAsync("file.txt"); // Task<string>
Console.WriteLine(text.Length);               // length of the Task, not the file
```

A missing `await` is a depressingly common bug, and `var` is the reason it slips through review. `string text = await File.ReadAllTextAsync(...)` is impossible to misread.

**Integer Division And Numeric Width**

```csharp
var ratio = 1 / 2;             // int, value 0
var big   = 100_000 * 100_000; // int, overflows silently
```

The right-hand side of each line looks like arithmetic to a reader, not a type declaration. `double ratio` and `long big` put the decision back onto the page where it can be questioned.

**Boxing Through `object`**

```csharp
var value = GetSomething(); // returns object
int n = (int)value;         // unboxing, NullReferenceException if value is null
```

A reviewer scanning `var value = ...` will mentally substitute whatever the variable name suggests. They won't notice the boxing penalty or the latent null cast.

**Nullable Types Pretending To Be Non-Nullable**

```csharp
var age = dbRow["Age"]; // object, possibly DBNull
int x = (int)age;       // boom, when the row had no age
```

`int? age = ...` forces the reader to confront nullability immediately. `var` hides it under a friendly-looking name.

**Anonymous Types And Scope**

```csharp
var client = new { Name = "Alice", Age = 30 };
```

Fine for a one-line projection inside a LINQ chain. Useless the moment you want to return, store, or pass the value, which is the moment a reviewer needs to understand its actual shape.

**Wrong-Collection-Type Assumptions**

```csharp
var clients = new[] { "Alice", "Bob" }; // string[], not List<string>
clients.Add("Charlie");                 // doesn't compile
```

The variable name suggests "a list of clients". The actual type is `string[]`, which has none of the `List<T>` API. `List<string> clients = [...]` removes the ambiguity for everyone.

**Generic API Soup**

```csharp
var json = JsonSerializer.Deserialize<object>("{}");
```

Whatever you wanted, `object` is what you got. Type the variable as `Dictionary<string, object>` (or the model you actually expect) and the line stops lying about its intent.

---

## When "var" Earns Its Keep

There is a narrow band where `var` is genuinely better than the explicit form: when the right-hand side already names the type and the explicit form would only repeat it. The textbook case used to be the constructor call, where `Client client = new Client()` says `Client` twice and `var client = new Client()` says it once. Since C# 9 that argument has lost its teeth, because `Client client = new()` is just as terse, keeps the type on the left where the reader scans for it, and reads identically to a reviewer.

What `var` still buys you is the `as` cast and the explicit cast. In those, the type sits inside the expression on the right, and writing it on the left would only duplicate it.

```csharp
var safeCast = obj as Client;   // alternative: Client safeCast = obj as Client
var cast     = (Client)source;  // alternative: Client cast = (Client)source
```

The pattern is simple. If the type is *on the line you're looking at*, `var` is fine. If the type lives behind a method signature in another file, `var` is a footgun pointed at the reviewer.

---

## A Debt To The Next Reader

Type declarations are not a tax. They're a contract that the line reads the same in every context the code will live in: your IDE, a GitHub diff, a Slack snippet, a printed page on a desk. `var` optimises for the writing context at the expense of every other one. Code is read far more often than it's written, and every implicit type is a wager that the next reader can still recover the meaning from the line alone.

Treat each `var` as a small debt to the next reader. Some debts are worth taking on. Most aren't.

---

## Conclusion

`var` is not the enemy. The enemy is the assumption that the type behind it is obvious to everyone, in every context, forever. That assumption fails most reliably in the exact place we'd want it to hold: code review. And the bugs it lets through are the ones reviewers cannot catch by reading alone.

When the type is on the right-hand side, use `var`. Everywhere else, type the variable explicitly and let the next reader keep their attention on the change in front of them.
