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