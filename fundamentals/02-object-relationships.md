# 02 — Nesneler Arası İlişkiler ve Tasarım Kalitesi



## 1. Nesneler neden ilişki kurar?

Bir yazılım tek bir büyük sınıftan oluşmaz. Örneğin sipariş alan bir sistemde `Order`, `OrderItem`, `Customer`, `PaymentService` ve `NotificationService` farklı sorumluluklara sahiptir. Sipariş, müşteriye ait olabilir; sipariş kalemlerini içerebilir; ödeme hizmetini kullanabilir. Bütün davranışları `Order` sınıfına toplarsak değişiklikler aynı sınıfta birikir, test yapmak zorlaşır ve sorumluluklar belirsizleşir.

OOP tasarımında asıl soru yalnızca **"Hangi sınıflar var?"** değildir. Şunları da sormalıyız:

1. Bir nesne başka bir nesneyi **kalıcı olarak biliyor mu**, yoksa yalnızca bir işlem sırasında mı kullanıyor?
2. Nesne, bağımlılığını **kendisi mi yaratıyor**, yoksa dışarıdan mı alıyor?
3. İlişkili nesnelerin **yaşam döngülerini** kim yönetiyor?
4. Bir sınıftaki değişiklik **kaç başka sınıfı** etkiliyor?
5. Her sınıfın sorumlulukları **birbirleriyle ne kadar ilişkili**?

Bu sorular bizi nesneler arası ilişkilere, coupling'e ve cohesion'a götürür.

---

## 2. Association (İlişki): Bir nesnenin başka bir nesneyi bilmesi

### Kavramsal açıklama

**Association**, iki nesne arasında anlamlı bir bağlantı bulunmasıdır. Örneğin bir `Doctor`, bir `Patient` ile çalışabilir. Bu ilişki, bir nesnenin diğerini alan/property olarak tutmasıyla ifade edilebilir. Ancak association tek başına nesnelerin birlikte yaratıldığını, birbirlerine sahip olduğunu ya da aynı anda yok olacağını söylemez.

İlişki **tek yönlü** olabilir: `Appointment`, `Doctor` nesnesini bilir ama `Doctor` her randevunun referansını tutmak zorunda değildir. **Çift yönlü** association ise her iki nesnenin birbirine erişebilmesini sağlar; gereksiz kullanılırsa durum senkronizasyonunu ve bağımlılıkları karmaşıklaştırabilir.

### C# — Tek yönlü association

```csharp
public class Doctor
{
    public string Name { get; }
    public Doctor(string name) => Name = name;
}

public class Appointment
{
    public Doctor Doctor { get; }
    public DateTime StartsAt { get; }

    public Appointment(Doctor doctor, DateTime startsAt)
    {
        Doctor = doctor ?? throw new ArgumentNullException(nameof(doctor));
        StartsAt = startsAt;
    }
}

var doctor = new Doctor("Dr. Deniz");
var appointment = new Appointment(doctor, DateTime.Today.AddHours(10));
```

**Kodun anlamı:** `Appointment`, dışarıda oluşturulmuş bir `Doctor` nesnesine referans tutar. Randevunun sona ermesi doktor nesnesinin de sona ermesi gerektiği anlamına gelmez. `Doctor` ayrı bir alan nesnesidir.

### Python ve Java'da kısa karşılıklar

**Python**

```python
class Appointment:
    def __init__(self, doctor, starts_at):
        self.doctor = doctor
        self.starts_at = starts_at
```

**Java**

```java
class Appointment {
    private final Doctor doctor;

    Appointment(Doctor doctor) {
        this.doctor = java.util.Objects.requireNonNull(doctor);
    }
}
```

Üç dilde de temel ilişki aynıdır: `Appointment`, `Doctor` referansını tutar. C# ve Java tipleri statik olarak tanımlar; Python örneğinde tip bildirimi zorunlu değildir.

**Tasarım notu:** Her association kötü coupling anlamına gelmez. Nesnelerin iş birliği yapması zaten gereklidir; sorun, değişime hassas ve gereksiz bağlantıların artmasıdır.

---

## 3. Aggregation: Bütün–parça ilişkisi, bağımsız yaşam döngüsü

### Kavramsal açıklama

**Aggregation**, association'ın özel bir yorumudur: Bir bütün, bazı parçaları bir arada tutar ancak parçalar bütünden bağımsız olarak var olabilir. Örneğin bir `Team` birçok `Developer` içerir; geliştirici takım dağıldığında da var olmaya devam edebilir ve farklı takımlarda görev alabilir.

UML'de aggregation çoğunlukla **içi boş elmas** ile gösterilir. Ancak gerçek kodda `List<Developer>` kullanılması tek başına aggregation'ı kanıtlamaz. Yaşam döngüsü ve sahiplik, sistemin iş kurallarıyla belirlenir; C#, Python ve Java'da buna özel bir `aggregation` anahtar sözcüğü yoktur.

### C# — Bağımsız oluşturulan üyeler

```csharp
public class Developer
{
    public string Name { get; }
    public Developer(string name) => Name = name;
}

public class Team
{
    private readonly List<Developer> _members = new();

    public void Add(Developer developer)
    {
        _members.Add(developer);
    }
}

var developer = new Developer("Ayşe");
var team = new Team();
team.Add(developer);  // developer, team dışında da var.
```

### Python ve Java'da kısa karşılıklar

**Python**

```python
class Team:
    def __init__(self):
        self.members = []

    def add(self, developer):
        self.members.append(developer)
```

**Java**

```java
class Team {
    private final java.util.List<Developer> members = new java.util.ArrayList<>();

    void add(Developer developer) {
        members.add(developer);
    }
}
```

**Yorum:** Bu örneklerde üyeler dışarıda oluşturulur ve takıma eklenir. Ancak aggregation'ın doğru adlandırılması yine uygulamanın alan modeline ve sahiplik kurallarına bağlıdır.

---

## 4. Composition: Bütün–parça ilişkisi ve güçlü sahiplik

### Kavramsal açıklama

**Composition**, parçaların mantıksal yaşam döngüsünün bütüne güçlü biçimde bağlı olduğu bir bütün–parça ilişkisidir. Örneğin sipariş kaleminin (`OrderLine`) ilgili sipariş (`Order`) olmadan iş alanında anlamlı olmaması, composition düşüncesine uygundur. Bütün, parçaların oluşturulmasını ve yönetilmesini üstlenebilir.

Burada önemli bir ayrıntı var: **Composition, bellekteki nesnenin çöp toplayıcı tarafından hemen silinmesi demek değildir.** C#, Java ve Python'da yaşam döngüsü yönetiminin teknik mekanizmaları farklıdır. UML composition ise esas olarak modeldeki **mantıksal sahipliği** anlatır.

### C# — Sipariş kendi kalemlerini oluşturuyor

```csharp
public class OrderLine
{
    public string ProductName { get; }
    public int Quantity { get; }

    internal OrderLine(string productName, int quantity)
    {
        if (quantity <= 0) throw new ArgumentOutOfRangeException(nameof(quantity));
        ProductName = productName;
        Quantity = quantity;
    }
}

public class Order
{
    private readonly List<OrderLine> _lines = new();
    public IReadOnlyList<OrderLine> Lines => _lines.AsReadOnly();

    public void AddLine(string productName, int quantity)
    {
        _lines.Add(new OrderLine(productName, quantity));
    }
}
```

**Kodun anlamı:** `Order` kendi `OrderLine` nesnelerini üretir ve koleksiyonu dışarıdan doğrudan değiştirilemez şekilde sunar. `internal` erişim, başka kodların aynı assembly içinde `OrderLine` oluşturmasını yine de mümkün kılar; bu örnek katı bir yaşam döngüsü garantisi değil, güçlü sahiplik niyetini gösterir.

### Python ve Java'da kısa karşılıklar

**Python**

```python
class Order:
    def __init__(self):
        self._lines = []

    def add_line(self, product, quantity):
        if quantity <= 0:
            raise ValueError("Quantity must be positive")
        self._lines.append(OrderLine(product, quantity))
```

**Java**

```java
class Order {
    private final java.util.List<OrderLine> lines = new java.util.ArrayList<>();

    void addLine(String product, int quantity) {
        lines.add(new OrderLine(product, quantity));
    }
}
```

Burada `OrderLine` sınıflarının ilgili dilde tanımlandığı varsayılır. Amaç bütün kodu tekrar etmek değil, **parçayı kimin oluşturduğunu** görmektir.

### Aggregation ve composition nasıl ayırt edilir?

| Soru | Aggregation örneği | Composition örneği |
|---|---|---|
| Parça, bütünün dışında anlamlı mı? | Çoğunlukla evet (`Developer`) | Alan modelinde çoğunlukla hayır (`OrderLine`) |
| Parçanın sahibi kim? | Bütün özel sahibi olmayabilir | Bütün güçlü sahiplik taşır |
| Bütün yok olduğunda ne olur? | Parça bağımsız kalabilir | Parçanın o bütüne ait kimliği/yaşamı sona erer |
| Kodda özel anahtar sözcük var mı? | Hayır | Hayır |

**Kritik ayrım:** UML composition ile günlük konuşmada kullanılan **"composition over inheritance"** önerisi aynı terimi kullansa da birebir aynı şey değildir. İlki sahiplik ve bütün–parça ilişkisi; ikincisi davranışı nesne iş birliğiyle yeniden kullanma yaklaşımıdır.

---

## 5. Dependency: Bir sınıfın başka bir sınıfa ihtiyaç duyması

### Kavramsal açıklama

Bir sınıf, işini yapabilmek için başka bir tipin davranışına ihtiyaç duyuyorsa aralarında **dependency** vardır. Bu bağımlılık kısa süreli olabilir (metot parametresi), nesne boyunca saklanabilir (field) veya sınıfın içinde doğrudan oluşturulabilir (`new`).

Dependency'nin varlığı kaçınılmazdır. OOP tasarımında amaç **tüm bağımlılıkları yok etmek** değil; bağımlılıkların yönünü, türünü ve değişime etkisini yönetmektir.

### C# — Üç kullanım biçimi

```csharp
// 1) Metot parametresi üzerinden kısa süreli kullanım
public class ReportPrinter
{
    public void Print(IReportFormatter formatter)
    {
        Console.WriteLine(formatter.Format());
    }
}

// 2) Constructor üzerinden verilen ve saklanan bağımlılık
public class ReportService
{
    private readonly IReportRepository _repository;
    public ReportService(IReportRepository repository) => _repository = repository;
}

// 3) Somut bağımlılığın içeride doğrudan oluşturulması
public class HardCodedReportService
{
    private readonly SqlReportRepository _repository = new SqlReportRepository();
}
```

Bu parça, `IReportFormatter`, `IReportRepository` ve `SqlReportRepository` tiplerinin tanımlı olduğunu varsayan bir **tasarım karşılaştırmasıdır**. İlk yaklaşım kısa süreli etkileşimdir; ikinci yaklaşım **Constructor Injection** örneğidir; üçüncü yaklaşım ise somut implementasyona daha sıkı bağımlılık kurar.

### Python ve Java'da kısa karşılıklar

**Python**

```python
class ReportService:
    def __init__(self, repository):
        self.repository = repository  # Dışarıdan sağlanan bağımlılık
```

**Java**

```java
class ReportService {
    private final ReportRepository repository;

    ReportService(ReportRepository repository) {
        this.repository = repository;
    }
}
```

**Önemli:** Constructor parametresi kullanmak tek başına DIP'e tam uyum demek değildir. Soyutlama seçimi, bağımlılıkların yönü ve sorumluluk dağıtımı da değerlendirilmelidir.

---

## 6. Coupling: Değişikliklerin sınıflar arasında ne kadar yayıldığı

### Kavramsal açıklama

**Coupling**, yazılım bileşenlerinin birbirlerine ne ölçüde bağımlı olduğunu ifade eder. Yüksek coupling, bir bileşenin ayrıntılarındaki değişimin başka bileşenlerde daha fazla düzenleme gerektirmesi riskini artırabilir. Düşük coupling ise ilgili bileşenlerin birbirinden daha bağımsız değişebilmesini destekler.

Yalnızca bir sınıfın kaç başka sınıfı kullandığını saymak coupling'i bütünüyle açıklamaz. **Bağımlı olunan şeyin türü** önemlidir: Bir dış servisin somut HTTP istemcisine sıkı bağlanmakla sabit bir veri yapısına ihtiyaç duymak farklı riskler taşır.

### C# — Somut implementasyona sıkı bağımlılık

```csharp
public class OrderService
{
    private readonly SqlOrderRepository _repository = new SqlOrderRepository();

    public void Save(Order order)
    {
        _repository.Save(order);
    }
}
```

Burada `OrderService`, `SqlOrderRepository` sınıfını hem **biliyor** hem **oluşturuyor**. Başka depolama yaklaşımına geçmek veya testte sahte repository kullanmak istediğimizde `OrderService` üzerinde değişiklik yapmamız gerekebilir.

### C# — Soyutlama ve DI ile daha esnek tasarım

```csharp
public interface IOrderRepository
{
    void Save(Order order);
}

public class OrderService
{
    private readonly IOrderRepository _repository;

    public OrderService(IOrderRepository repository)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
    }

    public void Save(Order order)
    {
        _repository.Save(order);
    }
}
```

Artık `OrderService`, belirli SQL implementasyonuna değil `IOrderRepository` sözleşmesine bağımlı. Farklı implementasyonlar kullanılabilir. Ancak **soyutlama eklemek otomatik olarak iyi tasarım oluşturmaz**: Tek kullanımlık ve değişmesi beklenmeyen basit bir sınıfa gereksiz interface koymak da karmaşıklık yaratabilir.

### Python ve Java'da kısa karşılıklar

**Python — Duck typing yaklaşımı**

```python
class OrderService:
    def __init__(self, repository):
        self._repository = repository

    def save(self, order):
        self._repository.save(order)
```

Python'da bu basit örnekte `repository` için `save` metodunu sağlayan bir nesne yeterlidir. Daha açık bir sözleşme gerekiyorsa `typing.Protocol` kullanılabilir.

**Java — Interface üzerinden bağımlılık**

```java
interface OrderRepository {
    void save(Order order);
}

class OrderService {
    private final OrderRepository repository;

    OrderService(OrderRepository repository) {
        this.repository = repository;
    }
}
```

### Düşük coupling neden faydalıdır?

- Bir altyapı uygulamasını değiştirirken iş mantığının daha az etkilenmesini sağlayabilir.
- Testlerde gerçek veritabanı yerine kontrollü bir test uygulaması kullanılmasını kolaylaştırabilir.
- Sorumlulukları daha açık birimlere ayırarak bakım maliyetini azaltabilir.

**Sınır:** Coupling'in düşük olması tek başına yeterli değildir. Sistem gereksiz interface ve sınıflara bölünürse kodun anlaşılması zorlaşabilir.

---

## 7. Cohesion: Bir sınıfın sorumluluklarının birbiriyle ilişkisi

### Kavramsal açıklama

**Cohesion**, bir sınıfın veya modülün içindeki sorumlulukların ne ölçüde ortak ve anlamlı bir amaca hizmet ettiğidir. Yüksek cohesion'a sahip bir sınıf genellikle belirli bir iş sorumluluğu etrafında toplanır. Düşük cohesion ise birbiriyle ilgisiz görevlerin aynı sınıfta birikmesiyle görülebilir.

Örneğin kullanıcı parolasını doğrulamak, PDF raporu üretmek ve e-posta göndermek tek bir `UserManager` içinde toplanırsa farklı nedenlerle değişen davranışlar aynı sınıfa yığılır. Bu, **Single Responsibility Principle (SRP)** ile de bağlantılıdır; fakat cohesion ve SRP aynı kavram değildir.

### C# — Düşük cohesion örneği

```csharp
public class UserManager
{
    public bool ValidatePassword(string password) => password.Length >= 12;

    public string BuildInvoicePdf() => "PDF bytes would be generated here";

    public void SendWelcomeEmail(string address)
    {
        Console.WriteLine($"Email to {address}");
    }
}
```

Bu sınıf üç farklı iş alanına dair davranışı toplar. Gerçek projede parola politikası, fatura biçimi veya mesajlaşma altyapısı bağımsız değişebilir.

### C# — Sorumlulukları ayırma

```csharp
public class PasswordPolicy
{
    public bool IsValid(string password) => password.Length >= 12;
}

public class InvoiceGenerator
{
    public string Generate() => "PDF bytes would be generated here";
}

public class WelcomeNotifier
{
    public void Send(string email) => Console.WriteLine($"Email to {email}");
}
```

Bu örneklerin metotları özellikle sadeleştirilmiştir; gerçek parola güvenliği, PDF üretimi ya da e-posta gönderimi implementasyonları değildir. Tasarım fikri, **farklı değişim nedenlerini ayrı sorumluluklara ayırmaktır**.

### Python ve Java'da kısa karşılıklar

**Python**

```python
class PasswordPolicy:
    def is_valid(self, password: str) -> bool:
        return len(password) >= 12

class WelcomeNotifier:
    def send(self, email: str) -> None:
        print(f"Email to {email}")
```

**Java**

```java
class PasswordPolicy {
    boolean isValid(String password) {
        return password.length() >= 12;
    }
}
```

**Önemli:** Çok fazla küçük sınıf oluşturmak her zaman yüksek cohesion anlamına gelmez. Bir sınıfın sınırını, sistemin gerçek sorumluluklarına göre seçmeliyiz.

### Coupling ve cohesion birlikte düşünülmeli

| | Coupling | Cohesion |
|---|---|---|
| Temel soru | Sınıf dışarıya ne kadar bağımlı? | Sınıfın iç görevleri ne kadar ilişkili? |
| Genellikle istenen | Gereksiz/sıkı bağımlılıkların azaltılması | Sorumlulukların anlamlı biçimde bütünleşmesi |
| Tipik sorun | Veri katmanı ayrıntısının iş mantığına yayılması | E-posta, raporlama ve kimlik doğrulamanın tek sınıfta birikmesi |
| İyileştirme aracı | Soyutlama, DI, sınırların düzenlenmesi | Sorumlulukların ayrılması, uygun sınıf tasarımı |



---

## 8. Inheritance mı, Composition mı? (`is-a` / `has-a`)

### Kavramsal açıklama

**Inheritance**, alt türün üst türün sözleşmesini devraldığı bir **is-a** ilişkisidir: `Dog` bir `Animal` olabilir. **Composition ile davranış kurma** ise bir nesnenin başka nesnelerin yeteneklerini kullanmasıdır: `ReportService`, bir `IReportFormatter` kullanır; kendisi bir formatter türü olmak zorunda değildir.

Inheritance bazen doğru modellemedir, özellikle gerçek bir alt tür ilişkisi ve substitutability varsa. Fakat yalnızca kod tekrarını azaltmak için kalıtım kullanmak, ilgisiz sınıfları aynı hiyerarşiye bağlayabilir. Composition çoğu zaman davranışı değiştirilebilir bileşenler hâlinde kurmaya izin verir.

### C# — Değiştirilebilir davranış için composition

```csharp
public interface IDiscountPolicy
{
    decimal Apply(decimal amount);
}

public class NoDiscount : IDiscountPolicy
{
    public decimal Apply(decimal amount) => amount;
}

public class TenPercentDiscount : IDiscountPolicy
{
    public decimal Apply(decimal amount) => amount * 0.90m;
}

public class CheckoutService
{
    private readonly IDiscountPolicy _policy;

    public CheckoutService(IDiscountPolicy policy) => _policy = policy;

    public decimal Total(decimal amount) => _policy.Apply(amount);
}

var checkout = new CheckoutService(new TenPercentDiscount());
Console.WriteLine(checkout.Total(100m)); // 90.0
```

**Neden composition?** `CheckoutService`, farklı indirim davranışlarını kendi alt sınıflarını üretmeden kullanabilir. `IDiscountPolicy` üzerinden iş birliği, yeni politikalar eklerken mevcut servis sınıfını daha az değiştirmemizi sağlayabilir.

### Python ve Java'da kısa karşılıklar

**Python**

```python
class CheckoutService:
    def __init__(self, policy):
        self.policy = policy

    def total(self, amount):
        return self.policy.apply(amount)
```

**Java**

```java
interface DiscountPolicy {
    double apply(double amount);
}

class CheckoutService {
    private final DiscountPolicy policy;
    CheckoutService(DiscountPolicy policy) { this.policy = policy; }
    double total(double amount) { return policy.apply(amount); }
}
```

Finansal uygulamalarda Java'da `double` yerine `BigDecimal` seçmek çoğunlukla daha uygundur; burada karşılaştırma yalnızca tasarım ilişkisini göstermek içindir.

**İnce ayrım:** Yukarıdaki `CheckoutService` ile `IDiscountPolicy` arasında nesne iş birliği/composition temelli bir tasarım vardır. Fakat UML'nin güçlü sahiplik anlamındaki composition ilişkisini çıkarmak için yalnızca constructor field'ı yetmez. `policy` dışarıdan sağlandığı ve paylaşılabildiği için UML açısından association olarak modellenmesi de uygundur.

---

## 9. Delegation: Bir davranışı başka nesneye devretmek

**Delegation**, bir nesnenin bir işlemin belirli kısmını başka bir nesneye yaptırmasıdır. Bir sınıfın tüm işi kendisinin üstlenmesi gerekmez; uygun uzman nesneden yardım alabilir. Örneğin `CheckoutService` toplam tutarın indirim hesabını `IDiscountPolicy` nesnesine devreder.

```csharp
public decimal Total(decimal amount)
{
    return _policy.Apply(amount); // Hesap davranışı policy nesnesine devrediliyor.
}
```

Delegation, composition tabanlı iş birliğinin önemli araçlarındandır. Ancak işin anlaşılmasını gereksiz yere çok sayıda küçük metoda dağıtmak da karmaşıklık yaratabilir.

---

## 10. C# / Python / Java tasarım karşılaştırması

| Konu | C# | Python | Java |
|---|---|---|---|
| Nesne referansı | Field/property | Attribute | Field |
| Interface sözleşmesi | `interface` | `Protocol` veya `ABC`; duck typing | `interface` |
| Constructor üzerinden bağımlılık | `public Service(IFoo foo)` | `def __init__(self, foo)` | `Service(Foo foo)` |
| Somut nesne oluşturma | `new Foo()` | `Foo()` | `new Foo()` |
| Liste ilişkisi | `List<T>` | `list` | `List<T>` |
| Yerleşik DI container | .NET ekosisteminde `Microsoft.Extensions.DependencyInjection` yaygın | Framework/kütüphane seçimine bağlı | Spring gibi framework'lerle yaygın |
| Aggregation / composition anahtar sözcüğü | Yok | Yok | Yok |

**Dikkat:** Java'daki Spring veya C#'taki .NET DI container bir ilişki türü değildir; bağımlılıkların yapılandırılması ve sağlanmasında kullanılan araçlardır. DI, container olmadan da uygulanabilir. Python'da da constructor ile bağımlılık sağlamak için özel bir DI kütüphanesi zorunlu değildir.

---



##  Kısa tekrar ve terim sözlüğü

| Terim | Hatırlanması gereken fikir |
|---|---|
| Association | Nesneler arasında anlamlı bağlantı |
| Aggregation | Bütün–parça; parça bağımsız var olabilir |
| Composition (UML) | Bütün–parça; güçlü sahiplik/yaşam döngüsü bağı |
| Dependency | Bir bileşenin işini yapmak için diğerine ihtiyaç duyması |
| Coupling | Bileşenler arasındaki bağımlılığın derecesi ve niteliği |
| Cohesion | Bileşenin kendi iç sorumluluklarının ilişkili olması |
| Inheritance | Alt türün üst tür sözleşmesine dahil olması (`is-a`) |
| Composition over inheritance | Yeniden kullanım için her zaman kalıtım yerine iş birliğinin de değerlendirilmesi |
| Delegation | Davranışın uygun başka nesneye devredilmesi |
| DI | Gerekli bağımlılığın nesneye dışarıdan sağlanması |

---

## Kaynaklar ve akademik notlandırma


- Microsoft Learn — [Object-oriented programming (C#)](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/oop) ve [Dependency injection in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection).
- Python Documentation — [Classes](https://docs.python.org/3/tutorial/classes.html) ve [typing.Protocol](https://docs.python.org/3/library/typing.html#typing.Protocol).
- Oracle Java Tutorials — [Object-Oriented Programming Concepts](https://docs.oracle.com/javase/tutorial/java/concepts/).
- Laigner, R., Mendonça, D., Garcia, A., & Kalinowski, M. (2022). *Cataloging dependency injection anti-patterns in software systems*. **Journal of Systems and Software, 184**, 111125. https://doi.org/10.1016/j.jss.2021.111125.

