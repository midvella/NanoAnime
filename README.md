# NanoAnime

Terminal ve GNU nano için anime teması kontrol paneli. Tek bir tema dosyasını yönetir, ondan `.nanorc` ve terminal ayarlarını üretir. Dağıtımdan bağımsız çalışır.

## Nasıl çalışır?

```
┌──────────────┐     ┌──────────────┐     ┌─────────────────────┐
│  Web Panel   │ ──▶ │  Çekirdek    │ ──▶ │  Adapter'lar        │
│  (localhost) │     │  (tema modeli)│     │  nano / kitty / ... │
└──────────────┘     └──────────────┘     └─────────────────────┘
```

1. Kullanıcı panelde resmi, karartma seviyesini ve renkleri seçer.
2. Panel seçimleri **tema modeline** yazar.
3. Çekirdek, kullanıcının terminalini algılar ve ilgili **adapter**'ı çağırır.
4. Adapter, temayı o programın kendi config formatına çevirip uygular.

## Katmanlar

### 1. Çekirdek (core)

Dağıtıma ve terminale bağlı değildir. Sadece tema modelini ve adapter kayıt sistemini tutar.

```json
{
  "name": "sakura",
  "bg_image": "~/Pictures/anime.png",
  "tint": 0.85,
  "accent": "#ff8fcf",
  "title_fg": "white",
  "title_bg": "magenta"
}
```

Tema dosyası `$XDG_CONFIG_HOME/nano-chan/themes/` altında saklanır (`XDG_CONFIG_HOME` yoksa `~/.config`).

### 2. Adapter arayüzü

Her hedef program (nano, kitty, wezterm...) aynı arayüzü uygular:

```python
class Adapter:
    name: str
    capabilities: set[str]   # {"colors", "opacity", "image", "live"}

    def detect(self) -> bool: ...
    def apply(self, theme: Theme) -> None: ...
    def revert(self) -> None: ...
```

- `detect()`: Bu program sistemde var mı / şu an içinde miyiz?
- `apply()`: Temayı programın config dosyasına yazar.
- `revert()`: Yedekten eski config'i geri yükler.
- `capabilities`: Panel buna bakarak desteklenmeyen ayarları gri gösterir.

### 3. Hazır adapter'lar

| Adapter | Renk | Opaklık | Arka plan resmi | Canlı uygulama |
|---|---|---|---|---|
| nano | ✅ | - | - | ❌ (sadece yeni açılan nano) |
| Kitty | ✅ | ✅ | ✅ | ✅ (`kitty @`) |
| WezTerm | ✅ | ✅ | ✅ | ✅ (config reload) |
| Konsole | ✅ | ✅ | ✅ | ⚠️ kısmen |
| Tilix | ✅ | ✅ | ✅ | ⚠️ kısmen |
| GNOME Terminal | ✅ | ✅ | ❌ | ✅ (`gsettings`/`dconf`) |
| Alacritty | ✅ | ✅ | ❌ | ✅ (config reload) |

> **Not:** Nano terminalin içinde çalışır, arka plan resmini kendisi çizemez. Resim her zaman terminal emülatörünün işidir. Nano adapter'ı bu yüzden metin alanına arka plan rengi vermez, resim görünsün diye sadece başlık ve durum çubuğunu renklendirir.

### 4. Terminal algılama

Sırayla şu ortam değişkenlerine bakılır:

| Değişken | Terminal |
|---|---|
| `KITTY_WINDOW_ID` | Kitty |
| `WEZTERM_EXECUTABLE` | WezTerm |
| `KONSOLE_VERSION` | Konsole |
| `TILIX_ID` | Tilix |
| `ALACRITTY_SOCKET` / `ALACRITTY_LOG` | Alacritty |
| `GNOME_TERMINAL_SCREEN` / `VTE_VERSION` | GNOME Terminal |
| `TERM_PROGRAM` | Yedek kontrol |

Desteklemeyen bir terminal algılanırsa panel, kullanıcıya resim destekleyen bir terminal (Kitty, WezTerm) kurmasını önerir.

### 5. Panel

Localhost'ta çalışan web paneli (FastAPI + basit HTML). Terminalden bağımsız olduğu için her dağıtımda ve her terminalde aynı açılır.

- Canlı önizleme
- Resim yükleme ve karartma slider'ı
- Renk seçiciler
- Algılanan terminal ve desteklenen özellikler listesi
- Uygula / Geri al butonları

Panel yalnızca `127.0.0.1` üzerinde dinler, dışarıya açılmaz.

## Dağıtımlar arası uyumluluk kuralları

- Yollar sabit yazılmaz: `$XDG_CONFIG_HOME` kullanılır.
- Paket yöneticisine (`apt`, `dnf`, `pacman`) bağımlılık yoktur.
- Mevcut config dosyaları değiştirilmeden önce `*.nano-chan.bak` olarak yedeklenir.
- Yalnızca kullanıcı dizininde yazılır, `sudo` gerekmez.

## Kurulum (planlanan)

```bash
pipx install nano-chan
nano-chan          # paneli açar: http://127.0.0.1:8777
```

Sonraki aşamalar: Flatpak ve AppImage.

## Yol haritası

- [ ] **v0.1:** Çekirdek + nano adapter + Kitty adapter + GNOME Terminal (sadece renk)
- [ ] **v0.2:** WezTerm, Konsole, Tilix, Alacritty adapter'ları
- [ ] **v0.3:** Hazır tema galerisi, tema içe/dışa aktarma
- [ ] **v0.4:** Flatpak / AppImage paketleme

## Yeni adapter eklemek

1. `adapters/` altına yeni bir dosya aç.
2. `Adapter` arayüzünü uygula.
3. `registry.py` içine kaydet.
4. Tablodaki yeteneklerini `capabilities` içine yaz.

## Lisans

Karar verilecek. Not: GNU nano'nun kendi kaynağını fork'lamak yerine yalnızca `.nanorc` ürettiği için, bu proje nano'nun GPLv3 lisansına bağlı değildir.
