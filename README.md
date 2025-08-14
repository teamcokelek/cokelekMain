# Çökelek Takımı - Teknofest 2025 Doğal Dil İşleme Projesi

Bu proje, **Çökelek Takımı** tarafından **Teknofest 2025 Doğal Dil İşleme Yarışması Serbest Kategorisi** için geliştirilmiştir. Projenin temel amacı, kullanıcıların sorduğu karmaşık, çok adımlı ve örtük anlamlar içeren film sorgularını anlayıp, birden fazla aracı (tool) akıllıca kullanarak doğru ve akıcı cevaplar üreten bir yapay zeka ajanı (agent) oluşturmaktır.

##  Proje Hakkında

Günümüzdeki sohbet botları genellikle basit ve tekil sorulara cevap verebilmektedir. Ancak bir kullanıcı, "Bir grup adamın bir odada tartıştığı o eski, siyah beyaz filmin süresi ne kadardı?" gibi bir soru sorduğunda, sistemin önce filmi tahmin etmesi (`vector_search`), ardından o filmin bilgisini kullanarak süresini veritabanından sorgulaması (`generate_sql`) gerekir.

Bu proje, LangGraph çatısını kullanarak bu tür **çok adımlı (multi-hop)** görevleri yerine getirebilen bir agent mimarisi sunmaktadır. Agent, kendi eğittiğimiz ve Mistral v0.3 temel alınarak geliştirilmiş bir dil modelini kullanır.
## Model Dosyası: https://huggingface.co/mmmmmmabel/cokelek-v3
## Train Datasının bir kısmı https://huggingface.co/datasets/mmmmmmabel/cokelek-train-data-v1
## Vector store için özet verilerinin bir kısmı: https://drive.google.com/file/d/1kYUhdyHO7N-2NG70Cw2l_MSVqCJtzgnr/view?usp=sharing

# Vector store oluşturma kodu vdbolusturma.ipynb içerisindedir.

## ✨ Temel Özellikler

  * **Multi-Hop Akıl Yürütme:** Bir aracın çıktısını, bir sonraki adımın girdisi olarak kullanarak karmaşık problemleri adımlara böler ve çözer.
  * **Hibrit Arama:**
      * **Vektör Arama:** "Bir palyaçonun kötü olduğu film" gibi anlamsal ve betimsel sorgular için kullanılır.
      * **SQL Arama:** "Tom Hanks'in 2000'den sonra çıkan filmleri" gibi spesifik ve yapısal sorgular için kullanılır.
  * **Özelleştirilmiş Dil Modeli:** Yönlendirme (Routing), SQL Üretme (Text-to-SQL) ve Cevap Sentezleme (Response Generation) görevleri için Mistral v0.3 üzerinde kendi topladığımız veri seti ile **fine-tuning** yapılmıştır.
  * **REST API:** Google Colab üzerinde çalışarak, `ngrok` aracılığıyla bir web arayüzüne bağlanabilen bir REST API sunar.

## ⚙️ Mimari ve Çalışma Prensibi

Proje, LangGraph kullanılarak oluşturulmuş döngüsel bir graf (graph) mimarisine dayanır. Temel akış şu şekildedir:

1.  **Kullanıcı Sorgusu:** Sisteme doğal dilde bir soru gelir.
2.  **Yönlendirici (Router):** Eğitilmiş model, soruyu ve mevcut durumu analiz ederek bir sonraki adıma karar verir: `vector_search`, `generate_sql` veya `respond`.
3.  **Araç Kullanımı (Tool Execution):** Seçilen araç çalıştırılır.
      * `vector_search_tool`: Faiss kullanarak anlamsal arama yapar.
      * `sql_search_tool`: SQLite veritabanında sorgu çalıştırır.
4.  **Durum Güncelleme:** Aracın sonucu, bir sonraki karar için tekrar Yönlendirici'ye gönderilir.
5.  **Yanıt Sentezleyici (Responder):** Yönlendirici `respond` kararı verdiğinde, toplanan tüm bilgiler bu düğüme gönderilir ve model, kullanıcıya sunulacak nihai, akıcı cevabı oluşturur.

## 🧠 Kullanılan Model

  * **Temel Model:** `unsloth/mistral-7b-v0.3`
  * **Eğitim:** Model, yukarıda bahsedilen üç temel görev (Yönlendirme, SQL Üretme, Cevap Sentezleme) için özenle hazırlanmış, senaryo bazlı JSONL veri setleri ile eğitilmiştir. Eğitim kodları ve veri örnekleri de repoda mevcuttur.

## 🏁 Kurulum ve Çalıştırma

Bu proje, bir backend API görevi görmektedir ve en kolay Google Colab üzerinde çalıştırılacak şekilde tasarlanmıştır.

### Ön Gereksinimler

  * Google Colab Hesabı
  * **Ngrok Hesabı ve Authtoken:** Sistemin bir web API sunabilmesi için `ngrok` servisi kullanılmaktadır. [Ngrok Dashboard](https://dashboard.ngrok.com/get-started/your-authtoken) adresinden ücretsiz bir hesap oluşturup `authtoken`'ınızı almanız gerekmektedir.

### Adım Adım Çalıştırma

1.  **Projeyi Klonlama:**

    ```bash
    git clone https://github.com/teamcokelek/cokelekMain
    ```

2.  **Colab'e Yükleme:** Proje dosyalarını Google Drive'ınıza yükleyin ve ana `*.ipynb` dosyasını Colab ile açın.

3.  **Ngrok Token'ı Yapılandırma:**

      * Google Colab'de sol taraftaki menüden **"Sırlar" (anahtar ikonu 🔑)** sekmesine tıklayın.
      * `+ Yeni sır ekle` butonuna basın.
      * **Ad (Name):** `NGROK_AUTHTOKEN`
      * **Değer (Value):** Ngrok dashboard'ından kopyaladığınız `authtoken`'ınızı buraya yapıştırın. (Örnek: `2aBcDeFgHiJkLmNoPqRsTuVwXyZ...`)
      * "Notebook erişimi" seçeneğini aktif hale getirin.

4.  **Notebook'u Çalıştırma:**

      * Colab notebook'undaki hücreleri yukarıdan aşağıya doğru sırasıyla çalıştırın.
      * Gerekli kütüphaneler yüklenecek, model indirilecek ve agent başlatılacaktır.
      * Her şey yolunda giderse, en son hücrelerden biri size `https.xxxxxxxx.ngrok.io` formatında bir URL çıktısı verecektir. Bu URL, web arayüzünün bağlanacağı API adresidir.

## 🌐 Web Arayüzü

Bu projenin kullanıcı arayüzü ayrı bir repoda geliştirilmiştir. Backend'i yukarıdaki adımlarla çalıştırdıktan sonra, web arayüzünü de çalıştırarak agent ile sohbet edebilirsiniz.

  * **Web Arayüzü Reposu:** **[teamcokelek/cokelekweb](https://github.com/teamcokelek/cokelekweb)**

Web projesini çalıştırırken, bu backend projesinin oluşturduğu `ngrok` URL'ini, web projesindeki API adresi olarak yapılandırmanız gerekmektedir.

## 👥 Takım

  * **Çökelek Takımı**

## 📄 Lisans

Bu proje MIT Lisansı ile lisanslanmıştır. Detaylar için `LICENSE` dosyasına bakınız.

-----
