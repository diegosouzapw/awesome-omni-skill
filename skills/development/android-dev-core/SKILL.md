---
name: android-dev-core
description: Android 功能开发核心规则。包含项目架构、MVI模式、Base类、Adapter、网络请求等基础规范。开发任何 Android 功能时都应加载此 skill。
metadata:
  category: android
  platform: android
---

# Android 功能开发核心规则

本 Skill 包含 Android 功能开发的核心规范，适用于所有功能开发场景。

---

## 1. 开发原则

You are an Android Developer that provides expert-level insights and solutions. Your responses
should include examples of code snippets (where applicable), best practices, and explanations of
underlying concepts.

Here are some rules:

- Adapt to existing project architecture while maintaining clean code principles; implement clean
  architecture with domain, data, and presentation layers.
- Follow unidirectional data flow with ViewModel and UI State.
- Use the latest stable Android SDKs and Android Jetpack libraries.
- Prioritize the use of `Coroutine` and `suspend` functions for asynchronous operations, and prioritize the use of `suspendCancellableCoroutine` or `callbackFlow` instead of the `callback` parameter. Improve code readability and maintainability.
- Prefer using Android's native tools and libraries over third-party dependencies.
- Follow the Android Kotlin Style Guide and official Android API Design Guidelines for style.
- Add code comments when logic is complex to aid understanding.
- Use Kotlin idioms and features such as coroutines and extension functions.
- Highlight any considerations, such as potential performance impacts, with advised solutions.
- Include links to reputable sources for further reading (when beneficial).
- Keep it simple, professional, and direct. Avoid unnecessary pleasantries (such as "okay" or "of
  course").
- Use Markdown to format responses, with code blocks enclosed in backticks.
- Never fabricate information or guess the results of tool execution.
- When encountering issues, explain the situation and attempt to resolve it rather than just
  apologizing.
- Provide necessary Chinese annotation explanations on complex logical functions and classes that require explanation of their functions.
- Please search and think about information in English, and finally translate the results into
  Chinese.

---

## 2. Import 规范

NOTICE:
When using encoding, prioritize encoding and importing packages according to the following format,
never put a full package name in a code block(e.g.,com.package.VLog.d()), always put an import statement at the beginning of the file.
adhere to these project standards to maintain code consistency:

### 2.1 Click Handling

```kotlin
import com.androidtool.common.extension.onClick

// Standard usage
button.onClick { /* handle click */ }

// With custom debounce delay
button.onClick(delay = 600) { /* handle click */ }
```

### 2.2 Dimension Conversion

```kotlin
import com.androidtool.common.extension.dp

// Usage
val padding = 16.dp
```

### 2.3 Screen & Device Information

```kotlin
import com.androidtool.common.utils.ScreenUtil

// Available methods
// Get StatusBar height
ScreenUtil.getStatusBarHeight()
// Get screen width
ScreenUtil.getScreenWidth()
// Get screen height
ScreenUtil.getScreenHeight()

```

### 2.4 Logging

```kotlin
import com.androidtool.common.log.VLog

// Always use "lilili" as primary tag with descriptive messages
VLog.d("lilili", "Retrieved user ID: $userId")
```


### 2.5 String Resource
```kotlin
import com.androidtool.common.utils.TranslateResource

binding.title =  TranslateResource.getStringResources("title")
```

### 2.6 collectWhenStarted
```kotlin
import com.androidtool.common.utils.collectWhenStarted

viewModel.sampleLoadMoreUiState.collectWhenStarted { state ->
    handleUiState(state)
}
```

### 2.7 Common LoadError Page
```kotlin
import com.androidtool.common.base.page.EmptyPage
import com.androidtool.common.base.page.ErrorPage
import com.androidtool.common.base.page.LoadingPage

class GiftRankListFragment : BaseBindingFragment<FragmentGiftRankListBinding>() {
    private val viewModel: GiftListViewModel by viewModels()

    private lateinit var id: String

    // Integration of LoadSir page placeholder
    override fun registerState() = binding.recyclerView

    private fun handleUiState(state: GiftListUiState<GiftRankListBean.GiftRankItem>) {
        when (state) {
            is GiftListUiState.Loading -> {
                showState<LoadingPage>()
            }

            is GiftListUiState.Empty -> {
                binding.refreshLayout.finishRefresh()
                showState<EmptyPage> ()
            }

            is GiftListUiState.Success -> {
                showSuccess() // LoadSir integration
                handleSuccessState(state)
            }

            is GiftListUiState.Error -> {
                binding.refreshLayout.finishRefresh()
                showState<ErrorPage> {
                    val hint = view.findViewById<TextView>(R.id.hint)
                    hint?.onClick {
                        viewModel.refresh()
                    }
                }
            }
        }
    }
}
```


### 2.8 RecycleView List Adapter
```kotlin
import com.model.UserBean
import com.chad.library.adapter.base.BaseQuickAdapter
import com.chad.library.adapter.base.BaseMultiItemQuickAdapter
import com.chad.library.adapter.base.module.LoadMoreModule
import com.chad.library.adapter.base.viewholder.BaseViewHolder

// Using https://github.com/CymChad/BaseRecyclerViewAdapterHelper v3.x

// Simple list adapter
class SampleLoadMoreAdapter : BaseQuickAdapter<SampleUserInfo, BaseViewHolder>(R.layout.item_sample_user),
    LoadMoreModule {}


// Complex multi-type list adapter
class InterestTagAdapter(private val chosenListListener: (List<UserBean>) -> Unit) :
    BaseMultiItemQuickAdapter<InterestTagShowBean, BaseViewHolder>() {}

```

### 2.9 All model classes are in the `com.model` package, import using
```kotlin
import com.model.SomeBean
```


Always use project-provided utilities and extensions instead of creating custom implementations or
introducing unnecessary third-party dependencies.

---

## 3. 项目结构

Note: This is a recommended project structure, but be flexible and adapt to existing project structures.
Do not enforce these structural patterns if the project follows a different organization.

Assume I want to implement the feature:"one_feature":

```filetree

app/src/main/java/com/package/one_feature
├── data
│   ├── repository
│   │   └── OneFeatureRepositoryImpl.kt
│   ├── datasource (optional)
│   │   ├── local
│   │   │   ├── OneFeatureDao.kt
│   │   │   └── OneFeatureDatabase.kt
│   │   └── remote
│   │       └── OneFeatureApiService.kt
│   ├── model
│   │   ├── OneFeatureEntity.kt
│   │   └── OneFeatureResponse.kt
│   └── mapper (optional)
│       └── OneFeatureMapper.kt
├── di (optional)
│   └── OneFeatureModule.kt
├── ui
│   ├── fragment
│   │   ├── OneFeatureFragment.kt
│   │   ├── TwoFeatureFragment.kt(optional)
│   │   ├── OneFeatureListAdapter.kt
│   │   ├── OneFeatureSubListViewModel.kt
│   │   └── OneFeatureListContract.kt(contain state,event,effect)
│   ├── view(optional)
│   │   ├── CustomViews.kt
│   │   └── OneFeatureItemView.kt
│   │── OneFeatureListContract.kt(contain state,event,effect)
│   ├── OneFeatureActivity.kt
│   └── OneFeatureViewModel.kt
└── domain(optional)
│     ├── usecase
│     │   ├── GetOneFeatureUseCase.kt
│     │   └── UpdateOneFeatureUseCase.kt
│     ├── repository
│     │   └── IOneFeatureRepository.kt
│     └── model
│           └── OneFeatureModel.kt

```

Note:
1. If not specifically mentioned, we need to implement the domain layer, but the corresponding OneFeatureRepository doesn't need to generate an interface and implement it in the data layer; we don't need to use usecases.
2. If I provide backend response types, please use the context-provided backend response data type to generate the ApiService. If I don't provide backend response types, please define a SampleBean to represent the return data type.
3. If I provide a specific package file path, please replace `com/package/one_feature` when importing.
4. Due to project constraints, we currently don't use any DI tools for the di layer.
5. Unless specifically mentioned, we don't need to distinguish between local and remote datasources in the data layer. Directly use OneFeatureApiService with retrofit suspend for remote fetching.
6. Unless specifically mentioned, we don't need to implement mappers in the data layer.

---

## 4. 功能通用规范

### 4.1 布局与命名

1. Prioritize the use of XML for layout, prefer ConstraintLayout, unless a simple layout is used
   with FrameLayout/LinearLayout and RelativeLayout.
2. Naming conventions:
    - Class names: PascalCase (UserRepository)
    - Variables/functions: camelCase (getUserData)
    - Constants: UPPER_SNAKE_CASE (MAX_RETRY_COUNT)
    - Use complete descriptive names
    - Interface: Usually not prefixed with 'I', unless to eliminate ambiguity.
    - Use complete and meaningful descriptive names (e.g., userProfileImageView instead of imgV).

### 4.2 Property Delegation with Lifecycle-Aware Components

When implementing complex logic in Activities and Fragments, use property delegation pattern to organize related functionality into specialized Impl classes. The delegate classes should implement DefaultLifecycleObserver and provide a bindLifecycle() method for automatic lifecycle management.

```kotlin
class FloatingVideoFragment : BaseBindingFragment<FragmentFloatingVideoBinding>(),
    DraggableBehavior by DraggableBehaviorImpl(),
    LoopVideoPlayerBehavior by LoopVideoPlayerBehaviorImpl() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        bindLifecycle(lifecycle)
    }
}

interface LoopVideoPlayerBehavior {
    fun bindLifecycle(lifecycle: Lifecycle)
    fun setEventVideo(urls: List<DemoEventVideo>)
    fun setupVideoPlayer(container: ViewPager2)
    fun isLooping(): Boolean
    fun startLoop()
    fun pauseLoop()
    fun releasePlayerResource()
}

class LoopVideoPlayerBehaviorImpl : LoopVideoPlayerBehavior, DefaultLifecycleObserver {

    override fun bindLifecycle(lifecycle: Lifecycle) {
        lifecycle.addObserver(this)
    }

    override fun onResume(owner: LifecycleOwner) { /* auto resume */ }
    override fun onPause(owner: LifecycleOwner) { /* auto pause */ }
    override fun onDestroy(owner: LifecycleOwner) { releasePlayerResource() }

    // ... other implementations
}

```

### 4.3 MVI Contract

State, Event, Effect should be integrated into a newly created `OneFeatureContract.kt` file

```kotlin
/**
 * Contract file for a single feature
 * Contains all state, event, and effect definitions for this feature
 */

/**
 * Data class representing UI state
 * Contains all data needed for UI display
 */
data class OneFeatureState(
    val isLoading: Boolean = false,
    val data: List<String> = emptyList(),
    val error: String? = null,
    val selectedItemId: String? = null
)

/**
 * Sealed class representing user intentions or system events
 * These events trigger state changes
 */
sealed interface OneFeatureEvent {
    data object LoadData : OneFeatureEvent()
    data object RefreshData : OneFeatureEvent()
    data class DeleteItem(val itemId: String) : OneFeatureEvent()
}

/**
 * Sealed class representing one-time side effects
 * These effects are typically one-time operations like navigation, Toast or Snackbar displays
 */
sealed interface OneFeatureEffect {
    data class ShowToast(val message: String) : OneFeatureEffect()
    data class ShareContent(val content: String) : OneFeatureEffect()
}
```

### 4.4 MVI Data Flow Patterns

When implementing MVI architecture, follow these data flow patterns:

- **Effect**: Use Channel to send one-time events (like navigation, Toast notifications) from
  ViewModel to View layer

  ```kotlin
  // In ViewModel
  private val _effect = Channel<MyEffect>(Channel.BUFFERED)
  val effect = _effect.receiveAsFlow()

  fun showToast(message: String) {
     viewModelScope.launch {
        _effect.send(MyEffect.ShowToast(message))
        }
  }

  // In View layer
  import com.androidtool.common.utils.collectWhenStarted

  viewModel.effect.collectWhenStarted { effect ->
     when (effect) {
        is MyEffect.ShowToast -> Toast.makeText(context, effect.message, Toast.LENGTH_SHORT).show()
        is MyEffect.Navigate -> findNavController().navigate(effect.destination)
     }
  }

  ```

- **State**: Use StateFlow to manage and expose UI state, ensuring consistency and observability

    ```kotlin
      // In ViewModel
      private val _state = MutableStateFlow(MyState())
      val state = _state.asStateFlow()
    ```

- **Event**: Trigger state updates by simply calling action methods on the ViewModel

    ```kotlin
      // In ViewModel
      fun onAction(event: MyEvent) {
        when (event) {
            is MyEvent.ItemClicked -> handleItemClick(event.item)
            is MyEvent.RefreshRequested -> loadData()
        }
      }

      // In View layer
      binding.refreshButton.onClick {
        viewModel.onAction(MyEvent.RefreshRequested)
      }
      recyclerView.adapter = adapter.apply {
      setOnItemClickListener {  adapter, view, position ->
          val item = adapter.getItem(position)
          viewModel.onAction(MyEvent.ItemClicked(item))
        }
      }

   ```

### 4.5 LoadSir 状态页框架

The project encapsulates LoadSir as a loading and error display framework, used as follows:
1. First, extend BaseBindingFragment or BaseBindingActivity
2. Second, override `open fun registerState(): View? = binding.recycleview` or
   `open fun registerState(): View? = binding.root`
3. Third, use `showState<LoadingPage>`, `showState<ErrorPage>`, `showState<EmptyPage>` to display state pages, and use
   `showSuccess()` to display the page used in the XML

### 4.6 BaseBindingFragment

The project encapsulates BaseBindingFragment, package in `package com.androidtool.common.base`:

```kotlin
package com.androidtool.common.base

import android.app.Activity
import android.content.Intent
import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.fragment.app.Fragment
import androidx.viewbinding.ViewBinding
import com.androidtool.common.loadsir.LoadLayout
import com.androidtool.common.loadsir.LoadSir
import com.scwang.smart.refresh.layout.SmartRefreshLayout

abstract class BaseBindingFragment<VB : ViewBinding> : Fragment(),
    IViewBinding<VB> by ViewBindingDelegate() {
    protected var lazyLoaded = false
    protected var initView = false

    // State page management
    lateinit var loadSir: LoadLayout
        private set

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?
    ): View? {
        lazyLoaded = false
        initView = false
        return createViewBinding(inflater, container, true)?.let { rootView ->
            // Register state page management
            registerState()?.let { compatibleSmartRefresh(it) } ?: run { rootView }
        }
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        initView()
        // observeData()
        // loadData()
    }

    override fun onDestroyView() {
        super.onDestroyView()
        lazyLoaded = false
    }

    override fun onResume() {
        super.onResume()
        if (!lazyLoaded) {
            lazyLoaded = true
            onLazyLoad()
        }
    }


    protected open fun initView() {}


    /**
     * For lazy loading of fragments in viewpager
     */
    open fun onLazyLoad() = Unit

    fun isLazyLoaded() = lazyLoaded

    /**
     * Compatible with SmartRefreshLayout
     */
    private fun compatibleSmartRefresh(target: View): View {
        loadSir = LoadSir.register(target, listener = { onStateClick() })
        (loadSir.parent as? SmartRefreshLayout)?.let {
            it.removeView(loadSir)
            it.setRefreshContent(loadSir)
        }
        return loadSir.rootView
    }

    // region------- State page management --------
    open fun registerState(): View? = null
    open fun onStateClick() = Unit

    fun showLoading() = loadSir.show<LoadingPage>()
    fun showSuccess(useAnim: Boolean = true) = loadSir.showSuccess(useAnim)

    /**
     * @param ifShow Condition to determine whether to show the state page: collection, Bool (default null, shows current state page)
     * @param useAnim Whether to use animation for state page transitions
     * @param block Can monitor current displayed state page: can get current displayed state page (for customizations)
     */
    inline fun <reified T : PageState> showState(
        ifShow: Any? = null,
        useAnim: Boolean = true,
        noinline block: (T.() -> Unit)? = null
    ) {
        // Whether to show current specified state page: based on condition
        val isShowState = when (ifShow) {
            is Collection<*> -> ifShow.isEmpty()
            is Array<*> -> ifShow.isEmpty()
            is Boolean -> ifShow
            else -> true
        }
        if (isShowState) loadSir.show<T>(
            useAnim = useAnim, block = block
        ) else loadSir.showSuccess()
    }
    // endregion
}
```

Usage example:

```kotlin
import com.androidtool.common.base.BaseBindingFragment
import com.androidtool.common.utils.collectWhenStarted

class SampleListFragment : BaseBindingFragment<FragmentGiftRankListBinding>() {
    private val viewModel: SampleListViewModel by viewModels()

    private lateinit var id: String

    // refresh placeholder usage
    override fun registerState() = binding.recyclerView

    private val adapter by lazy {
        GiftRankListAdapter()
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        id = arguments?.getString(DYNAMIC_ID) ?: "0"
        viewModel.initDataById(id = id)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        setupRefreshLayout()
        setupRecyclerView()
        observeUiState()
    }

    /**
     * Setup pull-to-refresh
     */
    private fun setupRefreshLayout() {
        binding.refreshLayout.setOnRefreshListener {
            viewModel.refresh()
        }
    }

    /**
     * Setup RecyclerView and load more
     */
    private fun setupRecyclerView() {
        binding.recyclerView.layoutManager = LinearLayoutManager(context)
        binding.recyclerView.adapter = adapter
        adapter.setOnItemClickListener { _, _, position ->
            val uId = adapter.data[position].uid
        }

        adapter.loadMoreModule.apply {
            isAutoLoadMore = true
            isEnableLoadMoreIfNotFullPage = false
            setOnLoadMoreListener {
                viewModel.loadMore()
            }
        }
    }

    /**
     * Observe UI state changes
     */
    private fun observeUiState() {
        viewModel.listUiState.collectWhenStarted { state ->
            handleUiState(state)
        }
        viewModel.listEffect.collectWhenStarted { event ->
            when (event) {
                is GiftListUiEvent.UpdatePeopleCount -> {
                    handleUpdatePeopleCount(event.count)
                }
            }

        }
    }

    companion object {
        const val DYNAMIC_ID = "dynamicID"
        fun newInstance(dynamicId: String): Fragment {
            val fragment = SampleListFragment()
            fragment.arguments = Bundle().apply {
                putString(DYNAMIC_ID, dynamicId)
            }
            return fragment

    }
}
```

### 4.7 BaseBindingActivity

The project encapsulates BaseBindingActivity, integrating LoadSir and ViewBinding. Note that this BaseBindingActivity does not implement TitleBar, you need to implement it yourself:
```kotlin
abstract class BaseBindingActivity<V : ViewBinding> : RootActivity(),
   IViewBinding<V> by ViewBindingDelegate() {
   // State page management
   lateinit var loadSir: LoadLayout
      private set

   override fun onCreate(savedInstanceState: Bundle?) {
      super.onCreate(savedInstanceState)
      val rootView = createViewBinding()
      // Register state page management
      registerState()?.let { target ->
         loadSir = LoadSir.register(target, listener = { onStateClick() })
         // Compatible with SmartRefreshLayout
         (loadSir.parent as? SmartRefreshLayout)?.let {
            it.removeView(loadSir)
            it.setRefreshContent(loadSir)
         }
      }
      setContentView((rootView.parent as? View) ?: rootView)
      initView()
      initData()
   }

   abstract fun initView()
   open fun initData() = Unit

   open fun registerState(): View? = null
   open fun onStateClick() = Unit
   fun showLoading() = loadSir.show<LoadingPage>()
   fun showSuccess(useAnim: Boolean = true) = loadSir.showSuccess(useAnim)

   /**
    * @param ifShow Condition to determine whether to show the state page: collection, Bool (default null, shows current state page)
    * @param useAnim Whether to use animation for state page transitions
    * @param block Can monitor current displayed state page: can get current displayed state page (for customizations)
    */
   inline fun <reified T : PageState> showState(
      ifShow: Any? = null,
      useAnim: Boolean = true,
      noinline block: (T.() -> Unit)? = null
   ) {
      // Whether to show current specified state page: based on condition
      val isShowState = when (ifShow) {
         is Collection<*> -> ifShow.isEmpty()
         is Array<*> -> ifShow.isEmpty()
         is Boolean -> ifShow
         else -> true
      }
      if (isShowState) loadSir.show<T>(
         useAnim = useAnim, block = block
      ) else loadSir.showSuccess()
   }
}
```

Usage example using `com.androidtool.common.widget.TitleBarView` to integrate title bar:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".ui.topic.TopicListActivity">

    <com.androidtool.common.widget.TitleBarView
        android:id="@+id/titleBar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/contentView"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <com.google.android.material.tabs.TabLayout
            android:id="@+id/tabLayout"
            style="@style/TopicTabLayout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginHorizontal="16dp"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            android:background="@android:color/transparent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:tabGravity="start"
            app:tabRippleColor="@null" />


        <androidx.viewpager2.widget.ViewPager2
            android:id="@+id/viewPager"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/tabLayout" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</LinearLayout>
```

SampleBindingActivity:

```kotlin
class TopicListActivity : BaseBindingActivity<ActivityTopicListBinding>() {
    private val viewModel: TopicListViewModel by viewModels()
    private var tabLayoutMediator: TabLayoutMediator? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setStatusBarTextColor(isLight = true)
        initView()
        observeData()
        viewModel.loadData()
    }

    override fun initView() {
        // Set title bar
        binding.titleBar.setTitle(TranslateResource.getStringResources("topic"))
        binding.titleBar.setOnBackClickListener {
            finish()
        }
    }

    override fun registerState(): View {
        return binding.contentView
    }

    private fun observeData() {
        viewModel.topicCategoryListUiState.collectWhenStarted {
            when (it) {
                TopicCategoryListUiState.Empty -> {
                    showState<EmptyPage>()
                }

                is TopicCategoryListUiState.Error -> {
                    showState<ErrorPage>()
                }

                TopicCategoryListUiState.Loading -> {
                    showState<LoadingPage>()
                }

                is TopicCategoryListUiState.Success -> {
                    showSuccess()
                    initTabLayoutAndViewPager(it.items)
                }
            }
        }
    }

    private fun initTabLayoutAndViewPager(categories: List<TopicCategoriesBean.TopicCategory>) {
        val pagerAdapter = TopicPagerAdapter(this, categories)

        binding.viewPager.adapter = pagerAdapter
        tabLayoutMediator?.detach()
        tabLayoutMediator =
            TabLayoutMediator(binding.tabLayout, binding.viewPager) { tab, position ->
                // Use custom layout
                val tabView = layoutInflater.inflate(R.layout.tab_item_topic, null)
                val tabTitle = tabView.findViewById<TextView>(R.id.tvTabTitle)
                tabTitle.text = categories[position].name
                tab.customView = tabView
            }.apply {
                attach()
            }

        if (categories.isNotEmpty()) {
            binding.viewPager.setCurrentItem(0, false)
        }
    }

    override fun onDestroy() {
        tabLayoutMediator?.detach()
        tabLayoutMediator = null
        super.onDestroy()
    }

    companion object {
        fun start(context: Context) {
            val intent = Intent(context, TopicListActivity::class.java)
            context.startActivity(intent)
        }
    }
}
```

### 4.8 TitleBarView API

```kotlin
binding.titleBar.setTitle(TranslateResource.getStringResources("topic"))
binding.titleBar.setOnBackClickListener {
    finish()
}
```

TitleBarView API:

```kotlin
/**
 * Set title text
 */
fun setTitle(title: CharSequence) {}
/**
 * Set back button click listener
 */
fun setOnBackClickListener(listener: () -> Unit) {}
/**
 * Set extra button (icon) on the right
 */
fun setExtraButton(@DrawableRes resId: Int, clickListener: (v: View) -> Unit) {}
/**
 * Set custom layout for the right side
 */
fun setExtraButtonLayout(view: View) {}
```

### 4.9 Adapter

Adapters should use `BaseRecyclerViewAdapterHelper V3.x`, as follows:

```kotlin
import com.model.UserBean
import com.chad.library.adapter.base.BaseQuickAdapter
import com.chad.library.adapter.base.BaseMultiItemQuickAdapter
import com.chad.library.adapter.base.module.LoadMoreModule
import com.chad.library.adapter.base.viewholder.BaseViewHolder

// Using https://github.com/CymChad/BaseRecyclerViewAdapterHelper v3.x

// Simple list adapter
class SampleLoadMoreAdapter : BaseQuickAdapter<SampleUserInfo, BaseViewHolder>(R.layout.item_sample_user),
   LoadMoreModule {}


// Complex multi-type list adapter
class InterestTagAdapter(private val chosenListListener: (List<UserBean>) -> Unit) :
   BaseMultiItemQuickAdapter<InterestTagShowBean, BaseViewHolder>() {}

```

### 4.10 Network Request

NOTICE: adhere to these project package import standards to maintain code consistency

- Repository Layer

```kotlin
import com.androidtool.common.extension.asResult
import com.androidtool.common.extension.requestToFlow

// Standard pattern
fun loadData(): Flow<Result<DataModel>> {
    return requestToFlow { apiService.fetchData() }.asResult()
}
```

- ViewModel Layer

```kotlin
// Follow this pattern for data loading
fun loadData() {
    viewModelScope.launch {
        updateLoadingState()
        repository.loadData()
            .catch { exception -> handleError(exception) }
            .collect { result ->
                result.fold(
                    onSuccess = { data -> handleSuccess(data) },
                    onFailure = { exception -> handleError(exception) }
                )
            }
    }
}
```

- API Service Definition

```kotlin
import com.androidtool.common.troll.BaseResponse
import com.androidtool.common.net.Apis


// 1. 项目中封装了 BaseResponse,所有的接口返回类型都为 BaseResponse
// 2. 项目中所有的参数都是通过Map<String, String>)进行请求的
interface ApiService {
    @GET(Apis.ENDPOINT_DATA)
    suspend fun getData(): BaseResponse<DataModel>

    @GET(Apis.ENDPOINT_PARAMS_DATA)
    suspend fun getParamsData(@QueryMap map: Map<String, String>): BaseResponse<List<DataModel>>
}
```
