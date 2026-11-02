# Push channel setup · Push kanalı kurulumu

## English

Use the Pushizo public API. The API key comes from the `PUSHIZO_API_KEY` environment variable; do not write it to a
file or repeat it in the chat.

1. Read `GET /v1/apps/{APP_ID}/channels` and check `ios_push` and `android_push`.
2. If a channel is missing, ask me for the credential file path (APNs `.p8` with Key ID, Team ID and bundle id, or
   the Firebase service account JSON) and save it with `PUT /v1/apps/{APP_ID}/channels/{channel}`. Do not echo the
   credential.
3. Run `POST /v1/apps/{APP_ID}/channels/{channel}/validate` and report the result.
4. Propose one test notification to test devices only (`target.test_devices_only: true`). Send it only after I
   confirm. Title example: "Your order is on its way"; keep the body under 120 characters; give English and Turkish
   copy separately.

## Türkçe

Pushizo public API'sini kullan. API anahtarı `PUSHIZO_API_KEY` ortam değişkeninden gelsin; dosyaya yazma, sohbette
tekrarlama.

1. `GET /v1/apps/{APP_ID}/channels` ile `ios_push` ve `android_push` kanallarını kontrol et.
2. Kanal yoksa kimlik bilgisi dosyasının yolunu iste (APNs `.p8` ile Key ID, Team ID ve bundle id ya da Firebase
   servis hesabı JSON'u) ve `PUT /v1/apps/{APP_ID}/channels/{channel}` ile kaydet. Kimlik bilgisini sohbete yazma.
3. `POST /v1/apps/{APP_ID}/channels/{channel}/validate` çalıştır ve sonucu bildir.
4. Yalnızca test cihazlarına (`target.test_devices_only: true`) tek bir test bildirimi öner. Ben onaylamadan
   gönderme. Başlık örneği: "Siparişin yola çıktı"; gövde 120 karakteri geçmesin; Türkçe ve İngilizce metni ayrı ver.
