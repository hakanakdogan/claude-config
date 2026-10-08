# Lean Workflow — Kullanım Kılavuzu

Mevcut global Claude kurulumunun (spec → tasks → next-task + 5 agent) kod kalitesini koruyup
**token tüketimini ve süreyi ciddi oranda düşüren** sadeleştirilmiş sürümü.
ANEW (ai-native-engineering-workspace) reposundan alınan iyi fikirler: tek doğrulama komutu,
dosyada yaşayan bağlam, reproduksiyon-önce bug fix, kısa context dosyası, deny listesi.

---

## 1. Token nereye gidiyordu? (teşhis)

| # | Sorun | Etkisi | Lean çözüm |
|---|---|---|---|
| 1 | `dotnet-claude-kit` plugin'i **global** açık: ~50 skill açıklaması + 13 agent + ~20 Roslyn MCP tool şeması | Her oturumun **ve her subagent çağrısının** başında, sen tek kelime yazmadan binlerce token | Global kapalı, sadece .NET projelerinde `.claude/settings.json` ile açık |
| 2 | 3 ayrı Opus aşaması: `/spec` (opus), `/tasks` (opus), `advisor` (opus, effort high) | En pahalı model 3 kez aynı dosyaları okuyor | Tek `/feat` komutu (opus, 1 kez). Advisor kaldırıldı → "önerilen AC" olarak aynı mesaja gömüldü |
| 3 | Spec'te soruların **tek tek** sorulması | Her soru = tüm bağlamın yeniden gönderildiği yeni tur (8 soru ≈ 8 tam tur) | Tüm sorular **tek mesajda**, her birinin önerilen cevabıyla |
| 4 | Testler iki kez: coder verify + reviewer her checkpoint'te tekrar verify | Build/test süresi ve log tokenı ikiye katlanıyor | Reviewer test **koşmaz**; coder'ın yeşil verify'ına güvenir |
| 5 | Her 2-4 task'ta bir checkpoint review + `/clear` + tekrar `/next-task` | 6 task'lık feature'da 2-3 review + insan beklemesi | Feature sonunda **tek** review; `/build` tüm task'ları tek seferde koşar |
| 6 | Build/test logları filtresiz bağlama giriyor | `dotnet test` çıktısı tek başına binlerce token | `scripts/verify`: başarıda 1 satır, hatada sadece hata satırları (max 40) |
| 7 | 4 ayrı dosya (SPEC, CONTRACT, TASKS, PROGRESS) | Orkestratör ve komutlar aynı bilgiyi farklı dosyalardan tekrar okuyor | Tek `PLAN.md` (spec + contract + task + durum) |
| 8 | be-coder / fe-coder neredeyse aynı iki agent | Gereksiz agent açıklaması her oturumda | Tek `coder` agent'ı |
| 9 | Küçük işler de spec akışına giriyor | 3 satırlık değişiklik için 3 komut + 4 agent | `/quick` ve `/fix`: subagent yok, doğrudan |
| 10 | Bağımsız task'lar sırayla koşuyor | Süre | Dosyaları çakışmayan task'lar **paralel** coder çağrısıyla |

## 2. Eski ve yeni karşılaştırması

```
ESKİ:  /spec (opus) → scout(haiku) → advisor(opus/high) → /tasks (opus)
       → /next-task → [coder → coder → reviewer+verify] → /clear → /next-task → ... (tekrar)

YENİ:  /feat (opus, 1 tur soru) → scout(haiku, gerekirse)
       → /build (sonnet) → coder ×N (paralel olabilir) → reviewer ×1 (test koşmaz)

KÜÇÜK İŞ:  /quick "..."     BUG:  /fix "..."
```

| | Eski | Yeni |
|---|---|---|
| Komutlar | spec, tasks, next-task | feat, build, quick, fix |
| Agent'lar | scout, advisor, be-coder, fe-coder, reviewer | scout, coder, reviewer |
| Opus çağrısı / feature | 3 | 1 |
| Soru turu / feature | soru sayısı kadar | 1 |
| Review / feature | checkpoint sayısı kadar (+ verify) | 1 (verify yok) |
| İnsan müdahalesi | her checkpoint'te /clear + /next-task | PLAN.md onayı + sonuç |

> Tahmini kazanç: sabit bağlam yükünde (plugin kapatma) oturum başına büyük düşüş; feature başına
> toplamda kabaca **%50-70 daha az token** ve belirgin şekilde daha kısa süre. Gerçek oranı
> `/cost` ile ölçerek doğrula (bkz. bölüm 7).

## 3. Klasör yapısı

```
lean-workflow/
├── install.ps1                 # Global kurulum (yedek alır)
├── KULLANIM-KILAVUZU.md        # Bu dosya
├── claude/                     # → ~/.claude/ içine kopyalanır
│   ├── CLAUDE.md               # Kısa global kurallar (her oturumda yüklenir)
│   ├── agents/  scout.md  coder.md  reviewer.md
│   └── commands/ feat.md  build.md  quick.md  fix.md
└── project-template/           # → her projenin köküne kopyalanır
    ├── CLAUDE.md               # ≤30 satırlık proje şablonu
    ├── .claude/settings.json   # .NET plugin'i burada açık + izinler + deny
    └── scripts/ verify  verify.conf.example
```

## 4. Kurulum

### 4.1 Global (bir kez)
PowerShell'de:
```powershell
cd C:\Users\hakan.akdogan\Desktop\claude-worflow-imp\lean-workflow
powershell -ExecutionPolicy Bypass -File .\install.ps1
```
Script şunları yapar:
1. `~/.claude/agents`, `commands`, `CLAUDE.md`, `settings.json` → `~/.claude/backups/lean-<tarih>/` altına yedekler.
2. Eski agent ve komutları kaldırır, lean olanları kopyalar. Eskiler yüklü kalırsa açıklamaları yine token yer.
3. `~/.claude/CLAUDE.md` yoksa kopyalar. Varsa üzerine yazmaz, sen birleştirirsin.
4. Global `settings.json` içinde `dotnet-claude-kit` plugin'ini kapatır.

**Geri alma:** yedek klasördeki dosyaları `~/.claude/` içine geri kopyala.

### 4.2 Her proje için (bir kez)
1. `project-template/` içeriğini projenin köküne kopyala.
2. `scripts/verify.conf.example` → `scripts/verify.conf` olarak kopyala; stack'ine uygun satırları bırak.
3. `CLAUDE.md` şablonunu doldur. **En fazla 30 satır.** Bu dosya her subagent'a da yüklenir; her satır her çağrıda para demek.
4. `.gitignore` içine `.verify.log` ekle.
5. .NET projesi değilse `.claude/settings.json` içindeki `enabledPlugins` bloğunu sil.
6. Test: `sh scripts/verify` → `VERIFY GREEN` görmelisin.

> Windows'ta `sh` için Git Bash gerekir (Claude Code zaten Git Bash kullanıyor).

## 5. Günlük kullanım

### Feature (yeni endpoint, ekran, veri modeli)
```text
/feat siparis-iptal Müşteri teslim edilmemiş siparişini iptal edebilsin
```
- Klasörde ANALYSIS.md varsa onu okur, yoksa verdiğin kısa açıklamayı kullanır. PDF ve .docx dosyalarını okumaz; bu dosyaları sen ANALYSIS.md'ye çevirirsin.
- Mevcut kod varsa ve CONTEXT.md yoksa `scout` (Haiku) ile CONTEXT.md üretir.
- **Tek mesajda** en fazla 6 soru ve en fazla 3 "önerilen AC" gelir. Hepsine tek cevapta yanıt ver, örneğin: `1 evet, 2 öneri ok, 3 hayır sadece admin, AC önerileri: 1 ve 3 kabul`.
- Sonuç: `docs/features/siparis-iptal/PLAN.md`.

**PLAN.md'yi oku ve düzelt.** En ucuz hata düzeltme anı burası. Sonra:
```text
/build siparis-iptal
```
- Bağımlılığı biten task'ları `coder`'a verir. Dosyaları çakışmıyorsa paralel çalıştırır.
- Her task için ayrı commit atar: `T01: ...`.
- Bitince `reviewer` bir kez çalışır. CRITICAL bulgu varsa tek fix turu yapılır ve `fix: review` commit'i atılır.
- MAJOR ve MINOR notlar PLAN.md'deki "Review notes" bölümüne eklenir.
- BLOCKED olursa durur ve nedenini 2 satırda söyler. Sorunu düzeltip `/build` komutunu tekrar çalıştır; kaldığı yerden devam eder.

### Küçük iş (≤ 3-5 dosya, yeni API yok)
```text
/quick Sipariş listesinde tarih kolonunu dd.MM.yyyy formatında göster
```

### Bug
```text
/fix İptal edilen sipariş stokta geri dönmüyor. Beklenen: stok +1, olan: değişmiyor
```
Sırası: önce kırmızı reproduksiyon testi, sonra kök neden, sonra en küçük düzeltme ve verify. 3 başarısız denemede durur.

### Büyük analiz (birden fazla feature)
`/feat` kapsamın büyük olduğunu fark edince bölmeyi önerir. Kabul edersen EPIC.md ve alt klasörler oluşur. Ardından her biri için sırayla `/feat <alt-klasör>` → `/build <alt-klasör>` çalıştırırsın.

## 5.1 dotnet-claude-kit ile birlikte kullanım

Plugin silinmedi. Global olarak kapalı, .NET projelerinde ise `.claude/settings.json` ile açık.
Plugin açık olduğunda:
- **Roslyn MCP tool'ları** `scout`, `coder` ve `reviewer` agent'larının `tools:` listesinde.
  - `scout`: `find_symbol`, `find_references`, `get_file_outline`, `get_project_graph`
  - `coder`: `find_symbol`, `find_references`, `get_file_outline`, `get_symbol_source`, `get_diagnostics`
  - `reviewer`: `find_references`, `get_symbol_source`, `get_diagnostics`

  Agent'lar C# kodunda dosyanın tamamını okumak yerine bu tool'ları kullanır. Bunlar kısa sonuçlar döndürdüğü için daha az token harcanır. `get_diagnostics`, coder'a tam build beklemeden hızlı bir derleme kontrolü sağlar.
- **Skill'ler** (`ef-core`, `minimal-api`, `modern-csharp`, `build-fix`…) konu açılınca kendiliğinden yüklenir.
- **Hook'lar** (tehlikeli komut koruması vb.) çalışmaya devam eder.
- **Plugin'in kendi agent'larını** (`code-reviewer`, `ef-core-specialist`…) lean akış çağırmaz. İstersen elle kullanabilirsin.

.NET dışı projelerde plugin kapalı olduğu için bu tool'lar bulunmaz. Agent'lar Grep ve Read ile çalışmaya devam eder.

## 6. Token disiplini (alışkanlıklar)

1. **Her feature'dan sonra `/clear`.** Bağlam dosyalarda (PLAN.md, CONTEXT.md) yaşıyor; sohbet geçmişine ihtiyaç yok.
2. **Model seçimi:** Günlük oturum modelini **Sonnet** tut (`/model sonnet`). Komutlar kendi modelini zaten seçiyor: `/feat` opus, diğerleri sonnet, scout haiku.
3. **Uzun oturumda `/compact`.** `/clear` yapamıyorsan bağlamı sıkıştır.
4. **Plugin ve MCP'leri sadece gereken projede aç.** `/context` ile sabit yükü gör. MCP tool'ları ve skill listesi orada görünür.
5. **CLAUDE.md'leri kısa tut.** Global ≤ 12 satır, proje ≤ 30 satır. Uzun dokümanı `docs/` altına koy, CLAUDE.md'den sadece referans ver.
6. **Küçük işte `/feat` kullanma.** `/quick` 1 oturumda biter. `/feat` + `/build` en az 3-4 model çağrısı demektir.
7. **Paralellik için iyi PLAN.md yaz.** Task'ların `Files:` listeleri net ve çakışmasızsa `/build` onları paralel koşar.

## 7. Ölçme (gerçekten düştü mü?)

Aynı büyüklükte bir feature'ı eski ve yeni akışla yap, karşılaştır:
- `/cost` → oturumun token ve maliyeti (subagent'lar dahil).
- `/context` → oturum açılışındaki sabit yük. Plugin kapatmadan önce ve sonra bak; farkı burada en net görürsün.
- Süre: `/feat` başlangıcından review sonucuna kadar geçen duvar saati.

## 8. Neyi bilerek kaldırdım / neye dikkat?

| Kaldırılan | Neden | Geri istersen |
|---|---|---|
| `advisor` agent'ı | Opus+high, ayrı tur. Aynı işi `/feat` tek mesajda "önerilen AC" olarak yapıyor | Eski `advisor.md`'yi yedekten geri koy, `/feat` 2. adımında çağır |
| Checkpoint başına review | Feature başına tek review yeterli. Coder zaten verify ve self-check yapıyor | `/build` 3. adımını her 3 task'ta bir çalıştıracak şekilde düzenle |
| Reviewer'ın verify koşması | Coder'lar yeşil verify ile dönüyor | Kritik projelerde reviewer.md'den "do NOT re-run" satırını sil |
| İkinci review turu | Nadiren fark yaratıp tam tur maliyeti var | Gerekirse `reviewer`'ı elle çağır: "review START..HEAD" |
| Ayrı SPEC/CONTRACT/TASKS/PROGRESS | Tek PLAN.md daha az okuma demek | — |

**Dikkat:**
- Paralel coder'lar aynı çalışma klasöründe çalışır. Bu yüzden PLAN.md'deki `Files:` listeleri doğru olmalı. Emin değilsen task'lara `deps:` ekleyerek sıralı çalıştır.
- `.claude/settings.json`'daki `allow` listesi onay sorularını azaltır ve akışı hızlandırır. Listeye sadece güvendiğin komutları ekle.
- Kod kalitesini koruyan kurallar korundu: tek task kapsamı, sözleşmeye birebir uyum, AC başına test, 3 deneme limiti, bağımsız reviewer, reproduksiyon-önce bug fix.
