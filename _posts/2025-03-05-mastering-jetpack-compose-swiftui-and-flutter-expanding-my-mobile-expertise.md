---
layout: post
title: "Mastering Jetpack Compose, SwiftUI, and Flutter: Expanding My Mobile Expertise"
date: "2025-03-05"
---

## A Decade of Mobile Development and Beyond
Over the past eleven years, I've worked on everything from high-performance microservices handling billions of hotel bookings to a government-backed mobile app that helps citizens record biodiversity in Australia. But no matter what I’ve worked on, mobile development has always been what I love most.

For almost nine years, I've been dedicated to creating mobile apps that users interact with daily, ensuring a smooth experience, optimal performance, and scalable architecture.

Since 2020, my journey has taken an exciting turn into freelancing, where I help businesses build, maintain, and scale their mobile applications. A large part of this expertise has come from co-creating and maintaining [Fabulous](https://fabulous.dev), a declarative mobile framework for .NET, inspired by architectures like SwiftUI and Elm. My contributions to this field have also been recognized through five consecutive Microsoft MVP awards.

With this foundation, it’s time to push my expertise further—by mastering Jetpack Compose, SwiftUI, and Flutter.

## Why Learn New Mobile Frameworks?
Mobile development is always evolving, and keeping up means embracing modern UI frameworks. While React Native is a major player, I prefer strongly typed languages, as I believe a proper type system prevents over 90% of common bugs. That’s one of many ways JavaScript falls short, which is why I’m drawn to Jetpack Compose (Kotlin), SwiftUI (Swift), and Flutter (Dart).

These frameworks have become the go-to choices for Android, iOS, and cross-platform development, and knowing all three lets me offer businesses the right tool for the job instead of a one-size-fits-all approach.

But it’s not just about picking up new technologies—it’s about seeing how different frameworks solve the same problems in their own way. Learning multiple approaches helps refine best practices, improve architecture, and ultimately build better apps.

## What Truly Separates a Great Mobile Developer?
A common misconception in hiring and evaluating engineers is focusing solely on years of experience with a specific technology. While experience is valuable, true expertise comes from understanding the core principles of software engineering—which translate across languages and frameworks.

Through my work over the years, I have already developed expertise in:

- **Business Goals** – Understanding user and stakeholder needs and translating them into features  
- **User Experience** – Designing smooth, performant, and accessible apps for a delightful experience  
- **Software Architecture** – Building scalable, maintainable, and modular systems  
- **State Management** – Choosing the right architecture for the app’s needs (MVU, MVVM, Redux, etc.)  
- **Performance Optimization** – Ensuring responsiveness and efficient memory use  
- **CI/CD Pipelines** – Automating builds, testing, and deployment for mobile apps  
- **App Distribution** – Navigating Apple & Google Play requirements for seamless deployment  

These skills aren't framework-dependent—they are fundamental to great mobile development.  

Mastering new mobile UI frameworks isn't about starting from scratch—it's about adapting existing expertise to different ecosystems.

## My Advantage: Building a Framework from the Inside Out
Most developers learn new frameworks by watching tutorials, reading documentation, and experimenting. My approach is different—I have already built a mobile UI framework.

For over seven years, my work on Fabulous has given me inside-out knowledge of what makes declarative UI frameworks powerful and scalable. This experience gives me a unique perspective when learning new frameworks, as I can analyze not just how they work, but why they were designed the way they are.

Through this journey, I aim to apply my deep understanding of declarative UI to Jetpack Compose, SwiftUI, and Flutter while refining my approach to mobile development across ecosystems.

## Understanding the Similarities Between Frameworks
One of the reasons I feel confident jumping into these new frameworks is that I already understand the core principles behind them. Declarative UI frameworks share fundamental concepts, including unidirectional data flow, state management, and component-based design.

To illustrate, here’s a simple counter app implemented in SwiftUI, Jetpack Compose, and Fabulous:

### SwiftUI
```swift
struct CounterView: View {
    @State private var count = 0

    var body: some View {
        VStack {
            Text("\(count)")
            Button("Increment") {
                count += 1
            }
        }
    }
}
```

### Jetpack Compose
```kotlin
@Composable
fun CounterView() {
    var count by remember { mutableStateOf(0) }

    Column {
        Text("$count")
        Button(onClick = { count++ }) {
            Text("Increment")
        }
    }
}
```

### Fabulous
```fsharp
let CounterView () =
    Component() {
        let! count = Context.State(0)

        VStack(spacing = 20.) {
            Text($"{count.Current}")
            Button("Increment", fun () -> count.Set(count.Current + 1))
        }
    }
```

These examples demonstrate how similar modern UI frameworks have become. While the syntax differs, the underlying concepts—such as state-driven rendering and UI composition—are nearly identical.

## Introducing Snapdex: A Production-Quality Playground for My Learning
As I embark on mastering these frameworks, I wanted a real-world project to apply my knowledge. That’s why I started working on Snapdex.

### What is Snapdex?
Inspired by the Pokédex from Pokémon, Snapdex is an app that lets users track Pokémon they encounter in the real world. Whether it’s a plush toy, a poster, a trading card, or even a sound clip from a video game or TV, users can take pictures or record audio, and the app will use AI-powered recognition to identify the Pokémon and unlock its corresponding entry in the Pokédex.

This project is an opportunity to demonstrate expertise across the full spectrum of mobile development, ensuring a seamless and high-quality user experience:
- **User Experience & Interface** – Designing clean, intuitive, and visually appealing UI with smooth animations.
- **Robust Architecture** – Implementing best practices in MVI, MVVM, and Clean Architecture for maintainability.
- **Data Management & Offline Support** – Handling API calls, local database persistence, and seamless offline syncing.
- **Security & Authentication** – Protecting user data with encrypted storage, secure tokens, and access controls.
- **Cloud & AI Integration** – Leveraging Firebase/AWS for backend services and AI-powered Pokémon recognition.
- **Platform-Specific Features** – Accessing camera, microphone, dark mode, system notifications, and deep linking.
- **Performance & Stability** – Optimizing image caching, monitoring analytics, and ensuring crash resilience.
- **Automation & Deployment** – Streamlining development with CI/CD, Gradle, Swift Package Manager, and automated testing.
- **Localization & Accessibility** – Adapting the app for multiple languages and inclusive design.

Each of these aspects will be carefully designed and implemented, ensuring that Snapdex is not just a functional app, but a showcase of best practices in modern mobile development.

Everything will be open-source on [GitHub](https://github.com/TimLariviere/Snapdex), allowing you to see my work firsthand and evaluate my expertise in real-world scenarios.

### Why Snapdex?
- It involves rich UI interactions, perfect for testing new frameworks  
- It requires high-performance rendering, a challenge across mobile platforms  
- It’s a fun and engaging project, keeping me motivated as I learn  

By developing Snapdex in Jetpack Compose, SwiftUI, and Flutter, I can directly compare how each framework handles UI composition, animations, performance optimizations, and scalability.

## Looking Ahead
Mastering these frameworks is more than just a technical challenge—it’s about broadening my expertise, increasing my flexibility, and preparing for the future of mobile development.

I believe that the best mobile developers aren't defined by a single framework, but by their understanding of core engineering principles. Jetpack Compose, SwiftUI, and Flutter represent the future of mobile development, and I’m excited to push my skills further by diving deep into each of them.

This is the next chapter of my journey—and I’m eager to share what I learn along the way.

### Final Thoughts
If you're also navigating the shift to modern declarative UI frameworks, I’d love to hear your thoughts! Have you transitioned to Jetpack Compose, SwiftUI, or Flutter? What challenges did you face, and how did your prior experience shape your learning process? Let’s connect and learn together.

### Follow My Progress:

- Find me at **[https://timothelariviere.com](https://timothelariviere.com)**
- Follow **[Snapdex on GitHub](https://github.com/TimLariviere/Snapdex)**
- Learn more about **[Fabulous](https://fabulous.dev)**
- Connect with me on **[LinkedIn](https://linkedin.com/in/timlariviere)**