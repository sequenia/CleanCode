# Основные принципы чистого кода

## Назначение

Облегчить чтение, тестирование и поддержку исходного кода командой разработчиков.

Минусы:
- На разработку тратится больше времени.

Плюсы:
- Значительно сокращаются затраты на работу с существующим исходным кодом.

## Именования

### Используйте осмысленные и легко произносимые названия переменных/функций/классов

#### Плохо

```swift
class MyParser {
    func parseA(yyyymmddhhmmss: String) -> Date {
        return f("yyyy-MM-dd'T'HH:mm:ssZ").date(from: yyyymmddhhmmss)!
    }

    func parseB(yyyymmdd: String) -> Date {
        return f("yyyy-MM-dd").date(from: yyyymmdd)!
    }

    func f(ff: String) -> DateFormatter {
        let df = DateFormatter()
        df.locale = Locale(identifier: "en_US_POSIX")
        df.dateFormat = ff
        return df
    }
}
```
- Суффиксы и префиксы "My", "Custom", “A”, “B” и подобные им не несут смысловой нагрузки, как и перечисления через “1”, “2”, “one”, “two”.
- Трудночитаемые и труднопроизносимые названия, такие как “yyyymmddhhmmss”, усложняют чтение и написание кода.
- Если название функции не отражает ее назначение, то без просмотра исходника невозможно понять, что она делает. То же самое относится к классам и переменным.

#### Хорошо

```swift
class DateParser {
    func parseIsoDateTime(isoDateTime: String) -> Date {
        return dateFormatter("yyyy-MM-dd'T'HH:mm:ssZ").date(from: isoDateTime)!
    }

    func parseIsoDate(isoDate: String) -> Date {
        return dateFormatter("yyyy-MM-dd").date(from: isoDate)!
    }

    func dateFormatter(dateFormat: String) -> DateFormatter {
        let dateFormatter = DateFormatter()
        dateFormatter.locale = Locale(identifier: "en_US_POSIX")
        dateFormatter.dateFormat = dateFormat
        return dateFormatter
    }
}
```

### Не используйте сокращения

#### Плохо

```javascript
let c = await findCars();

for (let i = 0; i < c.length; i++) {
  let cc = c[i];
  let o = await findOrders(cc);

  for (let j = 0; o.length; j++) {
    let oo = o[j];
    // Do some stuff with car and order.
    // Stop... what is a `c`?
    // ...and, what is a `oo`?
  }
}
```

- Чтобы прочитать код с сокращениями, нужно их изучить и запомнить. Это повышает нагрузку на программиста.
- Сокращенные названия индексов и других переменных легко перепутать, что будет трудно отладить.
- Исключение - известные аббревиатуры.


#### Хорошо

```javascript
let cars = await findCars();

for (let carIndex = 0; carIndex < cars.length; carIndex++) {
  let car = cars[carIndex];
  let orders = await findOrders(car);

  for (let orderIndex = 0; orderIndex < orders.length; orderIndex++) {
    let order = orders[orderIndex];
    // Do some stuff with car and order.
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

```javascript
function showItemsInPager(items, onPageChangeListener) {
  this.pager.setItems(items);

  setTimeout(
    () => { this.pager.setOnPageChangeListener(onPageChangeListener) },
    300
  );
}
```

```javascript
function potentialEnergy(mass, height) {
  return mass * height * 9.81;
}
```

- По не именованному значению в коде трудно понять его смысл и назначение.
- Если значение встречается несколько раз, то поменять его придется во всех местах.

#### Хорошо

```javascript
const PAGER_LISTENER_DELAY_TO_PREVENT_INITIAL_TRIGGERING = 300;

function showItemsInPager(items, onPageChangeListener) {
  this.pager.setItems(items);

  setTimeout(
    () => { this.pager.setOnPageChangeListener(onPageChangeListener) },
    PAGER_LISTENER_DELAY_TO_PREVENT_INITIAL_TRIGGERING
  );
}
```

```javascript
const gravitationConstant = 9.81;

function potentialEnergy(mass, height) {
  return mass * height * gravitationConstant;
}
```

### Заводите переменные для промежуточных вычислений

#### Плохо

```ruby
def parse_page(page_title)
  self.save_product_info(
    page_title.split('@')[1],
    page_title.split('@')[0].split('-')[0].strip
  )
end
```

- Трудно понять смысл вычислений без просмотра аргументов функции `saveProductInfo`.
- Часть одинакового исходного кода выполняется несколько раз.

#### Хорошо

```ruby
def parse_page(page_title)
  title_parts = page_title.split('@')

  product_title = title_parts[0].split('-')[0].strip
  shop_name = title_parts[1]

  self.save_product_info(product_title, shop_name)
end
```

#### Плохо

```ruby
def save_film(film)
  if self.user.roles.admin.any? || film.creator_id == self.user.id
    self.repository.save(film)
  end
end
```

- Трудно понять смысл проверки без детального изучения каждого условия.
- Код разросся по горизонтали.

#### Хорошо

```ruby
def save_film(film)
  is_admin = self.user.roles.admin.any?
  is_creator = film.creator_id == self.user.id

  if is_admin || is_creator
    self.repository.save(film)
  end
end
```

```ruby
def save_film(film)
  if self.is_admin || self.is_creator(film)
    self.repository.save(film)
  end
end

def is_admin
  self.user.roles.admin.any?
end

def is_creator(film)
  film.creator_id == self.user.id
end
```

## Функции

### Избегайте большого числа аргументов

#### Плохо

```java
public class ProfileScreen {
    public static void open(String firstName,
                            String lastName,
                            Gender gender,
                            Country country,
                            String city,
                            String street,
                            String house) {
        // Open new screen using SDK
    }
}

public class HomeScreen {
    public void onProfileButtonClicked() {
        String firstName = firstNameView.getText();
        String lastName = lastNameView.getText();
        // Get other values from views

        ProfileScreen.open(
          firstName, lastName, gender,
          country, city, street, house
        );
    }
}
```

- Легко запутаться в порядке аргументов и передать их на неправильных местах.
- Трудно отформатировать функцию, приходится делать переносы в описании аргументов.
- При необходимости добавить новую информацию о сущности, нужно будет рефакторить цепочку вызова функций.

#### Хорошо

```java
public class ProfileScreen {
    public static void open(PersonalInfo personalInfo, Address address) {
        // Open new screen using SDK
    }
}

public class HomeScreen {
    public void onScreenButtonClicked() {
        String firstName = firstNameView.getText();
        String lastName = lastNameView.getText();
        // Get other values from views

        PersonalInfo personalInfo = new PersonalInfo(firstName, lastName, gender);
        Address address = new Address(country, city, street, house);

        ProfileScreen.open(personalInfo, address);
    }
}
```

#### Плохо

```java
public class ImageUtils {
    public static File ensureCompatibility(
        File imageFile,
        String userAgent,
        boolean useCache,
        boolean debugOutput,
        boolean removeTempFiles) {

        // Image conversion logic
    }
}

public void main() {
    ImageUtils.ensureCompatibility(
        new File("../storage/image.webp"),
        "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36",
        false,
        true,
        true
    );
}
```

- При вызове функции трудно определить, за что отвечают похожие по типу не именованные аргументы.
- Если от значения аргументов зависит ветвление алгоритма, то при большом количестве аргументов становится трудно тестировать функцию.

#### Хорошо

```java
public class ImageCompatibilityService {
    private boolean useCache = true;
    private boolean debugOutput = false;
    private boolean removeTempFiles = true;

    // Setters

    public File convert(File imageFile, String userAgent) {
        // Image conversion logic
    }
}

public void main() {
    ImageCompatibilityService converter = new ImageCompatibilityService()
        .setUseCache(false)
        .setDebugOutput(true)
        .setRemoveTempFiles(true)

    converter.convert(
        new File("../storage/image.webp"),
        "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36",
    );
}
```

#### Решение через именованные аргументы

```ruby
class ImageUtils
  def self.ensure_compatibility(image_file:,
                                user_agent:,
                                use_cache: true,
                                debug_output: false,
                                remove_temp_files: true)
    # Image conversion logic
  end
end

# Somewhere in the code
ImageUtils.ensure_compatibility(
  image_file: File.new('../storage/image.webp'),
  user_agent: 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36',
  use_cache: false,
  debug_output: true,
  remove_temp_files: true
);
```

### Разбивайте комплексный код на простые функции

#### Плохо

```swift
func coloriseOrderStatus(order: Order) {
    var colorHex: String? = nil

    if order.creationDate != nil {
        colorHex = "#ff0000"
    } else if order.processingStartDate != nil {
        colorHex = "#00ff00"
    } else if order.deliveryStartDate != nil {
        colorHex = "#0000ff"
    } else {
        colorHex = "#000000"
    }

    statusView.backgroundColor = getUIColorByHex(colorHex)
}

func getUIColorByHex(hex: String) -> UIColor {
    // Create UIColor by hex
}
```

- В коде одной функции смешано несколько задач, среди которых не видно алгоритма и главного действия.

#### Хорошо

```swift
func coloriseOrderStatus(order: Order) {
    let status = getOrderStatus(order)
    let colorHex = getOrderColorHexByStatus(status)
    let color = getUIColorByHex(color)

    textView.backgroudColor = color
}

func getOrderStatus(order: Order) -> OrderStatus {
    if order.creationDate != nil {
        return .created
    } else if order.processingStartDate != nil {
        return .processing
    } else if order.deliveryStartDate != nil {
        return .delivery
    } else {
        return .unknown
    }
}

func getOrderColorHexByStatus(status: OrderStatus) -> String {
    switch status {
    case .created:
        return "#ff0000"
    case .processing:
        return "#00ff00"
    case .delivery:
        return "0000ff"
    case .unknown:
        return "#000000"
    }
}

func getUIColorByHex(hex: String) -> UIColor {
    // Create UIColor by hex
}
```

### Код в функции должен быть на одном уровне абстракции

#### Плохо

```java
public Product parseProductFromPage(Page page) {
    String name = null;
    Element titleElement = page.findElementByTag("title");

    if (titleElement != null) {
        String title = titleElement.getText();
        name = title.split("-")[0].trim();
    }

    String description = findDescriptionOnPage(page);
    Integer rating = findRatingOnPage(page);

    return new Product(name, description, rating);
}
```

- Чтение кода с разным уровнем абстракции заставляет мозг переключаться между уровнями, неявно выстраивая новый уровень абстракции самостоятельно.

#### Хорошо

```java
public Product parseProductFromPage(Page page) {
    String name = findNameOnPage(page);
    String description = findDescriptionOnPage(page);
    Integer rating = findRatingOnPage(page);

    return new Product(name, description, rating);
}

public String findNameOnPage(Page page) {
    Element titleElement = page.findElementByTag("title");

    if (titleElement == null) {
        return null;
    }

    String title = titleElement.getText();
    String name = title.split("-")[0].trim();

    return name;
}
```

### Избегайте побочных действий

#### Плохо

```ruby
class Table
  attr_accessor :items

  def add_items(items)
    if self.items.nil?
      self.items = items
    else
      self.items.push(*items)
    end
  end
end


items = [1, 2, 3]
table = Table.new
table.add_items(items)

# Somewhere in the other code
items.push(4, 5, 6) # Oops, items added to the table.
```

- Так как ссылка на один и тот же объект захвачена разными контекстами, то модификация объекта неявным образом влияет на работу каждого контекста.

#### Хорошо

```ruby
class Table
  attr_accessor :items

  def add_items(items)
    if self.items.nil?
      self.items = []
    end

    self.items.push(*items)
  end
end


items = [1, 2, 3]
table = Table.new
table.add_items(items)

# Somewhere in the other code
items.push(4, 5, 6) # This is OK
```

### Указывайте на скрытые условия в названии функции

#### Плохо

```swift
func runTask(task: Task) {
  if !task.isRunning() {
    task.run()
  }
}

// Working with already running task
let task = getTask()
runTask(task) // Why the task didn't run if i called the method????
```

- Невозможно понять весь алгоритм, если не посмотреть реализацию всех функций.

#### Хорошо

```swift
func runTaskIfNotRunning(task: Task) {
  if !task.isRunning() {
    task.run()
  }
}

// I know that task will not be runned if already runned.
let task = getTask()
runTaskIfNotRunning(task);
```

### Заменяйте вложенные условные операторы граничным оператором

#### Плохо

```java
public Integer getTotalAmount() {
    if (isArchived()) {
        return null;
    } else {
        if (this.cachedTotalAmount != null) {
            return this.cachedTotalAmount;
        } else {
            if (isEnoughData()) {
                return this.calculateTotalAmount();
            } else {
                return this.getDefaultTotalAmount();
            }
        }
    }
}
```

- Среди множества вложенных условий трудно выделить нормальный ход выполнения кода.
- Выделите все проверки специальных или граничных случаев в отдельные проверки и поместите их перед основным кодом, чтобы получить линейную структуру без вложенностей.

#### Хорошо

```java
public Integer getTotalAmount() {
    if (isArchived()) {
        return null;
    }

    if (this.cachedTotalAmount != null) {
        return this.cachedTotalAmount;
    }

    if (!this.isEnoughData()) {
        return this.getDefaultTotalAmount();
    }

    return this.calculateTotalAmount();
}
```

### Не используйте флаги в аргументах функции

#### Плохо

```javascript
class ProjectPage {
  loadProject() {
    this.projectUseCase
      .getById(this.projectId)
      .then((project) => {
        this.showProject(project);
        this.showMessage('Loaded', true);
      })
      .catch(() => {
        this.showMessage('Loading error', false);
      });
  }

  showMessage(messageText, isError) {
    if (isError) {
      // Show error message, for example via SnackBar
    } else {
      // Show success message, for example via Toast
    }
  }
}
```

- Флаг в аргументах вносит избыточное ветвление и делает функцию ответственной за несколько вещей.

#### Хорошо

```javascript
class ProjectPage {
  loadProject() {
    this.projectUseCase
      .getById(this.projectId)
      .then((project) => {
        this.showProject(project);
        this.showSuccessMessage('Loaded');
      })
      .catch(() => {
        this.showErrorMessage('Loading error');
      });
  }

  showSuccessMessage(messageText) {
    // Show success message, for example via Toast
  }

  showErrorMessage(messageText) {
    // Show error message, for example via SnackBar
  }
}
```

## Классы

### Не используйте God Object для хранения информации

#### Плохо

```kotlin
class Info(
  var id: Integer? = null,
  var name: String? = null,
  var status: String? = null,
  var description: String? = null,
  var price: Integer? = null
)

val product = Info()
product.id = 1
product.name = "Spoon"
product.description = "Material - Silver"
product.price = 3

val cart = Info()
cart.id = 1
cart.status = CartStatus.CHECKOUT.value
cart.price = 10
```

- С течением времени God Object бесконечно разрастается.
- Приходится работать с лишними полями.

#### Хорошо

```kotlin
class Product(
  var id: Integer? = null,
  var name: String? = null,
  var description: String = null,
  val price: Integer? = null
)

class Cart(
  var id: Integer? = null,
  var status: String? = null,
  var price: String? = null
)

val product = Product()
product.id = 1
product.name = "Spoon"
product.description = "Material - Silver"
product.price = 3

val cart = Cart()
cart.id = 1
cart.status = CartStatus.CHECKOUT.value
cart.setPrice = 10
```

### Создавайте data-классы для группировки полей по контексту

#### Плохо

```ruby
class City
  attr_accessor :city_id, :city_name, :region_id, :region_name
end

city = City.new
city.city_id = 1
city.city_name = "Novosibirsk"
city.region_id = 1
city.region_name = "Novosibirsk region"
```

- Невозможно независимо переиспользовать некоторые поля.
- Код разрастается из-за дублирования суффиксов.

#### Хорошо

```ruby
class City
  attr_accessor :id, :name, :region
end

class Region
  attr_accessor :id, :name
end

region = Region.nnew
region.id = 1
region.name = "Novosibirsk region"

city = City.new
city.id = 1
city.name = "Novosibirsk"
city.region = region
```

### Скрывайте методы, которые не будут использоваться снаружи

#### Плохо

```swift
public class CarCreator {
    private apiService = ApiService()

    public func create(car: Car) -> Car {
        let json = generateCarJson(car)
        let createdCar = apiService.createCar(car)
        return createdCar
    }

    public func generateCarJson(car: Car) -> String {
        // Some logic of json generation
    }
}

let car = Car()
let carCreator = CarCreator()
carCreator.generateCarJson(car) // Why do I need this function?
```

- Среда разработки при автодополнении показывает методы, которые не нужны программисту, который использует класс.

#### Хорошо

```swift
public class CarCreator {
    private apiService = ApiService()

    public func create(car: Car) -> Car {
        let json = generateCarJson(car)
        let createdCar = apiService.createCar(car)
        return createdCar
    }

    private func generateCarJson(car: Car) -> String {
        // Some logic of json generation
    }
}

let car = Car()
let carCreator = CarCreator()
carCreator.create(car) // I see only necessary functions
```

### Используйте рекурсию для описания глубокой вложенности

#### Плохо

```java
public class Service {
    private int id;
    private String name;
}

public class Category {
    private int id;
    private String name;
    private List<SubCategory> subCategories;
}

public class SubCategory {
    private int id;
    private String name;
    private List<SubSubCategory> subSubCategories;
}

public class SubSubCategory {
    private int id;
    private String name;
    private List<Service> services;
}

Category category = getCategory();
category.getSubCategories().getSubSubCategories().getServices();
```

- Трудно придумать хорошие отличимые названия для вложенных друг в друга классов.
- При появлении нового уровня вложенности нужно создавать новый класс.
- Если конечный объект может находиться на любом уровне вложенности, придется добавлять его в класс каждого уровня.

#### Хорошо

```java
public class Service {
    private int id;
    private String name;
}

public class Category {
    private int id;
    private String name;
    private List<Services> services;
    private List<Category> categories;
}

Category category = getRootCategory();
category.getCategories().getCategories().getServices();
```

### Перед наследованием подумайте, может лучше композиция

#### Плохо

```kotlin
class ProductScreen : Screen {
  fun showProduct(product: Product) {
    this.price.text = this.formatMoney(product.price)
  }
}

class ServiceScreen : Screen {
  fun showService(service: Service) {
    this.price.text = this.formatMoney(service.price)
  }
}

open class Screen : SomeSdkClass {
  protected fun formatMoney(money: Money): String {
    // Some formatting logic
    return "1.234.567,89 $"
  }
}

class CartSummaryCell : Cell {
  fun showSummary(cart: Cart) {
    // Whait! I don't have a formatMoney method here...
    this.amount.text = ""
  }
}
```

- Невозможно использовать метод из базового класса в другом дереве наследования.

#### Хорошо

```kotlin
class MoneyFormatter {
  fun format(Money money): String {
    // Some formatting logic
    return "1.234.567,89 $"
  }
}

class ProductScreen : Screen {
  fun showProduct(product: Product) {
    this.price.text = MoneyFormatter().format(product.price)
  }
}

class ServiceScreen : Screen {
  fun showService(service: Service) {
    this.price.text = MoneyFormatter().format(service.price)
  }
}

class CartSummaryCell : Cell {
  fun showSummary(cart: Cart) {
    this.amount.text = MoneyFormatter().format(cart.totalAmount)
  }
}
```

### Пользуйтесь геттерами и сеттерами

Используя геттеры и сеттеры вы получаете полный контроль при управлении объектом:
- Можно настраивать дополнительную логику при считывании значения, например, ленивая инициализация.
- Можно настраивать дополнительную логику при задании значения, например, сброс значения другого поля.
- Можно управлять областью видимости полей, чтобы было удобнее работать с объектом извне.

## Общее

### Комментируйте, что делает код, а не как делает код

#### Плохо

```java
public class SettingsProvider {
    public void getSettings(SettingsCallback callback) {
        // Get settings from cache
        Settings cachedSettings = getCachedSettings();

        // Needs to return cached if already exists.
        boolean returnCached = cachedSettings != null;

        // Needs to return remote only if cached does not exists.
        boolean returnRemote = cachedSettings == null;

        // Return cached settings if needs
        if (returnCached) {
            callback.onSuccess(cachedSettings);
        }

        // Always load actual settings from server
        loadSettingsFromServer(new SettingsCallback() {
            @Override
            public void onSuccess(Settings settings) {
                // Cache loaded settings
                cacheSettings(settings);

                // Return loaded settings if needs
                if (returnRemote) {
                    callback.onSuccess(settings);
                }
            }
        });
    }
}
```

- Комментарии к исполняемому коду дублируют сам код и, соответственно, не несут смысловой нагрузки.
- Комментарии к каждой строке делают код разрозненным, из-за чего его сложнее читать.
- В исполняемом коде есть неявности. Не понятно, по какой причине производятся некоторые действия.
- Чтобы понять, за что отвечает класс, нужно изучить его наполнение.

#### Хорошо

```java
/**
 * Provides the access to the application settings.
 * Can be used in the other code in order to abstract
 * from the source of data (API, Firebase, Cache, etc.)
 */
public class SettingsProvider {
    /**
    * Provides the current settings.
    * @param callback - A callback to get a result.
    */
    public void getSettings(SettingsCallback callback) {
        // At the request of the customer needs to
        // use the existing settings
        // if they have been downloaded at least once
        // but refresh the actual ones every time.
        Settings cachedSettings = getCachedSettings();

        boolean returnCached = cachedSettings != null;
        boolean returnRemote = cachedSettings == null;

        if (returnCached) {
            callback.onSuccess(cachedSettings);
        }

        loadSettingsFromServer(new SettingsCallback() {
            @Override
            public void onSuccess(Settings settings) {
                cacheSettings(settings);

                if (returnRemote) {
                    callback.onSuccess(settings);
                }
            }
        });
    }
}
```

### Код должен читаться сверху вниз

#### Плохо

```java
public class QueueProcessor {
    private static final int WAIT_TIMEOUT = 1;

    private Job currentJob;

    private void runNextJob() {
        Job job = getNextJob();

        if (job != null) {
            runJob(job);
        }
    }

    private void wait() {
        TimeUnit.SECONDS.sleep(WAIT_TIMEOUT);
    }

    private void clearCurrentJob() {
        this.currentJob = null;
    }

    private void setCurrentJob(Job job) {
        this.currentJob = job;
    }

    public void run() {
        while (true) {
            runNextJob();
            wait();
        }
    }

    private void runJob(Job job) {
        setCurrentJob(job);
        job.run();
        clearCurrentJob();
    }

    private Job getNextJob() {
        // Some logic to fetch the next job.
    }
}
```

- Для чтения кода приходится несколько раз пролистывать файл вверх-вниз, что создает дополнительную нагрузку для разработчика.

#### Хорошо

```java
public class QueueProcessor {
    private static final int WAIT_TIMEOUT = 1;

    private Job currentJob;

    public void run() {
        while (true) {
            runNextJob();
            wait();
        }
    }

    private void runNextJob() {
        Job job = getNextJob();

        if (job != null) {
            runJob(job);
        }
    }

    private void wait() {
        TimeUnit.SECONDS.sleep(WAIT_TIMEOUT);
    }

    private Job getNextJob() {
        // Some logic to fetch the next job.
    }

    private void runJob(Job job) {
        setCurrentJob(job);
        job.run();
        clearCurrentJob();
    }

    private void setCurrentJob(Job job) {
        this.currentJob = job;
    }

    private void clearCurrentJob() {
        this.currentJob = null;
    }
}
```

### Избавляйтесь от дублирования кода

- Если код дублируется, то при изменении алгоритма придется менять код в нескольких местах.
- Есть разные способы избавиться от дублирования кода. У каждого из них есть свои плюсы и минусы.
- Похожий код не всегда означает дублирование кода (What???).

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
    val personalInfo = user.getPersonalInfo();

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

class PersonalInfo {
  // Existing fields, getters and setters

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
class PersonalInfoNamesDecorator(val persinalInfo: PersonalInfo) {

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
val namesDecorator = PersonalInfoNamesDecorator(personalInfo)

namesDecorator.getFirstName()
namesDecorator.getLastName()
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
- Придерживайтесь стиля написания кода вашего языка/фреймворка.
- Используйте части речи по назначению
  - Используйте глаголы как основу для названий функций.
  - Используйте существительные для классов и переменных.
  - Для Boolean используйте прилагательные и причастия.

## Источники

1. Книга "Чистый код. Создание анализ и рефакторинг | Мартин Роберт К."
2. Сайт [Refactoring Guru](https://refactoring.guru/ru)
