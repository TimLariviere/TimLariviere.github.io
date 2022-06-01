---
layout: post
title: "Introducing Fabulous 2.0"
date: "2022-06-01"
---

# Introducing Fabulous 2.0

Almost a year of hard work, I am very excited to announce [Fabulous 2.0](https://github.com/fsprojects/Fabulous)!  
This new release is a complete rewrite of Fabulous, focusing on performance and developer experience.

Version 2.0 is the first of several major releases.  
It comes with a brand new DSL inspired by SwiftUI and improved performance to ensure the best possible development experience.

In later releases, we will introduce support for MAUI and support for components to help you scale your apps with ease.  
If you're interested, you can read more about it here: [Reasonable goals of v2](https://docs.fabulous.dev/v2/architecture/goals-of-v2)

I want to thank all Fabulous contributors for their contributions and support over the years.  
Special thanks to [Simon Korzunov (@twopSK)](https://twitter.com/twopSK) for helping design the new architecture and for his awesome work on performance improvements, and [Edgar Gonzalez (@efgpdev)](https://twitter.com/efgpdev) for his first contributions in Open Source by patiently mapping all Xamarin.Forms controls to Fabulous 2.0 and providing a lot of feedback.

## What's new in version 2.0

### New View DSL

We're introducing a new DSL for Fabulous, inspired by SwiftUI.  
This DSL is more concise, easier to extend and features proper type-safety to ensure your code is readable and always working as intended.

It makes use of 2 new concepts: Widgets and Modifiers.

Widgets are the building blocks of your UI. You compose them together to define what your app looks like.  
Some of those widgets will require essential properties to be passed as parameters: for example `Label` requires a `text` parameter, `Button` requires a `text` and `onClicked` parameters, etc.

In v1.0, properties were available as optional parameters and some as extension methods.  
Now in v2.0, properties are now all set through modifiers.

Those modifiers are accessible via the dot notation.


```fs
Label("Hello, World")
    .font(namedSize = NamedSize.Large, fontFamily = "Arial")
    .textColor(light = FabColor.Red, dark = FabColor.Blue)
    .horizontalTextAlignment(TextAlignment.Center)
    .padding(10.)
```

Thanks to the strongly-typed nature of Widget, the compiler will now prevent you from combining incompatible widgets.

![Error message when combining incompatible widgets](/assets/2022-06-01-introducing-fabulous-20/widget-incompatible-types.png)

And your IDE will only show you the modifiers available for the current widget.

![IDE experience in v2.0](/assets/2022-06-01-introducing-fabulous-20/ide-experience-v2.png)

For comparison, here is the same UI in both v1.0 and v2.0:

```fs
/// Fabulous 1.0
let view model dispatch =
    View.ContentPage(
        title = "Home",
        content =
            View.StackLayout(
                orientation = StackOrientation.Vertical,
                spacing = 8.0,
                children = [
                    View.Label(
                        text = "Welcome to Fabulous 1.0!",
                        fontSize = FontSize.FromNamedSize(NamedSize.Large)
                    )

                    View.Label(
                        text = $"Click count = {model.Count}"
                    )

                    View.Button(
                        text = "Click me!",
                        command = (fun () -> dispatch ButtonClicked),
                        foregroundColor = Color.FromHex("#FF0000")
                    )
                ]
            )
    )
    
// Fabulous 2.0
let view model =
    ContentPage("Home",
        VStack(spacing = 8.0) {
            Label("Welcome to Fabulous 2.0!")
                .font(namedSize = NamedSize.Large)

            Label($"Click count = {model.Count}")
            
            Button("Click me!", ButtonClicked)
                .foregroundColor(FabColor.fromHex "#FF0000")
        }
    )
```

You may have noticed `dispatch` is missing in Fabulous 2.0. We take care of dispatching messages for you now, so you can focus on the UI itself.

### Focus on performance

The second major improvement is reduced allocations on hot paths. This will make your apps faster and more responsive.

Almost all the internal infrastructure of Fabulous now works with structs instead of classes.  
This drastically reduces GC pressure on your apps by keeping as much as possible on the stack.

We've also optimized for CPU cache misses to achieve even better performances.

And finally, we optimized for trimming in Release builds to produce the smallest binaries possible.

_Comparison of dll sizes for FabulousContacts between Fabulous 1.0 and 2.0 (iOS release build with linker enabled)_

| File | 1.0 | 2.0 | Difference |
|---|---|---|---|
| Fabulous.dll | 57 KB | 209 KB | +152 KB |
| Fabulous.XamarinForms.dll* | 8.43 MB | 1.08 MB | -7.35 MB |
| Fabulous.XamarinForms.Maps.dll | 219 KB | N/A** | -219 KB |
| FabulousContacts.dll | 171 KB | 158 KB | -13 KB |
| Total | 8.87 MB | 1.44 MB | **-7.43 MB (-83.8%)** |
||||
| Estimated App Install Size*** | 48.5 MB | 42.0 MB | **-6.5 MB (-13.4%)** |

More than 80% reduction in dll size overall for Fabulous 2.0, leading to 13% size reduction for the FabulousContacts app itself!  
This is a huge improvement over 1.0.

\* Please note that for some dlls, the F# optimization and signature data can make for a big chunk of the dll size (2/3 in case of F.XF.dll, about ~0.7 MB).  
Those data are not used at runtime but the Xamarin linker doesn't remove them.  
After we start support for MAUI, the dll sizes will be much smaller thanks to the .NET 6.0 linker removing those unused data.

\** Fabulous.XamarinForms.Maps is not available yet for Fabulous 2.0.  
Instead the Map widget has been directly integrated into FabulousContacts.

\*** Decompressed IPA files

### Better 3rd party support and extensibility

With the new Widgets and Modifiers, extending an existing widget is as easy as declaring an extension method!  
Also makes it very easy to add 3rd party support.

Here is an example of adding support for the `Xamarin.Forms.Map` control:

```fs
// Declare a marker interface (used for strong-typing the widget)
type IMap = inherit Fabulous.XamarinForms.IView

module Map =
    // Tell Fabulous to instantiate Xamarin.Forms.Map when using Map widget
    let WidgetKey = Widgets.register<Xamarin.Forms.Map>()

    // Define which properties will be available
    let RequestedRegion = Attributes.defineSimpleScalarWithEquality<MapSpan> (...) // code omitted for brevity
    let HasZoomEnabled = Attributes.defineBindableBool Xamarin.Forms.Map.HasZoomEnabledProperty
    let HasScrollEnabled = Attributes.defineBindableBool Xamarin.Forms.Map.HasScrollEnabledProperty

[<AutoOpen>]
module MapBuilders =
    type Fabulous.XamarinForms.View with
        // Expose Map widget with a specific constructor (requestedRegion will be mandatory)
        static member inline Map<'msg>(requestedRegion) =
            WidgetBuilder<'msg, IMap>(
                Map.WidgetKey,
                Map.RequestedRegion.WithValue(requestedRegion)
            )

[<Extension>]
type MapModifiers =
    // Define the withZoom modifier
    [<Extension>]
    static member inline withZoom(this: WidgetBuilder<'msg, #IMap>, ?enabled: bool) =
        let value = match enabled with None -> true | Some v -> v
        this.AddScalar(Map.HasZoomEnabled.WithValue(value))

    // Define the withScroll modifier
    [<Extension>]
    static member inline withScroll(this: WidgetBuilder<'msg, #IMap>, ?enabled: bool) =
        let value = match enabled with None -> true | Some v -> v
        this.AddScalar(Map.HasScrollEnabled.WithValue(value))

// Usage
Map(MapSpan.FromCenterAndRadius(...))
    .withZoom()
    .withScroll(false)
```

## Support for other frameworks

One key feature of Fabulous is the possibility to be integrated with frameworks, other than Xamarin.Forms.

We made it even easier in version 2.0 by providing the whole reconciliation infrastructure directly as part of the Fabulous library.  
All you need to do to integrate Fabulous with your framework is to implement the widgets and modifiers you need.

Fabulous will take care of the rest.

## New Fabulous website

Along with this new release, we also launched a new website dedicated to Fabulous.

https://fabulous.dev

It's a great place to learn more about Fabulous, and to see how it's used in the wild.  
Documentation is available at https://docs.fabulous.dev

Right now, the website is in a very early stage, but we're working hard to make it as useful as possible.  
We are currently focusing on adding more documentation for 2.0.

We are looking for contributors to help us improve the website itself.  
If you're interested in contributing to the website, please contact us on the Fabulous [Discord channel](https://discord.gg/bpTJMbSSYK).

## Get started today

You can start right now by installing the [templates](https://www.nuget.org/packages/Fabulous.XamarinForms.Templates) available on NuGet!

Follow the Getting Started guide to learn start to use Fabulous 2.0: https://docs.fabulous.dev/v2/getting-started

If you have any questions, feel free to ask them on the [Discord channel](https://discord.gg/bpTJMbSSYK) or raise issues on GitHub: https://github.com/fsprojects/Fabulous  
We are also available on Twitter [@FabulousAppDev](https://twitter.com/FabulousAppDev).