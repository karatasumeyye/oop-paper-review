### 4.1. Intransigent Injection (IIJ)

Intransigent Injection, bir sınıfın oluşturulması sırasında henüz ihtiyaç duyulmayan bağımlılıkların constructor üzerinden DI container tarafından sağlanmasıyla ortaya çıkan anti-pattern'dir. *Copy-paste programming*, *ad hoc maintenance* ve test edilebilirliği kolaylaştırma amacı, bu durumun olası nedenleri arasında gösterilmektedir.

Buradaki temel problem bağımlılıkların sayısı veya tamamen gereksiz olmaları değil, **ihtiyaç duyulmadan önce oluşturulmaları ya da sağlanmalarıdır.** Özellikle oluşturulması maliyetli bağımlılıklar, başlangıç süresini ve bellek tüketimini artırarak performansı olumsuz etkileyebilir. Ayrıca bağımlılıkların kullanım zamanının belirsizleşmesi kodun anlaşılabilirliğini ve bakımını zorlaştırabilir.

Yazarlar, çözüm olarak *lazy injection* yaklaşımını ve bağımlılıkların yalnızca ihtiyaç duyulduğunda elde edilmesini sağlayan `Provider` interface'ini önermektedir. Bununla birlikte, bu yaklaşımın ek soyutlama ve karmaşıklık oluşturabileceği de dikkate alınmalıdır.

Bu anti-pattern, DI kullanımında yalnızca bağımlılıkların nasıl sağlandığının değil, **ne zaman sağlandığının da önemli olduğunu** ortaya koymaktadır. Dolayısıyla refactoring kararı, erken bağımlılık oluşturmanın maliyeti ile *lazy injection* uygulamasının getireceği karmaşıklık birlikte değerlendirilerek verilmelidir.

> IIJ, OOP'deki nesne oluşturma (object instantiation), nesne yaşam döngüsü (object lifecycle) ve composition kavramlarıyla ilişkilidir. Bir nesnenin oluşturulması, tüm bağımlılıklarının aynı anda kullanılmasını gerektirmez. Bu nedenle nesneler arasındaki ilişkilerin kurulması ile bağımlı nesnelerin oluşturulma zamanı ayrı değerlendirilmelidir. IIJ, OOP tasarımında yalnızca nesneler arasındaki ilişkilerin değil, yaşam döngülerinin yönetiminin de önemli olduğunu göstermektedir.


### 4.2. Concrete Class Injection (CCI)

Concrete Class Injection, Dependency Injection aracılığıyla sağlanan bir bağımlılığın interface yerine doğrudan somut bir sınıf (*concrete class*) üzerinden tanımlanmasıyla ortaya çıkan anti-pattern'dir. Bu yaklaşım, sınıfın belirli bir uygulama detayına bağımlı hâle gelmesine neden olarak Dependency Inversion Principle (DIP) ile çelişmektedir.

Yazarlar, bu anti-pattern'in ortaya çıkmasında *copy-paste programming*, dış framework bağımlılıkları ve hızlı çözüm geliştirme (*quick-fix*) yaklaşımlarını olası nedenler arasında göstermektedir. Somut sınıflara doğrudan bağımlılık kurulması; **changeability, extensibility, reusability ve testability** gibi kalite özelliklerini olumsuz etkileyebilir. Özellikle farklı uygulamaların kullanılması veya testlerde *mock* nesnelerle bağımlılıkların değiştirilmesi zorlaşabilir.

Önerilen refactoring, somut sınıfa olan bağımlılığın bir interface aracılığıyla yeniden düzenlenmesidir. Böylece bağımlılığı kullanan sınıf, belirli bir implementasyon yerine soyutlamaya bağımlı hâle gelir. Framework tarafından sağlanan somut bağımlılıklarda ise *Adapter* tasarım örüntüsü alternatif bir çözüm olarak değerlendirilmektedir.

Bununla birlikte, her somut sınıf bağımlılığının aynı düzeyde tasarım problemi oluşturduğu söylenemez. Özellikle küçük ölçekli, kısa ömürlü veya prototip niteliğindeki projelerde ek soyutlama oluşturulması gereksiz karmaşıklığa yol açabilir. Bu nedenle refactoring kararı, sistemin değiştirilebilirlik ve bakım gereksinimleri doğrultusunda değerlendirilmelidir.

Bu anti-pattern, DI mekanizmasının kullanılmasının tek başına DIP'e uygun bir tasarım sağlamadığını göstermektedir. **Bağımlılığın dışarıdan sağlanması kadar, hangi tür üzerinden tanımlandığı da önemlidir.**

> CCI, OOP'nin abstraction ve polymorphism mekanizmaları ile Dependency Inversion Principle (DIP) kapsamında değerlendirilebilir. Interface kullanımı, farklı sınıfların ortak bir sözleşme üzerinden kullanılmasını sağlar. Polymorphism ise bu sınıfların birbirinin yerine geçebilmesine olanak tanır. Somut sınıfa doğrudan bağımlılık kurulması bu esnekliği sınırlayabilir. Dolayısıyla CCI, DI ile bağımlılık sağlanmasının tek başına soyutlamaya dayalı bir OOP tasarımı oluşturmadığını göstermektedir.


### 4.3. Complex Producer Method (CPM)

Complex Producer Method, temel amacı bağımlılık sağlamak olan *Producer Method* içerisinde bu sorumluluğun kapsamını aşan işlemlerin gerçekleştirilmesiyle ortaya çıkan anti-pattern'dir. Bu durum genellikle uzun, karmaşık ve düşük cohesion'a sahip metot tasarımlarıyla ilişkilidir.

Yazarlar, CPM'nin ortaya çıkmasında iki temel neden belirlemektedir: *External Dependencies* ve *Complex Business Rules*. Bağımlılığın oluşturulması sırasında veritabanı veya harici uygulamalardan bilgi alınması gerekebilir. Benzer şekilde, hangi nesnenin oluşturulacağına karar vermek karmaşık iş kurallarının uygulanmasını gerektirebilir. Bu işlemlerin doğrudan Producer Method içerisinde yürütülmesi, metodun sorumluluklarını artırabilmektedir.

CPM'nin olumsuz etkileri özellikle **modularity** ve **understandability** kalite özelliklerinde görülmektedir. Karmaşık bağımlılık oluşturma süreçleri, değişen gereksinimlere uyum sağlamayı ve kaynak kodun anlaşılmasını zorlaştırabilir.

Önerilen refactoring yaklaşımında, bağımlılığın elde edilmesi için kapsamlı iş mantığı gerekiyorsa bu mantığın ayrı bir *business method* içerisinde yürütülmesi ve uygun durumlarda `@Produces` anotasyonunun kaldırılması önerilmektedir. Ayrıca metodun *cyclomatic complexity* değerinin azaltılması ve *Aspect-Oriented Programming (AOP)* kullanılarak bazı işlemlerin Provider Method'un yaşam döngüsüne bağlı biçimde yürütülmesi alternatif çözümler arasındadır.

Bununla birlikte, bağımlılığın oluşturulması zorunlu olarak karmaşık işlemler gerektiriyorsa veya legacy sistemlerde bağımlılıkların işleyişi yeterince anlaşılmıyorsa refactoring'in maliyeti sağlayacağı faydayı aşabilir. Bu nedenle çözümün uygulanabilirliği *cost–benefit analysis* ile değerlendirilmelidir.

Anti-pattern, endüstriyel projelerdeki kaynak kod incelemelerinden hareketle **tümevarımsal (inductive)** yöntemle belirlenmiştir. Yazarlar, karmaşık bağımlılık oluşturma süreçlerine az sayıda örnekte rastladıklarını belirtmektedir. Dolayısıyla CPM'nin pratikteki yaygınlığı ve farklı projelerdeki karşılığı, sonraki ampirik bulgular ışığında değerlendirilmelidir.

> **OOP ile İlişkisi:** CPM, özellikle **Single Responsibility Principle (SRP)** ve **High Cohesion (GRASP)** ile ilişkilidir. Bir metodun bağımlılık sağlama dışında farklı iş sorumluluklarını üstlenmesi, sorumlulukların ayrıştırılması ve ilişkili işlemlerin bir arada tutulması açısından sorun oluşturabilir. Ayrıca *modularity*, sınıfların ve metotların belirli görevler etrafında yapılandırılmasıyla ilişkilidir. Bu nedenle CPM, nesne yönelimli tasarımda yalnızca sınıfların değil, metotların da açık ve tutarlı sorumluluklara sahip olması gerektiğini göstermektedir.

### 4.4. Fat DI Class (FDC)

Fat DI Class, bir sınıfa DI aracılığıyla çok sayıda bağımlılık sağlanması sonucunda sınıfın karmaşıklaşması ve diğer bileşenlere olan bağımlılığının artmasıyla ortaya çıkan anti-pattern'dir. DI kullanımı, tek başına düşük *coupling* veya iyi bir sorumluluk dağılımı sağlamamaktadır.

Yazarlar, FDC'nin nedenlerini **Arbitrary Evolutionary Maintenance, Lack of Refactoring ve Inadequate Design Analysis** olarak açıklamaktadır. Yeni gereksinimlerle sınıfa sürekli bağımlılık eklenmesi ve tasarımın yeterince gözden geçirilmemesi, sınıfın zamanla farklı sorumluluklar üstlenmesine neden olabilir. Bu durum *understandability, changeability, extensibility, reusability* ve *testability* özelliklerini olumsuz etkileyebilir.

Önerilen çözüm, **Extract Class refactoring** ile ilişkili sorumlulukların ve bağımlılıkların uygun sınıflara ayrılmasıdır. Ancak bu işlem yeni bağımlılıklar ve bakım maliyetleri oluşturabileceğinden *cost–benefit analysis* gerektirmektedir. FDC, endüstriyel proje gözlemlerinden hareketle tümevarımsal (*inductive*) yöntemle belirlenmiştir.

>**OOP ile İlişkisi:** FDC, **Single Responsibility Principle (SRP), High Cohesion ve Low Coupling (GRASP)** ile ilişkilidir. Bir sınıfın çok sayıda bileşene bağımlı olması, sorumlulukların yeterince ayrıştırılmadığına işaret edebilir. SRP ve High Cohesion sorumlulukların tutarlı biçimde dağıtılmasını, Low Coupling ise sınıflar arasındaki gereksiz bağımlılıkların azaltılmasını destekler. Extract Class, bu ilkeler doğrultusunda sınıfın yeniden yapılandırılmasını sağlayabilir.

Bağımlılık sayısının fazla olması tek başına anti-pattern kanıtı değildir; bağımlılıkların kullanım amaçları ve sınıfın sorumlulukları birlikte incelenmelidir. Ayrıca sınıfın bölünmesi her durumda daha iyi bir tasarım sağlamayabilir.

### 4.5. Useless Injection (USI)

Useless Injection, DI aracılığıyla bir sınıfa sağlanan bağımlılığın, sınıf içerisinde kullanılmamasıyla ortaya çıkan anti-pattern'dir. Bağımlılık tanımlanmış olmasına rağmen sınıfın metotlarında bu bağımlılıktan yararlanılmamaktadır.

Yazarlar, problemin nedenlerini **Refactoring Residue, Speculative Design ve Fear of Breaking Code** olarak açıklamaktadır. Refactoring sonrasında kullanılmayan bağımlılıkların kaldırılmaması, gelecekte ihtiyaç duyulabileceği düşüncesiyle bağımlılık eklenmesi veya mevcut kodu bozma endişesi bu anti-pattern'in oluşmasına yol açabilmektedir.

USI özellikle *performance* ve *understandability* özelliklerini etkileyebilir. Kullanılmayan bağımlılığın DI container tarafından sağlanması gereksiz işlem maliyeti oluşturabilir. Bu etki, bağımlılığın oluşturulması maliyetli olduğunda veya *singleton* dışında bir yaşam döngüsüne sahip olduğunda artabilir. Ayrıca kullanılmayan bağımlılıkların kodda bulunması, sınıfın gerçek gereksinimlerinin anlaşılmasını zorlaştırabilir.

Önerilen refactoring, kullanılmayan bağımlılığın kaldırılmasıdır. Ancak bağımlılığın kalıtım yoluyla alt sınıflarda kullanılıp kullanılmadığı kontrol edilmelidir. USI, **tümevarımsal (inductive)** yöntemle belirlenmiştir. Yazarların incelediği commit geçmişinde, birçok örneğin refactoring sonrasında ortaya çıktığı gözlemlenmiştir.

>USI, özellikle **coupling ve sınıflar arası bağımlılık yönetimi** ile ilişkilidir. OOP'de bir sınıfın başka bir sınıfa bağımlı olması, belirli bir işlevsel gereksinimle gerekçelendirilebilmelidir. Kullanılmayan bağımlılıklar ise sınıfın ihtiyaç duymadığı ilişkileri sürdürmesine neden olur. Bu durum *Low Coupling (GRASP)* ilkesi açısından değerlendirilebilir.

 USI ile IIJ birbirinden farklıdır. IIJ'de bağımlılık daha sonra kullanılabilir ancak erken sağlanmaktadır. USI'de ise bağımlılık sınıf içerisinde hiç kullanılmamaktadır. Bu nedenle kaldırma işleminden önce kalıtım ilişkileri ve olası dolaylı kullanımlar kontrol edilmelidir.

### 4.6. Static Dependence Provider (SDP)

Static Dependence Provider, DI kullanılan bir sistemde bağımlılıkların DI container tarafından sağlanması yerine *Static Factory* veya *Service Locator* gibi mekanizmalar aracılığıyla doğrudan elde edilmesiyle ortaya çıkan anti-pattern'dir. Bu yaklaşım, bağımlılık yönetiminin sorumluluğunu yeniden bağımlılığı kullanan sınıfa aktararak DI'ın temel amaçlarından biri olan *Inversion of Control (IoC)* yaklaşımını zayıflatmaktadır.

Yazarlar, bu problemin özellikle mevcut sistemlerin DI mimarisine uyarlanması, harici kütüphanelerin kullanılması ve bağımlılık oluşturma süreçlerinin karmaşıklığı nedeniyle ortaya çıkabileceğini belirtmektedir. SDP, sınıfların bağımlılık sağlama mekanizmalarına doğrudan bağlanmasına neden olarak *coupling* düzeyini artırabilir; kodun anlaşılabilirliğini, değiştirilebilirliğini ve test edilebilirliğini olumsuz etkileyebilir.

Önerilen refactoring, bağımlılık oluşturma sorumluluğunun bir *Producer Method* veya DI yapılandırma bileşenine taşınması ve bağımlılığın doğrudan injection yoluyla sağlanmasıdır. Böylece sınıfın Service Locator veya Static Factory mekanizmasını bilme gereksinimi ortadan kaldırılmaktadır. Ancak harici kütüphaneler ve mevcut sistemlerin mimari kısıtları nedeniyle dönüşümün maliyeti ile sağlayacağı fayda birlikte değerlendirilmelidir.

> SDP, özellikle **Inversion of Control (IoC), Dependency Inversion Principle (DIP) ve Low Coupling (GRASP)** ile ilişkilidir. IoC, bağımlılık sağlama kontrolünün ilgili sınıfın dışına aktarılmasını; DIP, üst düzey bileşenlerin somut uygulamalar yerine soyutlamalara bağımlı olmasını destekler. Service Locator kullanımı, bağımlılık bir interface üzerinden alınsa bile sınıfı bağımlılık bulma mekanizmasına bağlayabilir. Bu nedenle SDP, soyutlamaya bağımlı olmanın tek başına yeterli olmadığını, **bağımlılığın elde edilme yönteminin de nesne yönelimli tasarım açısından önemli olduğunu** göstermektedir.

Static Factory veya Service Locator kullanımı her durumda anti-pattern olarak değerlendirilmemelidir. Özellikle legacy sistemlerde ve DI desteği bulunmayan kütüphanelerde mevcut yaklaşımın değiştirilmesi ek karmaşıklık oluşturabilir. Refactoring kararı sistemin mimari gereksinimleri doğrultusunda verilmelidir.

### 4.7. Direct Container Call (DCC)

Direct Container Call, bir sınıfın bağımlılıklarını DI mekanizması aracılığıyla almak yerine, **DI container'a doğrudan erişerek elde etmesiyle** ortaya çıkan anti-pattern'dir. Bu yaklaşım, uygulama kodunu belirli bir DI framework'ünün bileşenlerine bağımlı hâle getirerek *Dependency Inversion Principle (DIP)* ve *Inversion of Control (IoC)* yaklaşımlarıyla çelişmektedir.

Yazarlar, DCC'nin olası nedenlerinin Static Dependence Provider (SDP) ile benzer olduğunu belirtmektedir. Özellikle mevcut sistemlerin mimari kısıtları ve bazı bileşenlerde doğrudan injection desteğinin bulunmaması, geliştiricileri container çağrılarına yöneltebilir. Bu durum, framework'e yönelik *coupling* düzeyini artırarak özellikle **modularity** kalite özelliğini olumsuz etkileyebilir.

Önerilen refactoring, doğrudan container çağrılarının kaldırılarak bağımlılıkların Constructor Injection veya Field Injection gibi yöntemlerle sağlanmasıdır. Karmaşık bağımlılık oluşturma süreçlerinde *Provider* sınıfından yararlanılabilir. Ancak doğrudan container çağrılarının geliştirme süresini kısaltabildiği durumlarda, refactoring maliyeti ile uzun vadeli teknik borç birlikte değerlendirilmelidir.

DCC, endüstriyel projelerdeki gözlemlerden hareketle **tümevarımsal (inductive)** yöntemle belirlenmiştir.

> DCC, özellikle **IoC, DIP ve Low Coupling (GRASP)** ile ilişkilidir. Bir sınıfın bağımlılıklarını doğrudan container'dan araması, bağımlılık sağlama sorumluluğunu sınıfın kendisine yükler. Böylece sınıf, ihtiyaç duyduğu nesnenin yanında belirli bir framework'ün API'sini de bilmek zorunda kalır. IoC ve Low Coupling açısından amaç, bu altyapı bağımlılığını azaltarak nesnenin kendi işlevsel sorumluluğuna odaklanmasını sağlamaktır.

 DCC ile SDP benzer problemlerdir; ancak DCC doğrudan **DI container API'sine erişimi**, SDP ise *Static Factory* veya *Service Locator* üzerinden bağımlılık elde edilmesini ele alır. Ayrıca her container çağrısı anti-pattern değildir; uygulamanın başlangıcındaki merkezi bağımlılık yapılandırması ile iş mantığı içerisindeki doğrudan container erişimi birbirinden ayrılmalıdır.


### 4.8. Open Window Injection (OWI)

Open Window Injection, bir sınıfa DI aracılığıyla sağlanan bağımlılığın başka sınıfların kullanımına açılması veya aktarılmasıyla ortaya çıkan anti-pattern'dir. Bu durumda sınıf, kendi bağımlılıklarını kullanmanın yanında diğer bileşenler için bağımlılık sağlayan bir aracı hâline gelmektedir.

Yazarlar, bu yaklaşımın özellikle mevcut kodun yeniden kullanılması, bağımlılıklara erişimin kolaylaştırılması ve tasarımın yeterince gözden geçirilmemesi gibi nedenlerle ortaya çıkabileceğini belirtmektedir. Enjekte edilen nesnelerin metot parametreleri veya erişim metotları aracılığıyla dışarıya açılması, bileşenler arasındaki bağımlılık ilişkilerini karmaşıklaştırabilir ve *modularity* ile *understandability* özelliklerini olumsuz etkileyebilir.

Önerilen refactoring, bağımlılığın aracı sınıf üzerinden aktarılması yerine, ihtiyaç duyan sınıfa doğrudan DI mekanizmasıyla sağlanmasıdır. Böylece bağımlılık yönetimi ilgili bileşenin kendi yapısında tanımlanır. Ancak harici kütüphaneler veya mevcut mimari kısıtlar nedeniyle bu dönüşümün her durumda uygulanabilir olmayabileceği dikkate alınmalıdır.

> OWI, özellikle **Encapsulation, Low Coupling (GRASP) ve nesneler arası sorumluluk dağılımı** ile ilişkilidir. Encapsulation, bir nesnenin iç yapısının gereksiz yere dışarıya açılmasını sınırlandırır. Enjekte edilen bağımlılıkların başka sınıflara aktarılması ise sınıfların birbirlerinin iç bağımlılıklarını bilmesine neden olabilir. Low Coupling açısından önemli olan, nesnelerin ihtiyaç duydukları hizmetleri aracı sınıflara gereksiz bağımlılık oluşturmadan elde edebilmesidir.

Bir nesnenin metot parametresi olarak aktarılması tek başına anti-pattern değildir. OWI açısından belirleyici olan, DI ile sağlanan bağımlılığın başka bir sınıfın bağımlılık ihtiyacını karşılamak amacıyla dışarıya açılmasıdır. Refactoring kararı, bu aracılığın gerçekten gereksiz olup olmadığı değerlendirilerek verilmelidir.


### 4.9. Framework Coupling (FCO)

Framework Coupling, kaynak kodun belirli bir DI framework'üne özgü anotasyonlara veya yapılandırma bileşenlerine bağımlı hâle gelmesiyle ortaya çıkan anti-pattern'dir. Bu bağımlılık, uygulamanın farklı DI framework'lerine uyarlanmasını ve mevcut framework'ün değiştirilmesini zorlaştırabilir.

Yazarlar, FCO'nun olası nedeni olarak **Copy-paste Programming** yaklaşımını belirtmektedir. Geliştiricilerin mevcut kodları örnek alarak framework'e özgü anotasyonları kullanmaya devam etmesi, bu bağımlılığın sistem genelinde yayılmasına neden olabilir. FCO özellikle *modularity* ve *compatibility* kalite özelliklerini etkileyerek framework değişikliklerinde veya sürüm güncellemelerinde bakım maliyetini artırabilir.

Önerilen refactoring, framework'e özgü yapılar yerine programlama platformunun standart DI tanımlamalarının kullanılmasıdır. Makaledeki örnekte Spring framework'üne özgü `@Autowired` anotasyonu, JSR-330 standardındaki `@Inject` ile değiştirilmiştir. Böylece uygulama kodunun belirli bir framework'e bağımlılığı azaltılmaya çalışılmaktadır.

Bununla birlikte, mevcut sistemlerde DI framework'ünün tamamen değiştirilmesi nadir gerçekleşebilir. Bu nedenle framework bağımlılığını azaltmanın sağlayacağı fayda, dönüşüm maliyeti ve sistemin uzun vadeli gereksinimleriyle birlikte değerlendirilmelidir. FCO, açık kaynak projelerdeki gözlemlerden hareketle **tümevarımsal (inductive)** yöntemle belirlenmiştir.

>FCO, özellikle **Low Coupling (GRASP)** ve *modularity* ile ilişkilidir. Nesne yönelimli tasarımda bileşenlerin belirli uygulama ayrıntılarından mümkün olduğunca bağımsız olması, değiştirilebilirliği destekler. Framework'e özgü anotasyonların sınıflara yayılması ise bağımlılığın somut sınıflar arasında değil, altyapı düzeyinde oluşmasına neden olur. Bu durum, gevşek bağlılığın yalnızca sınıflar arasında değil, uygulama ile kullanılan framework arasında da değerlendirilmesi gerektiğini göstermektedir.

Framework'e özgü her anotasyonun kullanılması mutlaka refactoring gerektirmez. Ayrıca standart `@Inject` kullanılması, uygulamanın bütün framework bağımlılıklarını ortadan kaldırmaz. Bu değişiklik yalnızca bağımlılık tanımlama düzeyindeki framework bağımlılığını azaltabilir.


### 4.10. Open Door Injection (ODI)

Open Door Injection, DI container tarafından sağlanan bir bağımlılığın dış sınıflar tarafından değiştirilebilir hâle getirilmesiyle ortaya çıkan anti-pattern'dir. Özellikle enjekte edilen nesneye ait referansın `public setter` veya erişime açık alanlar üzerinden değiştirilmesi, bağımlılık yönetiminin DI container dışına taşınmasına neden olmaktadır.

Yazarlar, ODI'nin nedenleri arasında **Copy-paste Programming, Inadequate Design Analysis, Quick-fix Solutions** ve *encapsulation* ilkesine ilişkin farkındalık eksikliğini göstermektedir. Özellikle IDE tarafından otomatik oluşturulan getter/setter metotlarının kaldırılmaması veya bakım sırasında hızlı çözümler geliştirilmesi bu probleme yol açabilmektedir.

ODI; *reusability, extensibility, understandability* ve *modularity* özelliklerini olumsuz etkileyebilir. Enjekte edilen bağımlılığın çalışma zamanında dışarıdan değiştirilmesi, sınıfın beklenenden farklı davranmasına ve program akışının takip edilmesinin zorlaşmasına neden olabilir.

Önerilen refactoring, enjekte edilen bağımlılığın değiştirilmesine izin veren `public setter` gibi yapıların kaldırılması ve bağımlılığın DI mekanizmasının kontrolünde tutulmasıdır. Ancak bazı framework'lerin veya uygulama gereksinimlerinin dışarıdan erişim gerektirebileceği durumlarda bu kararın bağlama göre değerlendirilmesi gerektiği belirtilmektedir. ODI, **tümdengelimsel (deductive)** yöntemle belirlenmiştir.

>ODI, özellikle **Encapsulation, Open/Closed Principle (OCP) ve Inversion of Control (IoC)** ile ilişkilidir. Encapsulation, nesnenin iç durumunun kontrolsüz biçimde değiştirilmesini önlemeyi amaçlar. Enjekte edilen bağımlılığın dışarıdan değiştirilebilmesi ise nesnenin davranışının kendi kontrolü dışında farklılaşmasına neden olabilir. Ayrıca bağımlılık sağlama kontrolünün DI container'dan dış bileşenlere kayması IoC yaklaşımını zayıflatır.

ODI ile OWI birbirinden ayrılmalıdır. **OWI (4.8)** enjekte edilen bağımlılığın dışarıya açılmasına, **ODI (4.10)** ise dış bileşenlerin bu bağımlılığı değiştirebilmesine odaklanır. Her setter kullanımı anti-pattern değildir; belirleyici olan enjekte edilmiş bağımlılığın kontrolsüz biçimde yeniden atanabilmesidir.

### 4.11. Multiple Assigned Injection (MAI)

Multiple Assigned Injection, DI aracılığıyla sağlanan aynı nesne referansının bir sınıf içerisindeki birden fazla alana atanmasıyla ortaya çıkan anti-pattern'dir. Özellikle kalıtım ilişkilerinde üst sınıf ve alt sınıfın aynı bağımlılığı farklı alanlarda tutması, gereksiz referans tekrarına neden olabilmektedir.

Bu yaklaşım, bağımlılık referanslarının ayrı ayrı değiştirilebilmesi durumunda tutarsızlıklara yol açabilir. Ayrıca aynı bağımlılığın birden fazla alanda yönetilmesi, kodun *understandability* ve *maintainability* özelliklerini olumsuz etkileyebilir.

Yazarlar, çözüm olarak tekrarlanan alanların kaldırılmasını ve bağımlılığa erişimin uygun bir metot üzerinden sağlanmasını önermektedir. Makaledeki örnekte üst sınıfta soyut bir metot tanımlanmakta, alt sınıf ise bu metodu *override* ederek kendi bağımlılığını döndürmektedir. Böylece aynı referansın birden fazla alanda saklanması önlenmektedir. MAI, açık kaynak proje gözlemlerinden hareketle **tümevarımsal (inductive)** yöntemle belirlenmiştir.

>MAI, özellikle **Inheritance, Abstraction, Method Overriding ve Polymorphism** kavramlarıyla ilişkilidir. Üst sınıfta tanımlanan soyut metodun alt sınıfta gerçekleştirilmesi, ortak bir davranış sözleşmesi üzerinden bağımlılığa erişim sağlar. Polymorphism sayesinde üst sınıfın ihtiyaç duyduğu nesne, alt sınıfın uyguladığı metot aracılığıyla elde edilebilir. Bu yaklaşım, kalıtım hiyerarşisindeki gereksiz durum tekrarını azaltır.

Aynı nesneye birden fazla referans bulunması tek başına anti-pattern değildir. Problem, referansların farklı alanlarda saklanmasının gereksiz tekrar ve tutarsızlık riski oluşturmasıdır. Ayrıca önerilen refactoring'in mevcut kalıtım yapısına ve legacy sistemlerdeki değişiklik maliyetine uygunluğu değerlendirilmelidir.

### 4.12. Multiple Forms of Injection (MFI)

Multiple Forms of Injection, aynı bağımlılığın birden fazla injection yöntemiyle sağlanması sonucunda ortaya çıkan anti-pattern'dir. Makaledeki örnekte aynı alan için hem *Field Injection* hem de *Setter Injection* tanımlanmıştır. Bu durum, DI container'ın aynı bağımlılığı birden fazla injection noktası üzerinden işlemesine neden olabilir.

MFI, bağımlılıkların sağlanma sürecini karmaşıklaştırarak özellikle *understandability* ve *reliability* kalite özelliklerini olumsuz etkileyebilir. Framework'ün davranışına bağlı olarak aynı alana birden fazla atama yapılması veya beklenmeyen nesne durumlarının oluşması mümkündür. Ancak her framework'ün mutlaka farklı nesneler oluşturduğu varsayılmamalıdır.

Önerilen refactoring, aynı bağımlılık için tanımlanan gereksiz injection noktalarının kaldırılması ve tek bir injection yönteminin kullanılmasıdır. Böylece bağımlılığın nasıl sağlandığı daha açık ve öngörülebilir hâle gelir.

>MFI, özellikle **Encapsulation ve nesne durumunun tutarlılığı (object state consistency)** ile ilişkilidir. Bir nesnenin bağımlılıklarının farklı injection noktalarından değiştirilmesi, başlangıç durumunun ve davranışının öngörülebilirliğini azaltabilir. Tek bir injection yöntemi kullanılması, nesnenin bağımlılık yönetimini daha açık hâle getirerek durum tutarlılığını destekler.

Bir sınıfta farklı bağımlılıklar için farklı injection yöntemlerinin kullanılması tek başına MFI değildir. Problem, **aynı bağımlılığın birden fazla yöntemle sağlanmasıdır.** Ayrıca MFI, aynı referansın birden fazla alana atandığı Multiple Assigned Injection (MAI) ile karıştırılmamalıdır.


## DI Anti-pattern Kataloğunun Karşılaştırmalı Özeti

| Anti-pattern | Temel Problem | Olası Kalite Etkisi | Önerilen Refactoring | OOP ile İlişkisi |
|---|---|---|---|---|
| **IIJ – Intransigent Injection** | Bağımlılıkların ihtiyaç duyulmadan önce sağlanması | Performance, Understandability | Lazy Injection, Provider | Object Lifecycle, Composition |
| **CCI – Concrete Class Injection** | Soyutlama yerine somut sınıfa bağımlılık kurulması | Changeability, Extensibility, Reusability, Testability | Interface, Adapter | Abstraction, Polymorphism, DIP |
| **CPM – Complex Producer Method** | Producer Method'un karmaşık iş mantığı içermesi | Modularity, Understandability | Extract Method, iş mantığının ayrılması, AOP | SRP, High Cohesion |
| **FDC – Fat DI Class** | Sınıfın çok sayıda bağımlılık ve sorumluluk üstlenmesi | Understandability, Changeability, Extensibility, Reusability, Testability | Extract Class | SRP, Low Coupling, High Cohesion |
| **USI – Useless Injection** | Enjekte edilen bağımlılığın kullanılmaması | Performance, Understandability | Kullanılmayan bağımlılığın kaldırılması | Low Coupling |
| **SDP – Static Dependence Provider** | Bağımlılığın Static Factory veya Service Locator ile elde edilmesi | Coupling, Modularity, Testability | Producer Method, doğrudan Injection | IoC, DIP, Low Coupling |
| **DCC – Direct Container Call** | DI Container API'sinin uygulama kodunda doğrudan çağrılması | Modularity, Framework Coupling | Constructor/Field Injection, Provider | IoC, Low Coupling |
| **OWI – Open Window Injection** | Enjekte edilen bağımlılığın başka sınıflara aktarılması | Modularity, Understandability | Bağımlılığın ihtiyaç duyan sınıfa doğrudan enjekte edilmesi | Encapsulation, Low Coupling |
| **FCO – Framework Coupling** | Framework'e özgü DI anotasyonlarına bağımlılık | Modularity, Compatibility | Standart DI anotasyonlarının kullanılması | Low Coupling, Modularity |
| **ODI – Open Door Injection** | Enjekte edilen bağımlılığın dışarıdan değiştirilebilmesi | Modularity, Understandability, Reusability, Extensibility | Gereksiz Public Setter'ın kaldırılması | Encapsulation, IoC |
| **MAI – Multiple Assigned Injection** | Aynı enjekte edilmiş referansın birden fazla alanda tutulması | Understandability, Maintainability | Tek alan ve uygun erişim metodu | Inheritance, Abstraction, Polymorphism |
| **MFI – Multiple Forms of Injection** | Aynı bağımlılık için birden fazla Injection yönteminin kullanılması | Reliability, Understandability | Tek Injection yönteminin kullanılması | Encapsulation, Object State Consistency |



### Kataloğun Genel Değerlendirilmesi ve Geliştirme Önerileri

Önerilen 12 DI anti-pattern, bağımlılık yönetiminde karşılaşılabilecek farklı tasarım problemlerini sistematik biçimde ele almaktadır. Kataloğun güçlü yönü, yalnızca sorunları tanımlamakla kalmayıp bunların olası kalite etkilerini, OOP prensipleriyle ilişkilerini ve refactoring yaklaşımlarını da değerlendirmesidir. Bununla birlikte, CPM ve FDC gibi örneklerin genel nesne yönelimli tasarım problemleriyle de ilişkili olması, DI anti-pattern'leri ile genel OOP tasarım sorunları arasındaki ayrımın önemini ortaya koymaktadır.

Katalog incelendiğinde, **anti-pattern'lerin aynı düzeyde etki ve refactoring maliyetine sahip olmadığı** değerlendirilmektedir. Örneğin USI kapsamında kullanılmayan bir bağımlılığın kaldırılması görece sınırlı bir değişiklik gerektirebilirken FDC için önerilen Extract Class işlemi, sınıflar arasındaki sorumlulukların yeniden düzenlenmesini gerektirebilir. Bu nedenle kataloğun, anti-pattern'leri olası kalite etkileri, giderilme maliyetleri ve öncelik düzeyleri açısından da değerlendirecek şekilde genişletilmesi uygulayıcılara daha kapsamlı rehberlik sağlayabilir.

Bir diğer önemli nokta, önerilen refactoring işlemlerinin her durumda daha iyi tasarım sağlayacağının varsayılamamasıdır. Özellikle bağımlılıkları azaltmak amacıyla gereksiz soyutlamalar oluşturulması, sistemin karmaşıklığını ve bakım maliyetini artırabilir. Bu nedenle anti-pattern tespitlerinin kesin tasarım hataları olarak değil, sistemin bağlamı içerisinde değerlendirilmesi gereken olası problemler olarak ele alınması daha uygun görülmektedir. Refactoring öncesi ve sonrası kalite metriklerinin karşılaştırılması, önerilen çözümlerin gerçek etkilerinin belirlenmesine katkı sağlayabilir.

Son olarak, kataloğun Java ekosistemi temelinde geliştirilmesi, farklı programlama dillerine uygulanabilirliği konusunda araştırma fırsatı oluşturmaktadır. Özellikle C# ve Python gibi farklı bağımlılık yönetimi mekanizmalarına sahip ekosistemlerde yapılacak karşılaştırmalı çalışmalar, anti-pattern tanımlarının ne ölçüde genellenebileceğini ve hangi durumlarda dile veya framework'e özgü uyarlamalar gerektiğini ortaya koyabilir.