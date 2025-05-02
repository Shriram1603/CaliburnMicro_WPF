# CaliburnMicro_WPF

Caliburn Micro simplifies 5 things :

1. Bindings
2. Lifecycly management
3. Actions (eg : Button clicks)
4. Navigation
5. Dependency injection

BLAND in short

## 1. Data Binidng: Traditional vs Caliburn.Micro

Traditional WPF Binding:
```xaml
<TextBox Text="{Binding Username, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
<Button Command="{Binding LoginCommand}"/>
```
```csharp
// ViewModel
public ICommand LoginCommand => new RelayCommand(Login);

private string _username;
public string Username {
    get => _username;
    set {
        _username = value;
        OnPropertyChanged();
    }
}
```
Pain Points:

  * Manual property change notifications

  * Explicit command definitions

  * Verbose XAML binding syntax

Caliburn.Micro Binding :
```xaml
<TextBox x:Name="Username"/>
<Button x:Name="Login"/>
```
```csharp
public class LoginViewModel : Screen {
    private string _username;
    public string Username {
        get => _username;
        set => Set(ref _username, value);
    }
    
    public void Login() { ... }
}
```
Magic Sauce:
  * Automatic convention-based binding

  * Set() method handles property notifications

  * Method names match control names

  * No explicit command interfaces needed

Key Class: `PropertyChangedBase`

  * Base class providing Set() method

  * Implements INotifyPropertyChanged

  * Reduces boilerplate by 70%

## 2. Navigation: Traditional vs Caliburn.Micro
Traditional Navigation:
```csharp
// MainWindow.xaml.cs
private void ShowSettings_Click(object sender, RoutedEventArgs e) {
    var settings = new SettingsWindow();
    settings.DataContext = new SettingsViewModel();
    settings.ShowDialog();
}
```
Issues:

  * Tight coupling between views

  * Code-behind logic

  * Manual DI management

Caliburn.Micro Navigation
```csharp
public class ShellViewModel : Conductor<object> {
    public ShellViewModel() {
        ActivateItemAsync(new DashboardViewModel());
    }
}

public class DashboardViewModel : Screen {
    private readonly IWindowManager _windowManager;
    
    public DashboardViewModel(IWindowManager windowManager) {
        _windowManager = windowManager;
    }
    
    public async Task OpenSettings() {
        await _windowManager.ShowDialogAsync(new SettingsViewModel());
    }
}
```

Key Components:

1. `Conductor<T>`

    * Manages active screen

    * Handles lifecycle events

    * Enables parent-child relationships

2. `IWindowManager`

    * Abstracts window/dialog creation

    * Maintains MVVM purity

    * Provides ShowWindow/ShowDialog methods

Advantages:

  * Zero code-behind

  * Dependency-injected navigation

  * Lifecycle-aware components


## 3. Dependency Injection: Vanilla vs Caliburn

Traditional DI Setup
```csharp
// App.xaml.cs
protected override void OnStartup(StartupEventArgs e) {
    var container = new ServiceCollection()
        .AddSingleton<IMyService, MyService>()
        .BuildServiceProvider();
    
    var mainWindow = new MainWindow {
        DataContext = container.GetService<MainViewModel>()
    };
    mainWindow.Show();
}
```
Caliburn.Micro DI
```csharp
public class Bootstrapper : BootstrapperBase {
    private SimpleContainer _container;

    protected override void Configure() {
        _container = new SimpleContainer();
        _container.Instance(_container);
        _container
            .Singleton<IWindowManager, WindowManager>()
            .Singleton<IEventAggregator, EventAggregator>()
            .PerRequest<ShellViewModel>();
    }

    protected override object GetInstance(Type service, string key) {
        return _container.GetInstance(service, key);
    }
}
```
Key Classes:

1. `Bootstrapper`

  * Replacement for App.xaml startup

  * Central configuration point

  * Initializes framework components

2. `SimpleContainer`

  * Lightweight IoC container

  * Supports:

    * Singleton: One instance per app

    * PerRequest: New instance per resolution

    * Instance: Pre-built instance

## 4. Messaging: Traditional vs EventAggregator

Traditional Event Messaging

```csharp
// Publisher
public event EventHandler<UserLoggedInEventArgs> UserLoggedIn;

// Subscriber
public class DashboardViewModel {
    public DashboardViewModel(LoginViewModel login) {
        login.UserLoggedIn += OnUserLogin;
    }
}
```

Problems:

  * Tight coupling

  * Memory leak risks

  * No type safety

Caliburn's EventAggregator

```csharp
// Publisher
public class LoginViewModel {
    private readonly IEventAggregator _events;
    
    public async Task Login() {
        await _events.PublishOnUIThreadAsync(new UserLoggedInMessage(user));
    }
}

// Subscriber
public class DashboardViewModel : IHandle<UserLoggedInMessage> {
    public async Task HandleAsync(UserLoggedInMessage message, CancellationToken ct) {
        // Update UI
    }
}
```

Key Class: `IEventAggregator`

  * Pub/sub messaging system

  * Decouples components

  * Thread-safe UI marshalling

  * Strongly-typed messages

## 5. Lifecycle Management

Traditional Lifecycle

```csharp
public class MyViewModel : INotifyPropertyChanged {
    public void LoadData() { ... }
    
    public void Cleanup() { ... } // ← Manual cleanup needed
}
```

Issues:

  * No standard lifecycle hooks

  * Memory management challenges

  * No deactivation support

Caliburn's Lifecycle
```csharp
public class UserDetailsViewModel : Screen {
    protected override async Task OnInitializeAsync(CancellationToken ct) {
        await LoadUserData();
    }
    
    protected override Task OnActivateAsync(CancellationToken ct) {
        // When view becomes active
        return Task.CompletedTask;
    }
    
    protected override Task OnDeactivateAsync(bool close, CancellationToken ct) {
        // When view is closed/navigated from
        return Task.CompletedTask;
    }
}
```

Key Classes:

1. `Screen`

  * Base class for all view models

  * Provides activation/deactivation lifecycle

  * Implements IViewAware for view interaction

2. `Conductor<T>`

  * Manages collection of screens

  * Handles active item switching

  * Three variants:

      * OneActive (Tab-style)

      * AllActive (Dashboard-style)

      * Collection (Multiple children)

## Why Async Everywhere?

Caliburn.Micro embraces modern async patterns:

```csharp
public async Task HandleAsync(NavigateMessage message) {
    await DeactivateItemAsync(_currentView, true);
    await ActivateItemAsync(message.TargetViewModel);
}
```

Benefits:

  * Non-blocking UI operations

  * Proper activation/deactivation sequencing

  * Easy error handling with try/catch

  * Natural integration with async APIs

## When to Use Caliburn.Micro?

![Screenshot 2025-05-02 at 11 46 51 AM](https://github.com/user-attachments/assets/93b2652d-3799-437d-9eca-c700176a9322)

## Architecture Comparison
Traditional WPF Architecture:

![deepseek_mermaid_20250502_79e73e](https://github.com/user-attachments/assets/a44476e6-fa43-4243-902d-7725479521c3)

Caliburn.Micro Architecture:

![deepseek_mermaid_20250502_6255dc](https://github.com/user-attachments/assets/7ffb016d-7aba-4490-a103-eb9dc4bcbc5b)

Performance Considerations

![Screenshot 2025-05-02 at 11 49 40 AM](https://github.com/user-attachments/assets/70b461c5-7772-48d4-ad4f-1a9239f00a74)

Recommendation: Use Caliburn for apps with >5 screens or complex navigation requirements.

This comparison shows how Caliburn.Micro solves common WPF pain points through:

1. Convention over configuration
2. Strong DI integration
3. Managed lifecycles
4. Decoupled messaging
5. Async-first approach


