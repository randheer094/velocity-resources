# Architecture

The "shape of every code change" rules: MVI, Compose, Hilt, and
the coroutines / Flow patterns those three depend on.

## MVI (Model–View–Intent)

Every screen / feature uses MVI.

- Each feature exposes a single immutable `State` (data class), a
  sealed `Intent` (user actions + lifecycle events), and a sealed
  `Effect` (one-shot side effects — navigation, snackbars, toasts).
- `ViewModel` owns a `MutableStateFlow<State>` and a
  `Channel<Effect>(Channel.BUFFERED)`, and reduces `Intent` →
  `State` via a pure `reduce(state, intent): State`.
- State updates go through `state.update { it.copy(...) }` —
  never read-then-set, never plain assignment.
- I/O (network, DB, platform) lives in a use-case / repository
  layer invoked from the ViewModel.
- Views render `State` and forward user input as `Intent`.
- `SavedStateHandle` carries only the fields needed to reload the
  screen after process death (e.g. the `id` on a details page);
  derived or fetched data is re-loaded on restart.

## UI: Jetpack Compose

All UI is Jetpack Compose.

- A single host `Activity` per app (`MainActivity`) annotated
  `@AndroidEntryPoint`.
- Navigation uses Navigation 3 (`androidx.navigation3`).
- Composables consume state via
  `viewModel.state.collectAsStateWithLifecycle()` — never plain
  `collectAsState`.
- Composables are stateless: state is hoisted, callbacks flow
  down as `(Intent) -> Unit`.
- Side effects run inside `LaunchedEffect` / `DisposableEffect` /
  `rememberCoroutineScope`; never in the composition body.
- `LazyColumn` / `LazyRow` always pass `key = { it.id }`.
- State types are stable: data classes with immutable fields and
  `kotlinx.collections.immutable` collections (`ImmutableList`,
  `PersistentList`) — never mutable `List<T>` in `State`.
- Non-trivial Composables ship `@PreviewLightDark`, plus
  `@PreviewScreenSizes` where layout matters.
- Material 3 (`androidx.compose.material3`) is the design system;
  dynamic color on Android 12+, dark theme always supported.

## DI: Hilt

Hilt is the DI framework.

- `Application` is `@HiltAndroidApp`. The host `Activity` is
  `@AndroidEntryPoint`. ViewModels are `@HiltViewModel`.
- Hilt's annotation processor runs through KSP, not kapt.
- Collaborators reach the class via `@Inject constructor(...)`.
- Bindings live in `@Module @InstallIn(<Component>::class)`; use
  `@Binds` for interface→impl wiring.
- Scope with intent: `@Singleton` for app-lifetime,
  `@ActivityRetainedScoped` for ViewModel-shared state,
  `@ViewModelScoped` for per-screen collaborators. Default to the
  narrowest scope that works.
- Tests use `@HiltAndroidTest` + `@UninstallModules` to swap fakes.

## Coroutines & Flow

- `viewModelScope` for ViewModel work; `lifecycleScope` only when
  bound to UI lifecycle. Never `GlobalScope`.
- Cold flows surface to the UI through
  `stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), initial)`
  — survives config change, releases upstream after backgrounding.
- UI collection uses `collectAsStateWithLifecycle()` or
  `lifecycle.repeatOnLifecycle(STARTED) { flow.collect { ... } }`
  — never `LaunchedEffect { flow.collect { ... } }` for long-lived
  flows.
- Dispatchers are injected via Hilt qualifiers (`@IoDispatcher`,
  `@DefaultDispatcher`); tests substitute `StandardTestDispatcher`.
- `Dispatchers.Main.immediate` for UI-thread hops, `IO` for
  blocking I/O, `Default` for CPU-bound work.
