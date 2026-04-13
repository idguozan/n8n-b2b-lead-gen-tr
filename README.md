
# 🚀 Yapay Zeka Destekli B2B Müşteri Bulma Otomasyonu (n8n)

![n8n](https://img.shields.io/badge/n8n-FF6600?style=for-the-badge&logo=n8n&logoColor=white)
![LangChain](https://img.shields.io/badge/LangChain-1C3C3C?style=for-the-badge&logo=langchain&logoColor=white)
![Gemini](https://img.shields.io/badge/Google_Gemini-8E75B2?style=for-the-badge&logo=googlebard&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-34A853?style=for-the-badge&logo=googlesheets&logoColor=white)

## 📌 Genel Bakış
Bu proje, **n8n** ve **LangChain** (Google Gemini) kullanılarak geliştirilmiş, uçtan uca tam otomatik bir B2B (Kurumdan Kuruma) potansiyel müşteri (lead) bulma ve niteliklendirme sistemidir. 

Sistem, sadece satılmak istenen ürünün web adresini (URL) alır; ürünü analiz eder, doğru hedef kitleyi belirler, Google/LinkedIn üzerinden hedefe yönelik spesifik aramalar yapar, rakipleri/alakasız firmaları eler ve geriye kalan yüksek kaliteli müşteri adaylarını Google Sheets'e kaydeder.

## ⚙️ Sistem Mimarisi (5 Aşamalı İş Akışı)

Sistem, hatalara karşı dayanıklı (fail-safe) 5 ana fazdan oluşan bir mimariye sahiptir:

1. 🕷️ **Veri Toplama ve Temizlik (Web Scraping & Cleaning):** * Verilen ürün URL'sine giderek sitenin HTML kodunu indirir.
   * Özel JavaScript `Code` düğümü ile gereksiz etiketleri (script, style, nav, footer) ve gürültüyü temizleyerek sadece saf "ürün metnini" çıkarır.

2. 🧠 **Yapay Zeka Hedef Kitle Analizi (AI-1 Product Analyzer):**
   * Temizlenen metin LLM'e (Gemini) beslenir.
   * Yapay zeka, yuvarlak kelimeler kullanmadan (örn: "kurumsal firmalar" yasaktır) ürünü satın alma potansiyeli en yüksek 6 spesifik niş sektörü ve bu sektörlerin anahtar kelimelerini JSON formatında üretir.
   * *Kalite Kapısı:* JSON formatı bozuksa veya eksik veri varsa `try-catch` bloklarıyla sistem durdurulur ve hatalı veri engellenir.

3. 🔍 **Arama Sorgusu Üretimi (AI-2 Query Builder):**
   * İkinci LLM, ilk AI'dan gelen sektörleri kullanarak LinkedIn için özel arama dorkları (Query) oluşturur.
   * *Örnek Sorgu:* `site:linkedin.com/company ("Özel Okul" OR "Kolej") Türkiye -yazılım -danışmanlık`

4. 🌐 **Google Üzerinden Tarama (Search Execution):**
   * Üretilen sorgular **SearchAPI** üzerinden Google'a gönderilir ve eşleşen LinkedIn şirket profilleri çekilir.
   * API bağlantı kopmalarına karşı otomatik tekrar deneme (`Retry on Fail`) mekanizması aktiftir.

5. 🛡️ **Kalite Filtresi ve Dışa Aktarım (Lead Quality Gate & Export):**
   * Sistemin en kritik aşamasıdır. Gelen yüzlerce veri JavaScript filtrelerinden geçer.
   * Rakipler, yazılım şirketleri, ajanslar ve arama niyetine tam uymayan tüm şirketler acımadan elenir.
   * Geriye kalan %100 nitelikli potansiyel müşteriler, satış ekibinin önüne Google Sheets satırları olarak eklenir.

## 🚀 Öne Çıkan Özellikler
* **Sıfır Spam Felsefesi:** Gelişmiş negatif kelime filtreleri (Relevance Score) sayesinde satış ekibinin vaktini çalacak alakasız firmaları listeye almaz.
* **Zincirlenmiş LLM Ajanları:** Görevi tek bir modele yığıp hata payını artırmak yerine; işi "Analiz" ve "Arama Üretimi" olarak iki ayrı AI ajanına (LangChain) böler, yapay zeka halüsinasyon riskini sıfıra indirir.
* **Katı JSON Doğrulaması:** Yapay zekanın ürettiği tüm çıktılar katı JavaScript kurallarıyla parse edilir. Modele hiçbir inisiyatif bırakılmaz.

## 🛠️ Gereksinimler ve Kurulum

Bu otomasyonu kendi n8n sunucunuzda çalıştırmak için aşağıdaki bağlantılara ihtiyacınız vardır:

1. Çalışır durumda bir [n8n](https://n8n.io/) hesabı (Self-hosted veya Cloud)
2. **Google Gemini API Anahtarı** (LangChain düğümleri için)
3. **SearchAPI Hesabı** (Google aramalarını tetiklemek için)
4. **Google Sheets Bağlantısı** (Verileri yazdırmak için)

### Kurulum Adımları
1. Bu repository'deki `n8n-b2b-lead-gen-tr.json` dosyasını bilgisayarınıza indirin.
2. n8n arayüzünde yeni, boş bir workflow açın.
3. Sağ üst menüden **Import from File** (veya panodan yapıştır) diyerek indirdiğiniz JSON dosyasını içeri aktarın.
4. Sistem ekrana yüklendiğinde;
   * `Google Gemini` düğümlerine kendi API anahtarınızı bağlayın.
   * `SearchAPI Google` düğümünün URL parametrelerindeki API anahtarını kendinize göre güncelleyin.
   * `Write Leads` düğümüne kendi Google Sheets tablonuzu bağlayın.
5. İlk baştaki **Input URL** düğümüne test etmek istediğiniz ürünün linkini girin ve *Execute Workflow* butonuna basın!

## ⚠️ Uyarı ve Sorumluluk Reddi
Bu proje, B2B otomasyon mimarisi sergileme ve eğitim amaçlı tasarlanmıştır. Sistemi çok yüksek frekanslı aralıklarla (örneğin her dakika) çalıştırmak SearchAPI kotanızı hızlıca tüketebilir. Sistem canlıya alınırken saatlik veya günlük tetikleyiciler (`Schedule Trigger`) kullanılması şiddetle tavsiye ederim.