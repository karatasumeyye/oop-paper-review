# 05 — Code Smells, Anti-patterns ve Refactoring


## 1. Neden bu konuları öğreniyoruz?

Önceki notlarda sınıfları, nesneler arasındaki ilişkileri, SOLID/GRASP prensiplerini ve Dependency Injection'ı öğrendik. Ancak gerçek bir kod tabanında tasarım her zaman ideal olmayabilir. Bir servis zamanla çok fazla sorumluluk üstlenebilir, gereksiz bağımlılıklar edinebilir veya kullandığı framework'e gereğinden fazla bağlanabilir. Kod hâlâ çalışıyor olsa bile değiştirilmesi ve anlaşılması zorlaşabilir.

**Code smell**, kodda daha derin bir tasarım probleminin olabileceğini düşündüren belirtidir. **Refactoring**, dışarıdan gözlenen davranışı değiştirmeden kodun iç yapısını iyileştirmeyi amaçlayan kontrollü değişikliklerdir. **Anti-pattern** ise tekrar eden, ilk bakışta işe yarar görünen fakat belirli bir bağlamda istenmeyen sonuçlara yol açabilen problemli çözüm yaklaşımıdır.

Bu ayrımlar DI araştırması için gereklidir: **DI kullanılmış olması iyi tasarımı garanti etmez.** Öte yandan bir anti-pattern tespit aracının bir kod parçasını işaretlemesi de o parçanın her koşulda hatalı olduğunu kanıtlamaz. Bağlam, araç kuralları ve doğrulama sonuçları önemlidir.

## 2. Birbirine benzeyen ama farklı terimler

| Terim | Ne anlatır? | Nasıl yorumlanmalı? |
|---|---|---|
| **Bug (hata)** | Yazılımın beklenen davranıştan sapması | Davranışsal doğruluk problemidir. |
| **Code smell** | Olası tasarım sorununun belirtisi | Araştırma/inceleme gerektirir; kesin hüküm değildir. |
| **Anti-pattern** | Tekrarlanan problemli çözüm kalıbı | Bağlamı, etkisi ve daha iyi alternatifleri birlikte değerlendirilir. |
| **Technical debt** | Kısa vadeli tercihler nedeniyle sonraki değişikliklerin zorlaşması | Her borç bilinçsiz değildir; etkisi ve geri ödeme maliyeti vardır. |
| **Refactoring** | Davranışı koruyarak iç tasarımı düzenleme | Amaç yeni özellik geliştirmek değil, yapıyı iyileştirmektir. |
| **Design pattern** | Tekrarlayan tasarım sorununa bağlama duyarlı çözüm yaklaşımı | Her projeye mekanik biçimde uygulanmaz. |

**Örnek:** `InvoiceService` içinde 400 satırlık bir `ProcessInvoice()` metodu bulunması bir **Long Method** smell'idir. Eğer yanlış vergi hesaplıyorsa ayrıca bir **bug** vardır. Metodu küçük işlevlere ayırıp davranışını korumak bir **refactoring** olabilir. Oysa vergi hesaplama kuralını değiştirmek artık davranışsal değişikliktir; yalnızca refactoring diye adlandırılmamalıdır.

### Smell her zaman kusur mudur?

Hayır. Örneğin constructor'da çok sayıda parametre bulunması, sınıfın çok fazla sorumluluğu olduğuna işaret edebilir; ancak karmaşık bir koordinatörün gerçekten çok sayıda işbirlikçisi bulunması da mümkündür. **Kod bağlamını anlamadan refactoring kararı verilmemelidir.** Bu yüzden metrikler ve statik analiz araçları araştırmaya başlamak için faydalıdır, kararın tamamı değildir.

---

## 3. Bir code smell'i teknik açıdan nasıl incelerim?

Bir kod parçasını okuduğumda şu soruları sırasıyla sorarım:

1. **Sorumluluk:** Sınıf/metot kaç farklı nedenle değişebilir? SRP ve cohesion açısından durum nedir?
2. **Bağımlılık:** Hangi somut sınıfları tanıyor? Gereksiz coupling var mı?
3. **Gizlilik:** İç durumu dışarıya gereğinden fazla açıyor mu? Encapsulation bozuluyor mu?
4. **Değiştirilebilirlik:** Yeni bir iş kuralı eklediğimde kaç yerde değişiklik yapıyorum? OCP açısından ne görüyorum?
5. **Test edilebilirlik:** Davranışı çevre sistemleri olmadan sınayabiliyor muyum?
6. **Kanıt:** Gerçek bakım zorluğu gözlemlenmiş mi, yoksa yalnızca potansiyel risk mi var?

Bu sorular, “bu kod kötü” biçimindeki öznel yargıyı teknik gerekçeye dönüştürür.

---

## 4. Temel OOP code smell'leri: C# üzerinden öğrenelim

### 4.1. Long Method — Çok uzun metot

Tek bir metot doğrulama, hesaplama, veri erişimi ve bildirim gibi farklı işleri üstleniyorsa kodun izlenmesi zorlaşabilir. Uzunluk tek başına hata değildir; belirleyici unsur **farklı sorumlulukların tek yerde toplanmasıdır**.

**Problemli örnek:**

```csharp
public void CompleteOrder(Order order)
{
    if (order.Items.Count == 0)
        throw new InvalidOperationException("Boş sipariş");

    decimal total = order.Items.Sum(x => x.Price * x.Quantity);
    _database.Save(order, total);
    _email.Send(order.CustomerEmail, $"Toplam: {total}");
}
```

Burada hesaplama, kalıcılık ve bildirim aynı akışta bir arada. Basit bir uygulamada kabul edilebilir olsa da metot büyüdükçe kararları takip etmek zorlaşır.

**Daha okunabilir bir ayrıştırma:**

```csharp
public void CompleteOrder(Order order)
{
    Validate(order);
    decimal total = CalculateTotal(order);
    _database.Save(order, total);
    _email.Send(order.CustomerEmail, $"Toplam: {total}");
}

private static void Validate(Order order)
{
    if (order.Items.Count == 0)
        throw new InvalidOperationException("Boş sipariş");
}

private static decimal CalculateTotal(Order order) =>
    order.Items.Sum(x => x.Price * x.Quantity);
```

`Extract Method` uygulanarak okunabilirlik artırıldı; dış sistemlerle ilişkiler ise henüz değişmedi. **Bu örnek, her refactoring'in bütün tasarım sorunlarını tek seferde çözemeyeceğini gösteriyor.** `_database`, `_email`, `Order` ve ilişkili üyelerin daha geniş sınıf içinde tanımlandığı varsayılmıştır.

### 4.2. Large Class / God Class — Aşırı sorumluluk

Kullanıcı yönetimi, ödeme, raporlama ve e-posta gönderimi aynı sınıfa toplanmışsa sınıf birçok farklı nedenle değişmek zorunda kalır. Bu durum çoğu zaman düşük cohesion ve zor test edilen kod anlamına gelir.

**Problemli tasarım:**

```csharp
public class BusinessManager
{
    public void RegisterUser() { /* ... */ }
    public void ChargeCard() { /* ... */ }
    public void GenerateReport() { /* ... */ }
    public void SendEmail() { /* ... */ }
}
```

**Olası iyileştirme:** `UserRegistrationService`, `PaymentService`, `ReportService`, `NotificationService` gibi gerçekten bağımsız sorumluluklara ayırmak. Ancak yalnızca isimleri değiştirmek yetmez; sınıflar arasındaki işbirliği de doğru tasarlanmalıdır.

### 4.3. Duplicate Code — Yinelenen iş mantığı

Aynı iş kuralı birden fazla noktaya kopyalanırsa kural değiştiğinde tüm kopyaların tutarlı güncellenmesi gerekir. Bu, **bakım riski** oluşturur. Ancak benzer görünen her kodu ortaklaştırmak doğru değildir: İki parça farklı nedenlerle değişiyorsa erken soyutlama yeni bir coupling doğurabilir.

```csharp
// İki sınıfta ayrı ayrı bulunuyorsa kuralın güncellenmesi zorlaşabilir:
static decimal AddTax(decimal price) => price * 1.20m;
```

Örnekteki vergi oranı sadece temsili bir sabittir; gerçek vergi uygulaması değildir. İki modül aynı iş kuralını paylaşıyorsa kuralın sahibi olan ortak bileşene taşınması düşünülebilir.

### 4.4. Feature Envy — Başka nesnenin verileriyle aşırı uğraşmak

Bir metot, kendi nesnesinin durumundan çok başka bir nesnenin alanlarını kullanıyorsa davranışın yanlış yerde konumlandırılmış olabileceğini düşünürüz.

```csharp
// Sipariş hesaplamasının başka bir hizmette çok sayıda Order iç detayı okuması:
public decimal Total(Order order) =>
    order.Items.Sum(item => item.Price * item.Quantity);
```

Basit toplam hesabı `Order` nesnesinin kendi davranışı olarak sunulabilir:

```csharp
public class Order
{
    public IReadOnlyList<OrderItem> Items { get; init; } = [];
    public decimal CalculateTotal() =>
        Items.Sum(item => item.Price * item.Quantity);
}
```

Burada temel düşünce **Information Expert** (GRASP): Bilginin sahibi olan nesne, uygun davranışın da sahibi olabilir. Ancak tüm hesaplamaları entity içine taşıma zorunluluğu yoktur; dış servisler ve domain sınırları da önemlidir.

### 4.5. Data Class / Anemic Model — Davranıştan kopuk veri modeli

Sınıf yalnızca okunup yazılan alanlar içeriyor, ilgili bütün iş kuralları farklı sınıflara dağılıyorsa domain davranışları veriyle birlikte yönetilemiyor olabilir. Bununla birlikte **DTO, API sözleşmesi veya salt veri taşıyıcıları** için veri odaklı sınıflar doğaldır; otomatik olarak smell değildir.

### 4.6. Primitive Obsession — Alan kavramlarını primitive'lerle temsil etmek

Her yerde `string email`, `decimal money`, `string currency` kullanmak, kuralların farklı yerlere dağılmasına yol açabilir. Örneğin `Money` value object'ı para miktarını ve para birimini birlikte temsil edebilir. Ama basit uygulamalarda gereksiz value object üretmek de karmaşıklığı artırabilir.

```csharp
public readonly record struct Money(decimal Amount, string Currency);
```

Bu örnek yalnızca tip modellemesini gösterir; para birimi eşleşmesi veya aritmetik kuralları ayrıca uygulanmalıdır.

### 4.7. Shotgun Surgery ve Divergent Change

**Shotgun Surgery:** Tek bir iş kuralı değişikliğinin pek çok farklı dosyayı güncellemeyi gerektirmesi. Örneğin kargo ücretindeki bir değişiklik beş servisin aynı anda değiştirilmesini gerektiriyorsa ilgili mantık dağıtılmış olabilir.

**Divergent Change:** Tek bir sınıfın birbirinden ilgisiz birçok nedenle sık sık değiştirilmesi. Örneğin hem raporlama formatı hem ödeme sağlayıcısı değiştiğinde aynı sınıfa dokunuyorsak sorumluluk sınırı sorgulanmalıdır.

İki smell birbirinin aynısı değildir: İlki **bir değişikliğin birçok yere yayılmasını**, ikincisi **tek bir yerin birçok farklı değişim nedenine sahip olmasını** vurgular.

---

## 5. Refactoring nedir ve ne değildir?

**Refactoring**, yazılımın dışarıdan gözlenen davranışını korurken iç tasarımını iyileştirmeyi amaçlar. Örneğin metot ayırmak, değişken ismini açıklayıcı hâle getirmek veya uygun durumda koşul dallanmasını polymorphism ile değiştirmek birer refactoring olabilir.

**Refactoring olmayan örnek:** `CalculateDiscount()` metodunun indirim oranını %10'dan %15'e çıkarmak. Bu bir iş kuralı değişikliğidir.

### Sık kullanılan refactoring teknikleri

| Teknik | Ne zaman düşünülebilir? | Dikkat noktası |
|---|---|---|
| **Extract Method** | Uzun/karmaşık metotlar | Yeni metot anlamlı bir sorumluluk taşımalı. |
| **Extract Class** | Bir sınıfta ilgisiz sorumluluklar | Nesneler arası gereksiz delegasyon oluşturma. |
| **Move Method** | Davranış yanlış nesnede | Nesnenin veri ve sorumluluk sahipliğini koru. |
| **Encapsulate Field** | Dış kod iç durumu doğrudan değiştiriyor | Geçerli durum kurallarını yeni API ile koru. |
| **Introduce Parameter Object** | İlişkili parametre grupları tekrar ediyor | Alakasız parametreleri yapay olarak birleştirme. |
| **Replace Conditional with Polymorphism** | Tip bazlı dallanmalar sık genişliyor | Basit tek bir `if` için soyutlama eklemek şart değil. |
| **Introduce Dependency Injection** | Somut işbirlikçi değişimi/testi zorlaştırıyor | Her nesne için interface açmak gerekmez. |

### Güvenli refactoring akışı

1. Kodun **mevcut davranışını** ve korunması gereken sözleşmeyi belirle.
2. Varsa testleri çalıştır; yoksa davranışı sabitleyen **characterization test** eklemeyi düşün.
3. Tek ve küçük bir yapısal değişiklik yap.
4. Testleri yeniden çalıştır; mümkünse statik analiz ve derleme kontrolü de uygula.
5. Okunabilirliğin, sorumlulukların ve bağımlılıkların gerçekten iyileşip iyileşmediğini değerlendir.
6. Ayrı bir iş kuralı değişikliği yapacaksan onu refactoring'den ayır.

**Not:** Testlerin geçmesi her zaman eşdeğer davranışın matematiksel kanıtı değildir; yalnızca kapsanan davranışlar için güvence sağlar.

---

## 6. C# ile adım adım refactoring: Somut bağımlılıktan DI'a

Aşağıdaki örnek `NotificationService` sınıfında e-posta göndermeyi ele alır. Başlangıçtaki problem, sınıfın göndericiyi kendi içinde oluşturmasıdır.

**Önce — Değişmesi zor somut bağlılık:**

```csharp
public class NotificationService
{
    private readonly SmtpEmailSender _sender = new();

    public void Notify(string text) => _sender.Send(text);
}
```

`SmtpEmailSender` sınıfının önceden tanımlandığı varsayılmıştır. Gerçek SMTP bağlantısı testlerin dış ortam bağımlılığını artırabilir.

**Sonra — Sözleşme ve constructor injection:**

```csharp
public interface IMessageSender
{
    void Send(string text);
}

public class SmtpEmailSender : IMessageSender
{
    public void Send(string text)
    {
        // Gerçek SMTP gönderim ayrıntıları burada yer alır.
        Console.WriteLine($"E-posta: {text}");
    }
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

// Composition Root:
var service = new NotificationService(new SmtpEmailSender());
service.Notify("Sipariş hazır");
```

**Neler değişti?**

- `NotificationService`, `SmtpEmailSender`'ı oluşturma sorumluluğunu bıraktı.
- İşbirliği `IMessageSender` sözleşmesi üzerinden kuruldu.
- Testte farklı bir uygulama verilebilir.
- Nesne oluşturma işi uygulamanın birleştirme noktasına taşındı.

**Ancak:** Bu değişiklik her projede zorunlu değildir. Çok küçük ve değişmeyecek bir işbirliğine interface eklemek soyutlama maliyeti doğurabilir. DI için interface kullanmak da zorunlu değildir; burada DI ile DIP'i birlikte göstermek için tercih edilmiştir.

### C# — Test double ile aynı davranışı denetlemek

```csharp
public sealed class RecordingSender : IMessageSender
{
    public List<string> Messages { get; } = [];
    public void Send(string text) => Messages.Add(text);
}

var fake = new RecordingSender();
var service = new NotificationService(fake);
service.Notify("Merhaba");

Console.WriteLine(fake.Messages.Single() == "Merhaba"); // True
```

Bu test gerçek SMTP gönderimini değil, servisin işbirlikçisine doğru mesajı ilettiğini kontrol eder. Testin kapsamı ile iddia edilen davranışı karıştırmamalıyım.

### Python — Aynı tasarım fikri

```python
class NotificationService:
    def __init__(self, sender):
        self.sender = sender

    def notify(self, text):
        self.sender.send(text)

class RecordingSender:
    def __init__(self):
        self.messages = []

    def send(self, text):
        self.messages.append(text)

fake = RecordingSender()
NotificationService(fake).notify("Merhaba")
assert fake.messages == ["Merhaba"]
```

Python örneğinde interface bildirimine ihtiyaç olmadan yapısal davranış üzerinden işbirliği kuruldu. İstenirse `typing.Protocol` ile sözleşme açıkça tarif edilebilir.

### Java — Aynı tasarım fikri

```java
interface MessageSender {
    void send(String text);
}

final class NotificationService {
    private final MessageSender sender;

    NotificationService(MessageSender sender) {
        this.sender = java.util.Objects.requireNonNull(sender);
    }

    void notifyUser(String text) {
        sender.send(text);
    }
}
```

Üç dilde de **tasarım ilkesi aynı**: İşbirlikçi dışarıdan sağlanıyor. Ama interface türleri, tip denetimi, framework davranışı ve yaşam döngüsü yönetimi farklı olabilir.

---

## 7. Diğer refactoring örneği: Koşullardan polymorphism'e

**Önce — Yeni ödeme türlerinde büyüyen dallanma:**

```csharp
public decimal Fee(string method, decimal amount)
{
    if (method == "card") return amount * 0.02m;
    if (method == "transfer") return 0m;
    throw new ArgumentException("Bilinmeyen ödeme yöntemi");
}
```

**Sonra — Farklı davranışları nesnelere dağıtma:**

```csharp
public interface IFeePolicy
{
    decimal Calculate(decimal amount);
}

public sealed class CardFeePolicy : IFeePolicy
{
    public decimal Calculate(decimal amount) => amount * 0.02m;
}

public sealed class TransferFeePolicy : IFeePolicy
{
    public decimal Calculate(decimal amount) => 0m;
}

public sealed class FeeService
{
    private readonly IFeePolicy _policy;
    public FeeService(IFeePolicy policy) => _policy = policy;
    public decimal Calculate(decimal amount) => _policy.Calculate(amount);
}
```

Bu örnekte OCP ve polymorphism ile ilişkili bir tasarım kullanılıyor. **Fakat küçük, sabit bir koşulda bu soyutlama gereksiz olabilir.** Ayrıca eski örneğin “bilinmeyen yöntem” hatası yeni tasarımda bağımlılığı seçen üst katmana taşınmıştır; gerçek uygulamada aynı gözlemlenebilir davranışı korumak için bu seçimin de test edilmesi gerekir. Refactoring değerlendirmesinin yalnızca kodun daha kısa görünmesine dayanmaması gerekir.

---

## 8. Code smell'ler ile SOLID/GRASP bağlantısı

| Gözlenen belirti | Özellikle sorgulanabilecek ilkeler | Olası soru |
|---|---|---|
| Tek sınıfta farklı işler | SRP, High Cohesion | Kaç farklı nedenle değişiyor? |
| Somut sınıfa aşırı bağlılık | DIP, Low Coupling | Alternatif uygulamayı nasıl koyarım? |
| Sürekli büyüyen tip koşulları | OCP, Polymorphism | Yeni tür için mevcut kodu değiştirmek zorunda mıyım? |
| Başka nesnenin verilerine aşırı erişim | Information Expert, Encapsulation | Davranış yanlış yerde mi? |
| Çok büyük interface | ISP | Tüketici kullanmadığı metotlara bağımlı mı? |
| Alt sınıfın üst sınıf sözleşmesini bozması | LSP | Üst tipin yerine güvenle geçebilir mi? |
| Bağımlılıkların global yerden aranması | DIP, Explicit Dependencies | Sınıfın gerçek ihtiyaçları nereden anlaşılıyor? |

**Dikkat:** Bu eşleştirmeler kesin teşhis değildir. Bir smell birden fazla tasarım ilkesiyle ilişkili olabilir; tek bir ilkenin ihlalini kanıtlamak için somut bağlam gerekir.

---

## 9. Statik analiz, ölçütler ve yanlış pozitifler

Statik analiz, programı normal iş akışıyla çalıştırmadan kaynak kod veya derlenmiş yapı üzerinde kurallar uygulayarak olası sorunları arar. Örneğin bir araç constructor parametrelerini sayabilir, alanlara atama yollarını inceleyebilir veya DI container'a doğrudan erişim arayabilir.

Ancak **kuralın işaretlediği her bulgu gerçek problem değildir**. Genel kod analizi için iki önemli terim:

- **Precision:** Araç tarafından pozitif olarak işaretlenenlerin ne kadarının gerçekten pozitif olduğu. `TP / (TP + FP)`.
- **Recall:** Gerçek pozitiflerin ne kadarının tespit edildiği. `TP / (TP + FN)`.

Burada `TP` doğru pozitif, `FP` yanlış pozitif, `FN` kaçırılan gerçek pozitiftir. Bu ölçütler değerlendirme kümesinin ve etiketleme yönteminin kalitesinden etkilenir. Makaledeki **relative recall** gibi belirli ölçütleri yorumlarken yazarların nasıl tanımladığını ayrıca okumalıyım; genel recall formülünü otomatik olarak onların deneysel ölçüsüyle eşitlememeliyim.

**Mini örnek:** Bir araç 20 bulgu işaretlesin; uzman değerlendirmesi bunların 15'ini doğru, 5'ini yanlış bulsun. Bu örnekte precision `15 / 20 = 0.75` olur. Gerçek problem sayısı bilinmiyorsa recall hesaplanamaz.

### Code smell tespiti ≠ Nedensel kalite kanıtı

Bir projede çok sayıda anti-pattern tespit edilmiş olması, bakım maliyetinin ne kadar arttığını doğrudan ölçmez. Bunu araştırmak için örneğin gerçek değişiklik süreleri, hata oranları, test maliyeti veya kontrollü refactoring karşılaştırmaları gerekebilir. Bu ayrımı makalenin **bulgularını değerlendirme** kısmında özellikle koruyacağım.

---

## 10. Makaledeki 12 DI anti-pattern'e hazırlık

Laigner vd. (2022) çalışmasında DI kullanımını incelemek üzere **12 anti-pattern** kataloglanır. Aşağıdaki adlar ve kısaltmalar **makalenin kataloğuna dayanır**. Türkçe açıklamalar, bu çalışma notu için hazırlanan kısa kavramsal özetlerdir; özgün makaledeki tam tanım, koşullar, örnekler ve tespit kurallarının yerine geçmez.

| Kod | Makaledeki adı | Ön hazırlık için temel düşünce |
|---|---|---|
| **IIJ** | Intransigent injection | Oluşturma anında gerekli olmayan bağımlılığın erkenden enjekte edilmesi |
| **CCI** | Concrete class injection | Soyutlama yerine somut sınıfa dayalı injection kullanılması |
| **CPM** | Complex producer method | Bağımlılık üretmesi beklenen metodun bunun dışındaki karmaşık işleri de yapması |
| **FDC** | Fat DI class | Sınıfa çok sayıda bağımlılık verilerek modülerliğin zorlaşması |
| **USI** | Useless injection | Enjekte edilen bağımlılığın kullanılmaması |
| **SDP** | Static dependence provider | Bağımlılıkları elde etmek için statik fabrika veya Service Locator benzeri kullanımlar |
| **DCC** | Direct container call | İş kodundan DI container'a doğrudan başvurulması |
| **OWI** | Open window injection | Enjekte edilmiş nesnenin başka bir metoda aktarılıp dışarıya açılması |
| **FCO** | Framework coupling | Kodun kullanılan DI framework'ünün ayrıntılarına sıkı bağlı olması |
| **ODI** | Open door injection | Container'ın enjekte ettiği bağımlılığın dışarıya açık hâle getirilmesi |
| **MAI** | Multiple assigned injection | Enjekte edilen örneğin birden çok alana atanması |
| **MFI** | Multiple forms of injection | Aynı öğe için farklı injection biçimlerinin birlikte kullanılması |

> **Kaynak notu:** Bu tablo makalenin 4. bölümündeki katalogla ilişkilidir. Buradaki kısa cümleler özellikle **OWI, ODI, MAI ve MFI** gibi birbirine benzeyen durumlarda tam uygulama/tanı koymak için yeterli değildir; bunları asıl bölümde orijinal tanımları ve kodlarıyla ayrıntılı çalışacağız.



### C# örneği — FDC için ilk işaret

```csharp
public sealed class CheckoutService
{
    public CheckoutService(
        IInventory inventory,
        IPayment payment,
        IShipping shipping,
        IDiscounts discounts,
        ITaxCalculator taxes,
        IEmailSender email)
    {
        // Bağımlılıklar saklanıyor...
    }
}
```

Çok sayıda bağımlılık bir **inceleme sinyali** olabilir. Ama bu sınıfın mutlaka FDC olduğu sonucuna yalnızca parametre sayısına bakarak varamayız. Makaledeki tanım, eşik ve tespit stratejilerini ayrıca incelemek gerekir.

### C# örneği — DCC / Service Locator benzeri kod

```csharp
public sealed class ReportService
{
    private readonly IServiceProvider _provider;

    public ReportService(IServiceProvider provider) => _provider = provider;

    public void Generate()
    {
        var exporter = _provider.GetRequiredService<IReportExporter>();
        exporter.Export();
    }
}
```

Burada gerçek işbirlikçi `IReportExporter`, constructor imzasında açıkça görünmüyor; sınıf container bilgisine de bağlı. İş mantığının içinde container çağırmak DI kullanımındaki sorunlu yaklaşımlardan biri olabilir. **Ancak makaledeki DCC'nin Java bağlamındaki kesin tespit kuralı ile bu .NET örneğinin birebir özdeş olduğunu iddia etmiyoruz.** `GetRequiredService` için uygun .NET DI namespace'lerinin dahil edilmesi gerekir.

**Daha açık bağımlılık:**

```csharp
public sealed class ReportService
{
    private readonly IReportExporter _exporter;

    public ReportService(IReportExporter exporter) => _exporter = exporter;

    public void Generate() => _exporter.Export();
}
```

Bu örnekte sınıfın ihtiyaç duyduğu hizmet API üzerinden anlaşılır. Fakat bazı framework sınırlarında container'a erişim meşru olabilir; örneğin composition root içindeki servis çözümlemesi aynı anlama gelmez.

### Python ve Java'da olası karşılıklar

**Python — Global registry üzerinden bağımlılığı aramak:**

```python
services = {"exporter": SomeExporter()}  # SomeExporter önceden tanımlı

class ReportService:
    def generate(self):
        services["exporter"].export()
```

Bu, dış bağımlılığı örtülü hâle getirebilir. Birebir Java DCC tespiti değildir; **tasarım analojisi**dir.

**Java — Bir Service Locator üzerinden bağımlılık edinmek:**

```java
final class ReportService {
    void generate() {
        ReportExporter exporter = ServiceLocator.get(ReportExporter.class);
        exporter.export();
    }
}
```

`ServiceLocator` ve `ReportExporter` örnek için var sayılmıştır. Kodun amacı DI container ya da locator çağrısının iş kodu içine girmesini görünür kılmaktır.



## Terimler sözlüğü

| Terim | Açıklama |
|---|---|
| Code smell | Kodda olası tasarım problemine işaret eden belirti |
| Anti-pattern | Tekrarlanan ve bağlam içinde istenmeyen sonuçlara yol açabilen problemli çözüm yaklaşımı |
| Bug | Beklenen davranıştan sapma |
| Refactoring | Dışarıdan gözlenen davranışı koruyarak iç tasarımı iyileştirme |
| Technical debt | Daha sonra değişim/bakım maliyeti doğurabilen tasarım veya geliştirme tercihlerinin yükü |
| Extract Method | Bir kod bloğunu ayrı, anlamlı bir metoda taşıma |
| Extract Class | Ayrı sorumlulukları yeni sınıfa ayırma |
| Move Method | Davranışı daha uygun nesneye taşıma |
| Encapsulate Field | Doğrudan alan erişimini kontrollü API ile yönetme |
| Feature Envy | Bir davranışın ağırlıklı olarak başka nesnenin verilerini kullanması |
| Long Method | Bir metodun anlamayı zorlaştıracak kadar karmaşık/büyük olması |
| Large Class / God Class | Bir sınıfın aşırı ve farklı sorumlulukları üstlenmesi |
| Shotgun Surgery | Tek değişikliğin birçok yerde değişiklik gerektirmesi |
| Divergent Change | Bir sınıfın pek çok farklı nedenle değişmesi |
| Static Analysis | Programı olağan çalıştırma akışı olmadan kod üzerinde inceleme |
| False Positive | Gerçek problem olmadığı hâlde problem diye işaretlenen bulgu |
| False Negative | Gerçek problem olduğu hâlde tespit edilmeyen örnek |
| Precision | Pozitif işaretlenen örneklerin doğru çıkma oranı |
| Recall | Gerçek pozitif örneklerin tespit edilme oranı |
| Characterization Test | Mevcut sistem davranışını kayıt altına almayı amaçlayan test |

---


## Kaynaklar ve çalışma sınırları

**Ana makale (12 anti-pattern adlarının dayanağı):**

- Laigner, R., Mendonça, D., Garcia, A., & Kalinowski, M. (2022). *Cataloging dependency injection anti-patterns in software systems*. *Journal of Systems and Software*, 184, 111125. https://doi.org/10.1016/j.jss.2021.111125

**Ön hazırlık için temel başvuru eserleri (makalenin deneysel bulgularının kaynağı değildir):**

- Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code* (2. baskı). Addison-Wesley.
- Fowler, M. — Refactoring Catalog: https://refactoring.com/catalog/
- Microsoft Learn — Refactoring code (Visual Studio): https://learn.microsoft.com/en-us/visualstudio/ide/refactoring-in-visual-studio
- Microsoft Learn — Dependency injection guidelines: https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines


