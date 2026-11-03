# 1. Introduction — Analiz Notları

Dependency Injection (DI), yazılım bileşenlerinin ihtiyaç duydukları bağımlılıkların dışarıdan sağlanmasını esas alan bir mekanizmadır. Böylece bağımlılıkların yönetilmesini kolaylaştırmayı ve uygun tasarımlarda bileşenler arasındaki sıkı bağlılığı (tight coupling) azaltmayı amaçlamaktadır. DI, Dependency Inversion Principle (DIP) ve Inversion of Control (IoC) kavramlarıyla yakından ilişkilidir. DI kullanımının yaygınlaşmasıyla birlikte çeşitli framework'ler geliştirilmiş, ilerleyen süreçte bağımlılık yönetimini destekleyen mekanizmalar programlama dillerinin ve yazılım geliştirme ekosistemlerinin bir parçası hâline gelmiştir.

Bu gelişmeler DI'ın uygulanmasını kolaylaştırsa da beraberinde bazı tasarım problemlerini getirmiştir. DI mekanizmasını kullanmak, tek başına iyi bir nesne yönelimli tasarım oluşturulduğunu garanti etmemektedir. Mekanizmanın yanlış veya bilinçsiz kullanılması, azaltılması amaçlanan bağımlılık problemlerinin farklı biçimlerde ortaya çıkmasına neden olabilmektedir.

Makalenin ele aldığı temel problem, DI'ın hatalı kullanımlarına ilişkin geliştiricilere sunulan rehberliğin yetersiz olmasıdır. Literatürde çeşitli DI kötü uygulamaları tanımlanmış olsa da bu önerilerin kapsamı, pratikteki geçerliliği ve genellenebilirliği konusunda yeterli araştırma bulunmadığı vurgulanmaktadır. Bu nedenle DI anti-pattern'lerinin sistematik olarak belirlenmesi, belgelenmesi ve gerçek yazılım sistemleri üzerinden değerlendirilmesi gerektiği ifade edilmektedir.

Bu problemden hareketle araştırmanın temel amacı, DI kullanan yazılım sistemlerinin yapısal kalitesinin iyileştirilmesine katkı sağlayacak kapsamlı bir DI anti-pattern kataloğu önermek ve bu kataloğu değerlendirmektir. Önerilen katalog ile geliştiricilere DI ile ilişkili problemlerin tespit edilmesi, analiz edilmesi ve refactoring yoluyla giderilmesi konusunda rehberlik sağlanması hedeflenmektedir.

Araştırma dört temel soru üzerine şekillenmektedir:

- **RQ1:** DI uygulamasıyla ilişkili olup mevcut belgelenmiş kötü DI uygulamalarının yeterince kapsamadığı problem adayları var mıdır?
- **RQ2:** Önerilen DI anti-pattern'leri uygulamada ortaya çıkmakta mıdır?
- **RQ3:** Uzman geliştiriciler önerilen anti-pattern'leri uygulamada yararlı bulmakta mıdır?
- **RQ4:** Uygulayıcılar kaynak koddan DI anti-pattern örneklerini refactoring ile kaldırmaya istekli midir?

Bu araştırma sorularının birbirini tamamlayan bir yapıda olduğu görülmektedir. Öncelikle mevcut literatürdeki eksiklikler belirlenmekte, ardından önerilen anti-pattern'lerin gerçek yazılım projelerinde bulunup bulunmadığı incelenmektedir. Sonraki aşamalarda ise geliştiricilerin kataloğa yönelik değerlendirmeleri ve refactoring konusundaki tutumları araştırılmaktadır. Böylece çalışma, yalnızca teorik bir katalog önerisi sunmakla sınırlı kalmayıp önerilerin pratik açıdan ne ölçüde anlamlı olduğunu da değerlendirmeyi amaçlamaktadır.

Makalenin giriş bölümünde araştırmanın temel katkıları altı başlıkta belirtilmektedir:

1. DI üzerine kapsamlı araştırmaların eksikliğinin ortaya konulması; mevcut önerilerin geçerlilik ve genellenebilirlik açısından sınırlılıklarının incelenmesi.
2. Geliştiricilerin DI mekanizmasını yanlış kullanmaktan kaçınmalarına yardımcı olacak yeni bir DI anti-pattern kataloğunun önerilmesi.
3. DI kullanan Java projelerinde anti-pattern örneklerini tespit etmek amacıyla yüksek precision değerine sahip, GitHub üzerinden erişilebilen bir static analysis aracının geliştirilmesi.
4. Önerilen anti-pattern'lerin farklı açık ve kapalı kaynak yazılım projelerinde ortaya çıktığının gösterilmesi.
5. Geliştiricilerin kataloğu ilgili ve yararlı bulduklarının, ayrıca tespit edilen anti-pattern örneklerini refactoring ile gidermeye istekli olduklarının gözlemlenmesi.
6. Katılımcıların değerlendirmelerinin analiz edilerek kataloğun uygulayıcıların ihtiyaçlarını daha iyi karşılayacak şekilde güncellenmesi.

>DI'ın framework'ler ve programlama dillerindeki desteklerle yaygınlaşması, geliştiricilerin bu yaklaşımın arkasındaki tasarım prensiplerini yeterince kavradığı anlamına gelmemektedir. DI kullanımının yalnızca teknik bir uygulama olarak ele alınması, bağımlılık yönetiminde hedeflenen faydaların elde edilememesine yol açabilir. Bu nedenle mekanizmanın nasıl uygulandığının yanında neden kullanıldığının da anlaşılması önemlidir.

>Bununla birlikte, bağımlılıkları azaltma amacıyla gereğinden fazla soyutlama oluşturulması da yazılımın karmaşıklığını ve bakım maliyetini artırabilir. Her sınıf için interface oluşturulması veya her bağımlılığın mutlaka soyutlanması, tek başına daha iyi bir tasarım ortaya koymaz. Bu açıdan önemli olan, bağımlılıkları mümkün olduğunca azaltmak değil, sistemin ihtiyaçlarına uygun ve gerekçelendirilebilir tasarım kararları vermektir.

>Araştırmanın güçlü yönlerinden biri, önerilen anti-pattern'leri yalnızca teorik düzeyde ele almaması; gerçek projelerde tespit etmeye çalışması ve geliştirici görüşleriyle değerlendirmesidir. Bununla birlikte, bir anti-pattern'in projelerde sık görülmesi veya geliştiriciler tarafından yararlı bulunması, ilgili problemin giderilmesinin yazılım kalitesini doğrudan iyileştirdiğini tek başına kanıtlamamaktadır.

