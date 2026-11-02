# Email campaign draft · E-posta kampanya taslağı

## English

Use the Pushizo public API. Do not call send or schedule endpoints.

1. Write the subject, preheader and body in English and Turkish.
2. Save the email template as a draft with `POST /v1/apps/{APP_ID}/templates`; preview it with
   `POST /v1/apps/{APP_ID}/templates/{templateId}/render`.
3. Save the campaign as a draft with `POST /v1/apps/{APP_ID}/campaigns`. Do not set a send time.
4. Report `GET /v1/apps/{APP_ID}/campaigns/{campaignId}/email-risk`.

Do not call `POST …/send` or `POST …/schedule` unless I explicitly say "send".

## Türkçe

Pushizo public API'sini kullan. Gönderim ve zamanlama uçlarını çağırma.

1. Konu, ön başlık ve gövdeyi Türkçe ve İngilizce yaz.
2. `POST /v1/apps/{APP_ID}/templates` ile e-posta şablonunu taslak kaydet;
   `POST /v1/apps/{APP_ID}/templates/{templateId}/render` ile önizle.
3. `POST /v1/apps/{APP_ID}/campaigns` ile kampanyayı taslak kaydet. Gönderim zamanı yazma.
4. `GET /v1/apps/{APP_ID}/campaigns/{campaignId}/email-risk` sonucunu bildir.

Ben açıkça "gönder" demeden `POST …/send` veya `POST …/schedule` çağırma.
