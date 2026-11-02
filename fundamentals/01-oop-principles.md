# 01 — Nesneye Dayalı Programlama (OOP) Temelleri


## 1. Class ve Object: Tasarımı çalışan nesneye dönüştürmek

### Kavramsal açıklama

**Class (sınıf)**, belirli türdeki nesnelerin hangi verilere ve davranışlara sahip olabileceğini tanımlayan bir yapıdır. **Object (nesne)** ise bu sınıftan oluşturulan, bellekte kendi durumunu taşıyan somut örnektir. Örneğin `BankAccount` sınıfı hesapların ortak kurallarını tanımlarken `account1` ve `account2` ayrı bakiyelere sahip nesnelerdir.

OOP'nin önemli yönü, veriyi ve o veriye ilişkin davranışı bir arada düşünmesidir. Bir hesabın bakiyesini rastgele değiştirmek yerine para yatırma ve çekme davranışlarıyla yönetmek, iş kurallarının belirli bir yerde uygulanmasını sağlar. Bu, nesne tasarımına başlangıç noktasıdır; tek başına bütün tasarımın iyi olduğu anlamına gelmez.

### C# — Temel örnek

```csharp
public class BankAccount
{
    public string Owner { get; }
    public decimal Balance { get; private set; }

    public BankAccount(string owner, decimal initialBalance)
    {
        if (string.IsNullOrWhiteSpace(owner))
            throw new ArgumentException("Owner is required.");
        if (initialBalance < 0)
            throw new ArgumentOutOfRangeException(nameof(initialBalance));

        Owner = owner;
        Balance = initialBalance;
    }

    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentOutOfRangeException(nameof(amount));
        Balance += amount;
    }
}

var account1 = new BankAccount("Ayşe", 100m);
var account2 = new BankAccount("Mehmet", 250m);
account1.Deposit(50m);
// account1.Balance == 150m; account2.Balance == 250m
```

**Kodun analizi:** `BankAccount` sınıfın tanımıdır; `new` ile iki farklı nesne oluşturulur. Her nesne kendi `Balance` durumunu taşır. `Deposit` bir **method**'dur; `Owner` ve `Balance` ise **property**'dir. `Balance` için `private set` kullanılması, sınıf dışından `account1.Balance = -100m` gibi bir atamayı önler. Başlangıç ve para yatırma kuralları sınıf içinde korunur.

### Python — Aynı düşünce

```python
class BankAccount:
    def __init__(self, owner: str, balance: float):
        self.owner = owner
        self._balance = balance  # erişimi sınırlama niyeti (konvansiyon)

    def deposit(self, amount: float):
        if amount <= 0:
            raise ValueError("Amount must be positive")
        self._balance += amount

account = BankAccount("Ayşe", 100)
account.deposit(50)
```

Python'daki baştaki `_`, C#'taki `private` gibi katı bir erişim kuralı **değildir**; diğer geliştiricilere “iç kullanım” niyetini bildirir. Üretim amaçlı finansal tutarlarda kayan noktalı `float` yerine `Decimal` tercih edilmelidir. Burada örnek yalnızca nesne yapısını göstermektedir.

### Java — Aynı düşünce

```java
public class BankAccount {
    private final String owner;
    private int balance;

    public BankAccount(String owner, int balance) {
        if (balance < 0) throw new IllegalArgumentException();
        this.owner = owner;
        this.balance = balance;
    }

    public void deposit(int amount) {
        if (amount <= 0) throw new IllegalArgumentException();
        balance += amount;
    }
}

BankAccount account = new BankAccount("Ayşe", 100);
account.deposit(50);
```

Java'da alanlar (`field`) çoğunlukla `private` tutulur; kontrollü işlemler metodlarla sağlanır. Burada bakiyeyi `int` olarak düşünmek yalnızca örneği kısaltmak içindir; gerçek finansal uygulamalar için uygun para türü seçilmelidir.



## 2. Encapsulation (Kapsülleme): Durumu kurallarıyla birlikte korumak

### Kavramsal açıklama

**Encapsulation**, bir nesnenin iç durumunu ve bu durumla ilgili işlemleri anlamlı bir sınır içinde toplamasıdır. Pratikte erişim kontrolü (`private`, `public` vb.), property'ler ve yöntemlerle uygulanabilir. Ancak kapsüllemeyi yalnızca “tüm field'ları private yapmak” şeklinde yorumlamak eksiktir. Asıl amaç, nesnenin geçerli durumunu koruyacak kuralların dışarıdan gelişigüzel aşılmasını engellemektir.

Örneğin sipariş durumunu herhangi bir bileşenin serbestçe değiştirmesi, `Delivered` olmuş bir siparişin tekrar `Draft` durumuna döndürülmesine yol açabilir. Değişim, iş kuralını bilen davranış üzerinden gerçekleştirilirse geçersiz geçişler önlenebilir.

### C# — Kontrolsüz durumdan kontrollü davranışa

```csharp
// Sorunlu: Her yerden keyfi atama yapılabilir.
public class OrderBad
{
    public string Status { get; set; } = "Draft";
}

// Daha kontrollü yaklaşım:
public class Order
{
    public string Status { get; private set; } = "Draft";

    public void Ship()
    {
        if (Status != "Draft")
            throw new InvalidOperationException("Order cannot be shipped.");
        Status = "Shipped";
    }
}
```

**Neden daha iyi?** İkinci örnekte dış kod `Status` değerini serbestçe değiştiremez; `Ship()` mevcut durumu kontrol eder. Böylece sınıf kendi tutarlılığını (invariant) korur. Yine de gerçek bir sipariş modelinde daha fazla durum ve geçiş kuralı gerekebilir.

### Python

```python
class Order:
    def __init__(self):
        self._status = "Draft"

    @property
    def status(self):
        return self._status

    def ship(self):
        if self._status != "Draft":
            raise ValueError("Invalid transition")
        self._status = "Shipped"
```

Python `@property` ile kontrollü okuma sunar. `_status` alanına teknik olarak hâlâ erişilebilir; bu yüzden Python'daki kapsülleme büyük ölçüde dilin konvansiyonlarıyla desteklenir.

### Java

```java
public class Order {
    private String status = "Draft";

    public String getStatus() { return status; }

    public void ship() {
        if (!status.equals("Draft"))
            throw new IllegalStateException("Invalid transition");
        status = "Shipped";
    }
}
```


## 3. Abstraction (Soyutlama): Neye ihtiyaç duyulduğunu ayırmak

### Kavramsal açıklama

**Abstraction**, bir bileşeni kullanmak için gerekli davranışları öne çıkarıp uygulama ayrıntılarını arka planda bırakmaktır. Örneğin bir bildirimin *gönderilebilmesi* ile bildirimin *SMTP, HTTP veya başka bir protokolle nasıl gönderildiği* farklı meselelerdir. Çağıran bileşen yalnızca ilgili sözleşmeyi biliyorsa altyapı değişikliklerinden daha az etkilenebilir.

C#'ta `interface` bir davranış sözleşmesini tanımlayabilir. `abstract class` ise ortak durum ve kısmi uygulama paylaşımı da sağlayabilir. Bunlar her durumda aynı ihtiyacı karşılamaz. Her küçük sınıfa interface eklemek zorunlu değildir; soyutlamanın yararı değişebilirlik ve tasarım ihtiyacıyla değerlendirilmelidir.

### C# — Somut sınıf yerine sözleşme

```csharp
public interface IMessageSender
{
    void Send(string message);
}

public class EmailSender : IMessageSender
{
    public void Send(string message) => Console.WriteLine($"Email: {message}");
}

public class NotificationService
{
    private readonly IMessageSender _sender;

    public NotificationService(IMessageSender sender)
    {
        _sender = sender ?? throw new ArgumentNullException(nameof(sender));
    }

    public void Notify(string text) => _sender.Send(text);
}

var service = new NotificationService(new EmailSender());
service.Notify("Merhaba");
```

**Kodun analizi:** `NotificationService`, `EmailSender` sınıfının nasıl çalıştığını bilmek zorunda değildir. Yalnızca `IMessageSender.Send` sözleşmesine ihtiyaç duyar. `new EmailSender()` uygulamanın kurulum noktasındadır; `NotificationService` içinde değildir. Böylece başka bir `IMessageSender` uygulaması, servis sınıfını değiştirmeden kullanılabilir.



### Python — Protocol ile davranış sözleşmesi

```python
from typing import Protocol

class MessageSender(Protocol):
    def send(self, message: str) -> None: ...

class EmailSender:
    def send(self, message: str) -> None:
        print(f"Email: {message}")

class NotificationService:
    def __init__(self, sender: MessageSender):
        self.sender = sender
```

Python'daki `Protocol`, yapısal tipleme yaklaşımına imkân verir: sınıfın sözleşmeyi açıkça `implements` etmesi gerekmez, beklenen davranış biçimini karşılaması yeterlidir. Tip ipuçları varsayılan olarak çalışma zamanında zorunlu denetlenmez.

### Java — Interface ile sözleşme

```java
interface MessageSender {
    void send(String message);
}

class EmailSender implements MessageSender {
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

class NotificationService {
    private final MessageSender sender;

    NotificationService(MessageSender sender) {
        this.sender = sender;
    }
}
```

**Dil farkı:** C# ve Java interface uygulamasını nominal olarak bildirir (`: IMessageSender`, `implements MessageSender`). Python `Protocol` aracılığıyla yapısal uyumu ifade edebilir. Üçünde de temel hedef, kullanıcının ihtiyaç duyduğu davranışı uygulama ayrıntısından ayırmaktır.



## 4. Inheritance (Kalıtım): Ortak davranışı uzmanlaştırmak

### Kavramsal açıklama

**Inheritance**, bir alt sınıfın üst sınıfın belirli özellik ve davranışlarını devralmasını ve gerektiğinde uzmanlaştırmasını sağlar. Genellikle “is-a” (bir türüdür) ilişkisi için anlamlıdır. Örneğin `Circle`, bir `Shape` türü olarak düşünülebilir. Alt sınıfın üst sınıfın beklentilerini bozmaması önemlidir; aksi hâlde kalıtım, yeniden kullanım sağlasa da güvenilir tasarım oluşturmaz.

Kalıtımı yalnızca kod tekrarını azaltma yöntemi olarak kullanmak sakıncalıdır. Bir nesne başka bir nesneyi **kullanıyorsa**, çoğu zaman kalıtımdan ziyade **composition** ilişkisi uygundur. 

### C# — Abstract base class

```csharp
public abstract class Shape
{
    public abstract double Area();
}

public class Circle : Shape
{
    public double Radius { get; }

    public Circle(double radius)
    {
        if (radius <= 0) throw new ArgumentOutOfRangeException(nameof(radius));
        Radius = radius;
    }

    public override double Area() => Math.PI * Radius * Radius;
}

Shape shape = new Circle(2);
Console.WriteLine(shape.Area());
```

**Kodun analizi:** `Shape` soyut sınıfı alan hesaplama sözleşmesini verir. `Circle`, bu sözleşmeyi `override` ile uygular. `Shape` türündeki referans, çalışma zamanında bir `Circle` nesnesini gösterebilir. Bu durum kalıtım ile polymorphism arasındaki bağlantıyı ortaya koyar.

### Python

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

class Circle(Shape):
    def __init__(self, radius: float):
        self.radius = radius

    def area(self) -> float:
        return 3.14159 * self.radius ** 2
```

### Java

```java
abstract class Shape {
    abstract double area();
}

class Circle extends Shape {
    private final double radius;
    Circle(double radius) { this.radius = radius; }

    @Override
    double area() { return Math.PI * radius * radius; }
}
```

**Önemli dil farkı:** C# ve Java sınıflar arasında tekli kalıtım sunar; bir sınıf birden çok interface uygulayabilir. Python birden çok sınıftan kalıtımı destekler, fakat metot çözümleme sırası (MRO) ve tasarım karmaşıklığı dikkate alınmalıdır.


## 5. Polymorphism (Çok biçimlilik): Aynı sözleşme, farklı davranışlar

### Kavramsal açıklama

**Polymorphism**, farklı nesnelerin ortak bir sözleşme üzerinden kullanılabilmesi ve aynı çağrıya kendi uygulamalarına göre yanıt verebilmesidir. Örneğin bir ödeme servisi, ödeme yöntemini işleyen `Pay` davranışını bilir; kredi kartı veya havale ayrıntısını bilmek zorunda değildir.

Bu özellikle büyüyen `if/else` veya `switch` yapılarını azaltmaya yardımcı olabilir; ancak her koşullu ifade kötü tasarım değildir. Polymorphism'in yararı, değişen davranışların bağımsız ve genişletilebilir biçimde modellenebildiği durumlarda ortaya çıkar.

### C# — Interface üzerinden davranış seçimi

```csharp
public interface IPaymentMethod
{
    void Pay(decimal amount);
}

public class CardPayment : IPaymentMethod
{
    public void Pay(decimal amount) => Console.WriteLine($"Card: {amount}");
}

public class TransferPayment : IPaymentMethod
{
    public void Pay(decimal amount) => Console.WriteLine($"Transfer: {amount}");
}

public class CheckoutService
{
    public void Checkout(IPaymentMethod method, decimal amount)
    {
        if (amount <= 0) throw new ArgumentOutOfRangeException(nameof(amount));
        method.Pay(amount);
    }
}

var checkout = new CheckoutService();
checkout.Checkout(new CardPayment(), 100m);
checkout.Checkout(new TransferPayment(), 100m);
```

**Kodun analizi:** `CheckoutService` tek bir `IPaymentMethod` sözleşmesi kullanır; çağrılan gerçek uygulama nesnenin türüne göre belirlenir. Yeni bir ödeme yöntemi eklendiğinde `CheckoutService` sınıfının kodunu değiştirmek gerekmeyebilir. Buradaki çıktı üretme örneği, gerçek ödeme sistemindeki güvenlik ve işlem kurallarını temsil etmez.

### Python

```python
class CardPayment:
    def pay(self, amount):
        print(f"Card: {amount}")

class TransferPayment:
    def pay(self, amount):
        print(f"Transfer: {amount}")

def checkout(method, amount):
    method.pay(amount)

checkout(CardPayment(), 100)
checkout(TransferPayment(), 100)
```

Python örneği duck typing davranışını gösterir: `checkout`, nesnenin belirli bir üst sınıftan gelmesinden çok `pay` metodunu sunmasına dayanır. Gerekirse bu beklenti `Protocol` ile tip düzeyinde ifade edilebilir.

### Java

```java
interface PaymentMethod {
    void pay(int amount);
}

class CardPayment implements PaymentMethod {
    public void pay(int amount) { System.out.println("Card: " + amount); }
}

class CheckoutService {
    void checkout(PaymentMethod method, int amount) {
        method.pay(amount);
    }
}
```

**Kavram ayrımı:** Inheritance bir sınıf ilişkisini kurabilir; polymorphism farklı uygulamaların aynı sözleşme üzerinden kullanılabilmesini sağlar. Polymorphism için her durumda sınıf kalıtımı gerekli değildir; interface temelli yaklaşım da kullanılır.



## 6. Dört ilkenin birbiriyle ilişkisi ve makaleye hazırlık



| Kavram | Temel soru | C# üzerinde tipik araç | DI araştırması açısından düşünmem gereken |
|---|---|---|---|
| **Encapsulation** | Nesnenin durumunu kim, hangi kurallarla değiştirebilir? | `private`, property, method | Bağımlılıklar yönetilirken nesnenin sorumluluk sınırları korunuyor mu? |
| **Abstraction** | Bir bileşen başka bir bileşenin hangi davranışını bilmek zorunda? | `interface`, `abstract` | Bağımlılık davranış sözleşmesine mi, uygulama ayrıntısına mı bağlı? |
| **Inheritance** | Gerçekten bir “is-a” ilişkisi var mı? | `: BaseClass`, `override` | Kalıtım gereksiz kullanılarak tasarım sıkılaştırılmış mı? |
| **Polymorphism** | Aynı sözleşmeye farklı uygulamalar cevap verebilir mi? | Interface uygulaması, `virtual`/`override` | Farklı bağımlılık uygulamaları çağıran sınıfı değiştirmeden kullanılabilir mi? |

### Önemli ayrımlar

- **Encapsulation ≠ abstraction:** Kapsülleme, iç durumun ve değişim kurallarının korunmasıyla; soyutlama, gerekli davranışın ayrıntılardan ayrılmasıyla ilgilidir. Birlikte kullanılabilirler.
- **Inheritance ≠ polymorphism:** Kalıtım bir tür hiyerarşisi kurar; polymorphism, ortak sözleşmeyle farklı uygulamaları kullanabilmektir.
- **DI ≠ DIP:** DI bağımlılığın sağlanma biçimiyle ilgilidir. DIP ise üst düzey ve alt düzey modüllerin soyutlamalara bağımlı olması gerektiğini savunan tasarım ilkesidir. Somut bir sınıfı constructor'a vermek de teknik olarak DI olabilir; bu durum DIP'e tam uyulduğunu göstermez.
- **Interface kullanmak otomatik olarak iyi tasarım değildir:** Soyutlama maliyeti, değişiklik olasılığı, test gereksinimleri ve proje karmaşıklığı birlikte düşünülmelidir.



## 7. Üç dil arasında hızlı karşılaştırma

| Konu | C# | Python | Java |
|---|---|---|---|
| Nesne oluşturma | `new Car()` | `Car()` | `new Car()` |
| Constructor | `public Car(...)` | `def __init__(...)` | `public Car(...)` |
| Erişim kontrolü | `private`, `public`, `protected` | Büyük ölçüde konvansiyon (`_name`), name mangling (`__name`) | `private`, `public`, `protected` ve package-private |
| Soyut sözleşme | `interface`, `abstract class` | `Protocol`, `ABC` | `interface`, `abstract class` |
| Sınıf kalıtımı | `class B : A` | `class B(A)` | `class B extends A` |
| Interface uygulaması | `class B : IExample` | `Protocol` için açık uygulama zorunlu değil | `class B implements Example` |
| Override | `override` (uygun `virtual`/`abstract` üyede) | Aynı isimli metodu yeniden tanımlama | `@Override` (işaretleme; derleyici kontrolü) |

> **Dikkat:** Bu tablo sözdizimlerini hızlı karşılaştırır; dillerin tip sistemleri ve nesne modelleri birebir aynı değildir.



##  Başvuru kaynakları

Bu dosyanın açıklama ve örnekleri genel OOP bilgisi temelinde hazırlanmıştır. Makalenin özgün bulgularıyla karıştırılmamalıdır. Kavramları doğrulamak ve derinleştirmek için:

- Microsoft Learn — [Classes and objects (C#)](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/classes)
- Microsoft Learn — [Interfaces (C#)](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces)
- Python Documentation — [Classes](https://docs.python.org/3/tutorial/classes.html)
- Python Documentation — [`abc` module](https://docs.python.org/3/library/abc.html)
- Oracle Java Tutorials — [Object-Oriented Programming Concepts](https://docs.oracle.com/javase/tutorial/java/concepts/)
- Laigner, R., Mendonça, D., Garcia, A., & Kalinowski, M. (2022). *Cataloging dependency injection anti-patterns in software systems*. **Journal of Systems and Software, 184**, 111125. https://doi.org/10.1016/j.jss.2021.111125


