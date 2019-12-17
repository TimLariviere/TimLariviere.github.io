---
layout: post
title: "How to become a Fabulous developer"
date: "2019-12-21"
---

It's the 21st day of the [F# Advent Calendar 2019](https://sergeytihon.com/2019/11/05/f-advent-calendar-in-english-2019/), only 4 days left before Christmas!

If this year, like all the previous ones, you wrote Santa asking him to bring a new framework finally letting you write apps you'll love, with a clean, easy-to-use, reliable, maintainable and testable architecture, your present has come early!

Today, we will see together what Fabulous is, what it offers, and how it helps you build apps that you will finally be proud of.

# F# and the MVU architecture

Recently, frameworks like Elm, React-Redux and Flutter became really popular thanks in part to their alternative approach to app development, showing that mainstream MVC / MVVM architectures were not the only way to go.

Elm introduced its own architecture: MVU, standing for Model View Update, which inspired Redux.

The concept is to have a central state (the model) that can only be updated through messages sent by the UI when the user interacts with it, or by a "subscription" sending messages at any point in time (e.g. a push notification, an asynchronous task that completes, etc.).  
When the state is updated, the view is reevaluated to define how it should look for this particular state.

![Model View Update](https://elmprogramming.com/images/chapter-5/5.2-model-view-update-part-1/model-view-update.svg)
_Credits: Beginning Elm ([https://elmprogramming.com](https://elmprogramming.com))_

This architecture has several advantages over traditional ones:
- *Centralized state and updates*  
The immutability of the state is one of the greatest asset of MVU.  
It ensures that developers can only update the state through messages, declared in an enum-like type, thus centralizing all possible changes.  
No more implicit changes, sometimes hidden deep in the source code, that can be understood only if you know perfectly how the application works.

- *No concurrency issues*  
By updating the state only through messages, it allows the framework to apply concurrent messages one after the other.

- *Explicit views at each update*  
In a typical app framework, you would define your UI once and react to the user input by updating parts of it little by little. So to understand what made the UI look that way, you need to mentally reenact what the user did.  
In MVU, you're no longer responsible for that. Instead, each time the state is updated, you're asked to provide the complete view for this particular state (usually through a virtual representation) without having to think about what was the previous UI. The framework takes the responsability to update the UI.

- *Reproducible views*  
The fact your view is reevaluated each time the state is updated means that you can write side effect-free functions (aka pure functions) to generate the view according to a given state. This means that if you provide the same state, you will get the exact same view. Very convenient to test!

At any time, you know the complete state of your application and can trace what sequence of messages brought it to that state. (time-travel debugging FTW!)

For more information on MVU, please read [the Elm's website](https://guide.elm-lang.org/architecture/).

This architecture is a perfect match for functional programming.  
Pure functions, immutability, exhaustive pattern matching and so on. All are natural attributes of functional languages, like F#!

Hence, the architecture of Elm was ported to the F# programming language: [Elmish](https://elmish.github.io/elmish/) was born.

# What about Fabulous?

Fabulous is heavily inspired by Elmish. They, in fact, share a common approach to MVU.

Fabulous is an open-source, community driven set of .NET Standard 2.0 libraries to write apps in F# using an MVU architecture [available as NuGet packages](https://www.nuget.org/packages?q=Fabulous).  
The source code can be found on [GitHub](https://github.com/fsprojects/Fabulous).

![Fabulous and its ecosystem](/assets/2019-12-21-how-to-become-a-fabulous-developer/fabulous-ecosystem.png)

It started as `Elmish.XamarinForms`, providing only something called "static views" based on Xamarin.Forms. Later on, a "dynamic views" approach (still for Xamarin.Forms) has been introduced and it was renamed `Fabulous` for the occasion. Finally, it evolved in a set of platform-agnostic libraries, allowing you to use your favorite framework by adding support for it with a small adapter library.

The Xamarin.Forms part has been extracted into its own adapter library. But it remains the only officially supported framework as of today.

While Elmish decided to let the users "bring their own views" by not forcing a way to declare views, Fabulous provides several ways to define them: static, dynamic and adaptive views.

They all use the same building blocks:
- A model (usually named `Model`)
- A set of messages (usually named `Msg`)
- An `init` function giving the default model at the start of the application
- An `update` function applying the corresponding changes to the model for a given message, returning a new model
- A `view` function that will apply the new model to the UI

Once we have all those building blocks, we can assemble them in a `Program` that we will use to start the MVU application loop.

```fsharp
open Fabulous

type Model =
    { Count: int }

type Msg =
    | Increment
    | Decrement

let init () =
    { Count = 0 }

let update msg model =
    match msg with
    | Increment -> { model with Count = model.Count + 1 }
    | Decrement -> { model with Count = model.Count - 1 }

let view ... =
    // We will see how we can define our UI later in this post
    ...

let program = Program.mkSimple init update view
```

One of the advantages of this architecture is that it's completely composable.  
As the application grows, you can create sub-components with their own `Model`/`Msg`/`init`/`update`/`view` that you can call inside their parent component.

# The view function, your choice

All the examples will use Xamarin.Forms. Like said before, Fabulous was initially all about Xamarin.Forms so it remains the most fully supported framework.

To see the difference between each way to write a view, we will take the same application, a simple counter app.

<video height="420" controls loop>
  <source src="/assets/2019-12-21-how-to-become-a-fabulous-developer/counterapp.mov" type="video/mp4">
</video>

Fabulous has been tested on the following platforms supported by Xamarin.Forms:
- Android
- iOS
- UWP
- WPF
- macOS
- GTK

#### Static views

Fabulous.StaticView is a good way to gently move from an existing application to an F# MVU one, or make apps with heavy and complex UIs while benefiting from the advantages of Fabulous.

The concept is very similar to [Elmish.WPF](https://github.com/elmish/Elmish.WPF). You define the whole state of the application using MVU but the UI keeps getting declared in its native language (XAML for Xamarin.Forms, Storyboard for iOS, etc.).

In the view function, you expose a set of values and event handlers to the UI.  
With Xamarin.Forms, this is done through bindings.

```fsharp
// Binding.oneWay exposes a value from the current model
// Binding.msg creates a command that will send a message when executed
let view () =
    [ "CounterValue" |> Binding.oneWay (fun m -> m.Count)
      "IncrementCommand" |> Binding.msg Increment
      "DecrementCommand" |> Binding.msg Decrement ]
```

```xml
<ContentPage>
    <StackLayout>
        <Label Text="{Binding Path=[Count]}" HorizontalOptions="Center" />
        <Button Text="Increment" Command="{Binding Path=[IncrementCommand]}" />
        <Button Text="Decrement" Command="{Binding Path=[DecrementCommand]}" />
    </StackLayout>
</ContentPage>
```

Now that our `view` function is declared, we can start our application with our previously declared `program`.

```fsharp
type FabulousApp() as this = 
    inherit Xamarin.Forms.Application ()

    let runner =
        program
#if DEBUG
        |> Program.withConsoleTrace
#endif
        |> Program.runWithStaticView
    
    do this.MainPage <- runner.InitialMainPage
```

Notice the `Program.withConsoleTrace`?  
This allows us to check what's going on in our application. It's a logging mechanism that plugs directly in the MVU loop of Fabulous.  
Each time a model is initialized or a message is sent, we will see what the model was and how it was changed as well as if there was an error updating the UI.

See [Fabulous.StaticView on the Fabulous repository](https://github.com/fsprojects/Fabulous/tree/master/Fabulous.StaticView) for more information

#### Dynamic views

Fabulous "Dynamic Views" (shortened to Fabulous) is the most popular option of the 3.  
Instead of writing views using your framework's native view language, you write it directly in F# via a special `View` module that wraps all the supported controls.

```fsharp
let view model dispatch =
    View.ContentPage(
        View.StackLayout([
            View.Label(text = model.Count.ToString(), horizontalOptions = LayoutOptions.Center)
            View.Button(text = "Increment", command = fun() -> dispatch Increment)
            View.Button(text = "Decrement", command = fun() -> dispatch Decrement)
        ])
    )

type FabulousApp () as app = 
    inherit Application ()

    let runner =
        program
#if DEBUG
        |> Program.withConsoleTrace
#endif
        |> XamarinFormsProgram.run app
```

This has a lot of advantages over the previous approach:
- *One language to rule them all*  
Why having to learn and use 2 (or more) different languages, with uneven tooling support, to write an app? A lot of people tend to prefer to use a single language, even if it means no designer view or harder to write constructs like bindings.  
With “Dynamic Views”, you write everything in F#, including views. No more runtime errors because you provided the wrong value to a property (or mistyped that binding name once again...).  
The F# compiler will be there to help you.

- *Virtual views and incremental updates*  
For performance reasons, the views you write don't directly instantiate the actual controls. Instead, Fabulous will apply a diff'ing algorithm to check what changed between the new virtual view and the previous one, and apply only the required changes.  
This is called incremental updates.

- *Adapted to functional use*  
Since Fabulous provides its own wrappers over the controls, it's able to make them more functional-friendly.

```fsharp
/// Bindings
<Label Text="{Binding Value}" />
vs.
View.Label(text = model.Value)

/// Columns and rows
<Grid>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="Auto" />
        <ColumnDefinition Width="*" />
        <ColumnDefinition Width="2.5*" />
    </Grid.ColumnDefinitions>
    <Grid.RowDefinitions>
        <RowDefinition Width="Auto" />
        <RowDefinition Width="*" />
        <RowDefinition Width="2.5*whoops mistyped here" />
    </Grid.RowDefinitions>
</Grid>

vs.

View.Grid(
    coldefs = [ Auto; Star; Stars 2.5 ],
    rowdefs = [ Auto; Star; Stars 2.5 ]
)

/// DataTemplate / ItemsSource / Converter
<ListView ItemsSource="{Binding Items}">
    <ListView.ItemTemplate>
        <DataTemplate>
            <ViewCell>
                <Label Text="{Binding Name}"
                       TextColor="{Binding IsMinor, Converter={StaticResource BoolToTextColorConverter}}" />
            </ViewCell>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>

public class BoolToTextColorConverter : IValueConverter { ... }

<Application xmlns:local="...">
    <Application.Resources>
        <local:BoolToTextColorConverter x:Key="BoolToTextColorConverter" />
    </Application.Resources>
<Application>

vs.

View.ListView([
    for item in model.Items ->
        yield View.ViewCell(
            View.Label(
                text = item.Name,
                textColor = if item.IsMinor then Color.Red else Color.Blue
            )
        )
])
```

Those are only a few examples. And F# 4.7 will remove the need for `yield` and prefixing with `View.`.

As you may have seen in the last example, since this is F# code, you can easily write conditional views.

```fsharp
match model.UserData with
| None ->
    View.Button(text = "Log in", command = fun () -> dispatch StartLogin)
| Some user ->
    View.Label(
        text = user.Name,
        textColor = if user.IsPremium then Color.Gold else Color.Black
    )
```

Fabulous lets you write any kind of apps easily. Here's a few open-source ones using dynamic views: 

<img height="420" src="/assets/2019-12-21-how-to-become-a-fabulous-developer/fsharp-weekly.png" />
<img height="420" src="https://raw.githubusercontent.com/TimLariviere/FabulousContacts/master/docs/attachments/detail.png" />
<img height="420" src="https://raw.githubusercontent.com/TimLariviere/FabulousPlanets/master/docs/attachments/fabulousplanets-wip.gif" />

*(From left to right: [FSharp Weekly](https://github.com/Zaid-Ajaj/fsharp-weekly), [FabulousContacts](https://github.com/TimLariviere/FabulousContacts) and [FabulousPlanets](https://github.com/TimLariviere/FabulousPlanets))*

Templates are available to help you start a new solution.  
More information in the [Getting Started](https://fsprojects.github.io/Fabulous/Fabulous.XamarinForms/index.html#getting=started) guide and on the [Fabulous repository](https://github.com/fsprojects/Fabulous).

#### Adaptive views

[AdaptiveFabulous](https://github.com/dsyme/fabulous-adaptive) can be seen as a mix of the two previous approaches.

Like dynamic views, you declare the UI through a virtual representation that gets called on each update. But for increased performance (in intensive use cases, like real-time data), it also supports immediate bindings that will update the UI directly instead of waiting for a complete reevaluation.

This is achieved thanks to [FSharp.Data.Adaptive](https://github.com/fsprojects/FSharp.Data.Adaptive).

Currently, it is completely experimental and is developed by Don Syme.

A conversion is currently needed between the "classic" model and the adaptive one. But this will be eventually automated with [Adaptify](https://github.com/krauthaufen/Adaptify).
```fsharp
/// An adaptive version of the model. 
type AdaptiveModel = { Count: cval<int> }

/// Initialize an adaptive version of the model.
let ainit (model: Model) = { Text = cval model.Count }

/// Update an adaptive version of the model. 
let adelta (model: Model) (amodel: AdaptiveModel) =
    transact (fun () -> 
        if model.Count <> amodel.Count.Value then 
            amodel.Count.Value <- model.Count
    )
```

`c` and `cs` mean that the values are constants and won't change dynamically so there's no need to listen to their changes. `amodel.Count` on the other hand will change.

```fsharp
let view amodel dispatch =
    View.ContentPage(c
        View.StackLayout(cs [
            View.Label(text = amodel.Count, horizontalOptions = c LayoutOptions.Center)
            View.Button(text = c "Increment", command = c (fun() -> dispatch Increment))
            View.Button(text = c "Decrement", command = c (fun() -> dispatch Decrement))
        ])
    )
```

See [the AdaptiveFabulous repository](https://github.com/dsyme/fabulous-adaptive) for more information

# Testing the application

Our application is composed of 3 major parts: init, update and view.  
They're all functions, even more, they are pure, making them perfect candidates for unit testing!

Before we start testing them, we need to set identifiers to let our tests know which control is which.
This is done through `automationId`.

```fsharp
let view model dispatch =
    View.ContentPage(
        View.StackLayout([
            View.Label(automationId = "CountLabel", text = model.Count.ToString())
            View.Button(automationId = "IncrementButton", text = "Increment", command = fun() -> dispatch Increment)
            View.Button(automationId = "DecrementButton", text = "Decrement", command = fun() -> dispatch Decrement)
        ])
    )
```

Then, it's fairly typical unit testing for `init` and `update` (using [FsUnit](https://fsprojects.github.io/FsUnit/) and [NUnit](https://nunit.org/)).

```fsharp
open NUnit.Framework
open FsUnit
open FabulousApp
open Fabulous.XamarinForms

module ``Init tests`` =
    [<Test>]
    let ``Init should return a valid initial state``() =
        let initialState = { Count = 0 }
        App.init () |> should equal initialState

module ``Update tests`` =
    [<Test>]
    let ``Given the message Increment, Update should increment Count``() =
        let initialModel = { Count = 5 }
        let expectedState = { Count = 6 }
        App.update Increment initialModel |> should equal expectedState

    [<Test>]
    let ``Given the message Decrement, Update should decrement Count``() =
        let initialModel = { Count = 5 }
        let expectedState = { Count = 4 }
        App.update Decrement initialModel |> should equal expectedState
```

Testing `view` is a bit special.

Fabulous (dynamic views) stores your virtual view in a `ViewElement`, which is a glorified key-value dictionary.  
So, first we need to find our controls with the helper function `findViewElement` by providing it the previously declared AutomationIds.  
Then we feed the retrieved `ViewElement` to an helper class that will provide a nicely typed wrapper for a specific control.  
Each control has its matching `Viewer` helper class.

```fsharp
module ``View tests`` =
    [<Test>]
    let ``View should generate a valid interface``() =
        let model = { Count = 5 }
        let actualView = App.view model ignore

        let countLabel = actualView |> findViewElement "CountLabel" |> LabelViewer
        let incrementButton = actualView |> findViewElement "IncrementButton" |> ButtonViewer
        let decrementButton = actualView |> findViewElement "DecrementButton" |> ButtonViewer

        countLabel.Text |> should equal "5"
        incrementButton.Text |> should equal "Increment"
        decrementButton.Text |> should equal "Decrement"

    [<Test>]
    let ``Clicking the button Increment should send the message Increment``() =
        let mockedDispatch msg =
            msg |> should equal Increment

        let model = { Count = 5 }
        let actualView = App.view model mockedDispatch

        let incrementButton = actualView |> findViewElement "IncrementButton" |> ButtonViewer

        incrementButton.Command ()

    [<Test>]
    let ``Clicking the button Decrement should send the message Decrement``() =
        let mockedDispatch msg =
            msg |> should equal Decrement

        let model = { Count = 5 }
        let actualView = App.view model mockedDispatch

        let decrementButton = actualView |> findViewElement "DecrementButton" |> ButtonViewer

        decrementButton.Command ()
```

# But what if I don't want to use Xamarin.Forms?

Fabulous.StaticView and AdaptiveFabulous are not currently meant to be used without Xamarin.Forms.  
If you really want the same static views experience with another framework, take a look at other existing libraries that do it really well, like [Elmish.WPF](https://github.com/elmish/Elmish.WPF).

Fabulous (dynamic views), on the other hand, is platform-agnostic meaning you can plug your favorite framework, at the condition you or someone else wrote the corresponding adapter library (with the appropriate control wrappers).

This is greatly simplified by [Fabulous.CodeGen](https://github.com/fsprojects/Fabulous/tree/master/Fabulous.CodeGen) which will generate most of the code for you, if you provide it with the target framework dlls and instructions on what you want it to generate.  
This generated code can then be embedded in the adapter library.

[Example of the "Bindings" file](https://github.com/TimLariviere/Fabulous.WPF/blob/master/src/Fabulous.WPF/WPF.json) - [Example of the generated code](https://github.com/TimLariviere/Fabulous.WPF/blob/master/src/Fabulous.WPF/WPF.fs)

![Fabulous.CodeGen workflow](/assets/2019-12-21-how-to-become-a-fabulous-developer/fabulous-codegen-workflow.svg)

A few libraries are already making use of it:
- [Fabulous.XamarinForms](https://github.com/fsprojects/Fabulous/tree/master/Fabulous.XamarinForms) for Xamarin.Forms
- [Fabulous.WPF](https://github.com/TimLariviere/Fabulous.WPF) for WPF (Proof of Concept)
- [Fabulous.iOS](https://github.com/TimLariviere/Fabulous.iOS) for Xamarin.iOS (Proof of Concept)
- [Uno.Fabulous](https://github.com/jeromelaban/Uno.Fabulous) for [Uno Platform](https://platform.uno/) by Jérôme Laban (Proof of Concept)

There are similar alternatives to Fabulous targeting other frameworks, like [Elmish.WPF.Dynamic](https://github.com/cmeeren/Elmish.WPF.Dynamic) and [Avalonia.FuncUI](https://github.com/FuncUI/Avalonia.FuncUI).

# Accelerating the development process with LiveUpdate

Fabulous (dynamic views) has a tool called LiveUpdate, which is similar to Hot Reload (of React, Flutter and Xamarin.Forms).  
Just like Fabulous, LiveUpdate is technically compatible with all frameworks, though it will need an adapter for the specific framework you use.

With it, you can experience faster development feedback. After enabling it and starting a debug session (either on an emulator or an actual device), every time you make changes to the source code and save, the code is recompiled and injected in the running app. This allows you to quickly see the impact of your changes (1 to 2 seconds of wait), instead of stopping the app, rebuilding it, and once again debug.

To enable it, it's really simple.  
You'll need to install fabulous-cli if you haven't already via the command line.
>dotnet tool install -g fabulous-cli

Then, add the corresponding NuGet package (here it's [Fabulous.XamarinForms.LiveUpdate](https://www.nuget.org/packages/Fabulous.XamarinForms.LiveUpdate/)) to all projects in the solution.  
After that, in your shared project (where your app code is), add the following lines after the declaration of the `runner`.

```fsharp
open Fabulous.XamarinForms.LiveUpdate

#if DEBUG
    do runner.EnableLiveUpdate ()
#endif
```

When debugging, a message will be printed in the output window of your IDE.

```
LiveUpdate: Ready for connection. Will show this message 3 more times.

LiveUpdate: Connect using:
   fabulous --watch --webhook:http://192.168.1.53:9867/update
```

Execute the given command line and you're ready to go!

<video height="420" controls>
  <source src="/assets/2019-12-21-how-to-become-a-fabulous-developer/liveupdate.mov" type="video/mp4">
</video>

# What's next?

The new way of writing apps using patterns like MVU and virtual views is getting a lot of traction recently. Well known frameworks like Elm, React-Redux or Flutter are making use of it. Even major actors want to provide their own (Apple with SwiftUI, Google with Android Jetpack Compose).

One thing to note is that such patterns are perfectly adapted to functional languages.

In the .NET world, Fabulous achieves this with F#.  
A really great language, an easy - reliable - maintainable architecture, an ultra-complete .NET ecosystem; perfect recipe for an awesome app!

Even more, Fabulous lets you choose the way you want to write your views with your framework of choice: static, dynamic or adaptive views.  
All that with useful tools like LiveUpdate.

For the future, we're working on getting Fabulous to a v1.0 production ready release.  
We're also working on adding more support for other frameworks, beyond Xamarin.Forms.

If you want to get started today, please read the documentation of your preferred way: [static views](https://fsprojects.github.io/Fabulous/Fabulous.StaticView/), [dynamic views](https://fsprojects.github.io/Fabulous/Fabulous.XamarinForms/index.html#getting=started), or [adaptive views](https://github.com/dsyme/fabulous-adaptive).

# Contributing

If you're interested to contribute to Fabulous, you can head to [the Fabulous repository on GitHub](https://github.com/fsprojects/Fabulous). There, you will be able to participate through issue reports, pull requests, code reviews and discussion.

Also if you want to stay updated, follow me on Twitter [@Tim_Lariviere](https://twitter.com/Tim_Lariviere).

I wish you a happy Christmas, and a lot of other awesome [#FsAdvent](https://sergeytihon.com/2019/11/05/f-advent-calendar-in-english-2019/) posts!