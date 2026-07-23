# BizCard

Dijital bir kartvizit projesi. Şu an için tek sayfalık, sade bir HTML kartvizit —
isim, unvan ve iletişim bilgilerini gösterir. İleride React'e ve mobil uygulamaya
dönüşmesi planlanıyor.

Bu, yazılımcı olmayan iş insanları için hazırlanan bir kurs projesidir. Bu yüzden kod
bilinçli olarak basit tutuluyor: build aracı yok, npm yok, bundler yok.

## Dosyalar

- **`index.html`** — sade, düz HTML/CSS kartvizit.
- **`react.html`** — aynı kartvizitin React ile (CDN üzerinden React 18 + Babel
  standalone kullanarak, kurulum gerektirmeden) yeniden yazılmış hâli.

İkisini de tarayıcıda doğrudan açarak görüntüleyebilirsiniz.

## Kısıtlar

- Bu aşamada backend/veritabanı yok; tüm iletişim bilgileri örnek (demo) veridir.
- Görsel olarak sade tutulacak, gereksiz karmaşıklaşmayacak.

## Proje kuralları

Bileşen yapısı ve webhook veri sözleşmesi (JSON formatı) kuralları
[`.claude/skills/bizcard-conventions/SKILL.md`](.claude/skills/bizcard-conventions/SKILL.md)
dosyasında tanımlı.
