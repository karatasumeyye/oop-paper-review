# 03 — SOLID ve GRASP: Sorumluluk Dağıtımı ve Esnek Nesne Tasarımı


## 1. Tasarım prensibi nedir, neden gerekir?

Çalışan kod her zaman değişime hazır kod demek değildir. Örneğin bir sipariş sistemine yeni bir ödeme sağlayıcısı eklemek için ödeme, sipariş, fatura ve bildirim sınıflarının hepsini değiştirmek gerekiyorsa sorumluluklar gereğinden fazla iç içe geçmiş olabilir. Tasarım prensipleri bu tür kararları değerlendirmek için **rehber** sunar; derleyicinin otomatik doğruladığı yasalar değildir. Küçük bir projede ek bir interface gereksiz soyutlama yaratabilirken, aynı soyutlama değişen bir entegrasyon sınırında önemli olabilir.

**SOLID**, sınıf ve modül tasarımına yönelik beş ilkeyi ifade eder. **GRASP (General Responsibility Assignment Software Patterns)** ise nesnelere hangi sorumlulukların verilmesinin uygun olacağını düşünmeye yardımcı olan örüntüler bütünüdür. SOLID daha çok bağımlılık, arayüz ve değişim ilkeleriyle; GRASP ise sorumluluğun *hangi nesnede* bulunması gerektiğiyle ilgilenir. Aralarında önemli örtüşmeler vardır.

> **Terim:** *Responsibility* bir nesnenin **bilmesi** gereken bilgi veya **yapması** gereken iş olarak düşünülebilir. Her sınıfın yalnızca bir metodu olması gerektiği anlamına gelmez.

---

## 2. SOLID — Single Responsibility Principle (SRP)

### Anlamı ve gerekçesi

SRP'nin güçlü yorumu, bir modülün veya sınıfın **değişmesi için tek bir temel nedeni** olması gerektiğidir. Buradaki “neden”, çoğu zaman farklı iş gereksinimleri veya farklı aktörlerin talepleridir. “Bir sınıf tek işlem yapmalıdır” şeklinde yorumlamak fazla yüzeyseldir. Bir raporun hesaplanması ve dosyaya yazılması farklı nedenlerle değişebilir; bu işleri ayırmak değişikliklerin etkisini azaltabilir.

**C# — Sorumlulukları iç içe geçiren örnek:**

```csharp
public class InvoiceService
{
    public decimal CalculateTotal(decimal price, int quantity)
        => price * quantity;

    public void SaveToFile(string content)
        => File.WriteAllText("invoice.txt", content);
}
```

Bu sınıf hem tutar hesaplama kuralına hem dosya sistemine ilişkin kararlara bağlıdır. Vergi hesabındaki değişiklik ve kayıt biçimindeki değişiklik aynı sınıfı etkiler.

**C# — Sorumlulukları ayıran örnek:**

```csharp
public class InvoiceCalculator
{
    public decimal Total(decimal price, int quantity) => price * quantity;
}

public class InvoiceFileWriter
{
    public void Save(string content)
        => File.WriteAllText("invoice.txt", content);
}
```

Bu ayrımın değeri, ayrı ayrı değişen davranışların bağımsız geliştirilebilmesidir. Ancak her iki işlemi tek yerde tutmanın gerçek bir değişiklik maliyeti doğurmadığı çok küçük örneklerde ayrıştırma zorunlu değildir.

**Python — Aynı ayrım:**

```python
class InvoiceCalculator:
    def total(self, price, quantity):
        return price * quantity

class InvoiceFileWriter:
    def save(self, content):
        with open("invoice.txt", "w", encoding="utf-8") as f:
            f.write(content)
```

**Java — Aynı ayrım:**

```java
class InvoiceCalculator {
    double total(double price, int quantity) { return price * quantity; }
}
class InvoiceFileWriter {
    void save(String content) throws java.io.IOException {
        java.nio.file.Files.writeString(
            java.nio.file.Path.of("invoice.txt"), content);
    }
}
```

**Tasarım sorusu:** İki sorumluluğun gerçekten bağımsız değiştiğini gösteren gereksinim var mı? SRP, rastgele sınıf sayısını artırma çağrısı değildir.

---

## 3. SOLID — Open/Closed Principle (OCP)

### Anlamı ve gerekçesi

OCP, yazılım birimlerinin **yeni davranışlara genişletilebilir**, mevcut ve test edilmiş kodu sürekli düzenlemeye **daha az muhtaç** olacak şekilde tasarlanmasını savunur. Özellikle farklı stratejiler veya sağlayıcılar eklenecekse davranışları bir sözleşme üzerinden değiştirmek yararlı olabilir. Bu, mevcut kodun hiçbir zaman değiştirilmemesi gerektiği anlamına gelmez.

**C# — Her yeni türde değişen koşul:**

```csharp
public decimal Discount(string customerType, decimal total)
{
    if (customerType == "student") return total * 0.90m;
    if (customerType == "vip") return total * 0.80m;
    return total;
}
```

Yeni müşteri türleri ekledikçe aynı metot büyür. Değişim sık ve bağımsızsa stratejileri ayırabiliriz:

```csharp
public interface IDiscountPolicy
{
    decimal Apply(decimal total);
}

public sealed class StudentDiscount : IDiscountPolicy
{
    public decimal Apply(decimal total) => total * 0.90m;
}

public sealed class VipDiscount : IDiscountPolicy
{
    public decimal Apply(decimal total) => total * 0.80m;
}

public sealed class Checkout
{
    private readonly IDiscountPolicy _policy;
    public Checkout(IDiscountPolicy policy) => _policy = policy;
    public decimal Total(decimal amount) => _policy.Apply(amount);
}
```

Yeni indirim stratejisi eklemek için `Checkout` değişmez; ancak **hangi stratejinin seçileceği** başka bir yerde çözülmelidir.

**Python — Kısa karşılık:**

```python
class StudentDiscount:
    def apply(self, total): return total * 0.90

class Checkout:
    def __init__(self, policy): self.policy = policy
    def total(self, amount): return self.policy.apply(amount)
```

**Java — Kısa karşılık:**

```java
interface DiscountPolicy { double apply(double total); }
class StudentDiscount implements DiscountPolicy {
    public double apply(double total) { return total * 0.90; }
}
class Checkout {
    private final DiscountPolicy policy;
    Checkout(DiscountPolicy policy) { this.policy = policy; }
    double total(double amount) { return policy.apply(amount); }
}
```

**Dikkat:** C# `decimal` finansal tutarlar için uygun bir seçimdir; kısa Python/Java örneklerindeki ondalık türler dil karşılaştırması içindir, üretim ortamında para için sırasıyla `Decimal` ve `BigDecimal` düşünülebilir.

---

## 4. SOLID — Liskov Substitution Principle (LSP)

### Anlamı ve gerekçesi

Bir alt tür, üst türün kullanıldığı yerde **istemcinin beklediği davranış sözleşmesini bozmadan** kullanılabilmelidir. Yalnızca aynı metot adlarını sağlamak yeterli değildir. İstisnalar, önkoşullar, sonkoşullar ve durum değişmezleri de önemlidir.

**C# — Kötü tür ilişkisi:**

```csharp
public class Bird
{
    public virtual void Fly() { /* uçar */ }
}

public class Penguin : Bird
{
    public override void Fly()
        => throw new NotSupportedException();
}
```

`Bird` kullanan kod her kuşun uçacağını varsayıyorsa `Penguin` bu sözleşmeyi bozar. Sorun penguen sınıfının “eksik” olması değil, üst türün yanlış soyutlama sunmasıdır.

**C# — Sözleşmeyi doğru ayrıştırma:**

```csharp
public interface IBird { void Eat(); }
public interface IFlyingBird : IBird { void Fly(); }

public class Penguin : IBird
{
    public void Eat() { }
}

public class Sparrow : IFlyingBird
{
    public void Eat() { }
    public void Fly() { }
}
```

**Python:**

```python
from typing import Protocol

class FlyingBird(Protocol):
    def fly(self) -> None: ...

class Sparrow:
    def fly(self) -> None: print("flying")
```

**Java:**

```java
interface Bird { void eat(); }
interface FlyingBird extends Bird { void fly(); }
class Penguin implements Bird { public void eat() {} }
class Sparrow implements FlyingBird {
    public void eat() {}
    public void fly() {}
}
```

**Not** Kalıtım sözdizimsel bir ilişkidir; LSP ise *davranışsal uyumluluğu* sorgular. `is-a` ifadesinin günlük dilde doğru görünmesi tek başına yeterli değildir.

---

## 5. SOLID — Interface Segregation Principle (ISP)

### Anlamı ve gerekçesi

ISP, istemcilerin **kullanmadıkları operasyonlara bağımlı bırakılmamasını** önerir. Gereğinden büyük bir interface, farklı bileşenleri ilgisiz değişikliklerden etkileyebilir. Bir interface'e tek metot koymak zorunda değiliz; önemli olan istemcilerin gerçek ihtiyaçlarına göre anlamlı sınırlar oluşturmaktır.

**C# — Aşırı geniş sözleşme:**

```csharp
public interface IOfficeMachine
{
    void Print();
    void Scan();
    void Fax();
}
```

Yalnızca yazdırabilen cihaz `Scan` ve `Fax` uygulamak zorunda kalabilir. Daha anlamlı ayrım:

```csharp
public interface IPrinter { void Print(); }
public interface IScanner { void Scan(); }

public class BasicPrinter : IPrinter
{
    public void Print() { }
}

public class MultiFunctionPrinter : IPrinter, IScanner
{
    public void Print() { }
    public void Scan() { }
}
```

**Python:**

```python
from typing import Protocol

class Printer(Protocol):
    def print_document(self) -> None: ...
```

**Java:**

```java
interface Printer { void printDocument(); }
interface Scanner { void scan(); }
class BasicPrinter implements Printer {
    public void printDocument() {}
}
```

**LSP ile farkı:** ISP, *hangi operasyonların bir sözleşmede toplanacağını* sorgular. LSP, *bu sözleşmeyi uygulayan türün beklenen davranışı sağlayıp sağlamadığını* sorgular.

---

## 6. SOLID — Dependency Inversion Principle (DIP)

### Anlamı ve gerekçesi

Bu prensip makaleye hazırlık açısından en kritik olanıdır. DIP'nin özü iki ilişkili düşüncedir:

1. **Yüksek seviyeli modüller** (iş kurallarını koordine eden yapılar) **düşük seviyeli modüllerin** (veritabanı, e-posta, dosya, ağ gibi ayrıntıların) somut uygulamalarına doğrudan bağlanmamalıdır. Her ikisi de uygun soyutlamalara bağımlı olmalıdır.
2. Soyutlamalar ayrıntılara göre şekillenmek yerine, ayrıntılar uygun soyutlamaların sözleşmesine uyum sağlamalıdır.

Bu, her sınıfa zorunlu olarak interface yazmak demek değildir. Değişebilir altyapı veya test sınırı varsa soyutlama daha anlamlı hâle gelir.

### 6.1. C# — Somut bağımlılık problemi

```csharp
public class SmtpEmailSender
{
    public void Send(string message)
    {
        Console.WriteLine($"SMTP: {message}");
    }
}

public class OrderService
{
    private readonly SmtpEmailSender _sender = new();

    public void CompleteOrder()
    {
        // Sipariş iş kuralları ...
        _sender.Send("Sipariş tamamlandı");
    }
}
```

`OrderService` doğrudan `SmtpEmailSender` somut sınıfını seçer ve oluşturur. SMS ya da test amaçlı sahte göndericiye geçmek için iş kuralı sınıfını değiştirmek gerekebilir.

### 6.2. C# — DIP ile sınırı kurmak, DI ile bağımlılığı vermek

```csharp
public interface IMessageSender
{
    void Send(string message);
}

public sealed class SmtpEmailSender : IMessageSender
{
    public void Send(string message)
        => Console.WriteLine($"SMTP: {message}");
}

public sealed class OrderService
{
    private readonly IMessageSender _sender;

    public OrderService(IMessageSender sender)
    {
        _sender = sender ?? throw new ArgumentNullException(nameof(sender));
    }

    public void CompleteOrder()
    {
        // Sipariş iş kuralları ...
        _sender.Send("Sipariş tamamlandı");
    }
}

// Composition root: somut nesneler uygulama girişinde bağlanır.
var service = new OrderService(new SmtpEmailSender());
service.CompleteOrder();
```

**Adım adım tasarım analizi:**

- `OrderService`, e-posta altyapısının ayrıntılarını bilmek zorunda değildir; `IMessageSender` sözleşmesi üzerinden konuşur.
- `SmtpEmailSender`, bu sözleşmeyi uygular; SMS veya test göndericisi aynı ihtiyacı karşılayabilir.
- `OrderService` bağımlılığı **constructor** üzerinden alır: bu **Dependency Injection** örneğidir.
- Uygulamanın girişindeki `new OrderService(...)` bağımlılıkların bir araya getirildiği **composition root** rolündedir.
- İş kuralı modülünün somut altyapıdan ayrılması **DIP** ile ilgilidir. Bunlar bağlantılı olsa da aynı kavram değildir.

**C# — Test edilebilirliğin somut sonucu:**

```csharp
public sealed class FakeSender : IMessageSender
{
    public readonly List<string> Messages = new();
    public void Send(string message) => Messages.Add(message);
}

var fake = new FakeSender();
var service = new OrderService(fake);
service.CompleteOrder();

Console.WriteLine(fake.Messages.Count == 1); // True
```

İş kuralını gerçek SMTP sunucusuna bağlamadan test edebiliyoruz. Yine de her DI kullanımı otomatik olarak daha iyi bir tasarım üretmez: gereksiz interface, yanlış yaşam döngüsü veya uygunsuz nesne oluşturma biçimi yeni sorunlar doğurabilir.

### 6.3. Python — Aynı tasarımın kısa karşılığı

```python
from typing import Protocol

class MessageSender(Protocol):
    def send(self, message: str) -> None: ...

class SmtpSender:
    def send(self, message: str) -> None:
        print(f"SMTP: {message}")

class OrderService:
    def __init__(self, sender: MessageSender):
        self.sender = sender

    def complete_order(self):
        self.sender.send("Sipariş tamamlandı")

OrderService(SmtpSender()).complete_order()
```

`Protocol`, yapısal alt türleme için bir tip sözleşmesi sağlar; Python'da nesnenin bu protokolü açıkça `implements` etmesi gerekmez. Burada da nesne dışarıdan verilir; herhangi bir DI framework'ü zorunlu değildir.

### 6.4. Java — Makaledeki dile yakın karşılık

```java
interface MessageSender {
    void send(String message);
}

class SmtpSender implements MessageSender {
    public void send(String message) {
        System.out.println("SMTP: " + message);
    }
}

class OrderService {
    private final MessageSender sender;

    OrderService(MessageSender sender) {
        this.sender = java.util.Objects.requireNonNull(sender);
    }

    void completeOrder() {
        sender.send("Sipariş tamamlandı");
    }
}

// Uygulama başlangıcında:
OrderService service = new OrderService(new SmtpSender());
service.completeOrder();
```

Bu örnekte ne Spring ne de bir başka DI framework'ü kullanılır. **Constructor Injection, dil düzeyinde nesne oluşturma ve parametre ile gerçekleştirilebilir.** Framework, nesne grafiği ve yaşam döngüsü yönetimini otomatikleştirerek ek kolaylık sağlayabilir.

### 6.5. DIP, DI ve IoC arasındaki fark

| Kavram | Temel soru | Açıklama |
|---|---|---|
| **DIP** — tasarım prensibi | *Neye bağımlı olmalıyım?* | İş kuralları ile teknik ayrıntıların uygun soyutlamalar üzerinden ilişki kurmasını önerir. |
| **DI** — bağımlılık sağlama tekniği | *Bağımlılık nasıl verilecek?* | Gereken nesnenin içeride yaratılması yerine dışarıdan verilmesidir. |
| **IoC** — daha geniş kontrol yaklaşımı | *Akışın/oluşturmanın kontrolü kimde?* | Geleneksel kontrolün bir bölümünün dış mekanizma veya başka bileşenlere devredilmesidir; DI bunun yaygın biçimlerinden biridir. |
| **DI Container** | *Nesneleri kim kuracak ve yönetecek?* | Bağımlılıkları çözümleyip nesneleri oluşturabilen ve yaşam döngülerini yönetebilen araçtır. |

**İki kritik ayrım:**

- Constructor ile somut bir sınıf enjekte etmek **DI olabilir**, fakat yüksek seviyeli modül hâlâ somut tipe bağımlıysa DIP'in hedefi sınırlı kalır.
- Soyut bir interface tanımlamak tek başına **DI olduğu** anlamına gelmez; nesne hâlâ bağımlılığı içeride oluşturuyor olabilir.

### 6.6. C# DI Container ile kısa örnek

.NET uygulamalarında `Microsoft.Extensions.DependencyInjection` kullanılırken yaygın düzen şöyledir:

```csharp
using Microsoft.Extensions.DependencyInjection;

var services = new ServiceCollection();
services.AddTransient<IMessageSender, SmtpEmailSender>();
services.AddTransient<OrderService>();

using var provider = services.BuildServiceProvider();
var orderService = provider.GetRequiredService<OrderService>();
orderService.CompleteOrder();
```

Bu kod, önceki `IMessageSender`, `SmtpEmailSender` ve `OrderService` tanımlarının bulunduğu bir projede; ilgili paketin erişilebilir olması koşuluyla kullanılabilir. `Transient` her çözümlemede yeni örnek oluşturma semantiğini ifade eder. Diğer önemli yaşam döngüleri `Scoped` ve `Singleton`'dır; bunların uygulamada yanlış eşleştirilmesi ayrıca değerlendirilmelidir. **Container kullanmak DIP'e uygunluğu garanti etmez.**

---

## 7. GRASP — Nesnelere sorumluluk nasıl atanır?

SOLID'in yanında GRASP, davranışın hangi sınıfa verilmesi gerektiği konusunda daha doğrudan sorular sorar. GRASP'ın yaygın dokuz örüntüsü: **Information Expert, Creator, Controller, Low Coupling, High Cohesion, Polymorphism, Pure Fabrication, Indirection, Protected Variations**.

### 7.1. Information Expert

**Soru:** Bir işi yapmak için gereken bilgi hangi nesnede bulunuyor?  
Sipariş toplamını hesaplamak için ürün satırlarının fiyat ve miktar bilgisi gerekiyorsa, toplam hesabını bu bilginin bulunduğu `Order`/`OrderLine` çevresinde tutmak çoğu zaman mantıklıdır.

```csharp
public record OrderLine(decimal UnitPrice, int Quantity)
{
    public decimal Subtotal => UnitPrice * Quantity;
}

public class Order
{
    public List<OrderLine> Lines { get; } = new();
    public decimal Total => Lines.Sum(line => line.Subtotal);
}
```

Bu örnekte `Order` kendi satırlarının toplamından sorumludur. Ancak dış veri kaynakları veya vergi düzenlemeleri devreye girdiğinde farklı hizmetlere ihtiyaç doğabilir. Bilgi uzmanlığı tek başına bütün tasarım kararlarını çözmez.

### 7.2. Creator

**Soru:** Bir nesneyi kim oluşturmalı?  
Bir sınıf başka nesneleri içeriyor, yakından kullanıyor veya başlatmak için gerekli verilere sahipse oluşturma sorumluluğunu üstlenmesi mantıklı olabilir. Bununla birlikte DI kullanılan uygulamalarda servis bağımlılıklarının oluşturulması çoğu zaman **composition root/container** üzerinden gerçekleştirilir. GRASP Creator ile DI arasında her durumda çatışma yoktur: **alan nesnelerinin oluşturulması** ile **altyapı bağımlılıklarının kurulması** farklı kararlardır.

```csharp
public class Cart
{
    private readonly List<CartItem> _items = new();
    public void Add(string sku, int quantity)
        => _items.Add(new CartItem(sku, quantity));
}
public record CartItem(string Sku, int Quantity);
```

### 7.3. Controller

**Soru:** Dışarıdan gelen sistem olayını kim karşılamalı?  
Bir controller, UI/HTTP isteğini alıp alan işlemlerini koordine eder; her iş kuralını controller'a doldurmak iyi bir tasarım değildir.

```csharp
public class CheckoutController
{
    private readonly CheckoutService _service;
    public CheckoutController(CheckoutService service) => _service = service;
    public void Submit() => _service.Complete();
}
public class CheckoutService
{
    public void Complete() { /* iş akışı */ }
}
```

Buradaki `Controller`, ASP.NET Core MVC'ye özgü olmak zorunda değildir; örnek sistem olayını karşılayan genel bir nesneyi gösterir.

### 7.4. Low Coupling

**Soru:** Kararımız başka sınıflara gereksiz bağımlılık ekliyor mu?  
Bir sınıfın çok sayıda somut tipi, altyapı ayrıntısını ve başka nesnelerin iç durumunu bilmesi değişiklik maliyetini artırabilir. Soyutlamalar bağımlılıkları tümüyle yok etmez; **bağımlılığın yönünü ve türünü** düzenler.

### 7.5. High Cohesion

**Soru:** Aynı sınıftaki sorumluluklar anlamlı bir bütün mü oluşturuyor?  
Sipariş fiyatlandırma, veritabanı yönetimi, kullanıcı oturumu ve e-posta gönderimini tek sınıfta toplamak, ilgisiz değişiklik nedenlerini birleştirebilir. Ancak sadece küçük sınıflar üretmek de tek başına yüksek cohesion garantilemez.

### 7.6. Polymorphism

**Soru:** Türlere göre değişen davranışı nasıl temsil etmeliyim?  
Tekrarlanan `if/switch` yapıları yerine farklı uygulamalar aynı sözleşmeyi gerçekleştirebilir. Yine de sabit ve çok küçük bir koşul kümesi için soyutlama eklemenin maliyetini değerlendirmeliyiz.

```csharp
public interface IShippingCost
{
    decimal Calculate(decimal weight);
}
public class ExpressShipping : IShippingCost
{
    public decimal Calculate(decimal weight) => 25m + weight * 10m;
}
```

### 7.7. Pure Fabrication

**Soru:** Hiçbir alan nesnesine doğal biçimde ait olmayan teknik bir sorumluluğu nereye koymalıyım?  
Veritabanına kayıt, e-posta gönderimi gibi işleri alan nesnesine yüklemek yerine `Repository` veya ayrı bir servis tasarlanabilir. Bunlar alan kavramı olmasa da coupling/cohesion açısından yararlı olabilir.

### 7.8. Indirection

**Soru:** Doğrudan ilişki gereğinden fazla bağımlılık yaratıyorsa araya nasıl bir aracı koyabilirim?  
Adapter, mediator veya servis soyutlaması, iki bileşenin birbirinin ayrıntılarına doğrudan bağlanmasını azaltabilir. Fakat her dolaylılık katmanı yeni karmaşıklık getirir; amaçsız aracı sınıflar oluşturmamak gerekir.

### 7.9. Protected Variations

**Soru:** Hangi noktalar zaman içinde değişecek, bunları nasıl sınırlandırabilirim?  
Ödeme sağlayıcısı, veri kaynağı veya bildirim kanalı değişebilir. Bunların istikrarlı bir sözleşme arkasında tutulması değişimin sistem geneline yayılmasını sınırlayabilir. Bu, **OCP ve DIP ile güçlü biçimde ilişkili** bir düşüncedir.

### 7.10. GRASP'ın Python ve Java'daki kısa görünümü

Aşağıdaki iki örnekte **Information Expert** kullanılır: toplamın bilgisi sipariş satırlarında olduğu için hesaplama onlara yakın tutulur.

**Python:**

```python
from dataclasses import dataclass

@dataclass
class OrderLine:
    price: float
    quantity: int

    def subtotal(self): return self.price * self.quantity

class Order:
    def __init__(self, lines): self.lines = lines
    def total(self): return sum(line.subtotal() for line in self.lines)
```

**Java:**

```java
record OrderLine(double price, int quantity) {
    double subtotal() { return price * quantity; }
}
class Order {
    private final java.util.List<OrderLine> lines;
    Order(java.util.List<OrderLine> lines) { this.lines = lines; }
    double total() {
        return lines.stream().mapToDouble(OrderLine::subtotal).sum();
    }
}
```

Örneklerde basitlik için `float/double` kullanıldı; parasal hesapta hassasiyet gereksinimleri ayrıca ele alınmalıdır.

---

## 8. SOLID ile GRASP arasındaki bağlantılar

| Tasarım durumu | SOLID açısından | GRASP açısından |
|---|---|---|
| Bir sınıf hem hesaplama hem dosya kaydı yapıyor | SRP | High Cohesion, Pure Fabrication |
| Yeni sağlayıcı eklemek için iş kuralları sürekli değişiyor | OCP, DIP | Protected Variations, Polymorphism |
| Alt sınıf beklenen davranışı bozar | LSP | Polymorphism'in doğru uygulanmasıyla ilişkili |
| İstemci gereksiz metotlara bağımlı | ISP | Low Coupling |
| Üst düzey servis somut altyapıyı içeride oluşturuyor | DIP; DI ile iyileştirilebilir | Low Coupling, Indirection, Protected Variations |
| Bir metodun yapacağı işin sahibi belirsiz | SRP ile dolaylı ilişki | Information Expert, Creator, Controller |

**Yorum:** SOLID ve GRASP bir kontrol listesi gibi mekanik uygulanmamalı. Örneğin her sınıf için interface üretmek kodu uzatabilir; küçük ve değişmeyecek nesnelerde doğrudan bağımlılık kabul edilebilir. Tasarım kararlarını gerçek değişim ihtiyacı, test maliyeti, okunabilirlik ve ekip bağlamında gerekçelendirmek gerekir.

---

##  Kaynaklar ve doğrulama notu


- Robert C. Martin, *Design Principles and Design Patterns* (2000) — SOLID'in tarihsel temellerini incelemek için.
- Craig Larman, *Applying UML and Patterns* (3. baskı, 2004) — GRASP sorumluluk atama örüntüleri için.
- Mark Seemann, *Dependency Injection in .NET* (2011/2012 baskı bilgisi kullanılan nüshaya göre teyit edilmeli) — DI, composition root ve container kavramları için.
- Microsoft Learn, `.NET dependency injection` belgeleri — .NET container ve yaşam döngüsü davranışını uygulama düzeyinde kontrol etmek için.
- Laigner, R., Mendonça, D., Garcia, A. ve Kalinowski, M. (2022). *Cataloging dependency injection anti-patterns in software systems*. *Journal of Systems and Software, 184*, 111125. DOI: 10.1016/j.jss.2021.111125.

