Giriş
Nesne yönelimli programlamada (Object-Oriented Programming – OOP), yazılım sis-
temleri birbirleriyle etkileşim halindeki sınıflar ve nesneler üzerinden yapılandırılmaktadır.
Bir sınıfın işlevlerini gerçekleştirebilmek için başka sınıfların sunduğu hizmetlere ihtiyaç
duyması, bileşenler arasında bağımlılık ilişkilerinin oluşmasına neden olmaktadır. Bu bağım-
lılıkların uygun şekilde yönetilmemesi, bileşenler arasında sıkı bağlılığa (tight coupling) yol
açarak yazılımın değiştirilmesini, genişletilmesini, yeniden kullanılmasını ve test edilmesini
zorlaştırabilmektedir.
Bu problemin ele alınmasında nesne yönelimli tasarım ilkeleri önemli bir rol oyna-
maktadır. Özellikle Dependency Inversion Principle (DIP), bileşenlerin somut uygulama ay-
rıntıları yerine uygun soyutlamalara bağımlı olmasını desteklemektedir. Inversion of Control
(IoC) ise nesne oluşturma ve bağımlılık sağlama gibi bazı kontrol sorumluluklarının ilgili
bileşenin dışına aktarılmasını ifade etmektedir. Bu yaklaşımlarla ilişkili olan Dependency
Injection (DI), bir sınıfın ihtiyaç duyduğu bağımlılıkların sınıf içerisinde doğrudan oluşturul-
ması yerine dışarıdan sağlanmasını esas alan bir mekanizmadır. DI sayesinde bağımlılıkların
oluşturulması ve yönetilmesi ile bu bağımlılıkları kullanan bileşenlerin işlevsel sorumluluk-
ları birbirinden ayrılabilmektedir.
DI kullanımını kolaylaştırmak amacıyla çeşitli framework’ler geliştirilmiş ve bağımlı-
lık yönetimini destekleyen mekanizmalar yazılım geliştirme ekosistemlerinde yaygınlaşmış-
tır. Bununla birlikte, DI mekanizmasının kullanılması tek başına iyi bir nesne yönelimli tasa-
rım oluşturulduğunu garanti etmemektedir. Bağımlılıkların uygun olmayan biçimlerde tanım-
lanması, yönetilmesi veya diğer bileşenlere aktarılması; DI ile azaltılması hedeflenen tasarım
problemlerinin farklı biçimlerde ortaya çıkmasına neden olabilmektedir.
Laigner ve arkadaşlarının (2022) Cataloging Dependency Injection Anti-Patterns in
Software Systems başlıklı çalışması, DI mekanizmasının hatalı kullanımı sonucunda ortaya
çıkabilecek tasarım problemlerine odaklanmaktadır. Araştırmanın temel problemi, DI kötü
1
uygulamalarına ilişkin mevcut rehberliğin kapsam, ampirik geçerlilik ve genellenebilirlik açı-
sından sınırlı olmasıdır. Literatürde çeşitli kötü uygulamalar tanımlanmış olmakla birlikte, bu
problemlerin sistematik biçimde kataloglanması ve gerçek yazılım sistemleri üzerinde değer-
lendirilmesi konusunda eksiklikler bulunduğu belirtilmektedir.
Bu doğrultuda çalışmanın temel amacı, DI kullanımına ilişkin anti-pattern’leri tanım-
layan, olası kalite etkilerini ve refactoring yaklaşımlarını açıklayan kapsamlı bir katalog geliş-
tirmek ve bu kataloğun pratik geçerliliğini değerlendirmektir. Böylece geliştarım problemle-
rinin tespit edilmesi, anlaşılması ve giderilmesi konusunda sistematik biricilere DI kaynaklı
tasir rehberlik sunulması hedeflenmektedir.
1.1 Araştırma Soruları
Çalışma, birbirini tamamlayan dört araştırma sorusu üzerine yapılandırılmıştır:
• RQ1: DI uygulamasıyla ilişkili olup mevcut belgelenmiş kötü DI uygulamalarının ye-
terince kapsamadığı problem adayları var mıdır?
• RQ2: Önerilen DI anti-pattern’leri gerçek yazılım sistemlerinde ortaya çıkmakta mı-
dır?
• RQ3: Uzman geliştiriciler önerilen DI anti-pattern’lerini uygulamada yararlı bulmakta
mıdır?
• RQ4: Uygulayıcılar tespit edilen DI anti-pattern örneklerini refactoring yoluyla gider-
meye istekli midir?
Bu sorular doğrultusunda araştırma, öncelikle mevcut literatürde yeterince ele alın-
mamış DI problemlerinin belirlenmesine, ardından önerilen anti-pattern’lerin gerçek proje-
lerdeki varlığının incelenmesine odaklanmaktadır. Sonraki aşamalarda ise geliştiricilerin ka-
taloğa yönelik değerlendirmeleri ve refactoring konusundaki tutumları araştırılmaktadır. Böy-
lece katalog yalnızca teorik bir öneri olarak sunulmayıp ampirik bulgular ve uygulayıcı gö-
rüşleriyle değerlendirilmektedir.
1.2 Araştırmanın Temel Katkıları
Makalenin giriş bölümünde araştırmanın başlıca katkıları altı maddede özetlenmekte-
dir:
1. Literatürdeki eksikliklerin belirlenmesi: DI kötü uygulamalarına yönelik mevcut ça-
lışmaların kapsam, geçerlilik ve genellenebilirlik açısından sınırlılıklarının ortaya ko-
nulması.
2
2. DI anti-pattern kataloğunun geliştirilmesi: DI mekanizmasının yanlış kullanımın-
dan kaynaklanabilecek problemleri tanımlayan ve refactoring önerileri sunan yeni bir
kataloğun oluşturulması.
3. DIAnalyzer aracının geliştirilmesi: Java projelerinde DI anti-pattern örneklerini oto-
matik olarak tespit etmek amacıyla yüksek precision sonuçları elde edilen, açık kay-
naklı bir statik analiz aracının geliştirilmesi.
4. Gerçek yazılım projelerinde ampirik değerlendirme: Önerilen anti-pattern’lerin açık
ve kapalı kaynak projelerdeki görülme durumlarının incelenmesi.
5. Geliştirici görüşlerinin değerlendirilmesi: Kataloğun algılanan yararlılığının ve ge-
liştiricilerin tespit edilen problemleri refactoring yoluyla giderme istekliliklerinin araş-
tırılması.
6. Kataloğun güncellenmesi: Geliştirici geri bildirimleri doğrultusunda anti-pattern ta-
nımlarının ve uygulanma koşullarının yeniden değerlendirilerek kataloğun geliştiril-
mesi.
Bu katkılar birlikte değerlendirildiğinde çalışmanın, DI kullanımındaki tasarım prob-
lemlerini yalnızca tanımlamakla sınırlı kalmadığı; bunların otomatik tespiti, gerçek projeler-
deki varlığı ve geliştiriciler açısından yararlılığı arasında bir değerlendirme süreci oluştur-
duğu görülmektedir. Bununla birlikte, anti-pattern’lerin tespit edilmesi ve geliştiriciler tara-
fından yararlı bulunması, önerilen refactoring işlemlerinin yazılım kalitesine etkisini doğru-
dan kanıtlamamaktadır. Bu ayrım, araştırmanın yöntem ve bulgularının değerlendirilmesinde
önem taşımaktadır.

2 Kavramsal Çerçeve
İncelenen makalede ele alınan bağımlılık yönetimi problemlerinin ve tasarım ilkeleriyle ilişkilerinin
daha iyi anlaşılabilmesi için bu bölümde temel kavramlar kısaca açıklanmaktadır.
Coupling (bağlılık), yazılım bileşenlerinin birbirlerine olan bağımlılık düzeyini ifade etmektedir.
Bir sınıfın başka bir sınıfın somut uygulamasına sıkı biçimde bağlı olması, o uygulamada yapılan de-
ğişikliklerin kendisini de etkilemesine neden olabilir. Bu nedenle Low Coupling, bileşenler arasındaki
gereksiz bağımlılıkların azaltılmasını hedeflemektedir.
Cohesion (uyum), bir sınıfın üstlendiği sorumlulukların kendi içerisinde ne kadar ilişkili olduğunu
ifade etmektedir. Örneğin sipariş yönetiminden sorumlu bir sınıfın aynı zamanda kullanıcı yönetimi ve
raporlama gibi farklı görevleri üstlenmesi, sınıfın uyumunu azaltabilir. High Cohesion, birbiriyle ilişkili
sorumlulukların aynı bileşende toplanmasını amaçlamaktadır.
SOLID, nesne yönelimli tasarımda sınıfların sorumluluklarının ve bağımlılıklarının düzenlenme-
sine yönelik beş temel ilkeden oluşmaktadır. Özellikle Single Responsibility Principle (SRP), bir sınıfın
2
birbirinden bağımsız sorumlulukları üstlenmemesini; Dependency Inversion Principle (DIP) ise üst dü-
zey modüllerin belirli somut uygulamalara değil, uygun soyutlamalara bağımlı olmasını önermektedir.
Böylece bir uygulama değiştiğinde onu kullanan sınıfların değiştirilmesi ihtiyacı azaltılabilir [7].
GRASP (General Responsibility Assignment Software Patterns), yazılımda sorumlulukların
hangi nesnelere verilmesi gerektiğine yönelik tasarım ilkelerini kapsamaktadır. Özellikle Low Coupling
ve High Cohesion, bileşenlerin gereksiz bağımlılıklardan kaçınmasını ve ilişkili görevlerin bir arada
tutulmasını desteklemektedir [8].
Inversion of Control (IoC), bir bileşenin normalde kendisinin yürüttüğü belirli kontrol sorumlu-
luklarının dışarıdaki bir yapıya devredilmesidir. Bağımlılık yönetimi açısından bu, sınıfın ihtiyaç duy-
duğu nesneyi kendi içerisinde oluşturması veya araması yerine, dışarıdan hazır olarak alması anlamına
gelebilir. Örneğin OrderService sınıfının kendi içerisinde new EmailService() çağırması yerine,
EmailService nesnesinin dışarıda oluşturulup sınıfa verilmesi, nesne oluşturma ve sağlama sorumlu-
luğunun sınıf dışına taşındığını göstermektedir [9].
Dependency Injection (DI), bir sınıfın ihtiyaç duyduğu bağımlılıkların dışarıdan sağlanmasını ger-
çekleştiren mekanizmadır. Bağımlılıklar genellikle Constructor Injection (yapıcı metot), Setter Injection
(setter metodu) veya Field Injection (alan enjeksiyonu) yoluyla verilebilmektedir. Böylece sınıf, bağımlı-
lığın oluşturulmasından ziyade onu kullanarak gerçekleştireceği işlevlere odaklanabilmektedir [9].
DI Container, uygulamadaki nesnelerin oluşturulmasını, ihtiyaç duydukları bağımlılıkların be-
lirlenip sağlanmasını ve gerektiğinde yaşam döngülerinin yönetilmesini üstlenen yapıdır. Örneğin bir
OrderService sınıfı EmailService bağımlılığına ihtiyaç duyduğunda, Spring gibi bir DI container uy-
gun EmailService nesnesini oluşturarak veya mevcut nesneyi kullanarak bu bağımlılığı OrderService
sınıfına sağlayabilir. Böylece her sınıfın bağımlılıklarını kendisinin oluşturmasına gerek kalmaz. Bu-
nunla birlikte DI kullanmak için bir container zorunlu değildir; bağımlılıklar manuel olarak da dışarıdan
verilebilir.
Bu kavramlar birbirleriyle ilişkili olmakla birlikte farklı işlevlere sahiptir. DIP, bağımlılıkların hangi
yapılara yönelmesi gerektiğiyle ilgili bir tasarım ilkesi; IoC, kontrol sorumluluğunun dışarıya aktarılması
yaklaşımı; DI, bağımlılıkları dışarıdan sağlama mekanizması; DI container ise bu mekanizmayı otomatik
olarak yöneten araçtır.
3 Araştırma Yöntemi ve Kullanılan Teknolojiler
Laigner ve arkadaşları (2022)[1], DI anti-pattern kataloğunu geliştirmek ve değerlendirmek ama-
cıyla nitel ve nicel yöntemleri bir arada kullanan çok aşamalı bir araştırma yürütmüştür. Araştırma; ka-
taloğun oluşturulması, otomatik tespit aracının geliştirilmesi, gerçek yazılım projelerinin incelenmesi ve
geliştirici görüşlerinin değerlendirilmesi aşamalarından oluşmaktadır.
Kataloğun oluşturulmasında tümevarımsal ve tümdengelimsel yaklaşımlar birlikte kullanıl-
mıştır. Tümevarımsal yaklaşımda, araştırmacılardan birinin yedi ay boyunca üç kapalı kaynak Java pro-
jesinde gerçekleştirdiği bakım faaliyetleri sırasında gözlemlediği problemlerden yararlanılmıştır. Tüm-
dengelimsel yaklaşımda ise DI, SOLID ve GRASP ilkeleri temel alınarak olası tasarım problemleri belir-
lenmiştir. Böylece katalog hem gerçek yazılım geliştirme deneyimlerine hem de nesne yönelimli tasarım
ilkelerine dayandırılmıştır.
Önerilen anti-pattern’lerin gerçek projelerde tespit edilebilmesi amacıyla DIAnalyzer adlı statik
kod analiz aracı geliştirilmiştir. JavaParser kütüphanesinden yararlanan araç, Java kaynak kodlarını so-
yut sözdizim ağacı (Abstract Syntax Tree – AST) üzerinden inceleyerek önceden tanımlanmış kurallara
göre olası anti-pattern örneklerini belirlemektedir. Aracın tespit başarısı, araştırmacılar tarafından manuel
olarak oluşturulan referans veri kümesi kullanılarak precision ve relative recall ölçütleriyle değerlendiril-
miştir. Aracın doğrulanmasında Agilefant ve LibrePlan projelerinden seçilen 83 sınıfta belirlenmiş 141
3
anti-pattern örneğini içeren manuel referans veri kümesi kullanılmıştır.
Kataloğun uygulamadaki karşılığını araştırmak için DIAnalyzer kullanılarak dört açık kaynak ve
iki kapalı kaynak Java projesi incelenmiştir. Ardından geliştiricilerle gerçekleştirilen görüşmeler ve
çevrim içi anketler aracılığıyla anti-pattern’lerin algılanan yararlılığı ile geliştiricilerin refactoring yapma
isteklilikleri değerlendirilmiştir. Anket sürecinde Technology Acceptance Model (TAM) yaklaşımından
da yararlanılmıştır.
İncelenen projelerin özellikleri Tablo 1’de sunulmaktadır.
Tablo 1: Araştırmada İncelenen Yazılım Projeleri
Proje Kaynak Türü LOC
Agilefant Açık Kaynak 58.171
BroadleafCommerce Açık Kaynak 327.058
LibrePlan Açık Kaynak 284.090
Shopizer Açık Kaynak 109.792
Kapalı Kaynak Proje 1 Kapalı Kaynak (Spring) 29.405
Kapalı Kaynak Proje 2 Kapalı Kaynak (Guice) 32.204
Kaynak: Laigner vd. (2022)[1].
Geliştirici değerlendirme sürecinin aşamaları Tablo 2’de gösterilmektedir.
Tablo 2: Geliştirici Değerlendirme Aşamaları
Aşama Değerlendirme Yöntemi Katılımcı
1 Deneyimli geliştiricilerle yüz yüze görüşme 3
2 Uzman geliştiricilerle çevrim içi anket 6
3 Açık çevrim içi geliştirici anketi 9
Kaynak: Laigner vd. (2022)[1].
Son aşamada geliştiricilerden elde edilen geri bildirimler doğrultusunda katalog gözden geçirilmiş
ve anti-pattern’lerin hangi koşullarda problem oluşturabileceğini açıklayan bağlamsal düzenlemeler ya-
pılmıştır.

4 Bulgular ve Değerlendirme
4.1 DI Anti-Pattern Kataloğu ve Güncellenmesi
Laigner ve arkadaşları (2022)[1], Dependency Injection mekanizmasının kullanımında ortaya çıka-
bilecek 12 anti-pattern tanımlamıştır. Her anti-pattern için problemin tanımı, olası kalite etkileri ve çözüm
yaklaşımları açıklanmıştır.
Araştırmanın ilerleyen aşamalarında geliştiricilerden elde edilen geri bildirimler doğrultusunda ka-
talog yeniden değerlendirilmiştir. Bu süreçte anti-pattern sayısı korunmuş, bazı tanımlar netleştirilmiş ve
uygulanma koşullarını açıklayan Context alanı eklenmiştir.
Tablo 3, anti-pattern’lerin temel problemlerini, önerilen çözümleri ve güncellenen katalogdaki bağ-
lamsal açıklamaları karşılaştırmalı olarak sunmaktadır.
4
Tablo 3: DI Anti-Pattern Kataloğunun Karşılaştırmalı Değerlendirmesi
Anti-pattern / Kategori İlk Katalogdaki Problem Önerilen Çözüm Güncellenen Katalog / Bağlam
IIJ
Intransigent Injection
Performance
Bağımlılığın ihtiyaç duyulmadan erken oluşturulması
veya sağlanması, gereksiz başlangıç maliyeti ve bellek
tüketimi oluşturabilir.
Bağımlılığı ihtiyaç anında Provider veya lazy
injection ile sağlamak.
Problemin bağımlılığın oluşturulma maliyetine ve kulla-
nım zamanına bağlı olduğu vurgulanmıştır. Gecikmeli
oluşturma her durumda gerekli değildir.
CCI
Concrete Class Injection
Design
Bağımlılığın interface yerine somut sınıf üzerinden ta-
nımlanması, değiştirilebilirliği ve test edilebilirliği sı-
nırlayabilir.
Uygun bir interface veya soyutlama üzerinden
bağımlılık tanımlamak.
Küçük sistemlerde somut sınıf kullanımının her zaman
problem oluşturmayabileceği; soyutlamanın ihtiyaç ortaya
çıktığında eklenebileceği açıklanmıştır.
CPM / LPM
Complex Producer Method /
Long Producer Method
Design
Producer metodunun bağımlılık sağlama dışında kar-
maşık iş mantığını da üstlenmesi, sorumlulukların ka-
rışmasına neden olabilir.
Bağımlılık üretme işlemi ile iş mantığını ayır-
mak; uygun refactoring uygulamak.
Problemin yalnızca metot uzunluğu olmadığı, producer
metodunun temel sorumluluğu dışındaki işlemleri üstlen-
mesiyle ilişkili olduğu netleştirilmiştir. Güncel katalogda
LPM adı kullanılmıştır.
FDC
Fat DI Class
Design
Bir sınıfa çok sayıda bağımlılık enjekte edilmesi, so-
rumlulukların dağılmasına ve bakım maliyetinin art-
masına yol açabilir.
Sorumlulukları inceleyerek gerektiğinde Ext-
ract Class uygulamak.
Çok sayıda bağımlılığın tek başına yeterli olmadığı; bu ba-
ğımlılıkların gerekçesi ve sınıfın birden fazla sorumluluk
üstlenip üstlenmediğinin değerlendirilmesi gerektiği vur-
gulanmıştır.
USI
Useless Injection
Performance
Enjekte edilen bağımlılığın kullanılmaması, gereksiz
nesne oluşturma ve bağımlılık ilişkileri meydana geti-
rebilir.
Kullanılmayan injection tanımını kaldırmak. Injection anotasyonuyla tanımlanmış kullanılmayan alan-
ların IDE tarafından her zaman raporlanmaması açıklan-
mıştır. Kalıtım yoluyla kullanım da dikkate alınmalıdır.
SDP
Static Dependence Provider
Architecture
Bağımlılığın DI yerine static factory veya Service Lo-
cator üzerinden elde edilmesi, sınıflar arasında sıkı
bağlılık oluşturabilir.
Bağımlılığı DI mekanizması veya uygun bir
provider üzerinden sağlamak.
Farklı framework’lerin entegrasyonu ve container dışın-
daki bileşenlere bağımlılık sağlama gibi özel gereksinim-
lerin istisna oluşturabileceği belirtilmiştir.
DCC
Direct Container Call
Architecture
Uygulama sınıfının doğrudan DI container API’sini ça-
ğırarak bağımlılık elde etmesi, framework bağımlılı-
ğını artırabilir.
Bağımlılığı doğrudan enjekte etmek; gereki-
yorsa container erişimini provider içinde kap-
süllemek.
Çalışma zamanında dinamik bağımlılık çözümlemesi gere-
ken durumlar dikkate alınmış; container erişiminin uygun
bir sınırda kapsüllenmesi önerilmiştir.
Devamı sonraki sayfada
5
Tablo 3 – devamı
Anti-pattern / Kategori İlk Katalogdaki Problem Önerilen Çözüm Güncellenen Katalog / Bağlam
OWI
Open Window Injection
Design
Enjekte edilen nesnenin başka sınıflara parametre ola-
rak aktarılması veya dış erişime açılması, gereksiz ara-
cılık ve bağımlılık ilişkileri oluşturabilir.
Bağımlılığı onu gerçekten kullanan bileşene
doğrudan sağlamak.
Container dışında çalışan veya üçüncü taraf bileşenlere ba-
ğımlılık aktarılmasının gerekli olabileceği kabul edilmiş-
tir.
FCO
Framework Coupling
Standardization
Framework’e özgü injection anotasyonları ve API’leri,
uygulamanın belirli bir framework’e bağımlılığını ar-
tırabilir.
Uygun olduğunda standart DI tanımlamalarını
tercih etmek.
Standart anotasyonlara yönelmenin özellikle yeni proje-
lerde anlamlı olduğu; mevcut sistemlerde dönüşüm mali-
yetinin dikkate alınması gerektiği belirtilmiştir.
ODI
Open Door Injection
Design
Enjekte edilmiş bağımlılığın public setter gibi yön-
temlerle dışarıdan değiştirilebilmesi, nesne durumu-
nun denetlenmesini zorlaştırabilir.
Gereksiz dışarıdan değiştirme mekanizmasını
kaldırmak.
Makalenin Table 27 tablosunda bu örüntü için ayrıca bir
değişiklik raporlanmamıştır. Güncel katalogda da yer al-
maktadır.
MAI
Multiple Assigned Injection
Design
Enjekte edilen aynı nesne referansının birden fazla
alana atanması, gereksiz durum tekrarı oluşturabilir.
Tekrarlanan referansları kaldırmak; uygunsa
soyut metot üzerinden erişmek.
Table 27’de ayrıca bir güncelleme raporlanmamıştır. Gün-
cel katalogda kalıtım ilişkilerinden kaynaklanan bazı zo-
runlu atamalar istisna tutulmuştur.
MFI
Multiple Forms of Injection
Standardization
Aynı bağımlılığın birden fazla injection yöntemiyle
sağlanması, gereksiz işlem ve belirsizlik oluşturabilir.
Aynı bağımlılık için tek bir injection yönte-
mini kullanmak.
Table 27’de ayrıca bir değişiklik raporlanmamıştır. Güncel
katalogda yer almaya devam etmektedir.
Kaynak: Laigner vd. (2022)[1], Bölüm 4 ve Tablo 27; güncellenmiş katalog[2], Tablo 1.
6
Güncellenen katalogda anti-pattern’ler Design, Architecture, Performance ve Standardization ol-
mak üzere dört kategoride sınıflandırılmıştır. Bu sınıflandırma, problemlerin yalnızca sınıf düzeyindeki
tasarım kararlarıyla sınırlı olmadığını; mimari bağımlılıklar, performans ve standartlaşma açısından da
değerlendirildiğini göstermektedir.


4.2 SeçilmişAnti-Pattern’lerinOOPAçısındanİncelenmesi
Katalogdatanımlanananti-pattern’lerinnesneyönelimlitasarımilkeleriyleilişkisinisomutlaştırmak
amacıylaConcreteClassInjection(CCI)veDirectContainerCall (DCC)örnekleri incelenmiştir.Kod
örnekleri,güncellenmişkatalogdakiilgilişekillerdensadeleştirilerekuyarlanmıştır.
4.2.1 ConcreteClassInjection(CCI)
CCI,birsınıfınbağımlılığınısoyutlamayerinedoğrudansomutsınıfüzerindentanımlamasıylaor
tayaçıkmaktadır.
1 public class B {
2 @Inject
3 private ConcreteClassExample example;
4
5 public void foo() {
6 example.doSomething();
7 }
8 }
Kod1:CCI:Problemlikodyapısı
BuörnekteBsınıfı,bağımlılığınıDIaracılığıylaalmasınarağmendoğrudanConcreteClassExample
sınıfınabağlıdır.Budurum,bağımlılığındışarıdansağlanmasınıntekbaşınauygunbirsoyutlamaoluş
turmadığınıgöstermektedir.
1 public interface IExampleInterface {
2 void doSomething();
3 }
4
5 public class ConcreteClassExample
6 implements IExampleInterface {
7
8 public void doSomething() {
9 // Implementation
10 }
11 }
12
13 public class B {
14 @Inject
15 private IExampleInterface example;
16
17 public void foo() {
18 example.doSomething();
19 }
20 }
Kod2:CCI:Refactoringsonrasıkodyapısı
RefactoringsonrasındaBsınıfınınbağımlılığısomutsınıftanIExampleInterfacearayüzünetaşınmış
tır.Böylecesınıf,belirlibiruygulamayerineortakbirdavranışsözleşmesinebağımlıhâlegelmiştir.
7
OOP açısından değerlendirildiğinde, bu dönüşüm özellikle Dependency Inversion Principle
(DIP), abstraction ve polymorphism kavramlarıyla ilişkilidir. DI, bağımlılığın sınıfa nasıl sağlandı
ğını düzenlerken DIP, bağımlılığın uygun soyutlamalara yönelmesini esas almaktadır. Bu nedenle DI
kullanılması, tek başına DIP ilkesinin uygulandığı anlamına gelmemektedir.
Refactoring sonrasında soyutlama üzerinden kurulan bağımlılık, farklı uygulamaların kullanılma
sını ve testlerde alternatif gerçekleştirimlerin sağlanmasını kolaylaştırabilir. Bununla birlikte, her somut
sınıf için interface oluşturmak zorunlu değildir; sağlanacak fayda sistemin ihtiyaçlarına göre değerlendi
rilmelidir.
4.2.2 Direct Container Call (DCC)
1 public class F {
DCC, uygulama sınıfının ihtiyaç duyduğu bağımlılığı doğrudan DI container üzerinden elde etmesi
durumunda ortaya çıkmaktadır.
2
3
4
5
6
7
8
9
10
11
12
13
14 }
@Inject
private ApplicationContext context;
protected IDataSource getRepository() {
return (IDataSource)
context.getBean("ftpDataSource");
}
public void execute() {
IDataSource dataSource = getRepository();
// Use dataSource
}
Kod 3: DCC: Problemli kod yapısı
Bu örnekte F sınıfı, bağımlılığının sağlanması için doğrudan Spring ApplicationContext API’sini
kullanmaktadır. Böylece sınıf, yalnızca IDataSource hizmetine değil, bağımlılığın container üzerinden
nasıl elde edildiğine de bağlı hâle gelmektedir.
1 public class F {
2
3
4
5
6
7
8 }
@Inject
private IDataSource dataSource;
public void execute() {
// Use dataSource
}
Kod 4: DCC: Refactoring sonrası kod yapısı
Refactoring sonrasında container çağrısı uygulama sınıfından kaldırılmış ve IDataSource bağımlılığı
doğrudan enjeksiyon yoluyla sağlanmıştır. Böylece F sınıfı, bağımlılığı arama sorumluluğunu üstlenme
den yalnızca kendisine sağlanan nesneyi kullanmaktadır.
OOP açısından değerlendirildiğinde, bu dönüşüm Inversion of Control (IoC) ve Low Coupling
ilkeleriyle ilişkilidir. Bağımlılığın sağlanma sorumluluğunun uygulama sınıfından ayrılması, sınıfın con
tainer API’sine olan bağımlılığını azaltır. Ancak çalışma zamanında dinamik bağımlılık çözümlemesi
gereken sistemlerde doğrudan enjeksiyon her zaman yeterli olmayabilir. Bu tür durumlarda container
erişiminin ayrı bir Provider bileşeninde kapsüllenmesi değerlendirilebilir.
Kaynak: Laigner vd., Catalog of Java Dependency Injection Anti-Patterns[2], Şekil 1 ve Şekil 8’den uyarlanmıştır.
8
4.3 DIAnalyzer ve Gerçek Projelerdeki Bulgular
DIAnalyzer’ın değerlendirilmesinde, araştırmacılar tarafından Agilefant ve LibrePlan projelerindeki
83 sınıf üzerinden manuel olarak oluşturulan referans veri kümesi kullanılmıştır. Araç, bu kümede belir
lenen 141 anti-pattern örneğinin 130’unu tespit ederek %92,19 relative recall değerine ulaşmıştır. Proje
bazındaki ortalama precision sonuçları ise Agilefant için %97,78, LibrePlan için %89,80 olarak bildiril
miştir [1].
Bu ölçütlerden precision, araç tarafından raporlanan örneklerin ne kadarının doğru olduğunu değer
lendirmektedir. Relative recall ise manuel referans kümesindeki örneklerin ne kadarının araç tarafından
tespit edilebildiğini göstermektedir. Dolayısıyla raporlanan relative recall, tüm projelerdeki olası anti
pattern örneklerini kapsayan mutlak bir başarı ölçümü olarak yorumlanmamalıdır.
Tablo 4: DIAnalyzer doğrulama sonuçları
Ölçüt
Referans veri kümesindeki sınıf sayısı
Manuel belirlenen anti-pattern örneği
DIAnalyzer tarafından tespit edilen örnek
Relative recall
Agilefant ortalama precision
LibrePlan ortalama precision
Sonuç
83
141
130
%92,19
%97,78
%89,80
Kaynak: Laigner vd. (2022) [1], Tablo 7 ve Bölüm 5.2.
Araçla gerçekleştirilen proje incelemelerinde anti-pattern türlerinin görülme sıklığının projeler ara
sında farklılık gösterdiği raporlanmıştır. Buna karşılık CPM anti-pattern’ine incelenen altı projede de
rastlanmamıştır.
Ancak projelerin büyüklükleri ve kullandıkları DI framework’leri farklı olduğundan, ham tespit
sayıları doğrudan yazılım kalitesi karşılaştırması olarak değerlendirilmemelidir.
Ayrıca precision ve relative recall değerleri aracın tespit başarısını göstermektedir. Bu sonuçlar, tes
pit edilen her kod yapısının yazılım kalitesini olumsuz etkilediğini tek başına kanıtlamamaktadır.
4.4 Geliştirici Değerlendirmeleri ve Sonuçların Yorumlanması
Geliştirici görüşmeleri ve anketler, önerilen kataloğa yönelik genel olarak olumlu bir değerlendirme
ortaya koymuştur. Üç deneyimli geliştiriciyle gerçekleştirilen ilk görüşme aşamasında, toplam 39 değer
lendirmenin 37’sinde tam veya kısmi kabul bildirilmiştir. Altı uzman geliştiricinin katıldığı çevrim içi
ankette ise 36 TAM yanıtının 31’i olumlu yönde olmuştur [1]. Burada belirtilen 39 ve 36 değerleri katı
lımcı sayılarını değil, değerlendirme ve yanıt sayılarını ifade etmektedir. Bu nedenle bulgular, katılımcı
sayısı ve araştırmanın kapsamı dikkate alınarak yorumlanmalıdır.
Bununla birlikte, geliştiricilerin refactoring konusundaki tutumları anti-pattern türlerine göre fark
lılık göstermiştir. Bildirilen refactoring istekliliği, anti-pattern türüne ve katılımcıların uygulama bağla
mına ilişkin değerlendirmelerine göre değişmiştir. Bu nedenle anket yanıtları, bütün geliştiricilere genel
lenebilecek kesin refactoring oranları olarak yorumlanmamalıdır.
Bufarklılıklar, önerilen çözümlerin her yazılım sistemi için aynı ölçüde gerekli veya yararlı olmaya
bileceğini göstermektedir. Özellikle maliyetli bağımlılıkların gecikmeli oluşturulması, ek soyutlamaların
kullanılması ve framework bağımlılıklarının azaltılması gibi kararlarda sistemin gereksinimlerinin dik
kate alınması önem taşımaktadır.
Araştırmada elde edilen geliştirici görüşleri, kataloğun algılanan yararlılığı ve refactoring istekli
liği hakkında bilgi sunmaktadır. Ancak bu değerlendirmeler, önerilen değişikliklerin gerçek projelerde
9
uygulanmasının ardından yazılım kalitesinde ölçülebilir bir iyileşme meydana geldiğini doğrudan göster
memektedir.
5 Nesne Yönelimli Programlamayla İlişkisi
Laigner ve arkadaşlarının (2022)[1] çalışması, Dependency Injection mekanizmasının yalnızca ba
ğımlılıkların sağlanmasıyla ilgili teknik bir yaklaşım olmadığını, nesne yönelimli tasarım ilkeleriyle bir
likte değerlendirilmesi gerektiğini ortaya koymaktadır. Makalede tanımlanan anti-pattern’ler, DI kulla
nımına rağmen sınıflar arasında sıkı bağlılık, sorumlulukların uygun olmayan biçimde dağıtılması ve
kapsülleme problemleri oluşabileceğini göstermektedir.
Özellikle Concrete Class Injection (CCI), soyutlama yerine somut sınıflara bağımlılığın Dependency
Inversion Principle (DIP) ile ilişkisini ortaya koymaktadır. Fat DI Class (FDC) ve Complex Producer Met
hod (CPM) ise Single Responsibility Principle (SRP) ile GRASP’ın High Cohesion ve Low Coupling
ilkeleri açısından değerlendirilebilmektedir. Bu örnekler, sınıfların ve metotların üstlendikleri sorumlu
lukların bağımlılık yönetiminden ayrı düşünülemeyeceğini göstermektedir.
Open Window Injection (OWI) ve Open Door Injection (ODI), nesnelerin iç bağımlılıklarının dışa
rıya açılması veya dışarıdan değiştirilmesi nedeniyle kapsülleme (encapsulation) ilkesiyle ilişkilidir. Sta
tic Dependence Provider (SDP) ve Direct Container Call (DCC) ise bağımlılıkların doğrudan uygulama
kodu tarafından elde edilmesi nedeniyle Inversion of Control (IoC) ve düşük bağlılık hedefleri açısından
önem taşımaktadır.
Bu ilişkiler değerlendirildiğinde, DI mekanizmasının kullanılmasının tek başına düşük coupling
veya yüksek cohesion sağlamadığı anlaşılmaktadır. Bağımlılıkların nasıl tanımlandığı, hangi bileşenlere
verildiği ve sınıfların sorumluluklarıyla nasıl ilişkilendirildiği de tasarım kalitesini etkilemektedir.
Bununla birlikte, nesne yönelimli tasarım ilkelerinin her durumda katı kurallar olarak uygulanması
da uygun değildir. Örneğin yalnızca somut bir sınıfa bağımlılık bulunması, her koşulda yeni bir interface
oluşturulmasını gerektirmemektedir. Kataloğun geliştirici geri bildirimleri doğrultusunda güncellenmesi
de tasarım problemlerinin uygulamanın büyüklüğü, mimari gereksinimleri ve kullanım bağlamıyla bir
likte değerlendirilmesi gerektiğini desteklemektedir.
Bu yönüyle çalışma, bağımlılık sağlama mekanizması ile nesne yönelimli tasarım kararlarının ayrı
ancak birbiriyle ilişkili konular olduğunu vurgulamaktadır.

6 Temel Kaynaklar ve Alanın Tarihsel Gelişimi
Dependency Injection (DI) üzerine yapılan araştırmalar, mekanizmanın yazılım kalitesi üzerindeki
etkilerinin incelenmesinden farklı uygulama alanlarında kullanılmasına ve hatalı kullanım biçimlerinin
araştırılmasına doğru genişlemiştir. Laigner vearkadaşlarının (2022)çalışmasınınliteratürdeki konumunu
değerlendirmek amacıyla, makalenin kaynakçasında yer alan üç temel araştırma incelenmiştir.
Razina ve Janzen (2007)[3], Effects of Dependency Injection on Maintainability çalışmalarında,
DI’ın yazılımın bakım yapılabilirliğini artırdığı yönündeki teorik beklentiyi ampirik olarak sınamıştır.
Araştırmada 20’si Spring kullanan, 20’si kullanmayan toplam 40 açık kaynak proje, coupling ve cohesion
metrikleri üzerinden karşılaştırılmıştır. Sonuçlarda DI kullanımı ile bu metrikler arasında genel bir ilişki
doğrulanamamış,ancakDIkullanımoranı%10’unüzerindeolanprojelerdedahadüşükcouplingyönünde
bir eğilim gözlemlenmiştir. Bu çalışma, DI’ın beklenen tasarım faydalarının gerçek projelerde ölçülmesi
bakımından erken dönem araştırmalardan birini temsil etmektedir.
Crasso ve arkadaşları (2010)[4], Empirically Assessing the Impact of DI on the Development of
Web Service Applications çalışmalarında, DI araştırmalarını web servis tabanlı uygulama geliştirme ala
10
nına taşımıştır. DI ve Adapter tasarım örüntüsünü birleştiren DI4WS yaklaşımı geliştirilerek servis keşfi
başarısı ve kaynak tüketimi deneysel olarak incelenmiştir. Elde edilen bulgular, incelenen koşullarda ser
vis keşfinin iyileştirilebildiğini, ancak ek işlem ve bellek maliyetlerinin de oluşabildiğini göstermiştir.
Böylece araştırma odağı, DI’ın genel kalite etkisinin ölçülmesinden belirli uygulama alanlarında sağla
dığı faydaların ve maliyetlerin değerlendirilmesine genişlemiştir.
Aynı dönemde Roubtsov ve arkadaşları (2010)[5], Detecting Modularity Smells in Dependencies
Injected with Java Annotations çalışmalarında, DI kullanımının sağlayabileceği faydalardan farklı olarak,
hatalı uygulamaların oluşturabileceği tasarım problemlerine odaklanmıştır. Java anotasyonlarının modü
lerlik ilkeleriyle çelişebilecek kullanım biçimleri incelenmiş; yapılandırma bilgilerinin işlevsel kodla ka
rışması, bilgi tekrarı ve örtük bağımlılıklar gibi problemler sınıflandırılmıştır. Çalışmada bu problemlere
yönelik bir katalog ve çözüm önerileri sunulmuş, ancak bunların gerçek projelerdeki yaygınlığı kapsamlı
bir ampirik analizle değerlendirilmemiştir. Bu araştırma, DI’ın yalnızca sağladığı avantajların değil, yan
lış kullanımından doğabilecek tasarım sorunlarının da incelenmeye başlandığını göstermektedir.
Bu çalışmalar, DI araştırmalarının farklı yönlerini ortaya koymaktadır. İlk araştırmalarda DI’ın ya
zılım kalitesine ve geliştirme süreçlerine katkısı sorgulanırken, sonraki çalışmalarda mekanizmanın nasıl
kullanıldığı ve hangi koşullarda tasarım problemleri oluşturabileceği de araştırma konusu hâline gelmiş
tir. Bununla birlikte, bu çalışmalar DI kötü uygulamalarının kapsamlı biçimde tanımlanması ve gerçek
sistemlerde değerlendirilmesi konusunda sınırlı kalmıştır.
Bu araştırma birikimi üzerinde yükselen Laigner ve arkadaşları, 2019 yılında Java DI anti
pattern’lerine yönelik bir katalog önerisi sunmuş[6], 2022 yılındaki çalışmalarında ise bu yaklaşımı
genişleterek 12 anti-pattern içeren bir katalog geliştirmiştir. Önceki araştırmalardan farklı olarak DI
Analyzer adlı statik analiz aracıyla anti-pattern’lerin gerçek projelerdeki varlığı incelenmiş ve geliştirici
görüşleri doğrultusunda katalog güncellenmiştir.
Dolayısıyla alanın gelişimi, DI’ın olası faydalarının araştırılmasından kötü kullanım biçimlerinin
tanımlanmasınave bunların sistematik olarak değerlendirilmesine uzanan bir araştırma genişlemesi olarak
yorumlanabilir. Bununla birlikte, Laigner ve arkadaşlarının çalışması da anti-pattern’lerin giderilmesinin
gerçek bakım maliyetine ve yazılım kalitesine etkisini doğrudan ölçmemektedir. Bu durum, gelecekteki
araştırmalar için önemli bir çalışma alanı oluşturmaktadır.
7 Eleştirel Değerlendirme ve Geliştirme Önerileri
Laigner ve arkadaşlarının (2022) çalışmasının güçlü yönlerinden biri, Dependency Injection anti
pattern’lerini yalnızca teorik tasarım ilkeleri üzerinden tanımlamakla sınırlı kalmayıp gerçek yazılım pro
jeleri ve geliştirici görüşleri aracılığıyla değerlendirmesidir. Kataloğun tümevarımsal ve tümdengelimsel
yaklaşımlar kullanılarak oluşturulması, hem endüstriyel gözlemlerin hem de nesne yönelimli tasarım ilke
lerinin dikkate alınmasını sağlamıştır. Ayrıca DIAnalyzer aracının açık kaynak olarak sunulması, önerilen
tespit yaklaşımının incelenebilmesi ve geliştirilebilmesi açısından önemli bir katkıdır.
7.1 Araştırmanın Sınırlılıkları ve Bulguların Değerlendirilmesi
Araştırmada dikkat edilmesi gereken temel ayrımlardan biri, anti-pattern tespit başarısı ile yazı
lım kalitesi üzerindeki gerçek etkinin birbirinden farklı olmasıdır. DIAnalyzer’ın yüksek precision
ve relative recall değerlerine ulaşması, belirlenen kod örüntülerinin tespit edilmesindeki başarısını destek
lemektedir. Ancak bir kod örüntüsünün doğru tespit edilmesi, bu yapının yazılımın bakım maliyetini, de
ğiştirilebilirliğini veya test edilebilirliğini olumsuz etkilediğini doğrudan kanıtlamamaktadır. Bu nedenle
kataloğun geçerliliğinin yalnızca tespit sonuçlarıyla değil, söz konusu problemlerin yazılım kalitesine
etkileriyle de değerlendirilmesi gerektiği düşünülmektedir.
11
Bir diğer sınırlılık, kural tabanlı statik analizin uygulama bağlamını değerlendirme kapasite
sidir. Geliştirici geri bildirimleri sonucunda kataloğa Context alanının eklenmesi, belirli kod yapılarının
her durumda tasarım problemi olarak kabul edilemeyeceğini göstermektedir. Örneğin küçük bir sistemde
somut sınıfa bağımlılık kurulması veya çalışma zamanında dinamik bağımlılık çözümlemesi yapılması
mimari gereksinimlerden kaynaklanabilir. Bununla birlikte, katalogdaki bu bağlamsal düzenlemelerin
DIAnalyzer’ın tespit kurallarına ne ölçüde yansıtıldığı ayrıca değerlendirilmelidir. Bağlamın dikkate alın
maması, bazı tespitlerin yanlış pozitif olarak değerlendirilmesine neden olabilir.
Araştırmanın ampirik kapsamı açısından da bazı sınırlılıklar bulunmaktadır. İncelenen projelerin bü
yüklükleri, mimari özellikleri ve kullandıkları DI framework’leri farklıdır. Dolayısıyla anti-pattern’lerin
ham görülme sayılarının doğrudan karşılaştırılması, projelerin yapısal kalitesi hakkında yanıltıcı sonuç
lar oluşturabilir. Proje büyüklüğü, sınıf sayısı veya enjeksiyon kullanılan bileşen sayısı gibi değişkenler
dikkate alınarak yapılacak karşılaştırmaların daha anlamlı olabileceği değerlendirilmektedir.
Ayrıca incelenen projelerde Complex Producer Method (CPM) örneğinin tespit edilmemesi ve oluş
turulan referans veri kümesinde bu anti-pattern’e ait doğrulanmış örneklerin bulunmaması, ilgili tespit
kuralının başarısının değerlendirilmesini sınırlandırmaktadır. Bu durum, CPM’nin geçersiz olduğu anla
mına gelmemekle birlikte, aracın bu anti-pattern’e yönelik doğrulama kapsamının genişletilmesi gerekti
ğini göstermektedir.
Geliştirici görüşmeleri ve anketler, kataloğun algılanan yararlılığını değerlendirmesi açısından
önemli olmakla birlikte katılımcı sayılarının sınırlı olması ve değerlendirmelerin kişisel görüşlere dayan
ması, sonuçların genellenebilirliğini etkilemektedir. Daha önemlisi, geliştiricilerin refactoring yapmaya
istekli olduklarını belirtmeleri, bu değişiklikleri gerçek projelerde uygulayacaklarını veya uygulama
sonrasında yazılım kalitesinin iyileşeceğini garanti etmemektedir.
7.2 Geliştirme Önerileri
Çalışmanın geliştirilebileceği önemli alanlardan biri, anti-pattern’lerin zaman içerisindeki olu
şum ve değişim süreçlerinin incelenmesidir. Makalede proje geçmişlerine yönelik sınırlı incelemeler
bulunmakla birlikte, daha sistematik bir boylamsal analiz gerçekleştirilebilir. Git commit geçmişleri üze
rinden anti-pattern’lerin hangi kod değişiklikleri, mimari kararlar veya bakım faaliyetleri sonucunda oluş
tuğu araştırılabilir. Örneğin Useless Injection (USI) örneklerinin önceki refactoring işlemlerinden sonra
kullanılmayan bağımlılıkların kaldırılmaması nedeniyle ortaya çıkıp çıkmadığı incelenebilir. Böylece yal
nızca problemlerin varlığı değil, oluşum süreçleri de değerlendirilebilir.
Bir diğer öneri, DIAnalyzer’ın refactoring kararlarını destekleyecek şekilde geliştirilmesidir.
Mevcut katalogda farklı anti-pattern’ler için çözüm önerileri sunulsa da bu problemlerin giderilme mali
yetleri ve olası etkileri aynı değildir. Örneğin kullanılmayan bir bağımlılığın kaldırılması ile çok sayıda
sorumluluk üstlenen bir sınıfın yeniden yapılandırılması farklı düzeylerde değişiklik ve regresyon riski
oluşturabilir. Bu nedenle tespit edilen anti-pattern’lerin olası kalite etkisi, bağlamsal geçerliliği, giderilme
maliyeti ve önceliğine göre değerlendirilmesi geliştiricilere daha uygulanabilir bir rehberlik sağlayabilir.
Önerilen refactoring işlemlerinin etkilerinin öncesi ve sonrası kalite ölçümleriyle değerlendiril
mesi de araştırmayı güçlendirebilir. Coupling ve cohesion metriklerinin yanı sıra değişikliklerin kapsamı,
test edilebilirlik ve bakım faaliyetlerine ilişkin göstergeler incelenebilir. Bununla birlikte, yalnızca metrik
lerdeki değişimlerin değil, refactoring sırasında ortaya çıkan geliştirme maliyetinin ve olası yan etkilerin
de dikkate alınması gerekmektedir.
Son olarak, kataloğun Java dışındaki programlama ekosistemlerinde değerlendirilmesi önemli bir
araştırma fırsatıdır. Özellikle C# ve Python gibi dillerde kullanılan bağımlılık yönetimi mekanizmaları
farklılık göstermektedir. Bu nedenle DIAnalyzer’ın tespit kurallarının doğrudan aktarılması yerine, ilgili
dillerin ve framework’lerin özelliklerine uygun kurallar geliştirilerek yeni veri kümeleri üzerinde doğru
12
lama yapılması önerilebilir.
Genel olarak değerlendirildiğinde, çalışma DI kullanımındaki olası tasarım problemlerinin sistema
tik biçimde incelenmesine önemli bir temel sağlamaktadır. Bununla birlikte, bir anti-pattern’in tespit
edilmesi, mutlaka refactoring uygulanması gerektiği anlamına gelmemektedir. Tasarım kararlarının
sistem gereksinimleri, bakım maliyeti ve beklenen kalite kazanımları birlikte değerlendirilerek verilmesi
daha uygun bir yaklaşım olacaktır.
8 Sonuç
Bu raporda Laigner ve arkadaşlarının Dependency Injection anti-pattern’lerine ilişkin araştırması;
amacı, yöntemi, önerdiği katalog, otomatik tespit bulguları ve geliştirici değerlendirmeleri açısından in
celenmiştir. Çalışmanın temel katkısı, DI kullanımındaki olası tasarım problemlerini sistematik olarak
tanımlaması ve bunları gerçek yazılım projelerinden ve uygulayıcı görüşlerinden elde edilen kanıtlarla
değerlendirmesidir. Katalogdaki örnekler, DI mekanizmasının doğru nesne yönelimli tasarım kararla
rının yerini alamayacağını; sorumluluk dağılımı, uygun soyutlamalar ve bağımlılık ilişkilerinin ayrıca
değerlendirilmesi gerektiğini göstermektedir.
Bununla birlikte aracın tespit başarısı, her anti-pattern’in gerçek bakımmaliyetineetkisini veyaöneri
len refactoring’in sağlayacağı kalite kazanımını doğrudan ölçmemektedir. Bu nedenle kataloğun bağlam
sal bir tasarım inceleme rehberi olarak kullanılması; gelecek çalışmalarda boylamsal analizler, refactoring
öncesi ve sonrası kalite ölçümleri ve farklı programlama ekosistemlerinde doğrulamalarla geliştirilmesi
uygun görülmektedir.