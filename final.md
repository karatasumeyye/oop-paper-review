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