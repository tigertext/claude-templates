## Views

### ViewState

This pattern defines the possible states a view can take. Use this pattern when implementing views with finite states.

```swift
enum ViewState: Equatable {
    case loading
    case loaded(items: Items)
    case error(message: String)
}
```

Views `switch` on `viewModel.state` to decide what to render.

```swift
struct TCInboxSearchView: View {
    let viewModel: TCInboxSearchViewModel

    @ViewBuilder
    var content: some View {
        switch viewModel.state {
        case .loading:
            ProgressView()
        case .loaded(let items):
            InboxSearchListView(items: items)
        case .error(let message):
            TCErrorView(title: "AnErrorHasOccurred".localized(bundle: .module), message: message)
        }
    }
}
```