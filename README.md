# SwiftUI_Basic_2_Advanced
> [!IMPORTANT] SwiftUI Topics from Basic to Advanced

## 🟢 **State**
@State is a property wrapper used to store local, mutable state that belongs to a view.
When the value changes, SwiftUI automatically recomputes the view's body so the UI reflects the new value.

## 🟢 **Binding**
@Binding is used when another view needs to access and change that value. The second view can use the value, but it does not own it.
Binding is a two-way connection between a parent view and its child view. It allows the child view to read and modify a value of a parent view without owning it.

## 🟢 **Observable Object**
ObservableObject is a protocol that allows a reference-type object to publish changes so that SwiftUI views observing it can update when its data changes.
It is commonly used for ViewModels to separate UI state and business logic from the View.

Properties marked with @Published notify SwiftUI when their values change, allowing dependent views to update.

```swift
class UserViewModel: ObservableObject {
    @Published var name = "John"
}
```

@Published tells SwiftUI that when name changes, observers should be notified.
A view can observe it using:

```swift
@StateObject private var viewModel = UserViewModel()
```

or receive an existing instance:
```swift
@ObservedObject var viewModel: UserViewModel
```

### 🔵 Key Distinction

| Property / Protocol | Purpose |
|---|---|
| `ObservableObject` | Makes an object capable of publishing changes. |
| `@Published` | Marks properties whose changes should be published. |
| `@StateObject` | A view owns and manages the observable object's lifetime. |
| `@ObservedObject` | A view observes an object owned elsewhere. |

## 🟢 @StateObject
@StateObject is used when a view needs to create and own the lifecycle of an ObservableObject.
SwiftUI ensures that the object is initialized once for that view’s identity and preserved across view redraws.

```swift
class UserViewModel: ObservableObject {
    @Published var name = "John"
}

struct UserView: View {
    @StateObject private var viewModel = UserViewModel()

    var body: some View {
        Text(viewModel.name)
    }
}
```
### 🔵 Important Distinction: Ownership

`@StateObject` → This view owns the object<br>
`@ObservedObject` → This view observes an object owned elsewhere<br>
`@EnvironmentObject` → The object is supplied through the SwiftUI environment

## 🟢 @ObservedObject
@ObservedObject is a SwiftUI property wrapper used when a view needs to observe an ObservableObject that is owned and managed somewhere else.
When the object's published properties change, SwiftUI can update the view.

Example
```swift
class UserViewModel: ObservableObject {
    @Published var name = "John"
}

struct UserView: View {
    @ObservedObject var viewModel: UserViewModel

    var body: some View {
        Text(viewModel.name)
    }
}
```
The parent owns the object
```swift
struct ParentView: View {
    @StateObject private var viewModel = UserViewModel()

    var body: some View {
        UserView(viewModel: viewModel)
    }
}
```
Here

`@StateObject` Parent owns the UserViewModel<br>
`@ObservedObject` → Child observes the existing UserViewModel<br>
`@Published` → Notifies observers when a property changes

## 🟢 Environment
Environment in SwiftUI is a way to share data or dependencies across a view hierarchy.
Instead of passing the same object through every view’s initializer, We can put it into the environment at a higher level, and any child view that needs it can access it directly.

It’s useful when the same data or service is needed by multiple views.

## 🟢 StateObject
@StateObject is used when a SwiftUI View owns an ObservableObject.
It creates and maintains the object's lifecycle so the same object instance is preserved across View updates.

It's commonly used when a View creates its own ViewModel.

## 🟢 EnvironmentObject
@EnvironmentObject is used to access a shared ObservableObject from the SwiftUI environment.
Instead of passing the object through multiple view initializers, a parent can inject it into the environment, and any descendant view can access the same object using @EnvironmentObject.

The consuming view doesn't own the object.

Example
```swift
class UserSession: ObservableObject {
    @Published var username = "John"
}

struct ContentView: View {
    @StateObject private var session = UserSession()

    var body: some View {
        HomeView()
            .environmentObject(session)
    }
}

struct HomeView: View {
    @EnvironmentObject var session: UserSession

    var body: some View {
        Text(session.username)
    }
}
```
Here, ContentView injects session:
```swift
.environmentObject(session)
```
and HomeView retrieves it:
```swift
@EnvironmentObject var session: UserSession
```
The important point is that HomeView doesn't need:
```swift
HomeView(session: session)
```
and neither do intermediate views.

### 🔵 @Environment vs @EnvironmentObject
This is a good interview distinction:

`@Environment` → reads values/dependencies from SwiftUI's environment.<br>
`@EnvironmentObject` → specifically retrieves an ObservableObject that was injected into the environment.<br>
`@StateObject` → owns an observable object.<br>
`@ObservedObject` → observes an existing observable object.<br>
`@Binding` → provides read/write access to state owned elsewhere.

## 🟡 SwiftUI Property Wrapper Summary

| Property Wrapper | Purpose |
|---|---|
| `@State` | Stores local, temporary state owned by a View |
| `@Binding` | Creates a two-way connection to state owned by another View |
| `@StateObject` | Creates and owns an observable reference-type object |
| `@ObservedObject` | Observes an object that is owned elsewhere |
| `@EnvironmentObject` | Gets a shared observable object from the environment |
| `@Environment` | Reads values provided by SwiftUI's environment |
| `@AppStorage` | Persists a value using UserDefaults |
| `@SceneStorage` | Preserves state for a particular scene/session |
