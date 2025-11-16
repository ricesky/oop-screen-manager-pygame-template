# oop-screen-manager-pygame

Pada tutorial akan menjelaskan bagaimana membuat **Screen Manager** sederhana menggunakan PyGame, terdiri dari:

* **Main Menu Screen** (Start Game, High Score, Exit)
* **Game Screen**
* **High Score Screen**
* **Screen Manager** yang mengatur perpindahan antar screen
* **Button UI** yang merespons hover dan klik

---

# Capaian Pembelajaran

Mahasiswa mampu:

1. Memahami konsep **multi-screen** dalam game modern.
2. Mengimplementasikan **Screen Manager** berbasis OOP.
3. Membuat **komponen UI Button** interaktif di PyGame.
4. Mengatur perpindahan screen dengan **event handling**.

---

## Lingkungan Pengembangan

1. Platform: Python 3.12+
2. Bahasa: Python
3. Editor/IDE yang disarankan:
   - VS Code + Python Extension
   - Terminal
4. Library:
   - pygame 2.6.1

---

## Cara Menjalankan Project

1. Clone repositori project `oop-screen-manager-pygame` ke direktori lokal Anda:

   ```bash
   git clone https://github.com/USERNAME/oop-screen-manager-pygame.git
   cd oop-screen-manager-pygame
   ````

2. Buat dan aktifkan virtual environment:

   ```bash
   python -m venv .venv
   source .venv/bin/activate        # Linux/macOS
   .venv\Scripts\activate           # Windows
   ```

3. Install dependensi:

   ```bash
   pip install -r requirements.txt
   ```

4. Menjalankan program:

   ```bash
   python -m src.main
   ```

---

# TUTORIAL

# Tahap 1 — Menyiapkan Struktur Dasar Project

Sebelum membuat kode, kita perlu struktur yang jelas.

Buat folder berikut:

```
src/
├─ main.py
├─ screen_manager.py
├─ screens/
│  ├─ base.py
│  ├─ main_menu.py
│  ├─ game_screen.py
│  └─ high_score_screen.py
└─ ui/
   └─ button.py
```

Tujuan struktur ini:

* **screens/** berisi semua layar/game state
* **ui/** berisi komponen antarmuka (misal Button)
* **screen_manager.py** memutuskan screen mana yang aktif
* **main.py** menjalankan game loop

---

# Tahap 2 — Konsep Dasar “Screen” (Kelas Induk)

Pada PyGame, semua tampilan dibuat dalam 1 window. Namun dalam game, kita membutuhkan banyak layar:

* menu
* gameplay
* high score
* pause
* game over

Karena PyGame tidak menyediakan fitur manajemen layar, kita dapat membuatnya sendiri.

> Setiap screen mempunyai 3 fungsi penting:
>
> * `handle_event()`
> * `update()`
> * `draw()`

Ini akan menjadi **aturan/kontrak** bagi semua screen.

---

## Ketik kode berikut:

**`src/screens/base.py`**

```python
import pygame

class ScreenBase:
    def __init__(self, manager, screen_size: tuple[int, int]):
        self.manager = manager
        self.screen_width, self.screen_height = screen_size

    def handle_event(self, event):
        raise NotImplementedError

    def update(self, dt):
        raise NotImplementedError

    def draw(self, surface):
        raise NotImplementedError
```

**Tujuan:**
Ini adalah kelas dasar untuk semua screen lain.

---

# Tahap 3 — Membuat Komponen Button

Agar screen bisa memiliki tombol seperti:

* Start Game
* High Score
* Exit
* Back

kita harus membuat komponen Button yang:

* bisa di-hover
* bisa diklik
* memanggil callback function tertentu

---

## Ketik:

**`src/ui/button.py`**

```python
import pygame

class Button:
    def __init__(self, text, center_pos, size, callback, font,
                 bg_color=(70,70,70), hover_color=(100,100,100), text_color=(255,255,255)):
        
        self.text = text
        self.callback = callback
        self.font = font

        self.rect = pygame.Rect(0, 0, *size)
        self.rect.center = center_pos

        self.bg_color = bg_color
        self.hover_color = hover_color
        self.text_color = text_color

        self._is_hovered = False
        self.text_surface = self.font.render(text, True, text_color)
        self.text_rect = self.text_surface.get_rect(center=self.rect.center)

    def handle_event(self, event):
        if event.type == pygame.MOUSEMOTION:
            self._is_hovered = self.rect.collidepoint(event.pos)

        elif event.type == pygame.MOUSEBUTTONDOWN:
            if event.button == 1 and self.rect.collidepoint(event.pos):
                self.callback()

    def draw(self, surface):
        color = self.hover_color if self._is_hovered else self.bg_color
        pygame.draw.rect(surface, color, self.rect, border_radius=8)
        pygame.draw.rect(surface, (200,200,200), self.rect, 2, border_radius=8)
        surface.blit(self.text_surface, self.text_rect)
```

---

# Tahap 4 — Screen Manager

Screen Manager bertugas:

* menyimpan daftar screen
* mengatur screen mana yang sedang aktif
* melakukan perpindahan screen

---

## Ketik:

**`src/screen_manager.py`**

```python
from .screens.main_menu import MainMenuScreen
from .screens.game_screen import GameScreen
from .screens.high_score_screen import HighScoreScreen

class ScreenManager:
    def __init__(self, screen_size):
        self.screen_size = screen_size
        self.screens = {}
        self.current_screen = None

        self._register_screens()

    def _register_screens(self):
        self.screens["main_menu"] = MainMenuScreen(self, self.screen_size)
        self.screens["game"] = GameScreen(self, self.screen_size)
        self.screens["high_score"] = HighScoreScreen(self, self.screen_size)

        # Screen awal
        self.go_to("main_menu")

    def go_to(self, name):
        self.current_screen = self.screens[name]

    def handle_event(self, event):
        self.current_screen.handle_event(event)

    def update(self, dt):
        self.current_screen.update(dt)

    def draw(self, surface):
        self.current_screen.draw(surface)
```

---

# Tahap 5 — Main Menu Screen

Main Menu Screen adalah layar pertama yang muncul dan berisi tombol sebagai berikut:

* tombol Start Game
* tombol High Score
* tombol Exit

Ketika tombol ditekan, screen akan berpindah melalui `ScreenManager`.

---

## Ketik:

**`src/screens/main_menu.py`**

```python
import pygame
from .base import ScreenBase
from ..ui.button import Button

class MainMenuScreen(ScreenBase):
    def __init__(self, manager, screen_size):
        super().__init__(manager, screen_size)

        self.font_title = pygame.font.SysFont(None, 48)
        self.font_button = pygame.font.SysFont(None, 28)

        cx = screen_size[0] // 2
        cy = screen_size[1] // 2 - 40

        self.buttons = [
            Button("Start Game", (cx, cy), (180, 40), self.start_game, self.font_button),
            Button("High Score", (cx, cy + 60), (180, 40), self.go_high_score, self.font_button),
            Button("Exit", (cx, cy + 120), (180, 40), self.exit_game, self.font_button),
        ]

    def start_game(self):
        self.manager.go_to("game")

    def go_high_score(self):
        self.manager.go_to("high_score")

    def exit_game(self):
        pygame.quit()
        raise SystemExit

    def handle_event(self, event):
        for b in self.buttons:
            b.handle_event(event)

    def update(self, dt): pass

    def draw(self, surface):
        surface.fill((30, 30, 40))
        title = self.font_title.render("Galaxy Runner", True, (255,255,255))
        surface.blit(title, (self.screen_width//2 - 150, 120))

        for b in self.buttons:
            b.draw(surface)
```

**Apa yang harus terjadi:**
Muncul tiga tombol di layar. Hover -> tombol berubah warna. Klik -> screen berpindah.

---

# Tahap 6 — Game Screen

Game screen minimal menampilkan:

* objek bergerak (lingkaran)
* tombol Back untuk kembali ke menu

---

## Ketik:

**`src/screens/game_screen.py`**

```python
import pygame
from .base import ScreenBase
from ..ui.button import Button

class GameScreen(ScreenBase):
    def __init__(self, manager, screen_size):
        super().__init__(manager, screen_size)

        self.font = pygame.font.SysFont(None, 28)

        self.back_button = Button("Back", (70, 30), (100, 36),
                                  self.go_back, self.font)

        self.player_pos = [300, 300]
        self.speed = 200
        self.radius = 20

    def go_back(self):
        self.manager.go_to("main_menu")

    def handle_event(self, event):
        self.back_button.handle_event(event)

    def update(self, dt):
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            self.player_pos[0] -= self.speed * dt
        if keys[pygame.K_RIGHT]:
            self.player_pos[0] += self.speed * dt
        if keys[pygame.K_UP]:
            self.player_pos[1] -= self.speed * dt
        if keys[pygame.K_DOWN]:
            self.player_pos[1] += self.speed * dt

    def draw(self, surface):
        surface.fill((10, 10, 30))
        pygame.draw.circle(surface, (0,200,255),
                           (int(self.player_pos[0]), int(self.player_pos[1])),
                           self.radius)
        self.back_button.draw(surface)
```

---

# Tahap 7 — High Score Screen

Layar ini menampilkan daftar skor yang akan bisa dikembangkan untuk:

* membaca file JSON
* menulis skor
* animasi

---

## Ketik:

**`src/screens/high_score_screen.py`**

```python
import pygame
from .base import ScreenBase
from ..ui.button import Button

class HighScoreScreen(ScreenBase):
    def __init__(self, manager, screen_size):
        super().__init__(manager, screen_size)

        self.font_title = pygame.font.SysFont(None, 36)
        self.font_text = pygame.font.SysFont(None, 24)

        self.back_button = Button("Back", (70, 30), (100, 36),
                                  self.go_back, self.font_text)

        self.scores = [1500, 1100, 900]

    def go_back(self):
        self.manager.go_to("main_menu")

    def handle_event(self, event):
        self.back_button.handle_event(event)

    def update(self, dt): pass

    def draw(self, surface):
        surface.fill((20,20,20))
        title = self.font_title.render("High Scores", True, (255,255,255))
        surface.blit(title, (self.screen_width//2 - 80, 120))

        y = 200
        for s in self.scores:
            txt = self.font_text.render(str(s), True, (220,220,220))
            surface.blit(txt, (self.screen_width//2 - 20, y))
            y += 40

        self.back_button.draw(surface)
```

---

# Tahap Akhir — Menjalankan Game

**`src/main.py`**

```python
import pygame
from .screen_manager import ScreenManager

def main():
    pygame.init()

    screen_size = (800, 600)
    screen = pygame.display.set_mode(screen_size)
    pygame.display.set_caption("Screen Manager – PyGame OOP")

    clock = pygame.time.Clock()
    manager = ScreenManager(screen_size)

    running = True
    while running:
        dt = clock.tick(60) / 1000.0

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            else:
                manager.handle_event(event)

        manager.update(dt)
        manager.draw(screen)
        pygame.display.flip()

    pygame.quit()

if __name__ == "__main__":
    main()
```

Jalankan program menggunakan perintah sebagai berikut:

   ```bash
   python -m src.main
   ```


---
