# Основные принципы чистого кода

## Введение

Цель - облегчить чтение, тестирование и поддержку исходного кода командой разработчиков.

При составлении документа использовались следующие источники:
1. Книга "Чистый код. Создание анализ и рефакторинг | Мартин Роберт К."
2. Сайт [Refactoring Guru](https://refactoring.guru/ru)

## Именования

### Используйте осмысленные и легко произносимые названия переменных/функций/классов

#### Плохо

```kotlin
class MyParser {

    fun parseA(yyyymmddhhmmss: String): Date {
        return parse(yyyymmddhhmmss, "yyyy-MM-dd'T'HH:mm:ssZ")
    }

    private fun parse(ds: String, f: String): Date {
        val ff = getF(f)
        return LocalDate.parse(ds, ff)
    }

    private fun getF(f: String): DateTimeFormatter {
        return DateTimeFormatter.ofPattern(f)
    }
}
```
- Суффиксы и префиксы "My", "Custom", "A", "B" и подобные им не несут смысловой нагрузки, как и перечисления через "1", "2", "one", "two".
- Трудночитаемые и труднопроизносимые названия, такие как "yyyymmddhhmmss", усложняют чтение и написание кода.
- Если название функции не отражает ее назначение, то без просмотра исходника невозможно понять, что она делает. То же самое относится к классам и переменным.

#### Хорошо

```kotlin
class DateTimeParser {

    fun parseIsoDateTime(isoDateTime: String): Date {
        return parseDateTime(isoDateTime, "yyyy-MM-dd'T'HH:mm:ssZ")
    }

    private fun parseDateTime(dateTime: String, format: String): Date {
        val formatter = getDateTimeFormatter(format)
        return LocalDate.parse(dateTime, formatter)
    }

    private fun getDateTimeFormatter(format: String): DateTimeFormatter {
        return DateTimeFormatter.ofPattern(format)
    }
}
```

### Не используйте сокращения

#### Плохо

```kotlin
fun showTxt(txt: String) {
    this.vb.tv.text = txt

    // Стоп... что значит "vb"? Visual Basic? Volleyball? Или View Binding?
    // И что такое "tv"? Television? Tommy Vercetti? Или Text View?
    // И почему свойство "text" и похожая переменная "txt" отличаются по именам?
}
```

```kotlin
val c: List<Car> = findCars()

c.forEach {
    val o: List<Order> = findOrders(it)

    o.forEach {
        // Стоп... А как мне обратиться к автомобилю?
        // И что вообще сейчас означает "it"?
        // А что такое "c", я уже забыл...
    }
}
```

- Чтобы прочитать код с сокращениями, нужно их изучить и запомнить. Это повышает нагрузку на программиста.
- Сокращенные названия индексов и других переменных легко перепутать, что будет трудно отладить.
- Исключение - известные аббревиатуры.


#### Хорошо

```kotlin
fun showText(text: String) {
    this.viewBinding.textView.text = text
}
```

```kotlin
val cars: List<Car> = findCars()

cars.forEach { car ->
    val orders: List<Order> = findOrders(car)

    orders.forEach { order ->
        // Можно взаимодействовать с "car" и "order"
    }
}
```

### Используйте одинаковые слова для именования переменных/функций одного типа

#### Плохо

```kotlin
fun showContract(trade: Contract) {
    showDealName(trade)
    showDealDate(trade)
}

fun showDealName(cotract: Contract) {
    this.dealName.text = contract.name
}
```

- Визуально кажется, что производится работа с разными типами данных. Особенно в языках с динамической типизацией.
- Затруднен поиск по исходному коду. Нужно искать несколько раз с разными названиями.

#### Хорошо

```kotlin
fun showContract(contract: Contract) {
    showContractName(contract)
    showContractDate(contract)
}

fun showContractName(contract: Contract) {
    this.contractName.text = contract.name
}
```

## Переменные

### Не используйте магические значения

#### Плохо

```kotlin
object Calculator {

    fun calculatePotentialEnergy(mass: Float, height: Float): Float {
        return mass * height * 9.81f;
    }
}
```

```kotlin
class Storage {

    fun getUser(): User? {
        val userJSON = this.sharedPreferences.getString("USER", null) ?: return null
        return parseUserFromJSON(userJSON)
    }

    fun setUser(user: User?) {
        val userJSON = if (user == null) null else convertUserToJSON(user)
        this.sharedPreferences
            .edit()
            .putString("USER", userJSON)
            .apply()
    }
}
```

- По не именованному значению в коде трудно понять его смысл и назначение.
- Если значение встречается несколько раз, то поменять его придется во всех местах.

#### Хорошо

```kotlin
const val GRAVITATION_CONSTANT: Float = 9.81f

object Calculator {

    fun calculatePotentialEnergy(mass: Float, height: Float): Float {
        return mass * height * GRAVITATION_CONSTANT;
    }
}
```

```kotlin
const val USER_KEY: String = "USER"

class Storage {

    fun getUser(): User? {
        val userJSON = this.sharedPreferences.getString(USER_KEY, null) ?: return null
        return parseUserFromJSON(userJSON)
    }

    fun setUser(user: User?) {
        val userJSON = if (user == null) null else convertUserToJSON(user)
        this.sharedPreferences
            .edit()
            .putString(USER_KEY, userJSON)
            .apply()
    }
}
```

### Заводите переменные или функции для промежуточных вычислений

#### Плохо

```kotlin
fun parseProductFromPage(pageTitle: String): Product {
    return buildProduct(
        pageTitle.split("@")[0].split("-")[0].trim(),
        pageTitle.split("@")[1]
    )
}
```

- Трудно понять смысл вычислений без просмотра аргументов функции.
- Часть одинакового исходного кода выполняется несколько раз.

#### Хорошо

```kotlin
fun parseProductFromPage(pageTitle: String): Product {
    val titleParts = pageTitle.split("@")

    val productTitle = titleParts[0].split("-")[0].trim()
    val shopName = titleParts[1]

    return buildProduct(productTitle, shopName)
}

// Или

fun parseProductFromPage(pageTitle: String): Product {
    val titleParts = pageTitle.split("@")

    return buildProduct(
        productTitle = titleParts[0].split("-")[0].trim(),
        shopName = titleParts[1]
    )
}
```

#### Плохо

```kotlin
fun saveFilm(film: Film)
    if (this.user.roles.any { role -> role.isAdmin() } || film.creatorId == this.user.id) {
        this.repository.save(film)
    }
}
```

- Трудно понять смысл проверки без детального изучения каждого условия.
- Код разросся по горизонтали.

#### Хорошо

```kotlin
fun saveFilm(film: Film)
    val isAdmin = this.user.roles.any { role -> role.isAdmin() }
    val isCreator = film.creatorId == this.user.id

    if (isAdmin || isCreator) {
        this.repository.save(film)
    }
}
```

```kotlin
fun saveFilm(film: Film)
    if (isAdmin() || isCreator(film)) {
        this.repository.save(film)
    }
}

fun isAdmin(): Boolean {
    return this.user.roles.any { role -> role.isAdmin() }
}

fun isCreator(film: Film): Boolean {
    return film.creatorId == this.user.id
}
```

## Функции

### Избегайте большого числа аргументов

#### Плохо

```kotlin
class HomeScreen {

    fun onProfileButtonClicked() {
        val firstName = this.firstNameView.text
        val lastName = this.lastNameView.text
        // Получение остальных значений с отображения

        ProfileScreen.open(
            firstName, lastName, gender,
            country, city, street, house
        )
    }
}

class ProfileScreen {

    companion object {

        fun open(
            firstName: String,
            lastName: String,
            gender: Gender,
            country: Country,
            city: String,
            street: String,
            house: String
        ) {
            // Открыть экран с помощью SDK
        }
    }
}
```

- Легко запутаться в порядке аргументов и передать их на неправильных местах.
- Трудно отформатировать функцию, приходится делать переносы в описании аргументов.
- При необходимости добавить новую информацию о сущности, нужно будет рефакторить цепочку вызова функций.

#### Хорошо

```kotlin
class HomeScreen {

    fun onScreenButtonClicked() {
        val firstName = this.firstNameView.text
        val lastName = this.lastNameView.text
        // Получение остальных значений с отображения

        val personalInfo = PersonalInfo(firstName, lastName, gender)
        val address = Address(country, city, street, house)

        ProfileScreen.open(personalInfo, address)
    }
}

class ProfileScreen {

    companion object {

        fun open(personalInfo: PersonalInfo, address: Address) {
            // Открыть экран с помощью SDK
        }
    }
}
```

#### Плохо

```kotlin
fun main() {
    ImageUtils.convertToSupportedFormat(
        File("../storage/image.webp"),
        false,
        true,
        true
    )
}

object ImageUtils {

    fun convertToSupportedFormat(
        imageFile: File,
        useCache: Boolean,
        debugOutput: Boolean,
        removeTempFiles: Boolean
    ): File {
        // Преобразование изображения
    }
}
```

- При вызове функции трудно определить, за что отвечают похожие по типу не именованные аргументы.

#### Хорошо

```kotlin
fun main() {
    ImageUtils.convertToSupportedFormat(
        imageFile = File("../storage/image.webp"),
        useCache = false,
        debugOutput = true,
        removeTempFiles = true
    )
}

object ImageUtils {

    fun convertToSupportedFormat(
        imageFile: File,
        useCache: Boolean,
        debugOutput: Boolean,
        removeTempFiles: Boolean
    ): File {
        // Преобразование изображения
    }
}
```

### Разбивайте комплексный код на простые функции

#### Плохо

```kotlin
fun coloriseOrderStatus(order: Order) {
    var colorHex: String? = null

    if (order.creationDate != null) {
        colorHex = "#ff0000"
    } else if (order.processingStartDate != null) {
        colorHex = "#00ff00"
    } else if (order.deliveryStartDate != null) {
        colorHex = "#0000ff"
    } else {
        colorHex = "#000000"
    }

    this.statusView.backgroundColor = getColorByHex(colorHex)
}

fun getColorByHex(hex: String): Int {
    // Создать цвет по hex-коду
}
```

- В коде одной функции смешано несколько задач, среди которых не видно алгоритма и главного действия.

#### Хорошо

```kotlin
fun coloriseOrderStatus(order: Order) {
    val status = getOrderStatus(order)
    val colorHex = getOrderColorHexByStatus(status)
    val color = getColorByHex(color)

    this.statusView.backgroudColor = color
}

fun getOrderStatus(order: Order): OrderStatus {
    return if (order.creationDate != null) {
        OrderStatus.CREATED
    } else if (order.processingStartDate != null) {
        OrderStatus.PROCESSING
    } else if (order.deliveryStartDate != null) {
        OrderStatus.DELIVERY
    } else {
        OrderStatus.UNKNOWN
    }
}

fun getOrderColorHexByStatus(status: OrderStatus): String {
    return when (status) {
        OrderStatus.CREATED -> "#ff0000"
        OrderStatus.PROCESSING -> "#00ff00"
        OrderStatus.DELIVERY -> "0000ff"
        OrderStatus.UNKNOWN -> "#000000"
    }
}

fun getColorByHex(hex: String): Int {
    // Создать цвет по hex-коду
}
```

### Код в функции должен быть на одном уровне абстракции

#### Плохо

```kotlin
class UserViewHolder {

    fun showUser(user: User) {
        val formattedUserRoles = user
            .roles
            .map { role -> role.name }
            .joinToString(", ")

        this.nameTextView.text = formatUserName(user)
        this.createdAtTextView.text = formatUserCreatedAt(user)
        this.rolesTextView = formattedUserRoles
    }

    private fun formatUserName(user: User): String {
        // Форматирование имени пользователя
    }

    private fun formatUserCreatedAt(user: User): String {
        // Форматирование даты создания пользователя
    }
}
```

- Чтение кода с разным уровнем абстракции заставляет мозг переключаться между уровнями, неявно выстраивая новый уровень абстракции самостоятельно.

#### Хорошо

```kotlin
class UserViewHolder {

    fun showUser(user: User) {
        this.nameTextView.text = formatUserName(user)
        this.createdAtTextView.text = formatUserCreatedAt(user)
        this.rolesTextView = formatUserRoles(user)
    }

    private fun formatUserName(user: User): String {
        // Форматирование имени пользователя
    }

    private fun formatUserCreatedAt(user: User): String {
        // Форматирование даты создания пользователя
    }

    private fun formatUserRoles(user: User): String {
        return user
            .roles
            .map { role -> role.name }
            .joinToString(", ")
    }
}
```

### Избегайте модификации объектов, пришедших в классы и функции извне

#### Плохо

```kotlin
fun main() {
    val items = mutableListOf(
        Item(id = 1),
        Item(id = 2),
        Item(id = 3)
    )

    val adapter = ItemsAdapter()
    adapter.setItems(items)

    // Упс... Item появился и в Adapter-е
    items.add(
        Item(id = 4)
    )
}

class Item(val id: Int)

class ItemsAdapter {

    private lateinit var items: MutableList<Item>

    fun setItems(items: MutableList<Item>) {
        this.items = items
    }
}
```

- Так как ссылка на один и тот же объект захвачена разными контекстами, то модификация объекта неявным образом влияет на работу каждого контекста.

#### Хорошо

```kotlin
fun main() {
    val items = mutableListOf(
        Item(id = 1),
        Item(id = 2),
        Item(id = 3)
    )

    val adapter = ItemsAdapter()
    adapter.setItems(items)

    // Всё хорошо, items в Adapter-е не изменились
    items.add(
        Item(id = 4)
    )
}

class Item(val id: Int)

class ItemsAdapter {

    private val items: MutableList<Item> = mutableListOf()

    fun setItems(items: List<Item>) {
        this.items.clear()
        this.items.addAll(items)
    }
}
```

### Указывайте на скрытые условия в названии функции

#### Плохо

```kotlin
fun main() {
    val task = Task()

    runTask(task) // Ожидаем, что задача запустилась
    runTask(task) // Ожидаем повторного запуска, но его не произошло
}

fun runTask(task: Task) {
    if (!task.isRunning()) {
        task.run()
    }
}
```

- Невозможно понять весь алгоритм, если не посмотреть реализацию всех функций.

#### Хорошо

```kotlin
fun main() {
    val task = Task()

    runTaskIfNotRunning(task) // Ожидаем, что задача запустилась
    runTaskIfNotRunning(task) // Не ожидаем повторного запуска, всё ок
}

fun runTaskIfNotRunning(task: Task) {
    if (!task.isRunning()) {
        task.run()
    }
}
```

### Заменяйте вложенные условные операторы граничным оператором

#### Плохо

```kotlin
fun getTotalAmount(): Int? {
    if (isArchived()) {
        return null
    } else {
        if (this.cachedTotalAmount != null) {
            return this.cachedTotalAmount
        } else {
            if (isEnoughData()) {
                return this.calculateTotalAmount()
            } else {
                return this.getDefaultTotalAmount()
            }
        }
    }
}
```

- Среди множества вложенных условий трудно выделить нормальный ход выполнения кода.
- Выделите все проверки специальных или граничных случаев в отдельные проверки и поместите их перед основным кодом, чтобы получить линейную структуру без вложенностей.

#### Хорошо

```kotlin
fun getTotalAmount(): Int? {
    if (isArchived()) {
        return null
    }

    if (this.cachedTotalAmount != null) {
        return this.cachedTotalAmount
    }

    if (!this.isEnoughData()) {
        return this.getDefaultTotalAmount()
    }

    return this.calculateTotalAmount()
}
```

### Не используйте флаги в аргументах функции

#### Плохо

```kotlin
class ProjectScreen {

    suspend fun loadProject() {
        try {
            val project = this.projectUseCase.getById(this.projectId)
            this.showProject(project)
            this.showMessage("Loaded", true)
        } catch (e: Exception) {
            this.showMessage("Loading error", false)
        }
    }

    fun showMessage(messageText: String, isError: Boolean) {
        if (isError) {
            // Показ соообщения об ошибке в специальном дизайне для ошибок
        } else {
            // Показ информационного сообщения
        }
    }
}
```

- Флаг в аргументах вносит избыточное ветвление и делает функцию ответственной за несколько вещей.

#### Хорошо

```kotlin
class ProjectScreen {

    suspend fun loadProject() {
        try {
            val project = this.projectUseCase.getById(this.projectId)
            this.showProject(project)
            this.showSuccessMessage("Loaded")
        } catch (e: Exception) {
            this.showErrorMessage("Loading error")
        }
    }

    fun showSuccessMessage(messageText: String) {
        // Показ сообщения об успехе в специальном дизайне для успеха
    }

    fun showErrorMessage(messageText: String) {
        // Показ соообщения об ошибке в специальном дизайне для ошибок
    }
}
```

## Классы

### Не используйте God Object для хранения информации

#### Плохо

```kotlin
fun main() {
    val product = Info(
        id = 1,
        name = "Spoon",
        description = "Material - Silver",
        price = 3
    )

    val cart = Info(
        id = 1,
        status = CartStatus.CHECKOUT.value,
        price = 10
    )
}

class Info(
    var id: Integer? = null,
    var name: String? = null,
    var status: String? = null,
    var description: String? = null,
    var price: Integer? = null
)
```

- С течением времени God Object бесконечно разрастается.
- Приходится работать с лишними полями.

#### Хорошо

```kotlin
fun main() {
    val product = Product(
        id = 1,
        name = "Spoon",
        description = "Material - Silver",
        price = 3
    )

    val cart = Cart(
        id = 1,
        status = CartStatus.CHECKOUT.value,
        price = 10
    )
}

class Product(
    var id: Integer? = null,
    var name: String? = null,
    var description: String? = null,
    val price: Integer? = null
)

class Cart(
    var id: Integer? = null,
    var status: String? = null,
    var price: String? = null
)
```

### Создавайте дополнительные классы для группировки полей по контексту

#### Плохо

```kotlin
fun main() {
    val city = City(
        cityId = 1,
        cityName = "Novosibirsk",
        regionId = 1,
        regionName = "Novosibirsk region"
    )
}

class City(
    val cityId: Int? = null,
    val cityName: String? = null,
    val regionId: Int? = null,
    val regionName: String? = null
)
```

- Невозможно независимо переиспользовать некоторые поля.
- Код разрастается из-за дублирования суффиксов.

#### Хорошо

```kotlin
fun main() {
    val city = City(
        id = 1,
        name = "Novosibirsk",
        region = Region(
            id = 1,
            name = "Novosibirsk region"
        )
    )
}

class City(
    val id: Int? = null,
    val name: String? = null,
    val region: Region? = null
)

class Region(
    val id: Int? = null,
    val name: String? = null
)
```

### Скрывайте методы, которые не будут использоваться снаружи

#### Плохо

```kotlin
fun main() {
    val car = Car()
    val carCreator = CarCreator()
    carCreator.generateCarJson(car) // Почему эта функция доступна?
}

class CarCreator {

    private val apiService = ApiService()

    fun create(car: Car): Car {
        val json = generateCarJson(car)
        val createdCar = this.apiService.createCar(car)
        return createdCar
    }

    fun generateCarJson(car: Car): String {
        // Логика по созданию JSON-а
    }
}
```

- Среда разработки при автодополнении показывает методы, которые не нужны программисту, который использует класс.

#### Хорошо

```kotlin
fun main() {
    val car = Car()
    val carCreator = CarCreator()
    carCreator.create(car) // Видны только необходимые функции
}

class CarCreator {

    private val apiService = ApiService()

    fun create(car: Car): Car {
        val json = generateCarJson(car)
        val createdCar = this.apiService.createCar(car)
        return createdCar
    }

    private fun generateCarJson(car: Car): String {
        // Логика по созданию JSON-а
    }
}
```

### Используйте рекурсию для описания глубокой вложенности

#### Плохо

```kotlin
class Category(
    val id: Int,
    val name: String,
    val subCategories: List<SubCategory>
)

class SubCategory(
    val id: Int,
    val name: String,
    val subSubCategories: List<SubSubCategory>
)

class SubSubCategory(
    val id: Int,
    val name: String,
    val services: List<Service>
)

class Service(
    val id: Int,
    val name: String
)
```

- Трудно придумать хорошие отличимые названия для вложенных друг в друга классов.
- При появлении нового уровня вложенности нужно создавать новый класс.
- Если конечный объект может находиться на любом уровне вложенности, придется добавлять его в класс каждого уровня.

#### Хорошо

```kotlin
class Category(
    val id: Int,
    val name: String,
    val services: List<Service>,
    val categories: List<Category>
)

class Service(
    val id: Int,
    val name: String
)
```

### Перед наследованием подумайте, может лучше композиция

#### Плохо

```kotlin
class ProductFragment : BaseFragment {

    private var productPriceTextView: TextView? = null

    fun showProduct(product: Product) {
        productPriceTextView?.text = formatMoney(product.price)
    }
}

class ServiceFragment : BaseFragment {

    private var servicePriceTextView: TextView? = null

    fun showService(service: Service) {
        servicePriceTextView?.text = formatMoney(service.price)
    }
}

open class BaseFragment : Fragment {

    protected fun formatMoney(money: Money): String {
        // Некая логика форматирования цены
        return "1.234.567,89 $"
    }
}

class CartSummaryViewHolder : RecyclerView.ViewHolder {

    private var priceTextView: TextView? = null

    fun showSummary(cart: Cart) {
        // Стоп... а как отформатировать цену,
        // если форматирование находится во фрагменте?
        priceTextView?.text = ""
    }
}
```

- Невозможно использовать метод из базового класса в другом дереве наследования.

#### Хорошо

```kotlin
class MoneyFormatter {

    fun format(Money money): String {
        // Некая логика форматирования цены
        return "1.234.567,89 $"
    }
}

class ProductFragment : Fragment {

    private var productPriceTextView: TextView? = null
    private val moneyFormatter = MoneyFormatter()

    fun showProduct(product: Product) {
        productPriceTextView?.text = moneyFormatter.format(product.price)
    }
}

class ServiceFragment : Fragment {

    private var servicePriceTextView: TextView? = null
    private val moneyFormatter = MoneyFormatter()

    fun showService(service: Service) {
        servicePriceTextView?.text = moneyFormatter.format(service.price)
    }
}

class CartSummaryViewHolder : RecyclerView.ViewHolder {

    private var priceTextView: TextView? = null
    private val moneyFormatter = MoneyFormatter()

    fun showSummary(cart: Cart) {
        priceTextView?.text = moneyFormatter.format(cart.totalAmount)
    }
}
```

## Общее

### Комментируйте, что делает код, а не как делает код

#### Плохо

```kotlin
class SettingsProvider {

    fun getSettings(onSuccess: (settings: Settings) -> Unit) {
        // Достать настройки из кеша
        val cachedSettings: Settings = getCachedSettings()

        // Сразу вернуть закешированные настройки, если они есть
        val returnCached = cachedSettings != null

        // Возвращать скачанные с сервера настройки только если кеш пустой
        val returnRemote = cachedSettings == null

        // Вернуть кеш, если нужно
        if (returnCached) {
            onSuccess.invoke(cachedSettings)
        }

        // Всегда скачиваем актуальные настройки с сервера
        loadSettingsFromServer(
            onSuccess = { settings: Settings ->
                cacheSettings(settings)

                if (returnRemote) {
                    onSuccess.invoke(settings)
                }
            }
        )
    }
}
```

- Комментарии, повторяющие исходный код, не несут смысловой нагрузки и не дают информации о причине действий.
- Комментарии к каждой строке делают код разрозненным, из-за чего его сложнее читать.

#### Хорошо

```kotlin
/**
 * Предоставляет доступ к настройкам приложения.
 * Может использоваться в коде с целью абстрагироваться
 * от источника данных (API, Firebase, кеш и т.д.)
 */
class SettingsProvider {

    fun getSettings(onSuccess: (settings: Settings) -> Unit) {
        // По просьбе заказчика нужно использовать настройки
        // из кеша, если они там есть, но актуализировать кеш
        // в любом случае.
        val cachedSettings: Settings = getCachedSettings()

        val returnCached = cachedSettings != null
        val returnRemote = cachedSettings == null

        if (returnCached) {
            onSuccess.invoke(cachedSettings)
        }

        loadSettingsFromServer(
            onSuccess = { settings: Settings ->
                cacheSettings(settings)

                if (returnRemote) {
                    onSuccess.invoke(settings)
                }
            }
        )
    }
}
```

### Код должен читаться сверху вниз, как журнальная статья

#### Плохо

```kotlin
class QueueProcessor {

    companion object {
        const val WAIT_TIMEOUT: Int = 1
    }

    var currentJob: Job? = null

    private suspend fun runNextJob() {
        val job: Job? = getNextJob()

        if (job != null) {
            runJob(job)
        }
    }

    private suspend fun wait() {
        delay(WAIT_TIMEOUT)
    }

    private fun clearCurrentJob() {
        this.currentJob = null
    }

    private fun setCurrentJob(job: Job) {
        this.currentJob = job
    }

    public fun run() {
        CoroutineScope(Dispatchers.Default).launch {
            while (isActive) {
                runNextJob()
                wait()
            }
        }
    }

    private suspend fun runJob(job: Job) {
        setCurrentJob(job)
        job.run()
        clearCurrentJob()
    }

    private fun getNextJob(): Job? {
        // Some logic to fetch the next job.
    }
}
```

- Для чтения кода приходится несколько раз пролистывать файл вверх-вниз, что создает дополнительную нагрузку для разработчика.

#### Хорошо

```kotlin
class QueueProcessor {

    companion object {
        const val WAIT_TIMEOUT: Int = 1
    }

    var currentJob: Job? = null

    public fun run() {
        CoroutineScope(Dispatchers.Default).launch {
            while (isActive) {
                runNextJob()
                wait()
            }
        }
    }

    private suspend fun runNextJob() {
        val job: Job? = getNextJob()

        if (job != null) {
            runJob(job)
        }
    }

    private suspend fun wait() {
        delay(WAIT_TIMEOUT)
    }

    private fun getNextJob(): Job? {
        // Some logic to fetch the next job.
    }

    private suspend fun runJob(job: Job) {
        setCurrentJob(job)
        job.run()
        clearCurrentJob()
    }

    private fun setCurrentJob(job: Job) {
        this.currentJob = job
    }

    private fun clearCurrentJob() {
        this.currentJob = null
    }
}
```

### Избавляйтесь от дублирования кода

- Если код дублируется, то при изменении алгоритма придется менять код в нескольких местах.
- Есть разные способы избавиться от дублирования кода. У каждого из них есть свои плюсы и минусы.
- Похожий код не всегда означает дублирование кода.

#### Дублирование в одном классе

##### Плохо

```kotlin
class UsersListScreen {

    fun showClient(client: User, clientView: ClientView) {
        clientView.firstName.text = client.firstName
        clientView.lastName.text = client.lastName

        val description =
            "Email: " + client.email + "\n" +
            "Address: " + client.address + "\n" + 
            "Registered at: " + client.registeredAt

        clientView.description.text = description
    }

    fun showAdmin(admin: User, adminView: AdminView) {
        adminView.fullName = admin.fullName

        val description =
            "Email: " + admin.email + "\n" +
            "Address: " + admin.address + "\n" + 
            "Registered at: " + admin.registeredAt

        adminView.description.text = description
    }
}
```

##### Хорошо

```kotlin
class UsersListScreen {

    fun showClient(client: User, clientView: ClientView) {
        clientView.firstName.text = client.firstName
        clientView.lastName.text = client.lastName
        clientView.description = getDescription(client)
    }

    fun showAdmin(admin: User, adminView: AdminView) {
        adminView.fullName = admin.fullName
        clientView.description = getDescription(admin)
    }

    private fun getDescription(user: User): String {
        return "Email: " + admin.email + "\n" +
            "Address: " + admin.address + "\n" + 
            "Registered at: " + admin.registeredAt
    }
}
```

Если код продублирован в одном классе, можно вынести его в функции этого же класса.
Однако, не получится переиспользовать код в других классах.
Пользуйтесь таким способом только в том случае, если нет предпосылок для его использования в других классах.

#### Дублирование в разных классах одной иерархии

##### Плохо

```kotlin
class ProfileScreen : BaseUserScreen {

    fun showProfile(personalInfo: PersonalInfo) {
        val parts = personalInfo.fullName.split(" ")

        val firstName = parts[0]
        val lastName = parts[1]

        this.firstNameTextView.text = firstName
        this.lastNameTextView.text = lastName
    }
}

class UsersListScreen : BaseUserScreen {

    fun showUser(user: User, userView: UserView) {
        var parts = user.personalInfo.fullName.split(" ")

        val firstName = parts[0]
        val lastName = parts[1]

        userView.firstName.text = firstName
        userView.lastName.text = lastName
    }
}
```

##### Хорошо

```kotlin
class ProfileScreen : BaseUserScreen {

    fun showProfile(personalInfo: PersonalInfo) {
        this.firstNameTextView.text = getFirstName(personalInfo)
        this.lastNameTextView.text = getLastName(personalInfo)
    }
}

class UsersListScreen : BaseUserScreen {

    fun showUser(user: User, userView: UserView) {
        val personalInfo = user.personalInfo

        userView.firstName.text = getFirstName(personalInfo)
        userView.lastName.text = getLastName(personalInfo)
    }
}

open class BaseUserScreen {

    protected fun getFirstName(PersonalInfo personalInfo): String {
        return getFullNameParts(personalInfo)[0]
    }

    protected fun getLastName(PersonalInfo personalInfo): String {
        return getFullNameParts(personalInfo)[1]
    }

    protected fun getFullNameParts(PersonalInfo personalInfo): List<String> {
        return personalInfo.fullName.split(" ")
    }
}
```

Можно вынести дублирующийся код в базовый класс.
Но получится переиспользовать код только в рамках одной иерархии.

#### Дублирование в классах из разных иерархий

##### Плохо

```kotlin
class ProfileScreen : BaseObjectScreen {

    fun showProfile(personalInfo: PersonalInfo) {
        val parts = personalInfo.fullName.split(" ")

        val firstName = parts[0]
        val lastName = parts[1]

        this.firstNameTextView.text = firstName
        this.lastNameTextView.text = lastName
    }
}

class UsersListScreen : BaseListScreen {

    fun showUser(user: User, userView: UserView) {
        var parts = user.personalInfo.fullName.split(" ")

        val firstName = parts[0]
        val lastName = parts[1]

        userView.firstName.text = firstName
        userView.lastName.text = lastName
    }
}
```

##### Хорошо

```kotlin
class ProfileScreen : BaseObjectScreen {

    fun showProfile(personalInfo: PersonalInfo) {
        this.firstNameTextView.text = personalInfo.getFirstName()
        this.lastNameTextView.text = personamInfo.getLastName()
    }
}

class UsersListScreen : BaseListScreen {

    fun showUser(user: User, userView: UserView) {
        userView.firstName.text = user.personalInfo.getFirstName()
        userView.lastName.text = user.personalInfo.getLastName()
    }
}

class PersonalInfo(val fullName: String) {

    fun getFirstName(): String {
        return this.fullNameParts()[0]
    }

    fun getLastname(): String {
        return this.fullNameParts()[1]
    }

    private fun getFullNameParts(): List<String> {
        return this.fullName.split(" ")
    }
}
```

- Если код продублирован в разных классах одной иерархии, можно вынести его в другой класс, который используется в дублирующемся коде.

#### Тоже хорошо, если нельзя нарушать ответственность

```kotlin
class PersonalInfoLogic(val persinalInfo: PersonalInfo) {

    fun getFirstName(): String {
        return this.fullNameParts()[0]
    }

    fun getLastName(): String {
        return this.fullNameParts()[1]
    }

    private fun getFullNameParts(): List<String> {
        return this.personalInfo.fullName.split(" ")
    }
}


val personalInfo = getPersonalInfo()
val personalInfoLogic = PersonalInfoLogic(personalInfo)

personalInfoLogic.getFirstName()
personalInfoLogic.getLastName()
```

- Избавляйтесь от дублирующегося кода.
- Уделите внимание месту, куда будет вынесен одинаковый код (отсортировано по возможности переиспользования от худшего к лучшему):
  - В тот же класс.
  - В базовый класс иерархии.
  - В существующий класс, который используется в дублирующемся коде.
  - В новый специальный класс.

#### Исключения

Если код выглядит одинаково, то это не обязательно дублирование.

```kotlin
class Model(val id: Integer, val name: String) {}
class Brand(val id: Integer, val name: String) {}
```

Здесь дублирование, это случайность, в рамках которой набор полей и их названий повторяются между двумя несвязанными классами.

### И ещё несколько рекомендаций

- Используйте единый стиль именования для одинаковых вещей.
- Не делайте много преждевременной оптимизации.
- Удаляйте неиспользуемый и закомментированный код.
- Вместо покрытия кода комментариями пытайтесь отраизить свои намерения в коде с помощью продуманного именования, введения промежуточных переменных и функций.
- Придерживайтесь стиля написания кода вашего языка/фреймворка.
- Используйте части речи по назначению
  - Используйте глаголы как основу для названий функций.
  - Используйте существительные для классов и переменных.
  - Для Boolean используйте прилагательные и причастия.
