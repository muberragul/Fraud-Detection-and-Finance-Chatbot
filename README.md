# Auto Loan Chatbot Demo Using Llama 3

## Görev: Araç Finansmanı Chatbot Tasarım

**Senaryo Açıklaması:** Müşteriler mobil uygulama tarafından sunulan sohbet robotu yardımıyla bankamızda taşıt finansmanı ön başvurusu yapacaktır. Taşıt finansmanı yeni veya 2. (ikinci) el araçlar için yapılacaktır. Müşteriden alınması gereken bilgiler ve bu bilgilerin sağlaması gereken kriterler araç finansmanı türüne göre aşağıdaki gibidir:

**Yeni Taşıt Finansmanı için:**

- Araç proforma fatura değeri (7M üzeri araçlar için başvuru yapılamaz)
- Araç modeli (Ticari Modeller için başvuru yapılamaz)
- Kefil TCKN (Araç Fiyatı 5M ve üzeri olan başvurular için gereklidir. Diğer durumlar kefil gerektirmez)
- İstenen finansman tutarı (Araç Fiyatının en fazla %60’u talep edilebilir)

**İkinci El Finansmanı için:**

- Araç kasko değeri
- Araç Yaşı (5 yaş üstü araçlar için başvuru oluşturulamaz)
- İstenen finansman tutarı (Araç Kasko değerinin en fazla %40’ı veya üst barem olan 3M turarını aşamaz)
- Satıcı T.C. kimlik numarası (Bu alan zorunlu değildir. Boş bırakılabilir.)

Bu kapsamda chatbot’umuz müşterinin araç finansmanı türünü (yeni veya ikinci el) netleştirecek, sonrasında da araç finansmanı türüne göre alması gereken bilgileri toplayacak ve bu bilgilerin sağlaması gereken koşulları kontrol edecektir. Tasarlanan chatbot müşterilerin kişisel bilgilerini banka dışına çıkarmadan çalışmalıdır. Süreci başarıyla ilerleyen müşteriler için toplanan bilgiler son kez müşteriye teyit ettirilecektir. Müşteri bu bilgileri güncelleyebilecektir. Sonrasında ise chatbot tarafından veri tabanına ön başvuru kaydı atılması sağlanacaktır.

Ayrıca kurumda araç finansmanı ile ilgili sıkça sorulan sorular derlenip süreci anlatan bir doküman haline getirilmiştir. Chatbot’un bu dokümanı kullanarak müşterinin araç finansmanıyla ilgili olası sorularını yanıtlayabilmesi de beklenmektedir.

Taşıt finansmanı ön başvurusunu başarıyla yapan müşteriler için çapraz satış politikası kapsamında HGS ürünü isteyip istemediği de sorulacaktır.

Yerel sunucularımız üzerinde kurulu 2 adet her biri 48 GB olacak şekilde toplam 96 GB GPU kaynağımız mevcuttur.

Bu chatbot’u bulutta yer alan veya yerel’e indirilen açık kaynaklı büyük dil modelleriyle geliştirebilirsiniz.
