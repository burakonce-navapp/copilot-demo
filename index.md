---
title: Seyahat Talep Sistemi Kullanım Rehberi INTEC
description: INTEC özelleştirme projesi için Dynamics 365 Business Central Seyahat Talep Sistemi kullanım adımları, temel alanlar ve onay süreçleri hakkında detaylı eğitim dokümanı.
---

<meta name="description" content="INTEC özelleştirme projesi için Dynamics 365 Business Central Seyahat Talep Sistemi kullanım adımları, temel alanlar ve onay süreçleri hakkında detaylı eğitim dokümanı.">

# INTEC Seyahat Talep Sistemi Kullanım Kılavuzu

// İçeriğiniz buradan devam ediyor...

## Giriş ve Genel Bakış

## Seyahat Talep Sistemi Nedir?

INTEC Customization projesi kapsamında geliştirilen Seyahat Talep Sistemi, çalışanların kurumsal seyahatlerini yönetmek için tasarlanmış kapsamlı bir çözümdür. Bu sistem şirkete ait uçak biletleri, konaklama, araba kiralama gibi seyahat giderlerini organize etmek, takip etmek ve raporlamak için kullanılır.

## Sistem Bileşenleri

Seyahat Talep Sistemi aşağıdaki ana bileşenlerden oluşur:

- Ana Talep Tablosu: Seyahat taleplerinin ana bilgilerini içerir
- Alt Bileşenler: Bilet, Konaklama, Araba Kiralama
- Seyahat Günlüğü: Faturalı işlemleri takip eder
- Defter Girdileri: Muhasebe entegrasyonu
- Onay İşlevi: Workflow tabanlı onay süreci

## Ana Tablo: INTCTravelRequest

## Temel Alanlar

## No (Kod[20])

Seyahat talebinin benzersiz numarası. Otomatik numara serisi tarafından oluşturulur. Numara serisi CustomizationSetup.TravelRequestFormNo tablosundan alınır. Sistem içinde bir talep oluşturulduğunda, No Series mekanizması tarafından otomatik olarak bir sonraki numara atanır.

## Status (Enum INTCDocumentStatus)

Seyahat talebinin şu anki durumunu gösterir:

- Open: Yeni oluşturulan talep, düzenleme aşaması. Tüm alanlar değiştirilebilir. Kullanıcı bu durumdayken talepte değişiklik yapabilir.
- PendingApproval: Onay bekleniyor. Talep yöneticiye gönderilmiştir. Düzenleme yapılamaz.
- Released: Onaylanmış, dış taraflar ile iletişime geçilebilir. Bu durumda alt bileşenler günlüğe aktarılabilir.

## FillingBy (Kod[20])

Talebi dolduran kişinin çalışan numarası. Otomatik olarak sistem kullanıcısından alınır. User Setup.NappWBSEmployeeNo alanından çekilen bilgidir. Talep oluşturulduğunda otomatik dolar, değiştirilebilir.

## RequestedBy (Kod[20])

Seyahat yapacak olan çalışan numarası. Manuel olarak seçilir. Pozisyon ve ad bilgileri otomatik olarak Employee tablosundan alınır. Lookup aracılığıyla çalışan seçilir.

## Seyahat Detayları

## County (Text[50])

Seyahat yapılacak ilçe. INTCTRDestination tablosundan lookup yapılır. County seçilmesi otomatik olarak City ve Country alanlarını doldurur. Destinasyon kayıtları önceden sistem yöneticisi tarafından girilmiştir.

## City (Text[50])

Seyahat şehri. County seçilmesi sonrası otomatik dolar. Düzenlenebilir değildir (Editable = false).

## Country (Text[50])

Seyahat yapılacak ülke. County seçilmesi sonrası otomatik dolar.

## AdvanceCurrency (Code[10])

Para birimi seçimi. Currency tablosundan lookup yapılır. Örneğin: USD, EUR, TRY. İlerleme avansı hangi para biriminde verileceğini belirtir.

## AdvanceAmount (Decimal)

İşletme avansı tutarı. Seyahat öncesi avans olarak verilen para miktarı. Talep oluşturulurken, bütçe aşılmadığını kontrol etmek için kullanılabilir.

## DocumentDate (Date)

Seyahat talebinin oluşturulduğu tarih. Otomatik olarak sistem tarihinden alınır (Today). Tüm alt bileşenlerin tarihleri bu tarihten eski olamaz.

## JobNo (Code[20])

Seyahat nedenini belirtmek için Job (İş/Proje) numarası seçilir. Job tablosundan lookup yapılır. Project Management modülü ile entegredir.

## JobDescription (Text[100])

JobNo seçildiğinde otomatik dolar. İş tanımı bilgisi. Örneğin "Ankara'da müşteri ziyareti" vs.

## MainCostCategory (Text[50])

Ana maliyet kategorisi. Dimension Value tablosundan seçilir. Muhasebe boyutlandırması için kullanılır. CustomizationSetup.MainCostCategoryDimensionValue'den belirtilen boyut değerleri gösterilir.

## SubCostCategory (Text[50])

Alt maliyet kategorisi. Ana kategoriye bağlı boyut değeri. Örneğin ana kategori "Seyahat" ise, alt kategori "Uçak Bileti" olabilir.

## Department (Text[50])

Çalışanın departmanı. Dimension Value tablosundan seçilir. Raporlama ve maliyet analizi için kullanılır.

## Alt Bileşenler: Seyahat Türleri

## Bilet (INTCTRTicket)

## TransportationType (Enum INTCTRTransportationType)

Taşıma türü seçilir: Bus, Plane, Ship, Train. Her tip için farklı hub'lar (istasyonlar, havaalanları) tanımlanabilir.

## FromHub ve ToHub (Text[100])

Kalkış ve varış noktaları. Transportation Hub tablosundan lookup yapılır. TransportationType seçilmesi sonrası uygun hub'lar filtrelenerek gösterilir.

## NumberOfPeople (Integer)

Seyahat edecek kişi sayısı. Örneğin 3 kişi ortak olarak seyahat ediyorsa, bu alanda 3 girilir.

## DepartureDate (Date)

Kalkış tarihi. DocumentDate'den eski olamaz. CheckDocumentDate procedure'ü tarafından kontrol edilir.

## ReturnDate (Date)

Dönüş tarihi. DepartureDate'den sonra olmalıdır. ReturnDate - DepartureDate otomatik olarak Days alanında hesaplanır.

## Days (Integer)

Seyahat gün sayısı. ReturnDate seçildiğinde otomatik hesaplanır. Maliyetlendirme için önemlidir.

## FlexibleTravelDescription (Text[100])

Esnek seyahat seçeneği. Örneğin "Gidiş için 2 seçenek" gibi açıklamalar. İptal veya değişiklik durumunda kullanılır.

## AlternativeDepartureDate ve AlternativeArrivalDate

Alternatif tarihler. İlk tarihler uygun değilse, yedek olarak başka tarihler girilir. AlternativeDays otomatik hesaplanır.

## ReasonCode (Code[10])

Bilet iptali veya değişikliğinin nedeni. Reason Code tablosundan seçilir. Raporlama amaçlı kullanılır.

## Konaklama (INTCTRAccommodation)

## CheckInDate (Date)

Otel girişi tarihi. DocumentDate'den eski olamaz.

## CheckOutDate (Date)

Otel çıkışı tarihi. CheckInDate'den sonra olmalıdır. CheckOutDate - CheckInDate otomatik Days'e yazılır.

## City (Text[30])

Konaklama şehri. Post Code tablosundan lookup yapılır. INTCShowOnTravelRequest = true olan şehirler gösterilir.

## NumberOfPeople (Integer)

Oda paylaşacak kişi sayısı veya grup büyüklüğü.

## Description (Text[2048])

Otel adı, oda tipi, özel istekler vb. açıklamalar. Detaylı bilgi alanıdır.

## Araba Kiralama (INTCTRCarRent)

## PickUpDate (DateTime)

Araç alış tarihi ve saati. DateTime tipinde tutulur, saat bilgisi de kaydedilir.

## DropOffDate (DateTime)

Araç iadesinin tarihi ve saati. PickUpDate'den sonra olmalıdır. Hata alınırsa "Pick Up Date cannot be later than Drop Off Date" uyarısı gösterilir.

## Days (Integer)

Kiralama gün sayısı. DropOffDate - PickUpDate otomatik hesaplanır. Maliyetlendirme için kullanılır.

## PickUpLocation ve DropOffLocation (Text[100])

Araç alış ve iade yerleri. Örneğin "Havaalanı", "Şehir merkezi" vb.

## DriverName (Text[100])

Araç kiralayan şoför adı. Belirtilmezse opsiyoneldir.

## CurrencyCode (Code[10])

Araç kiralama parasının hangi biriminde ödeneceği. Currency tablosundan seçilir.

## Amount (Decimal)

Toplam kiralama ücreti. Değiştirildiğinde AmountLCY otomatik hesaplanır.

## AmountLCY (Decimal)

Tutarın yerel para (LCY) cinsinden değeri. Döviz kuruna göre otomatik hesaplanır.

## CostPerDayLCY (Decimal)

Günlük maliyet yerel para cinsinden. AmountLCY / Days formülü ile hesaplanır.

## Seyahat Günlüğü: INTCTravelJournal

## Amaç ve Kullanım

Seyahat talepleri onaylandıktan sonra, alt bileşenleri (bilet, konaklama, araba kiralama) Seyahat Günlüğü'ne taşınır. Bu, fatura oluşturuluncaya kadar geçici bir depolama alanıdır. Günlük, "Get Travel Requests" işlemi tarafından dolulur.

## TravelRequestNo (Code[20])

Bağlı olduğu seyahat talebinin numarası. Bu alan seçildiğinde, TravelRequest kaydından otomatik bilgiler (Position, RequestedBy, MainCostCategory, SubCostCategory, Department, DimensionSetId, PostingDate) çekilerek doldurulur.

## LineNo (Integer)

Alt bileşenin satır numarası. Talep içindeki bilet, konaklama veya arabanın satır numarası.

## TravelRequestLineType (Enum INTCTravelRequestLineType)

Satırın türü: Accommodation, Ticket, CarRent. Hangi alt tablodan verileri geldiğini gösterir.

## InvoiceDate ve InvoiceNo

Fatura tarihi ve numarası. Hizmet sağlayıcıdan gelen faturanın bilgileri girilir. Satıcı faturası oluşturulurken bu bilgiler kullanılır.

## ConfirmationNo (Code[20])

Rezervasyon onay numarası. Özellikle bilet ve konaklama için kullanılır. Fatura numarası olmadan da, onay numarası ile fatura oluşturulabilir.

## PartnerCompanyName (Text[200])

Hizmet sağlayıcı (havayolu, otel, kiralama şirketi) adı. INTCTJPartnerCompany tablosundan seçilir. Seçilmesi VendorNo'yu otomatik doldurur.

## VendorNo (Code[20])

Satıcı numarası. Partner Company seçilmesi sonrası otomatik dolar. Satın Alma Faturası oluştururken bu satıcı kullanılır.

## CurrencyCode ve Amount

Para birimi ve tutarı. Dışarıdan gelen fatura bilgileridir.

## AmountLCY (Decimal)

Yerel para cinsinden tutar. CalcAmountLCY procedure'ü ile döviz kuruna göre hesaplanır.

## CostPerDayLCY (Decimal)

Günlük maliyet. CalcCostPerDay procedure'ü ile, alt bileşenin gün sayısına bölünerek hesaplanır. Örneğin 3 gün konaklama 900 TL ise, günlük 300 TL.

## MainCostCategory, SubCostCategory, Department

Muhasebe kategorileri. TravelRequestNo seçildiğinde otomatik dolar. Defter girdisine aktarılırken korunur.

## BookingNo (Text[100])

Rezervasyon numarası. Özellikle uçak biletleri için PNR numarası (Passenger Name Record).

## WhoBuys (Code[20])

Kimi tarafından satın alındığını gösterir. User Setup tablosundan seçilir. Örneğin "Muhasebe müdürü" vs.

## PaymentMethod (Code[10])

Ödeme yöntemi. Payment Method tablosundan seçilir. Kredi kartı, çek, nakit vb.

## Vehicle, PickUp, DropOff, Route (Text[100])

Araba kiralama özgü bilgiler. Günlük oluştururken, alt bileşenlerden otomatik aktarılır.

## TripType (Enum INTCTJTripType)

Seyahat tipi: OneWay (tek yön) veya RoundTrip (gidiş-dönüş). Bilet türünü belirtir.

## AddressOrHostName (Text[200])

Konaklama yeri adresi veya otel adı. Veya restoran, işletme adresi vb.

## PostingDate (Date)

Muhasebe kayıt tarihi. Defter girdisinde bu tarih kullanılır. Varsayılan olarak bugünkü tarih girilir.

## Posted (Boolean)

Günlüğün Defter Girdilerine aktarılıp aktarılmadığını gösterir. Yayımlandıktan sonra true olur.

## InvoiceCreated ve InvoiceCreatedNo

Satın Alma Faturası oluşturulup oluşturulmadığını ve fatura numarasını gösterir.

## Defter Girdileri: INTCTravelLedgerEntry

## Amaç ve Özellikler

Seyahat Günlüğü yayımlandıktan sonra, kalıcı kayıt olarak Seyahat Defter Girdi'sine aktarılır. Bu, muhasebe raporları ve analizler için kullanılır. Okuma yalnızıdır, düzenlenemez. Otomatik olarak günlük yayımlandığında oluşturulur.

## Alan Yapısı

Defter Girdi'sinin neredeyse tüm alanları Günlük'ten aktarılır. TravelRequestNo, LineNo, TravelRequestLineType, InvoiceDate, InvoiceNo, PartnerCompany, CurrencyCode, Amount, AmountLCY, CostPerDayLCY, MainCostCategory, SubCostCategory, RequestedBy, Position, BookingNo, WhoBuys, PickUp, DropOff, Route, Vehicle, TripType, AddressOrHostName, PostingDate, FlexibleTravel, Department, DimensionSetId.

## Ek Bilgiler

Defter Girdisinde ConfirmationNo ve GeneralProdPostingGroup ve VatProdPostingGroup de kaydedilir. Ekler (attachments) başvuru için saklanır. Atanan çalışanlar görüntülenebilir.

## Onay Süreci (Workflow)

## Durum Değişim Diyagramı

Open (Yeni Talep)
↓ [Send Approval Request]
PendingApproval (Onay Bekleniyor)
↓ [Approve] [Reject] [Delegate]
Released (Onaylandı) veya Open (Geri Döndü)

## INTCTravelRequestApprovalsMgt Codeunit'i

Bu codeunit, Workflow yönetimini sağlar. Onay taleplerini gönder, iptal et, onay durumu kontrol et gibi işlemler yapılır.

## CheckApprovalPossible()

Workflow onay süreci etkin mi ve talep uygun mu kontrol eder. Etkin değilse hata verir: "This record is not supported by related approval workflow."

## IsApprovalWorkflowEnabled()

Talep için onay workflow'u aktif mi denetler. WorkflowManagement.CanExecuteWorkflow() metodunu kullanır.

## OnSendForApproval()

Onay isteği göndermek için çağrılır. Integration Event'tir. Status alanını PendingApproval'a değiştirir. WorkflowManagement.HandleEvent() event'i tetikler.

## OnCancelApprovalRequest()

Pending Approval durumundaki talep için onay iptal eder. Integration Event'tir. Workflow'da cancel event'i tetikler.

## Event Subscribers

Workflow Event Handling'e event'ler kaydedilir. Onay öncesi-sonrası event'ler tanımlanır. Approval Entry kaydının RelationValue'su INTCTravelRequest olarak ayarlanır.

## Release Süreci (INTCReleaseTravelRequest)

## Amaç

Onaylanan seyahat taleplerini "Released" durumuna getirmek. Released durumdaki talepler:
- Dış taraflarla paylaşılabilir
- Günlüğe aktarılabilir
- Alt bileşenleri değiştirilemez

## PerformManualRelease()

Kullanıcı tarafından "Release" butonu tıklandığında çağrılır. Onay süreci tamamlanmış olmalıdır. PendingApproval durumdaysa hata verir. Status alanını Released'a değiştirir.

## Reopen()

Released durumundaki talep tekrar Open'a dönüştürülür. Düzenlemelerin yapılması gerektiğinde kullanılır.

## TestReleasePossible()

Release yapılmadan önce kontroller yapılır. Approval workflow aktifse ve hala PendingApproval durumdaysa izin vermez.

## Seyahat Sayfaları ve İşlemler

## INTCTravelRequests (Sayfa 50002) - Liste Görünümü

Bu sayfa tüm seyahat taleplerini listeler. Temel bilgiler (No, Status, City, Country, County, FillingBy, RequestedBy) gösterilir. Yeni talep oluşturulamaz (InsertAllowed = false). Detay görmek için talep numarasını tıklayarak INTCTravelRequest kartını açarsınız.

## INTCTravelRequest (Sayfa 50003) - Kart Görünümü

Ayrıntılı seyahat talep kartı. "Seyahat Talepleri Listesi"nden bir talep seçildiğinde açılır. CardPageId = INTCTravelRequest olarak ayarlanmıştır.

## Layout Bölümleri

Genel Bilgiler: No, Status, FillingBy, FillingByFullName, FillingByPosition, RequestedBy, RequestedByFullName, RequestedByPosition, DocumentDate, County, City, Country, JobNo, JobDescription, MainCostCategory, SubCostCategory, Department, AdvanceCurrency, AdvanceAmount.

## Alt Sayfalar (Parts)

Part(Accomodation; INTCTRAccommodation): Konaklama alt bileşenleri. SubPageLink = TravelRequestNo = field(No).
Part(Ticket; INTCTRTicket): Bilet alt bileşenleri.
Part(CarRent; INTCTRCarRent): Araba kiralama alt bileşenleri.

## Fact Boxes

INTCTravelRequestAttchmntsSubP sayfası ek'leri gösterir.

## Eylemler - Navigation Grubu

## ShowTravelLedgerEntries

Talep için oluşturulan tüm Defter Girdileri'ni listeler. INTCTravelLedgerEntry tablosundan filtrelenerek gösterilir.

## Dimensions

Muhasebe boyutlarını düzenleme sayfasını açar. DimensionManagement.EditDimensionSet() metodunu kullanır.

## Approval Grubu

## ApproveAction

Talep üzerine açık onay girişi varsa, bunu onayla. ApprovalsMgmt.ApproveRecordApprovalRequest() çağrılır. Visible = OpenApprovalEntriesExistForCurrUser.

## RejectAction

Talep üzerine açık onay girişi varsa, bunu reddet. ApprovalsMgmt.RejectRecordApprovalRequest() çağrılır.

## DelegateAction

Onay yapılacak kişiyi değiştir. ApprovalsMgmt.DelegateRecordApprovalRequest() çağrılır.

## Request Approval Grubu

## SendApprovalRequestAction

Talep onay için gönder. ApprovalsMgmt.CheckApprovalPossible() ve OnSendForApproval() çağrılır. Enabled = not OpenApprovalEntriesExist (yani zaten açık onay girişi yoksa).

## CancelApprovalRequestAction

Gönderilen onay isteğini iptal et. OnCancelApprovalRequest() çağrılır. Enabled = OpenApprovalEntriesExist.

## Release Grubu

## ReleaseAction

Onaylanan talep sürümü alındığında çağrılır. ReleaseTravelRequest.PerformManualRelease() çağrılır. Enabled = (Status = Open) and not OpenApprovalEntriesExist. ShortCutKey = 'Ctrl+F9'.

## ReopenAction

Released talep tekrar Open'a dönüştürülür. ReleaseTravelRequest.PerformManualReopen() çağrılır.

## Seyahat Günlükleri (INTCTravelJournals - Sayfa 50009)

Bu sayfa, onaylanmış seyahat taleplerinden bilet, konaklama, araba kiralama satırlarının fatura ve muhasebe işlemlerini yönetir. Worksheet türü sayfadır. SourceTableView = where(Posted = const(false)) - yayımlanmamış günlükler gösterilir.

## Get Travel Requests Eylemi

- Sürüm alınmış taleplerinden (Released) henüz günlüğe aktarılmamış satırları bulur
- INTCTravelRequestBuffer'a geçici olarak yükler
- Kullanıcı seçim yapmasını sağlar
- Seçilen satırlar INTCTravelJournal'a aktarılır
- Kaynak tabloda HasJournalLine = true olur

## Post Travel Journal Eylemi

- Seçilen günlük satırlarını Defter Girdisine aktarır
- Her satır için INTCTravelLedgerEntry kaydı oluşturur
- Posted = true olur
- Kaynakta Posted = true olur

## Create Purchase Invoice Eylemi

- Seçilen günlük satırlarından Satın Alma Faturası oluşturur
- Her satır için Purchase Line oluşturulur
- VendorNo, Amount, CurrencyCode, Dimensions vb. aktarılır
- INTCTravelRequestNo, INTCTravelRequestLineNo, INTCTravelRequestLineType purchase line'a yazılır
- InvoiceCreated = true, InvoiceCreatedNo atanır

## Defter Girdileri Sayfası (INTCTravelLedgerEntries - Sayfa 50010)

Bu sayfa, tamamlanmış ve muhasebeleştirilmiş seyahat işlemlerinin kalıcı kaydını gösterir. Editable = false, InsertAllowed = false, DeleteAllowed = false. Yalnızca görüntüleme amaçlıdır.

## Dimensions Eylemi

Defter girdisinin muhasebe boyutlarını görüntüle. DimensionManagement.ShowDimensionSet() çağrılır.

## Assigned Employees Eylemi

Bu defter girdisine atanan çalışanları listeler. INTCTJAssignedUser kaydından filtrelenerek gösterilir.

## Ek Bileşenler

## INTCTravelRequestBuffer (Tablo 50009)

Get Travel Requests işleminde geçici veri depolama için kullanılır. TravelRequestNo, LineNo, LineType, Description, FlexibleTravel, PickUp, DropOff, Route alanlarını içerir. UserSession'ın sonunda silinir.

## INTCTravelRequestAttachment (Tablo 50010)

Seyahat talebine eklenen dosyaları saklar. FileName, Description, Attachment (Media) alanları. ImportFile() ve ExportFile() procedure'leri vardır. TravelRequestNo başına multiple attachment destekler.

## INTCTJPartnerCompany (Tablo 50004)

Havayolu, otel, kiralama şirketi gibi hizmet sağlayıcıları tanımlar. TravelRequestLineType'a göre filtreli. Vendor'a bağlantılı. Seyahat Günlüğünde seçilir.

## INTCTJAssignedUser (Tablo 50011)

Bir seyahat satırına birden fazla çalışan atanabilir. Grup seyahatlerinde tüm katılımcılar kayıt edilir. TravelRequestNo, TravelRequestLineNo, TravelRequestLineType, EmployeeNo, EmployeeName alanları vardır.

## INTCTRDestination (Tablo 50002)

Sık seyahat yapılan hedefler önceden tanımlanır. No, County, City, Country, VendorNo alanları. Numara serisi CustomizationSetup.DestinationNo'dan alınır.

## INTCTRTransportationHub (Tablo 50012)

Ulaşım merkez ve düğümleri (havaalanları, tren istasyonları, otobüs terminalleri). Code, TransportationType, HubName, CityName, CountryName alanları. Bilet türü seçilmesi sonrası uygun hub'lar filtrelenerek gösterilir.

## Enumeration (Numaralandırma) Türleri

## INTCDocumentStatus

- Open (0): Yeni, düzenleme aşaması
- Released (1): Onaylanmış
- PendingApproval (2): Onay bekleniyor

## INTCTravelRequestLineType

- Accommodation (1): Konaklama
- Ticket (2): Bilet
- CarRent (3): Araba kiralama

## INTCTRTransportationType

- Bus: Otobüs
- Plane: Uçak
- Ship: Gemi
- Train: Tren

## INTCTJTripType

- OneWay: Tek yön
- RoundTrip: Gidiş-dönüş

## INTCTravelRequestMgt Codeunit (50003)

Bu codeunit, seyahat taleplerini günlüğe aktarmak ve yönetmek için temel işlevleri sağlar.

## InitTravelRequestLines()

- Released durumdaki tüm taleplerini bulur
- Her talepte henüz HasJournalLine = false olan satırları arar (Ticket, Accommodation, CarRent)
- Bu satırları INTCTravelRequestBuffer'a yükler
- Bilgiler: TravelRequestNo, LineNo, LineType, Description vb.

## SelectTravelRequestLines()

- InitTravelRequestLines() çalıştırıldıktan sonra
- INTCTravelRequestBuffer sayfası (INTCTravelRequestBuffer) açılır
- Kullanıcı istediği satırları seçer (multi-select)
- Seçilen satırlar INTCTravelJournal'a aktarılır
- HasJournalLine = true olur

## UpdateRequestLineHasJournalAndPosted()

- Kaynak satırın HasJournalLine ve Posted alanlarını günceller
- SetPosted = true ise Posted = true yapılır
- SetHasJournal = true ise HasJournalLine = true yapılır
- Parametreler: TravelRequestNo, TravelRequestLineType, TravelRequestLineNo, SetPosted, SetHasJournal

## CleanRequestLines()

- Günlük satırı silinmişse, kaynak satırda HasJournalLine = false yapılır
- Günlük sayfasında Delete işlemi yapıldığında çağrılır

## CleanAssignedUsers()

- Silinen günlük satırının atanan çalışanlarını siler
- AssignedUser kaydından TravelRequestNo, TravelRequestLineType, TravelRequestLineNo göre filtreler ve siler

## CheckDocumentDate()

- Girilen tarihin DocumentDate'den eski olmadığını kontrol eder
- Çift yönlü olarak Ticket ve Accommodation'da OnValidate trigger'ında çağrılır
- Eski tarihe girilmişse uyarı gösterilir

## UpdateReasonCodeInTravelRequestLine()

- Günlükte ReasonCode değiştirilmişse, kaynak satırın ReasonCode'unu günceller
- Ticket, Accommodation, CarRent'de Reason Code lookup'ları filtredir

## LookupTravelRequestLineNo()

- Purchase Line içinde Travel Request Line seçmek için kullanılır
- LineType'a göre uygun sayfa açılır
- Ticket için INTCTRTicketSubPg, Accommodation için INTCTRAccommodationSubPg, CarRent için INTCTRCarRentSubPg

## İş Akışı - Tam Örnek

## 1. Talep Oluştur

Seyahat Talepleri listesine girilir. +Yeni düğmesi tıklanır. Sistem otomatik olarak No, Status, FillingBy, DocumentDate atar. RequestedBy, hedef (County/City/Country), AdvanceAmount, MainCostCategory, SubCostCategory, Department ve Job seçilir.

## 2. Alt Bileşen Ekle

Talep kartında Ticket, Accommodation, CarRent sayfalarında yeni satır eklenir. Her birine tarih, yer, para vb. bilgiler girilir.

## 3. Boyut Ata (Opsiyonel)

Talep kartında "Dimensions" butonu tıklanarak muhasebe boyutları tamanlanır.

## 4. Onay İsteği

"Send Approval Request" butonu tıklanır. Status = PendingApproval olur. Onaylayanın Inbox'ına talep gider.

## 5. Onay Alma

İlgili yönetici Approval Entry'yi görür. "Approve" (Onayla), "Reject" (Reddet) veya "Delegate" (Devret) seçer.

## 6. Onay Sonrası

Onay tamamlandıktan sonra, talep otomatik olarak Released durumuna gelir (INTCReleaseTravelRequest.PerformManualRelease() çağrılır). Ya da kullanıcı "Release" butonunu tıklayabilir.

## 7. Günlüğe Aktar

Seyahat Günlükleri sayfasına girilir. "Get Travel Requests" butonu tıklanır. Released taleplerinin satırları yüklenir. İstenen satırlar seçilir. Günlüğe aktarılır.

## 8. Günlüğü Yayımla

Günlük satırlarında InvoiceDate, InvoiceNo, ConfirmationNo, Amount, CurrencyCode, WhoBuys vb. fatura bilgileri doldurulur. "Post Travel Journals" butonu tıklanır. Defter Girdileri oluşturulur. Posted = true olur.

## 9. Fatura Oluştur (Opsiyonel)

"Create Purchase Invoice" butonu tıklanır. Satış Faturası oluşturulur. GeneralProdPostingGroup ve VatProdPostingGroup alanları satıcı ve ürün grubuna göre doldurulur.

## 10. İnceleme ve Muhasebeleştirme

Defter Girdileri sayfasında tüm kayıtlar incelenir. Dimensions kontrol edilir. Muhasebe işlemi tamamlanır.

## Temel Validasyonlar ve Kurallar

## Tarih Validasyonları

- Bilet ve Konaklama'da tüm tarihler DocumentDate'den eski olamaz
- ReturnDate > DepartureDate olmalıdır
- DropOffDate > PickUpDate olmalıdır
- Uygulamada eski tarihe girilmişse uyarı/hata mesajı gösterilir

## Durum Validasyonları

- Open durumdaki talep düzenlenebilir
- PendingApproval durumdaki talep düzenlenemez
- Released durumdaki talep alt bileşenleri değiştiremez, sadece bilgi saklanabilir
- Posted = true olan günlük satırı silinebilir, değiştirilemez

## İş Mantığı Validasyonları

- Sürüm alınmadan günlüğe aktarılamaz
- Günlüğe aktarılmayan satırlar defter girdisine yazılamaz
- Fatura oluşturmadan önce InvoiceNo veya ConfirmationNo zorunludur
- Partner Company seçilmesi VendorNo'yu otomatik doldurur

## Döviz Dönüşümü

Bütün para birimleri dönüştürülür. Örneğin EUR ile 100 € girilmişse:
- AmountLCY = 100 * ExchangeRate(DocumentDate, "EUR")
- CostPerDayLCY = AmountLCY / Days (eğer Days > 0 ise)

## En İyi Uygulamalar

## Talep Hazırlanırken

- Tüm tarihler önceden belirleyin
- Konaklama şehri, bilet rotası vb. açık olmalıdır
- Maliyet kategorisini doğru seçin (Raporlama için önemli)
- Ek'ler'e rezervasyon belgesi ekleyin

## Onay Süreci

- Zaman kaybı yaşamak için erken onay isteyin
- Onaylayan kişinin Approval Setup'ında tanımlanmış olduğundan emin olun
- Delegasyon varsa, yeni onaylayıcıya kontrol verin

## Günlük İşlemleri

- Fatura numarası ve onay numarasını doğru girin
- Para birimini seçmeyi unutmayın
- Satıcı seçilmesi otomatik olur, kontrol edin
- Eklerden fatura örneğini yükleyin

## Sorun Giderme

## "Talep sürümü alınamıyor"

Çözüm: Onay süreci tamamlanmış mı kontrol edin. Status = PendingApproval durumdaysa onay yapılıncaya kadar bekleyin.

## "Tarih hatasıdır aldığı tarih"

Çözüm: Bilet/Konaklama tarihini DocumentDate'den önce giremezsiniz. DocumentDate'i doğru seçin.

## "Günlük oluşturulamıyor"

Çözüm: Talep Released durumunda mı kontrol edin. Open veya PendingApproval durumdaysa sürüm almalısınız.

## "Fatura oluşturulamıyor"

Çözüm: InvoiceNo veya ConfirmationNo zorunludur. Satırı seçtiğinizde bu alanların doldurulup doldurulmadığını kontrol edin.

## "Partner Company seçilemiyor"

Çözüm: Partner Company LineType'ı seçili satırın LineType'ı ile aynı olmalıdır. Örneğin Ticket satırında Accommodation Partner Company seçemezsiniz.

## Sonuç

Seyahat Talep Sistemi, şirketin seyahat giderlerini merkezi olarak yönetmek, kontrol etmek ve muhasebe entegrasyonu sağlamak için tasarlanmıştır. Doğru şekilde kullanıldığında, operasyonel verimliliği artırır, gider kontrolünü sağlar ve raporlama altyapısını kuvvetlendirir.

Sistem içinde:
- Talep oluşturma, değişiklik ve silme basitçe yapılır
- Onay süreci workflow'undan yönetilir
- Muhasebe boyutlandırması detaylı yapılır
- Döviz dönüşümü otomatik yapılır
- Raporlama ve analiz veri tabanında tutulur

Herhangi bir sorunla karşılaşırsanız, sistem yöneticisine başvurunuz.