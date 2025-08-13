Cityscapes Color → YOLO-Seg Converter
Bu repo, Cityscapes isimlendirmesine uyan dosya çiftlerinden (aynı klasördeki X_gtFine_color.png + X_leftImg8bit.png) YOLO segmentation (YOLO-Seg) biçiminde poligon etiketleri üretir ve görüntü/etiket yapısını standart YOLO dizin düzeninde (images/ & labels/) hazırlar.

🔧 Ne yapar?
*_gtFine_color.png maskelerindeki renkleri sınıflara map’leyerek her sınıf için dış konturları çıkarır.

Her kontur için poligon noktalarını [0,1] aralığında normalize ederek YOLO-Seg satırları yazar.

Aynı ada sahip görüntüyü (*_leftImg8bit.png) images/ altına kopyalar (istersen taşıyabilir).

Çıktı olarak:

markdown
Kopyala
Düzenle
yolo_out/
  images/
    aachen_000000_000019_leftImg8bit.png
  labels/
    aachen_000000_000019_leftImg8bit.txt
  dataset.yaml
  color_map.json
üretir.

🧠 Neden “color” maskeden?
Cityscapes’te *_gtFine_color.png dosyaları renk kodlu semantik sınıfları içerir (ör. yol = 128,64,128). Bu betik, bu renkleri doğrudan trainId (19 sınıf) eşlemesine map’leyerek YOLO-Seg poligonları çıkarır.

Gerçek instance ayrımı istiyorsan, *_instanceIds.png ile farklı bir akış gerekir. Bu betik semantik maskeyi bağlı bileşenlere bölerek “instance benzeri” poligonlar üretir.

✨ Özellikler
Cityscapes’in 19 sınıflık paleti hazır (trainId).

Kontur sadeleştirme (Douglas–Peucker), min alan filtresi, anti-alias yuvarlama seçenekleri.

Görüntüyü kopyalama/taşıma tercihi.

İstersen Cityscapes paleti kapatıp maskeden renkleri otomatik keşfetme (custom veri için).

📦 Gereksinimler
Python 3.8+

OpenCV, NumPy

pip install opencv-python numpy
📁 Dosya yapısı (girdi)
Aynı klasörde bu tip çiftler olmalı (örnek):

python-repl

aachen_000000_000019_gtFine_color.png
aachen_000000_000019_leftImg8bit.png
...
Betik, klasördeki tüm *_gtFine_color.png dosyalarını tarar ve karşılık gelen *_leftImg8bit.png dosyasını eşler.

▶️ Kullanım
Betiği repo köküne kaydet: cityscapes_color_to_yoloseg_with_images.py

Ayarları düzenle:

ROOT → çiftlerin bulunduğu klasör

OUT_DIR → çıktı klasörü

COPY_IMAGES / MOVE_IMAGES → görüntüyü kopyala/taşı

Gerekirse: MIN_AREA, APPROX_EPS, ROUND_TO

Çalıştır:

bash
python cityscapes_color_to_yoloseg_with_images.py
Önemli ayarlar
python

ROOT      = Path(r"...\train\aachen")  # giriş klasörü
OUT_DIR   = ROOT / "yolo_out"          # çıktı kökü

WRITE_BBOX_TOO = False   # True yaparsan aynı konturlardan bbox .txt de yazar
MIN_AREA = 50            # piksel alanı küçük olan parçaları at
APPROX_EPS = 0.003       # kontur sadeleştirme oranı (0.001–0.01 arası deneyebilirsin)
ROUND_TO = 0             # 0 kapalı; 8 yaparsan maskede anti-alias renkleri yuvarlar
USE_CITYSCAPES_PALETTE = True  # Cityscapes 19 sınıf paletini kullan
COPY_IMAGES = True       # görüntüyü images/ altına kopyala
MOVE_IMAGES = False      # görüntüyü kopyalamak yerine TAŞI (riskli; orijinali taşır)
🧾 Çıktı formatı
YOLO-Seg etiket satırı:

php-template
<class_id> x1 y1 x2 y2 x3 y3 ... xN yN
Her satır tek bir poligonu temsil eder.

x, y koordinatları görüntü boyutuna göre normalize edilir (0–1).

Aynı görüntüde bir sınıfa ait birden çok poligon olabilir (çoklu satır).

dataset.yaml:
Cityscapes paleti açıkken 19 sınıf ismi otomatik yazılır:

yaml
names:
  - road
  - sidewalk
  - building
  - wall
  - fence
  - pole
  - traffic light
  - traffic sign
  - vegetation
  - terrain
  - sky
  - person
  - rider
  - car
  - truck
  - bus
  - train
  - motorcycle
  - bicycle
color_map.json:
Kullanılan renk → sınıf id eşleşmesini hex biçiminde saklar (debug için faydalı).

🎨 Cityscapes renk → trainId eşlemesi
Betiğe gömülü palet:

RGB	trainId	name
(128, 64,128)	0	road
(244, 35,232)	1	sidewalk
(70, 70, 70)	2	building
(102,102,156)	3	wall
(190,153,153)	4	fence
(153,153,153)	5	pole
(250,170,30)	6	traffic light
(220,220,0)	7	traffic sign
(107,142,35)	8	vegetation
(152,251,152)	9	terrain
(70,130,180)	10	sky
(220,20,60)	11	person
(255,0,0)	12	rider
(0,0,142)	13	car
(0,0,70)	14	truck
(0,60,100)	15	bus
(0,80,100)	16	train
(0,0,230)	17	motorcycle
(119,11,32)	18	bicycle

Not: Cityscapes’te bazı sınıflar “void / ignore” olabilir. Betik, IGNORE_COLORS = {(0,0,0)} ile siyahı yok sayıyor; gerekiyorsa genişletebilirsin.

🛠️ İpuçları
Çok detaylı poligonlar varsa APPROX_EPS değerini biraz artır (örn. 0.005) → daha az noktalı, daha hızlı.

Gürültülü/alias’lı maskeler için ROUND_TO = 8 gibi bir değer, renk varyasyonlarını tek renge “yaklaştırır”.

Çok küçük parçaları temizlemek için MIN_AREA’yı yükselt (örn. MIN_AREA = 150).

Sadece bbox istiyorsan WRITE_BBOX_TOO = True yap; ayrıca poligon yerine YOLO detection formatı da üretir (ayrı dosya).

🧪 Doğrulama
Üretilen labels/*.txt dosyalarını bir YOLO çerçevesinde (Ultralytics/YOLOv5–11 vb.) görselleştirerek poligonların doğru oturduğunu kontrol edebilirsin.

Poligon sayıları beklediğinden az/çok ise: IGNORE_COLORS, MIN_AREA, APPROX_EPS, ROUND_TO ayarlarıyla oyna.

⚠️ Sınırlamalar
Bu akış semantik maskeden bağlı bileşenleri “instance gibi” ele alır. Gerçek instance ayrımı için Cityscapes’in *_instanceIds.png dosyalarına dayalı ayrı bir mantık gerekir.

Çok büyük görüntülerde kontur çıkarma yavaş olabilir; alt örnekleme ya da çoklu işlem (multiprocessing) ekleyebilirsin.

▶️ Betik
Betiğin adı: cityscapes_color_to_yoloseg_with_images.py
(Repoda bu dosyayı ve README’yi birlikte paylaş.)

