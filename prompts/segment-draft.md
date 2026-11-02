# Segment draft · Segment taslağı

## English

Use the Pushizo public API. Do not export the user list.

1. Express the rule as one of: tag, last activity, subscription channel, external id list.
2. Check it with `POST /v1/apps/{APP_ID}/segments/preview`. Summarize the count; do not paste raw user ids.
3. Save it with `POST /v1/apps/{APP_ID}/segments`, with an English and a Turkish name.

## Türkçe

Pushizo public API'sini kullan. Kullanıcı listesini dışarı aktarma.

1. Kuralı şu türlerden biriyle yaz: etiket, son etkinlik, abonelik kanalı, dış kimlik listesi.
2. `POST /v1/apps/{APP_ID}/segments/preview` ile dene. Sayıyı özetle; ham kullanıcı kimliklerini sohbete dökme.
3. `POST /v1/apps/{APP_ID}/segments` ile Türkçe ve İngilizce adla kaydet.
