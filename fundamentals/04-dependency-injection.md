# 04 — Dependency Injection: Bağımlılıkların Yönetimi ve DI Container

## 1. Dependency (bağımlılık) nedir?

Bir nesne görevini yerine getirmek için başka bir nesnenin sunduğu davranışa ihtiyaç duyuyorsa aralarında bir **dependency** vardır. Örneğin siparişi tamamlayan bir `OrderService`, ödeme almak için `PaymentGateway` adlı bir nesneyle çalışabilir. Bu ilişki tek başına kötü değildir: Nesne yönelimli sistemler zaten iş birliği yapan nesnelerden oluşur. Asıl tasarım sorusu, **bağımlılığın hangi düzeyde tanımlandığı, kimin oluşturduğu ve değiştirilmesinin ne kadar zor olduğudur**.

**C# — Bağımlılığın sınıfın içinde oluşturulması:**

```csharp
public sealed class EmailSender
{
    public void Send(string message) => Console.WriteLine(message);
}

public sealed class NotificationService
{
    private readonly EmailSender _sender = new();

    public void Notify(string message) => _sender.Send(message);
}
```

`NotificationService`, doğrudan `EmailSender`'ın somut uygulamasını oluşturuyor. Başka bir mesajlaşma kanalı eklemek veya testte gerçek gönderici yerine sahte bir uygulama kullanmak için bu sınıfı değiştirmek gerekebilir. Bu, **yüksek coupling** yaratma eğilimindeki tasarımlardan biridir. Ancak her `new` ifadesinin kötü tasarım olduğu sonucu çıkmaz: Basit ve değişmeyen değer nesnelerini oluşturmak için DI kullanmak çoğu zaman anlamsızdır.

### Terim: Coupling ve cohesion

- **Coupling:** Modüllerin veya sınıfların birbirini ne ölçüde tanıdığı ve birindeki değişikliğin diğerini ne kadar etkilediği.
- **Cohesion:** Bir sınıfın üstlendiği sorumlulukların ortak bir amaca ne ölçüde hizmet ettiği.

DI özellikle bağımlılığın **edinilme biçimi** üzerinde kontrol sağlar. Tek başına tüm coupling türlerini çözmez ve yüksek cohesion garantilemez.

---

## 2. Dependency Injection tam olarak nedir?

**Dependency Injection (DI)**, bir nesnenin ihtiyaç duyduğu bağımlılıkların nesnenin kendi içinde oluşturulması veya global bir yerden aranması yerine **dışarıdan verilmesi** yaklaşımıdır. Bu dışarıdaki taraf bir üst seviye uygulama kodu, fabrika veya DI container olabilir. DI uygulamak için framework kullanmak zorunlu değildir.

**C# — Constructor Injection ile daha esnek tasarım:**

```csharp
public interface IMessageSender
{
    void Send(string message);
}

public sealed class EmailSender : IMessageSender
{
    public void Send(string message) => Console.WriteLine($"Email: {message}");
}

public sealed class NotificationService
{
    private readonly IMessageSender _sender;

    public NotificationService(IMessageSender sender)
    {
        _sender = sender ?? throw new ArgumentNullException(nameof(sender));
    }

    public void Notify(string message) => _sender.Send(message);
}

// Uygulamanın nesneleri birleştirdiği yer:
var service = new NotificationService(new EmailSender());
service.Notify("Sipariş tamamlandı");
```

Burada `NotificationService`, mesajın nasıl gönderildiğini bilmek zorunda değildir; `IMessageSender` sözleşmesini kullanır. Gerçek gönderici dışarıda oluşturulup constructor'a verilir. **Bağımlılığı dışarıdan vermek (DI)** ile **soyutlamaya bağımlı olmak (DIP ile uyumlu tasarım)** burada birlikte kullanılıyor. Bunlar birbirinin yerine geçen kavramlar değildir.

### Python — Aynı fikir

```python
class NotificationService:
    def __init__(self, sender):
        self.sender = sender

    def notify(self, message):
        self.sender.send(message)

service = NotificationService(EmailSender())  # EmailSender tanımlı varsayılıyor
```

Python'da dinamik tip sistemi sayesinde basit bir DI örneğinde resmi interface tanımlamak zorunlu değildir. Daha açık bir sözleşme istenirse `typing.Protocol` veya `abc.ABC` kullanılabilir.

### Java — Aynı fikir

```java
interface MessageSender {
    void send(String message);
}

class NotificationService {
    private final MessageSender sender;

    NotificationService(MessageSender sender) {
        this.sender = java.util.Objects.requireNonNull(sender);
    }

    void notifyUser(String message) { sender.send(message); }
}
```

Java da C# gibi bir interface aracılığıyla somut uygulamayı çağıran koddan ayırabilir. Makale Java sistemlerini incelediği için bu constructor kalıbını tanımak özellikle önemlidir.

---

## 3. DI, DIP ve IoC arasındaki fark

Birbirine çok yakın oldukları için bu üç terim sık karıştırılır:

| Kavram | Türü | Temel soru |
|---|---|---|
| **DIP — Dependency Inversion Principle** | Tasarım prensibi | Üst düzey iş kuralları hangi soyutlamalara bağımlı olmalı? |
| **DI — Dependency Injection** | Bağımlılık sağlama tekniği | Nesnenin ihtiyaç duyduğu işbirlikçiler ona nasıl verilecek? |
| **IoC — Inversion of Control** | Daha genel kontrol akışı yaklaşımı | Nesnelerin oluşturulmasını veya uygulama akışını kim yönetecek? |

**Önemli ayrım:** DI, bir somut sınıfın da constructor'a dışarıdan verilmesiyle uygulanabilir. Bu durumda **DI vardır**, fakat DIP'i destekleyecek anlamlı bir soyutlama kurulmuş olmayabilir. Benzer şekilde yalnızca interface oluşturmak DI yapıldığı anlamına gelmez.

Bir framework uygulama açılırken nesneleri oluşturup gerekli bağımlılıkları kendisi sağlıyorsa kontrolün bir kısmı uygulama kodundan framework'e geçmiştir; bu, IoC yaklaşımının bir örneğidir. Ancak IoC sadece DI'dan ibaret değildir; callback, event-driven framework ve template method gibi yapılarda da görülebilir.

**Sunumda söyleyebileceğim ayrım:** “DIP bana bağımlılığın hangi yöne kurulması gerektiği hakkında ilke verir; DI bağımlılığı dışarıdan sağlamanın yoludur; IoC ise kontrolün uygulama kodundan daha üst bir mekanizmaya devredilmesidir.”

---

## 4. DI yöntemleri

### 4.1. Constructor Injection

Gerekli bağımlılıklar nesne oluşturulurken constructor aracılığıyla verilir. **Zorunlu bağımlılıklar** için genellikle ilk tercihtir; nesne eksik bağımlılıkla kurulmaz ve alanlar `readonly` tutulabilir.

```csharp
public sealed class ReportService
{
    private readonly IReportRepository _repository;

    public ReportService(IReportRepository repository)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
    }
}
```

**Artısı:** Bağımlılıklar açıkça görülebilir; nesne oluşturulduğunda temel gereksinimler karşılanmış olur. **Dikkat:** Çok sayıda constructor parametresi, sınıfın aşırı sorumluluk taşıdığının belirtisi olabilir; çözüm her zaman parametreleri gizlemek değildir.

### 4.2. Property Injection

Bağımlılık, nesne oluşturulduktan sonra bir property aracılığıyla atanır.

```csharp
public sealed class ExportService
{
    public ILogger? Logger { get; set; }

    public void Export()
    {
        Logger?.LogInformation("Export başladı");
    }
}
```

Bu örnekte `Logger` isteğe bağlıdır; verilmemesi nesneyi çalışamaz hâle getirmemektedir. Zorunlu bir bağımlılığı set edilebilir property'de tutmak ise nesnenin eksik veya geçersiz durumda kullanılmasına yol açabilir. **.NET'in yerleşik DI container'ı genel amaçlı property injection'ı otomatik yapmaz**; burada yalnızca teknik gösterilmektedir.

### 4.3. Method Injection

Bağımlılık nesneye kalıcı olarak verilmez; onu kullanan metoda parametre olarak aktarılır.

```csharp
public sealed class FileImporter
{
    public void Import(Stream input, IImportValidator validator)
    {
        validator.Validate(input);
        // Doğrulama sonrası içe aktarma işlemi...
    }
}
```

Belirli bir işleme özgü ve her çağrıda değişebilen yardımcılar için uygundur. Her metot parametresi “DI container injection” anlamına gelmez; burada kavramsal bir method injection örneği vardır.

### Üç yaklaşımın karşılaştırması

| Yöntem | Bağımlılık ne zaman verilir? | Uygun kullanım | Olası risk |
|---|---|---|---|
| Constructor | Nesne oluşturulurken | Zorunlu işbirlikçiler | Fazla parametre, karmaşık sınıf |
| Property | Oluşturulduktan sonra | Gerçekten isteğe bağlı özellik | Eksik/sonradan değişen durum |
| Method | Metot çağrısında | İşleme özgü bağımlılık | İhtiyacın çok sayıda metoda taşınması |

**Java notu:** Spring uygulamalarında constructor injection yaygın olarak önerilir; field injection da mümkündür ancak bağımlılıkları gizleyebilir ve sınıfı container olmadan test etmeyi zorlaştırabilir. **Python notu:** Constructor ve method parametreleri aynı DI ilkesini doğal biçimde destekler.

---

## 5. Composition Root: Nesneler nerede oluşturulmalı?

**Composition Root**, uygulamanın çalışması için gerekli nesne grafiğinin bir araya getirildiği üst düzey noktadır. Burada somut implementasyonlar seçilir ve bileşenler birbirine bağlanır. Sınıfların her birinin kendi bağımlılıklarını rastgele üretmesi yerine oluşturma kararlarının yönetilebilir bir yerde toplanması amaçlanır.

**C# — Container olmadan (manual DI):**

```csharp
IMessageSender sender = new EmailSender();
var notificationService = new NotificationService(sender);
notificationService.Notify("Merhaba");
```

Bu da geçerli DI'dır. Küçük uygulamalarda container eklemek yerine böyle açık bir nesne birleştirme kodu kullanmak daha anlaşılır olabilir.

### DI Container nedir?

**DI Container**, hangi soyutlamanın hangi somut tiple eşleştiğini, nesnelerin nasıl oluşturulacağını ve hangi yaşam döngüsüyle tutulacağını yöneten araçtır. Container, DI'ı uygulamayı kolaylaştırabilir; fakat DI'ın kendisi değildir.

.NET'te `Microsoft.Extensions.DependencyInjection`, Java ekosisteminde Spring, Python'da FastAPI'nin dependency sistemi gibi araçlar, farklı özellik ve kapsamlarla bu problem alanına çözüm sunar. **Bunların kullanım modeli birebir aynı değildir.**

---

## 6. C#/.NET ile gerçek DI Container kullanımı

Aşağıdaki örnekte `Microsoft.Extensions.DependencyInjection` ve önceki `IMessageSender`, `EmailSender`, `NotificationService` tipleri kullanılır.

```csharp
using Microsoft.Extensions.DependencyInjection;

var services = new ServiceCollection();
services.AddTransient<IMessageSender, EmailSender>();
services.AddTransient<NotificationService>();

using var provider = services.BuildServiceProvider();

var notification = provider.GetRequiredService<NotificationService>();
notification.Notify("Container aracılığıyla oluşturuldu");
```

**Çalışma sırası:** Önce kayıtlar yapılır (`AddTransient`). Ardından provider oluşturulur. `NotificationService` istendiğinde container, constructor'daki `IMessageSender` gereksinimini görür; kayıtlı `EmailSender` tipini oluşturur ve servise sağlar.

Bu örnek **.NET'in yerleşik container'ının nasıl kullanıldığını** gösterir. ASP.NET Core uygulamalarında aynı kayıtlar genellikle uygulamanın `builder.Services` koleksiyonuna eklenir:

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddScoped<IOrderRepository, EfOrderRepository>();
builder.Services.AddScoped<OrderService>();
var app = builder.Build();
```

Burada `IOrderRepository`, `EfOrderRepository` ve `OrderService` uygulamada tanımlanmış tipleri temsil eder. Bu parça tek başına çalıştırılabilir örnek değil, **servis kayıt kalıbıdır**.

### Registration, resolution, object graph

- **Registration:** Arayüz ile implementasyonun veya oluşturma fonksiyonunun container'a tanıtılması.
- **Resolution:** Bir bileşen istendiğinde gerekli bağımlılıkların bulunup oluşturulması.
- **Object graph:** Bir nesnenin bağımlılıkları ile onların bağımlılıklarından oluşan ağ.

Bir container, yanlış tasarlanmış sınıfları otomatik olarak iyi tasarıma dönüştürmez. Kayıt edilen nesne grafiğinin anlaşılır ve tutarlı kalması geliştiricinin sorumluluğudur.

---

## 7. Service Lifetime: Transient, Scoped ve Singleton

Bir DI container'ın yalnızca **neyi** oluşturduğunu değil **ne kadar süre** yaşatacağını da yönetmesi gerekir. Özellikle web uygulamalarında yanlış lifetime seçimi durum paylaşımına, kaynakların gereğinden uzun tutulmasına ve beklenmedik hatalara yol açabilir.

| Lifetime | Genel davranış | Tipik örnek |
|---|---|---|
| `Transient` | Container tarafından her çözümlemede yeni örnek oluşturulur | Hafif, durumsuz işlem servisi |
| `Scoped` | Bir scope boyunca aynı örnek kullanılır | Web isteği kapsamındaki veri erişimi servisi |
| `Singleton` | Container ömrü boyunca aynı örnek kullanılır | Paylaşılan, thread-safe olması gereken servis |

**Not:** ASP.NET Core'da scoped yaşam döngüsü çoğunlukla **bir HTTP isteği** boyunca geçerlidir. Ancak scope kavramı yalnızca HTTP'ye bağlı değildir; konsol/arka plan uygulamalarında da açıkça scope oluşturulabilir.

**C# kayıt örnekleri:**

```csharp
services.AddTransient<IFormatter, TextFormatter>();
services.AddScoped<IUnitOfWork, UnitOfWork>();
services.AddSingleton<IClock, SystemClock>();
```

Tipler örnek amaçlıdır; uygun lifetime, servisin duruma ve bağımlılıklara sahip olma biçimine göre seçilmelidir.

### Kritik hata: Captive Dependency

Bir singleton servisin constructor üzerinden scoped servis alması genellikle tehlikelidir:

```csharp
services.AddScoped<IRequestContext, RequestContext>();
services.AddSingleton<AuditService>();

public sealed class AuditService
{
    public AuditService(IRequestContext context) { }
}
```

`AuditService` tek örnek olarak tutulurken `IRequestContext` normalde daha kısa bir scope'a aittir. Bu durumda **lifetime mismatch** oluşur. Scope validation açıksa .NET container bu sorunu yakalayabilir; validation ayarları kapalıysa beklenmedik yaşam döngüsü davranışları oluşabilir.

Çözümler bağlama bağlıdır: `AuditService` da scoped olabilir veya uzun ömürlü servis, ihtiyaç anında uygun bir scope oluşturma mekanizması kullanabilir. **Yalnızca sorunu susturmak için servislerin hepsini singleton yapmak doğru değildir.**

### Singleton ve thread safety

Singleton, aynı nesnenin eşzamanlı istekler tarafından kullanılabilmesi anlamına gelebilir. İçerisinde değişebilir durum varsa thread safety düşünülmelidir. DI container'ın nesneyi singleton olarak oluşturması, nesnenin otomatik olarak thread-safe olduğu anlamına gelmez.

### IDisposable ve kaynak yönetimi

Container tarafından oluşturulan `IDisposable`/`IAsyncDisposable` nesnelerin yaşam döngüsü ayrıca önemlidir. Gereğinden uzun yaşayan nesneler bağlantı veya bellek gibi kaynakları gereğinden uzun tutabilir. Bir container'dan alınan nesneyi kimin dispose edeceği, nesnenin container tarafından mı yoksa uygulama tarafından mı oluşturulduğuna bağlıdır.

---

## 8. DI kullanımında yaygın tasarım sorunları

Aşağıdaki örnekler **genel mühendislik açıklamalarıdır**. Bunları Laigner vd. makalesindeki katalog öğelerinin birebir tanımları veya makaleden alıntılar olarak düşünmemeliyim.

### 8.1. Service Locator: Bağımlılığı saklamak

```csharp
public sealed class OrderService
{
    private readonly IServiceProvider _provider;

    public OrderService(IServiceProvider provider) => _provider = provider;

    public void PlaceOrder()
    {
        var gateway = _provider.GetRequiredService<IPaymentGateway>();
        gateway.Pay();
    }
}
```

Bu tasarımda `OrderService`'in gerçek ihtiyaçları constructor imzasından açıkça anlaşılmıyor. Bağımlılığı dışarıdan **almak** yerine kodun içinde container'dan **aramak**, özellikle uygulama iş kurallarında Service Locator tarzı bir kullanıma dönüşebilir. Daha açık alternatif:

```csharp
public sealed class OrderService
{
    private readonly IPaymentGateway _gateway;

    public OrderService(IPaymentGateway gateway) => _gateway = gateway;

    public void PlaceOrder() => _gateway.Pay();
}
```

`IServiceProvider` kullanımının tamamı yanlış değildir: Composition Root, özel factory senaryoları veya framework entegrasyon sınırlarında kontrollü biçimde kullanılabilir. Sorun **bağımlılıkların gizlenmesi ve oluşturma sorumluluğunun iş mantığına yayılmasıdır**.

### 8.2. Gereksiz interface ve aşırı soyutlama

Her sınıf için otomatik olarak `IFoo` interface'i eklemek iyi tasarım garantilemez. Alternatif implementasyon, test sınırı veya modülerlik ihtiyacı yoksa bu yaklaşım gereksiz dosya ve kavram maliyeti oluşturabilir. DI, **anlamlı bağımlılık sınırlarını** yönetmek için kullanılmalı; sırf container'a kayıt yapabilmek amacıyla uygulanmamalıdır.

### 8.3. Çok sayıda constructor bağımlılığı

```csharp
public CheckoutService(
    ICart cart,
    IPaymentGateway payment,
    IInvoiceWriter invoice,
    IEmailSender email,
    IDiscountPolicy discount,
    IAuditLogger audit,
    IInventoryRepository inventory)
{ /* ... */ }
```

Uzun constructor parametre listesi kendi başına kesin hata değildir; ancak çok farklı sorumlulukların tek sınıfa yüklendiğine işaret edebilir. SRP ve cohesion açısından sınıfı yeniden değerlendirmek gerekir.

### 8.4. Circular Dependency

```text
OrderService  ──needs──> PaymentService
PaymentService ──needs──> OrderService
```

İki servisin birbirini constructor'da talep etmesi, nesne grafiğinde döngü oluşturur. .NET'in varsayılan container'ı bu tür constructor döngülerini çözümleme sırasında hata olarak bildirebilir. Çoğu durumda yapılması gereken döngüyü hileyle aşmak değil, **sorumlulukları ve işbirliği yönünü yeniden düşünmektir**.

### 8.5. Alanlara doğrudan erişim ve gizli global durum

`static` olarak tutulan değişebilir servisler veya her yerden erişilen global container, bağımlılıkları görünmez hâle getirebilir ve testlerin birbirini etkilemesine yol açabilir. `static` kullanımının tamamı kötü değildir; burada risk, **kontrol edilmeyen paylaşılan durum** ve açık olmayan nesne ilişkileridir.

### 8.6. Container'ı aşırı kullanmak

DI container'ın varlığı her nesnenin container tarafından oluşturulmasını gerektirmez. Basit DTO, immutable value object veya geçici hesaplama nesneleri çoğunlukla normal `new` ile oluşturulabilir. Tasarım kararını verebilmek için bağımlılığın **değişebilir bir işbirlikçi** mi yoksa **sıradan bir veri/değer** mi olduğunu ayırt etmek gerekir.

---

## 9. C#, Python ve Java: Framework karşılaştırması

Aynı temel DI fikri farklı ekosistemlerde farklı araçlarla kullanılır. Aşağıdaki parçalar temel kalıbı göstermeyi amaçlar; tam bir uygulamanın bütün import, yapılandırma ve veri tipi tanımlarını içermez.

### 9.1. C# — ASP.NET Core

```csharp
builder.Services.AddScoped<IUserRepository, SqlUserRepository>();
builder.Services.AddScoped<UserService>();

public sealed class UserService
{
    private readonly IUserRepository _users;

    public UserService(IUserRepository users) => _users = users;
}
```

Container, kayıtlar ve constructor parametreleri üzerinden nesne grafiğini kurar. Kullanıcı kodunun `UserService` içerisinde `SqlUserRepository` üretmesi gerekmez.

### 9.2. Python — FastAPI

```python
from fastapi import Depends, FastAPI

app = FastAPI()

class UserRepository:
    def list_users(self):
        return ["Ada", "Lin"]

def get_repository():
    return UserRepository()

@app.get("/users")
def read_users(repo: UserRepository = Depends(get_repository)):
    return repo.list_users()
```

`Depends`, endpoint için bağımlılığı çözümlemesini FastAPI'ye bildirir. Bu örnekte kullanılan mekanizma **istek/endpoint bağımlılığı çözümleme sistemi**dir; .NET'in `AddSingleton`/`AddScoped` kayıtlarıyla birebir aynı lifetime modeli değildir. FastAPI'nin request-level caching, `yield` ile cleanup ve dependency override gibi ayrı özellikleri vardır.

### 9.3. Java — Spring

```java
public interface UserRepository {
    java.util.List<String> listUsers();
}

@org.springframework.stereotype.Service
public class UserService {
    private final UserRepository users;

    public UserService(UserRepository users) {
        this.users = users;
    }
}
```

Spring, `UserService`'i bir bean olarak yönetebilir. Bu kodun çalışabilmesi için `UserRepository` arayüzünü uygulayan uygun bir bean'in de tanımlanmış olması gerekir. Tek constructor varsa Spring çoğu modern uygulamada ayrıca `@Autowired` yazılmadan injection yapabilir.

### Karşılaştırma

| Özellik | C# / .NET | Python / FastAPI | Java / Spring |
|---|---|---|---|
| Ana yaklaşım | Servis kaydı ve constructor injection | `Depends` üzerinden dependency çözümleme; normal Python DI da mümkün | Bean container ve constructor injection |
| Soyutlama | `interface`, `abstract class` | `Protocol`, `ABC` veya duck typing | `interface`, `abstract class` |
| Yaşam döngüsü | Transient, Scoped, Singleton | Dependency caching / request çözümleme, bağımlılığa özel kaynak yönetimi | Singleton, prototype, request vb. bean scope'ları |
| Sık görülen risk | Lifetime mismatch, Service Locator | Belirsiz bağımlılık sözleşmesi, yanlış scope/cache beklentileri | Bean döngüleri, field injection, scope uyumsuzlukları |

**Sonuç:** Dil ve framework farklı olsa bile önemli OOP sorusu aynıdır: *Bir bileşenin hangi işbirlikçiye neden ihtiyaç duyduğunu açıkça gösterebiliyor muyum ve bu ilişkileri uygun yerde yönetebiliyor muyum?*

---

## 10. Test edilebilirlik ve DI

DI'ın önemli pratik yararlarından biri, testte gerçek altyapı nesnesi yerine kontrollü bir test nesnesi verebilmemizdir.

**C# — Test double ile örnek:**

```csharp
public sealed class FakeMessageSender : IMessageSender
{
    public string? LastMessage { get; private set; }

    public void Send(string message) => LastMessage = message;
}

var fake = new FakeMessageSender();
var service = new NotificationService(fake);
service.Notify("Test mesajı");

if (fake.LastMessage != "Test mesajı")
    throw new Exception("Beklenen mesaj gönderilmedi.");
```

Burada gerçek e-posta sunucusuna ihtiyaç yoktur. Yine de **test edilebilirlik tek başına doğru mimari kanıtı değildir**: Uygun olmayan bir abstraction da mock'lanabilir. Testler tasarımın davranışını denetler; mimari kararların gerekçesini ayrıca değerlendirmek gerekir.

---


## Terimler sözlüğü

| Terim | Açıklama |
|---|---|
| Dependency | Bir bileşenin görevini yapmak için ihtiyaç duyduğu işbirlikçi veya hizmet |
| Injection | Bağımlılığın nesneye dışarıdan sağlanması |
| DIP | Modüllerin soyutlamalar üzerinden ilişki kurmasını destekleyen SOLID prensibi |
| IoC | Kontrolün veya oluşturma sorumluluğunun daha üst bir mekanizmaya devredilmesi |
| Composition Root | Nesne grafiğinin bir araya getirildiği uygulama sınırı |
| DI Container | Bağımlılık oluşturma ve yaşam döngüsü yönetimini kolaylaştıran araç |
| Registration | Soyutlama/implementasyon eşleşmesinin tanımlanması |
| Resolution | İstenen servisin ve bağımlılıklarının elde edilmesi |
| Object Graph | Bir nesne ve ilişkili bağımlılıklar ağı |
| Transient | Genellikle çözümleme başına yeni servis örneği |
| Scoped | Belirli scope boyunca paylaşılan servis örneği |
| Singleton | Container ömrü boyunca paylaşılan servis örneği |
| Captive Dependency | Uzun ömürlü servisin daha kısa ömürlü bağımlılığı uygunsuz biçimde tutması |
| Service Locator | Kodun bağımlılıklarını gizli bir merkezden arayarak edinmesi kalıbı |
| Circular Dependency | Bağımlılık grafiğinde bir döngü bulunması |
| Test Double | Gerçek işbirlikçinin yerine testte kullanılan fake, stub veya mock vb. nesne |

---

## Kaynaklar ve okuma önerileri


- Microsoft Learn — Dependency injection in .NET: https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection
- Microsoft Learn — Service lifetimes: https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#service-lifetimes
- Microsoft Learn — Dependency injection guidelines: https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines
- FastAPI documentation — Dependencies: https://fastapi.tiangolo.com/tutorial/dependencies/
- Spring Framework Reference — The IoC Container: https://docs.spring.io/spring-framework/reference/core/beans.html


