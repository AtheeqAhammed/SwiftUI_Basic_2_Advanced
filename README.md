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

## 🟢 Identifiable
Identifiable is a protocol used to give an object or model a unique identity.
SwiftUI uses this identity to track individual items when displaying dynamic collections, especially with ForEach and List.

For example, if I have a list of users, each user can have a unique id. When the data changes, SwiftUI can use that ID to determine which item was added, removed, or updated, instead of treating every item as a completely new view.

### Why is it needed?
SwiftUI is a declarative framework.
When the underlying data changes, SwiftUI needs to compare the old and new data and figure out what changed.

Identifiable provides a stable identity that helps SwiftUI efficiently update only the views that need to change.

What does Identifiable contain?
It mainly requires an id property whose type conforms to Hashable.
## 🟢 Hashable
Hashable is a protocol that allows a value to produce a hash value, so Swift can efficiently store and find it in collections like Set and Dictionary.
A Hashable type is also Equatable, meaning Swift can determine whether two values are equal.

Identifiable requires its ID type to be Hashable because SwiftUI needs a reliable, comparable identity for each item.

🟡 Hashing converts a value into a hash value that helps Swift quickly locate that value in hashed collections such as Set and Dictionary.
## 🟢 Equatable
Equatable is a protocol that allows two values to be compared to determine whether they are equal.
#### 🛒 Example: Shopping App
Imagine your app has products:
```swift
struct Product {
    let id: Int
    let name: String
    let price: Double
}
```

Now let's see where each protocol becomes useful.
1. 🔵 Identifiable → Displaying products in SwiftUI
You have a product list:
```swift
struct Product: Identifiable {
    let id: Int
    let name: String
    let price: Double
}
```
Then:
```swift
struct ProductListView: View {
    let products: [Product]

    var body: some View {
        List(products) { product in
            Text(product.name)
        }
    }
}
```

Why Identifiable?

Because SwiftUI needs to know:
"Which product is this row?"<br>
For example:<br>
`Product ID 101` → iPhone<br>
`Product ID 102` → MacBook<br>
`Product ID 103` → AirPods

If the product list changes, SwiftUI can track those products by their IDs.
Real app scenario
Product lists, chat messages, notifications, orders, contacts, etc.

2. 🔵 Equatable → Detecting whether something changed
Now imagine your product details screen.<br>
The user changes the quantity:

iPhone
Price: $999
Quantity: 1

You want to determine whether the product configuration has changed.<br>
You could make your model:
```swift
struct CartItem: Equatable {
    let productID: Int
    let quantity: Int
}
```
Then:
```swift
let oldItem = CartItem(productID: 101, quantity: 1)
let newItem = CartItem(productID: 101, quantity: 2)

if oldItem != newItem {
    print("Cart item changed")
}
```

Real app scenarios for Equatable<br>
You see this a lot in apps:
| Old state | New state |
|---|---|
| LoginState | LoginState |
| CartState | CartState |
| UserProfile | UserProfile |
| FilterState | FilterState |


You can ask:
oldState == newState

This can be useful for deciding:<br>
"Do I actually need to update the UI / perform an API call / save something?"

3. 🔵 Hashable → Favorites / Recently Viewed
Now imagine your shopping app has a Favorites feature.<br>
You don't want the same product added twice.

A Set is perfect:
```swift
struct Product: Hashable {
    let id: Int
    let name: String
    let price: Double
}
```
Then:
```swift
var favorites: Set<Product> = []

let iphone = Product(
    id: 101,
    name: "iPhone",
    price: 999
)

favorites.insert(iphone)
favorites.insert(iphone)
```

The second insertion doesn't create another copy.
You can do:
```swift
if favorites.contains(iphone) {
    print("Already in favorites")
}
```
Why Hashable?<br>
Because Set needs its elements to be Hashable.

4. 🔵 Hashable → Dictionary
Another real-world example is caching products.
Suppose you want:

Product ID → Product

You could have:
```swift
var productCache: [Int: Product] = [:]
```
Int is already Hashable, so this works.
```swift
productCache[101] = iphone
```
Later:
```swift
let product = productCache[101]
```
The dictionary can efficiently find the product using the key.

5. 🔵 All three together

In a real SwiftUI app, you might actually have:
```swift
struct Product: Identifiable, Equatable, Hashable {
    let id: Int
    let name: String
    let price: Double
}
```
And each protocol has a different job:
```text
Product
   │
   ├── Identifiable
   │      ↓
   │   SwiftUI knows:
   │   "Which product is this?"
   │
   ├── Equatable
   │      ↓
   │   App knows:
   │   "Are these two products equal?"
   │
   └── Hashable
          ↓
       Set / Dictionary:
       "Where can I efficiently find this product?"
```

⭐ A very practical example
Imagine your app has:
```swift
let products: [Product]
```
You show them:
```
ForEach(products) { product in
    Text(product.name)
}
```
→ Identifiable
User selects a product and you compare it:
```
if selectedProduct == product {
    // selected
}
```
→ Equatable
User adds it to favorites:

favorites.insert(product)

→ Hashable
That's the easiest way to understand them:
| Protocol | Easy way to remember |
|---|---|
| Identifiable | "Who are you?" |
| Equatable | "Are you the same as this?" |
| Hashable | "How can I efficiently store/find you?" |

## 🟢 EquatableView
EquatableView is a wrapper that tells SwiftUI to compare the old and new values of a view using Equatable.
If the values are equal, SwiftUI can skip updating the view's content.

It can be useful for optimizing expensive views when their inputs don't change frequently.

## 🟢 Diffing
Diffing means comparing the previous state of your UI with the new state to figure out what actually changed.

💬 Real app example, Imagine a chat app.<br>
Initially:
```text
Chat
----------------
John: Hi
Sam: Hello
Mike: How are you?
```
A new message arrives:
```text
Chat
----------------
John: Hi
Sam: Hello
Mike: How are you?
John: I'm good!
```
SwiftUI can identify that:
```text
John: Hi            → unchanged
Sam: Hello          → unchanged
Mike: How are you?  → unchanged
John: I'm good!     → NEW
```
So it doesn't conceptually need to treat the entire chat as brand-new.
That's one reason stable identity is important.

### 🔵 Diffing vs Equatable
| Concept | Meaning |
|---|---|
| Diffing | SwiftUI's general process of figuring out what changed. |
| EquatableView | A way to give SwiftUI an explicit equality check for a particular view. |

Diffing is the general process SwiftUI uses to determine what changed when the state or data changes.
EquatableView is a specific optimization mechanism where I make a view Equatable and tell SwiftUI to use that equality comparison.

If the old and new values are equal, SwiftUI can avoid updating that view's subtree.

So diffing is the broader mechanism, while EquatableView gives SwiftUI an explicit equality check for a specific view.

## 🟢 @AppStorage
@AppStorage is a SwiftUI property wrapper that persists a value in UserDefaults and automatically keeps the SwiftUI view in sync when that value changes.
It's useful for small, user-preference-type values such as dark mode preference, onboarding completion, selected settings, etc.

Example
```swift
struct SettingsView: View {
    @AppStorage("isDarkMode") private var isDarkMode = false

    var body: some View {
        Toggle("Dark Mode", isOn: $isDarkMode)
    }
}
```
When isDarkMode changes, SwiftUI writes the value to UserDefaults.
When the app launches again, the stored value can be restored.

## 🟢 @SceneStorage
@SceneStorage is a SwiftUI property wrapper used to preserve small amounts of UI state for a particular scene or window.
SwiftUI can restore that state when the scene is recreated, such as after being backgrounded or terminated and restored.

Example
```swift
struct ContentView: View {
    @SceneStorage("selectedTab") private var selectedTab = 0

    var body: some View {
        TabView(selection: $selectedTab) {
            Text("Home")
                .tabItem { Text("Home") }
                .tag(0)

            Text("Profile")
                .tabItem { Text("Profile") }
                .tag(1)
        }
    }
}
```
If the scene is recreated, SwiftUI can restore the previously selected tab.
### 🔵 @AppStorage vs @SceneStorage
This is the important interview distinction:
| @AppStorage | @SceneStorage |
|---|---|
| **Purpose** | Persistent app/user preferences | Restore UI state for a scene |
| **Scope** | App/user-wide | Individual scene/window |
| **Backed by** | UserDefaults | Scene-specific state restoration |
| **Example** | Dark-mode preference | Selected tab, navigation/UI state |
