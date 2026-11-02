<!-- pushizo:header -->
<p align="center">
  <a href="https://pushizo.com"><img src="https://raw.githubusercontent.com/pushizo/.github/main/profile/assets/mark.svg" alt="Pushizo" width="56" height="56"></a>
</p>
<p align="center">
  <a href="https://docs.pushizo.com/docs/integrations/ai-prompts"><img src="https://img.shields.io/badge/docs-docs.pushizo.com-5b4bdb" alt="Docs"></a>
  <a href="https://docs.pushizo.com/docs/api"><img src="https://img.shields.io/badge/API-reference-b44fc4" alt="API reference"></a>
  <a href="https://github.com/pushizo/pushizo-ai-skills/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-16130f" alt="License: MIT"></a>
</p>
<p align="center"><a href="#english">English</a> · <a href="#türkçe">Türkçe</a></p>

## English

**Pushizo AI skills** — Agent skills (SKILL.md) and Cursor rules that teach AI coding agents to integrate Pushizo correctly: install and initialize the client SDKs, call the REST API with the server SDKs, verify webhook signatures and keep API keys out of source code. Works with Claude Code, Cursor and any agent that reads Markdown instructions.

```bash
# Claude Code (all skills, current project)
git clone --depth 1 https://github.com/pushizo/pushizo-ai-skills /tmp/pushizo-ai-skills
mkdir -p .claude/skills && cp -R /tmp/pushizo-ai-skills/skills/* .claude/skills/

# Cursor rules
mkdir -p .cursor/rules && cp /tmp/pushizo-ai-skills/cursor/rules/*.mdc .cursor/rules/
```

The detailed guide below is in Turkish; code samples are language-neutral. Full documentation:
[docs.pushizo.com/docs/integrations/ai-prompts](https://docs.pushizo.com/docs/integrations/ai-prompts) · API reference: [docs.pushizo.com/docs/api](https://docs.pushizo.com/docs/api).

## Türkçe
<!-- /pushizo:header -->

# Pushizo AI skills

Yapay zekâ kodlama ajanlarının (Claude Code, Cursor ve Markdown talimat okuyan diğer ajanlar) Pushizo'yu doğru
entegre etmesi için skill, kural ve hazır istemler. Skill dosyaları ajanların en tutarlı çalıştığı dil olan
İngilizcedir; ajan kullanıcıya kendi dilinde yanıt verir. Hazır istemler iki dillidir.

## İçerik

| Yol | Ne işe yarar |
|-----|--------------|
| `skills/pushizo-sdk-setup/SKILL.md` | İstemci SDK kurulumu: web, iOS, Android, Flutter, React Native/Expo, Capacitor, Cordova, MAUI, Unity, React/Vue/Angular; izin isteme ve `login` |
| `skills/pushizo-server-api/SKILL.md` | Sunucu SDK'ları (Node, Python, PHP, Go, Java, .NET, Ruby, Rust) ve REST API: gönderim, kullanıcılar, segment/şablon/kampanya, hatalar, yeniden deneme, idempotency, sayfalama, kimlik doğrulama |
| `skills/pushizo-webhooks/SKILL.md` | Webhook alıcısı: ham gövdede `Pushizo-Signature` doğrulaması, batch, tekrar ayıklama, secret rotasyonu |
| `skills/pushizo-drafts/SKILL.md` | Gönderim yapmadan taslak hazırlama: push kanalı, e-posta şablonu ve taslak kampanya, segment |
| `cursor/rules/pushizo.mdc` | Cursor proje kuralı (aynı kuralların kısa hâli) |
| `prompts/*.md` | Kopyala-yapıştır istemler (İngilizce + Türkçe): push kanalı kurulumu, e-posta kampanya taslağı, segment taslağı |

Tüm içerik aynı güvenlik kurallarını uygular: API anahtarı, SDK secret'ı, webhook secret'ı ve push kimlik bilgileri
koda, loga veya sohbete yazılmaz; istemci uygulamaya yalnızca App ID verilir; ajan kullanıcı açıkça istemeden gerçek
gönderim, zamanlama veya silme çağrısı yapmaz.

## Kurulum

**Claude Code** (proje düzeyi; tüm projeler için `~/.claude/skills/` kullanın):

```bash
git clone --depth 1 https://github.com/pushizo/pushizo-ai-skills /tmp/pushizo-ai-skills
mkdir -p .claude/skills
cp -R /tmp/pushizo-ai-skills/skills/* .claude/skills/
```

Claude Code skill'leri açıklamalarına göre gerektiğinde kendisi yükler; doğrudan çağırmak için `/pushizo-sdk-setup`
gibi adlarını kullanabilirsiniz.

**Cursor:**

```bash
mkdir -p .cursor/rules
cp /tmp/pushizo-ai-skills/cursor/rules/pushizo.mdc .cursor/rules/
```

**Diğer ajanlar:** ilgili `SKILL.md` dosyasını ajanın talimat dosyasına (ör. `AGENTS.md`) ekleyin veya istem olarak
verin.

**MCP:** ajanınız MCP destekliyorsa Pushizo verilerini okumak ve taslak oluşturmak için uzak MCP sunucusunu da
ekleyin: `claude mcp add --transport http pushizo https://api.pushizo.com/mcp`
([pushizo-mcp](https://github.com/pushizo/pushizo-mcp)).

## Sürümler

İçerik her Pushizo SDK sürümüyle birlikte güncellenir; sürüm etiketleri SDK sürümleriyle aynıdır (`v0.1.0` …).
Belgelerin tamamı: [docs.pushizo.com/docs](https://docs.pushizo.com/docs).

<!-- pushizo:footer -->
---

## Support · Destek

- **Guide · Rehber:** [docs.pushizo.com/docs/integrations/ai-prompts](https://docs.pushizo.com/docs/integrations/ai-prompts)
- **API reference & OpenAPI · API referansı ve OpenAPI:** [docs.pushizo.com/docs/api](https://docs.pushizo.com/docs/api) ·
  [pushizo-openapi](https://github.com/pushizo/pushizo-openapi)
- **Bugs & ideas · Hata ve öneri:** [Issues](https://github.com/pushizo/pushizo-ai-skills/issues)
- **Security · Güvenlik:** please do not open public issues; email [security@pushizo.com](mailto:security@pushizo.com) ·
  herkese açık issue açmayın, bu adrese yazın.
- **Help · Destek:** [support@pushizo.com](mailto:support@pushizo.com)

This repository is mirrored automatically from the Pushizo main repository on every release. Please open an issue
instead of a pull request; fixes land in the main repository and ship with the next release.
Bu depo her sürümde Pushizo ana deposundan otomatik yansıtılır. Pull request yerine issue açın; düzeltmeler ana depoda
yapılır ve bir sonraki sürümle buraya gelir.

Released under the MIT License · MIT lisanslıdır — [LICENSE](https://github.com/pushizo/pushizo-ai-skills/blob/main/LICENSE).
<!-- /pushizo:footer -->
