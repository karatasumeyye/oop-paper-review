## 7. Updating the Catalog — Kataloğun Güncellenmesi

Yazarlar, geliştirici görüşmeleri ve anketlerden elde edilen geri bildirimler doğrultusunda DI anti-pattern kataloğunu yeniden değerlendirmiştir. Katılımcıların bazı anti-pattern'lerin uygulanabilirliğine yönelik çekinceleri, mevcut tanımların belirli koşulları yeterince açıklamadığını ortaya koymuştur. Bu nedenle katalogdaki 12 anti-pattern korunurken, problemlerin kapsamı ve uygulanma koşulları daha açık biçimde tanımlanmıştır.

Güncellemenin temel amacı, bir kod örüntüsünün hangi durumlarda gerçek bir tasarım problemi oluşturduğunu ve önerilen refactoring işlemlerinin hangi koşullarda anlamlı olduğunu açıklığa kavuşturmaktır.

### 7.1. İlk Katalog ile Güncellenmiş Kataloğun Karşılaştırılması

Aşağıdaki tablo, makalenin 7. bölümündeki **Table 27** ve güncellenmiş katalog temel alınarak hazırlanmıştır.

| Anti-pattern | Önceki Katalog / Tartışılan Problem | Güncellenmiş Katalog |
|---|---|---|
| **IIJ – Intransigent Injection** | İhtiyaç duyulmadan erken sağlanan bağımlılıklar performans problemi olarak değerlendiriliyordu. | Problemin bağlama bağlı olduğu vurgulandı. Bağımlılığın daha geç oluşturulmasının gereksiz iş yükünü azaltabileceği durumlar netleştirildi. |
| **CCI – Concrete Class Injection** | Somut sınıfa bağımlılık kurulması yerine interface kullanılması öneriliyordu. | Küçük sistemlerde interface kullanımının her zaman gerekli olmayabileceği, ihtiyaç ortaya çıktığında soyutlama oluşturulabileceği belirtildi. |
| **CPM – Complex Producer Method** | Karmaşık Producer Method yapısının genel *Long Method* problemiyle örtüşüp örtüşmediği sorgulandı. | Problemin yalnızca uzun metotlardan ibaret olmadığı; Producer Method'un bağımlılık sağlama sorumluluğu dışındaki işlemleri üstlenmesiyle ilişkili olduğu netleştirildi. |
| **FDC – Fat DI Class** | Çok sayıda bağımlılık içeren sınıfların genel *God Class* problemiyle ilişkisi sorgulandı. | Problemin özellikle gerekçelendirilmeden eklenen injection bağımlılıkları ve bunların sınıfın diğer bileşenlerle ilişkisini artırmasıyla ilgili olduğu vurgulandı. |
| **USI – Useless Injection** | Kullanılmayan bağımlılıkların DI'a özgü bir anti-pattern olup olmadığı sorgulandı. | Injection anotasyonu bulunan kullanılmayan alanların IDE tarafından her zaman raporlanmaması, problemin DI bağlamındaki ayırt edici yönü olarak açıklandı. |
| **SDP – Static Dependence Provider** | Static Factory veya Service Locator kullanımı, bağımlılık yönetiminde sorun olarak değerlendiriliyordu. | Farklı framework'lerin entegrasyonu gibi özel gereksinimlerin bulunabileceği ve bu durumların anti-pattern kapsamının dışında değerlendirilebileceği açıklandı. |
| **DCC – Direct Container Call** | DI container'a doğrudan erişim, mimari bağımlılık problemi olarak tanımlanıyordu. | Çalışma zamanında dinamik bağımlılık çözümlemesi gereken durumlar dikkate alındı. Bu tür ihtiyaçlarda container çağrısının bir Provider sınıfında kapsüllenmesi önerildi. |
| **OWI – Open Window Injection** | Enjekte edilen bağımlılığın başka bileşenlere aktarılması problem olarak değerlendiriliyordu. | DI container dışında çalışan bileşenlere veya üçüncü taraf sistemlere bağımlılık aktarımının gerekli olabileceği belirtildi. |
| **FCO – Framework Coupling** | Framework'e özgü DI yapıları yerine standart tanımlamaların kullanılması öneriliyordu. | Standart DI tanımlamalarına geçişin özellikle yeni projelerde daha anlamlı olabileceği, mevcut sistemlerde dönüşüm maliyetinin dikkate alınması gerektiği vurgulandı. |
| **Katalog Yapısı** | Anti-pattern tanımlarında uygulanma koşulları ayrı ve sistematik bir alan olarak sunulmuyordu. | **Context** alanı eklenerek her anti-pattern'in hangi koşullarda geçerli olduğu açıkça belirtilmeye başlandı. |

**Not:** Table 27'de ODI, MAI ve MFI için ayrıca bir güncelleme raporlanmamıştır. Bu üç anti-pattern güncellenmiş katalogda bulunmaya devam etmektedir. CPM ise güncellenmiş katalogda *Long Producer Method* adıyla sunulmaktadır.

### 7.2. Güncellenmiş Kataloğun Sınıflandırılması

Güncellenmiş katalogda 12 DI anti-pattern, ele aldıkları temel problem türlerine göre dört kategoride gruplandırılmıştır.

| Kategori | Kapsam | Anti-pattern'ler |
|---|---|---|
| **Design** | Sınıf tasarımı, sorumluluk dağılımı, encapsulation ve bağımlılık ilişkileri | CCI, OWI, ODI, FDC, MAI, CPM/LPM |
| **Architecture** | Bağımlılık sağlama mekanizmaları, IoC ve DIP ile ilişkili mimari problemler | SDP, DCC |
| **Performance** | Gereksiz bağımlılıkların oluşturulması veya yönetilmesinden kaynaklanabilecek kaynak tüketimi | IIJ, USI |
| **Standardization** | DI framework'lerine bağımlılık ve injection kullanımındaki tutarsızlıklar | FCO, MFI |

Bu sınıflandırma, anti-pattern'lerin farklı kalite ve tasarım boyutlarına odaklandığını göstermektedir. Özellikle Design ve Architecture kategorilerinin ayrılması, sınıf düzeyindeki tasarım problemleri ile bağımlılık yönetiminin mimari düzeydeki problemlerini birbirinden ayırmayı kolaylaştırmaktadır.

### 7.3. Eleştirel Değerlendirme

Güncellemeler incelendiğinde, kataloğun en önemli gelişiminin **anti-pattern tanımlarına uygulama bağlamının dâhil edilmesi** olduğu görülmektedir. İlk tanımlar üzerinden problemli kabul edilebilecek bazı kod örüntülerinin, belirli mimari veya işlevsel gereksinimler doğrultusunda kabul edilebilir olabileceği açıklığa kavuşturulmuştur. Bu yaklaşım, nesne yönelimli tasarım ilkelerinin uygulama koşullarından bağımsız ve kesin kurallar olarak değerlendirilmemesi gerektiğini desteklemektedir.

Özellikle CCI ve FCO örnekleri, tasarım esnekliği ile geliştirme ve bakım maliyetleri arasındaki dengeyi göstermektedir. Soyutlamaların artırılması veya framework bağımlılıklarının azaltılması teorik olarak avantaj sağlayabilse de bu işlemler her sistem için aynı ölçüde gerekli olmayabilir. Dolayısıyla iyi tasarım, yalnızca belirli OOP prensiplerine uyulmasıyla değil, bu prensiplerin sistem gereksinimlerine uygun uygulanmasıyla ilişkilidir.

Bununla birlikte, **bağlamsal koşulların kataloğa eklenmesi, bu koşulların DIAnalyzer tarafından otomatik olarak değerlendirilebildiğini göstermemektedir.** Güncellenen tanımlar ile aracın tespit kuralları arasındaki uyumun ayrıca incelenmesi önemli bir geliştirme alanıdır. Özellikle mimari bağlamın dikkate alınması, yanlış pozitif sonuçların azaltılmasına ve geliştiricilere sunulan tespitlerin daha anlamlı hâle getirilmesine katkı sağlayabilir.

Kataloğun gelecekte anti-pattern'leri yalnızca tanımlamakla kalmayıp **uygulanma koşulları, olası kalite etkileri ve refactoring maliyetleri** açısından da değerlendiren bir yapıya dönüştürülmesi önerilebilir. Böylece geliştiriciler, tespit edilen problemin yalnızca varlığını değil, giderilmesinin gerekli ve öncelikli olup olmadığını da değerlendirebilir.



