# Tarsau Arşivleme Aracı

`tarsau`, Linux/Unix işletim sistemleri için C diliyle yazılmış, kararlı ve hafif bir komut satırı arşivleme aracıdır. Herhangi bir sıkıştırma uygulamadan birden fazla ASCII metin dosyasını tek bir `.sau` arşiv dosyasında birleştirmeyi ve bu dosyaları orijinal UNIX izinlerini (okuma, yazma, çalıştırma) tam olarak koruyarak arşivden çıkarmayı sağlar.

## Özellikler
- **Katı Format Denetimi**: Giriş dosyalarının tamamının geçerli ASCII metin dosyaları (karakter başına 1 bayt, karakter değeri 0–127 aralığında) olduğunu doğrular. Uyumsuz bir dosya tespit edildiğinde, ekrana tam olarak `<dosya_adı> giriş dosyasının formatı uyumsuzdur!` yazdırarak güvenli bir şekilde sonlanır.
- **Kapasite Sınırları**: En fazla 32 giriş dosyası ve toplamda 200 MB kümülatif dosya boyutu limitlerini katı bir şekilde uygular.
- **UNIX İzinlerinin Korunması**: POSIX stat ve chmod API'lerini kullanarak, arşivden çıkarılan dosyaların izin seviyelerinin arşivlenmeden önceki halleriyle birebir aynı olmasını garanti eder.
- **Güvenlik ve Bütünlük**: Kararlı bir yerleşim düzeni (layout) bütünlüğü kontrolü ve metadata doğrulaması gerçekleştirir. Eğer `.sau` dosyası kırpılmış, değiştirilmiş veya hatalı formatlanmışsa, ekrana `Arşiv dosyası uygunsuz veya bozuk!` yazdırarak temiz bir şekilde sonlanır.

---

## Derleme Adımları
Programı bir Linux/Unix sisteminde derlemek için bu dizine gidin ve aşağıdaki komutu çalıştırın:

```bash
make
```

Bu komut, `tarsau.c` dosyasını aşağıdaki GCC bayraklarını (flags) kullanarak `tarsau` adında bir çalıştırılabilir (executable) dosyaya derler:
`-Wall -Wextra -O2 -std=c99`

Derlenen ikili (binary) dosyayı temizlemek için şu komutu çalıştırın:
```bash
make clean
```

---

## Komut Satırı Parametreleri ve Kullanım

### 1. Paketleme Modu (`-b`)
Birden fazla dosyayı tek bir `.sau` arşiv dosyasında birleştirir.

**Kullanım:**
```bash
./tarsau -b <input_files...> -o <output_archive.sau>
```

- `-b`: Paketleme modunu etkinleştirir. Bu parametreden sonra gelen tüm argümanlar (`-o` ve değeri hariç) giriş dosyası olarak kabul edilir.
- `-o`: Çıktı arşiv dosyasının adını belirtir (isteğe bağlıdır; belirtilmezse varsayılan olarak `a.sau` adı kullanılır).

**Örnek:**
```bash
./tarsau -b t1.txt t2.txt sub/t3.dat -o my_archive.sau
```

---

### 2. Arşivden Çıkarma Modu (`-a`)
Bir `.sau` arşivine paketlenmiş dosyaları dışarı çıkarır.

**Kullanım:**
```bash
./tarsau -a <archive_name.sau> [<target_directory>]
```

- `-a`: Arşivden çıkarma modunu etkinleştirir.
- `<archive_name.sau>`: Açılacak `.sau` arşiv dosyasının yolu.
- `<target_directory>`: Dosyaların çıkarılacağı hedef dizin (isteğe bağlıdır). Eğer belirtilen dizin mevcut değilse, otomatik olarak (iç içe ise rekürsif şekilde) oluşturulur. Belirtilmezse, dosyalar doğrudan geçerli çalışma dizinine çıkarılır.

**Örnek:**
```bash
./tarsau -a my_archive.sau output_dir/
```

---

## Teknik Mimari ve Dosya Yerleşimi
Bir `.sau` arşiv dosyasının yapısı iki ana bölümden oluşur:

1. **Organizasyon Bölümü (Metadata)**:
   - **İlk 10 Bayt**: Organizasyon bölümünün tamamının (bu 10 bayt dahil) desimal (onluk tabanda) boyutunu içeren ve sol tarafı sıfırlarla doldurulmuş (örneğin `0000000084`) ASCII sayısal değeri.
   - **Dosya Kayıtları**: Sıralı kayıtlar halinde temsil edilir ve her bir kayıt tam olarak `|dosya_adi,izinler,boyut|` (örneğin `|t1.txt,0644,12|t2.txt,0755,34|`) formatındadır.

2. **Veri Bölümü**:
   - Doğrudan organizasyon bölümünün bitiminden sonra başlar.
   - Arşivlenen dosyaların ham içeriklerini, aralarında herhangi bir ayırıcı karakter olmaksızın peş peşe birleştirilmiş şekilde barındırır. Her bir dosyanın kesin boyutu metadata bölümünde tanımlı olduğundan, arşiv açma işlemi sırasında veriler ilgili bayt sınırlarından okunarak ayrıştırılır.