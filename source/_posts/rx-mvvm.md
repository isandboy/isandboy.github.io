---
title: Reactive-MVVM
date: 2018-01-15 20:57:24
tags:
---

原文：https://medium.com/@mecid/mastering-mvvm-on-ios-f875d2b99816

译者：Extra Mo

## Mastering MVVM on iOS

网上有很多关于app架构的文章，今天，我将展示一些使用MVVM架构开发iOS应用时的技巧。下面我不将介绍其他架构，如果你需要了解更多，这有一篇不错的文章 (https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52).

### Why MVVM?

MVC的主要问题是混合责任，这导致了一些问题的出现，比如重量级视图控制器。

我们都知道`UIViewController`是苹果iOS SDK的主要组件，所有的操作都是在这里面开始和构建。尽管是叫`UIViewController`，但它更`View`和来自于MVC(或MVP)的一个经典控制器(或`Presenter`)相比，因为它里面有`viewDidLoad`、`viewWillLayoutSubviews`和其他视图相关方法的回调。这就是为什么我们应该忽略名称中的Controller关键字，并将其作为视图使用，而实际控制器的角色则采用ViewModel。

ViewModel是视图的完整数据表示。每个视图应该仅仅只能拥有一个ViewModel实例。通常，ViewModel使用一个管理器来获取数据并将其转换为试图需要的格式。请看以下列子：

```
import Foundation

class ItemsViewModel {
    var items: [Item] = []
    var error: Error?
    var refreshing = false

    private let dataManager: DataManager
    init(dataManager: DataManager) {
        self.dataManager = dataManager
    }

    func fetch(completion: @escaping () -> Void) {
        refreshing = true
        dataManager.fetchItems { [weak self] (items, error) in
            self?.items = items ?? []
            self?.error = error
            self?.refreshing = false
            completion()
        }
    }
}
```
这里我们的ViewModel，通过DataManager获取`items`并将其保存在某个变量中。它还有一个错误和刷新状态的变量，这就带来了在所有需要的条件下构建UI的机会。

```
import UIKit

class ItemsViewController: UIViewController {
    @IBOutlet private weak var tableView: UITableView!
    private var viewModel: ItemsViewModel

    init(viewModel: ItemsViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        viewModel.fetch { [weak self] in
            self?.tableView.reloadData()
        }
    }
}

extension ItemsViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return viewModel.items.count
    }
}

```

正如您在上面看到的，我们的`ItemsViewController`包含显示列表数据的`UITableView`。它含有ViewModel实例，并要求它在`viewDidLoad`回调中获取数据。我们还传递闭包，当数据被获取时，它将重新加载`UITableView`。  `UITableViewDataSource`方法也使用ViewModel来获取cell个数。

### Reactive Bindings

View和ViewModel之间的绑定是MVVM模式的主要思想，开发人员可以在ViewModel中编写逻辑代码，在View中做布局。在第一个例子中，我们使用了闭包，因为iOS SDK不支持绑定。在实际应用中，您可以使用一些流行的FRP扩展，比如`ReactiveCocoa`、`RxSwift`或`Bond`。我倾向于使用`Bond`，因为它简单。

```
import Bond

class ItemsViewModel {
    let items = Observable<[Item]>([])
    let error = Observable<Error?>(nil)
    let refreshing = Observable<Bool>(false)

    private let dataManager: DataManager
    init(dataManager: DataManager) {
        self.dataManager = dataManager
    }

    func fetch() {
        refreshing.value = false
        dataManager.fetchItems { [weak self] (items, error) in
            self?.items.value = items ?? []
            self?.error.value = error
            self?.refreshing.value = false
        }
    }
}
```
这是相同的ItemsViewModel，但现在我们使用响应式编程来观察变化。

```
class ItemsViewController: UIViewController {
    @IBOutlet private weak var tableView: UITableView!
    private let activityIndicator = ActivityIndicatorView()
    private var viewModel: ItemsViewModel

    init(viewModel: ItemsViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bindViewModel()
        viewModel.fetch()
    }

    func bindViewModel() {
        viewModel.refreshing.bind(to: activityIndicator.reactive.isAnimating)
        viewModel.items.bind(to: self) { strongSelf, _ in
            strongSelf.tableView.reloadData()
        }
    }

    func setupUI() {
        view.addSubview(activityIndicator)
    }
}

extension ItemsViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return viewModel.items.value.count
    }
}
```
让我们看一下`bindViewModel`方法，这里我们将把ViewModel绑定到视图。每当刷新值改变时，它设置activityIndicator的isAnimating属性。或者当items被更改时，`UITableView`重新加载。正如您所看到的，在大多数情况下，响应式绑定简化了代码。

### ViewModel Composition

有时，我们有多个数据源的复杂视图。例如，在`Instagram`应用中的用户资料中，我们有一些关于用户的信息以及所有与该用户有关的照片。这里好的做法是将这个逻辑分成两个或多个视图模型。但是我们有一个规则:每个视图应该只有一个ViewModel。在这种情况下，最好的选择是使用`ViewModel`组合。让我们来看看例子:

```
import Bond
import ReactiveKit

class UserProfileViewModel {
    let refreshing = Observable<Bool>(false)
    let username = Observable<String>("")
    let photos = Observable<[Photos]>([])

    private let userViewModel: UserViewModel
    private let photosViewModel: PhotosViewModel

    init(userManager: UserManager, photoManager: PhotoManager) {
        userViewModel = UserViewModel(manager: userManager)
        photosViewModel = PhotosViewModel(manager: photoManager)

        userViewModel.username.bind(to: username)
        photosViewModel.photos.bind(to: photos)
        combineLatest(userViewModel.refreshing, photosViewModel.refreshing)
            .map { $0 || $1 }
            .bind(to: refreshing)
    }

    func fetch() {
        userViewModel.fetch()
        photosViewModel.fetch()
    }
}

class UserViewModel {
    let refreshing = Observable<Bool>(false)
    let username = Observable<String>("")

    func fetch() {
        refreshing.value = true
        manager.fetch(user: id) { [weak self] (user, error) in
            self?.username.value = "@" + user.username
            self?.refreshing.value = false
        }
    }
}

class PhotosViewModel {
    let refreshing = Observable<Bool>(false)
    let photos = Observable<[Photo]>([])

    func fetch() {
        refreshing.value = true
        manager.fetch(for user: id) { [weak self] (photos, error) in
            self?.photos.value = photos ?? []
            self?.refreshing.value = false
        }
    }
}
view rawViewModelComposition.swift hosted with ❤ by GitHub

```

正如您所看到的，我们使用`UserProfileViewModel`，它含有两个视图模型并从它们中获取数据。我们还有一个刷新新的状态，它包含了两个内部视图模型的刷新状态。第二个重要点是在第36行中，ViewModel将数据格式化为所需的表单数据。视图只需要将组件绑定到ViewModel并显示数据。

## Conclusion

ViewModel是将表示逻辑层分离到其他层的一种非常简单的方式，它帮助我们避免了重量级的视图控制器，使我们更容易控制和覆盖单元测试。这就是我们的目的，简单而可测试的架构。
