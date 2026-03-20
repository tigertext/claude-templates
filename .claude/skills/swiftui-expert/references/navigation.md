## Navigation

### Pure SwiftUI NavigationStack + NavigationPath

```swift
struct MyFeatureRootView: View {
    @State private var path = NavigationPath()

    var body: some View {
        NavigationStack(path: $path) {
            MyFeatureListView(onSelectItem: { item in
                path.append(MyRoute.detail(item))
            })
            .navigationDestination(for: MyRoute.self) { route in
                switch route {
                case .detail(let item):
                    MyFeatureDetailView(item: item)
                }
            }
        }
    }
}

enum MyRoute: Hashable {
    case detail(MyItem)
}
```

### UIKit/SwiftUI Bridging

When SwiftUI views live inside UIKit via `UIHostingController`, use the Route + Navigator pattern:

**Route enum:**

```swift
enum TCInboxSearchRoute {
    case conversationPreview(id: String)
    case userDetails(user: User)
}
```

**Navigator struct** — pure UIKit imperative logic, no ViewModel or View knowledge:

```swift
@MainActor
struct Navigator {
    static func navigate(
        _ route: Route,
        using navigationController: UINavigationController?
    ) {
        switch route {
        case .userDetails(let user):
            let vc = UserDetailsViewController(user: user)
            navigationController?.pushViewController(vc, animated: true)
        case .conversationPreview(let id):
            let viewModel = ConversationViewModel(id: id)
            let vc = ConversationViewController(viewModel: viewModel)
            viewModel.onNavigate = { [weak vc] route in
                Self.navigate(route, using: vc?.navigationController)
            }
            navigationController?.pushViewController(vc, animated: true)
        }
    }
}
```

**ViewController wires it up** via an `onNavigate` closure on the ViewModel:

```swift
private func setupHandlers() {
    viewModel.onNavigate = { [weak self] route in
        guard let self else { return }
        let navController = navigationController ?? sourceViewController?.navigationController
        TCInboxSearchNavigator.navigate(
            route,
            using: navController,
            onNotFound: { [weak self] in self?.handleFailedNavigation(for: route) }
        )
    }
}
```
