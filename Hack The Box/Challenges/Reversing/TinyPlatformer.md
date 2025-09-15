# TinyPlatformer

## Recon

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_tinyplatformer]
└─$ file TinyPlatformer 
TinyPlatformer: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.24, BuildID[sha1]=75f8346ea2dbfaafcc12f6682d4424f06b9a8ddd, stripped
```

Run:

![[Pasted image 20250915182214.png]]

## Reversing

This is a PyInstaller executable.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace/rev_tinyplatformer]
└─$ pyinstxtractor-ng TinyPlatformer
```

Decompile:

```py
def display_game_end_screen(self):
    overlay = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
    overlay.fill((0, 0, 0, 200))
    self.screen.blit(overlay, (0, 0))
    large_font = pygame.font.Font(FONT, 72)
    small_font = pygame.font.Font(FONT, 36)
    secret_flag = False
    if self.win:
        title_text = large_font.render("YOU WIN!", True, (255, 255, 0))
        title_shadow = large_font.render("YOU WIN!", True, (200, 150, 0))
        secrets = [[self.player.secret[i] for i in range(6)]]
        secrets += [[self.player.secret[i] for i in range(6, 11)]]
        secrets += [[self.player.secret[i] for i in range(11, len(self.player.secret))]]
        secrets = [
            secrets[0][0] > secrets[0][2],
            secrets[0][1] < secrets[0][4],
            secrets[0][2] > secrets[0][5],
            secrets[0][3] > secrets[0][4],
            secrets[0][5] > secrets[0][3],
            secrets[1][0] > secrets[1][4],
            secrets[1][1] < secrets[1][4],
            secrets[1][2] < secrets[1][3],
            secrets[1][3] < secrets[1][1],
            secrets[2][0] > secrets[2][1],
            secrets[2][2] < secrets[2][1],
            secrets[2][2] > secrets[2][3],
        ]
        secret_flag = secret_flag not in secrets
    else:
        title_text = large_font.render("GAME OVER", True, (255, 50, 50))
        title_shadow = large_font.render("GAME OVER", True, (180, 0, 0))
    title_x = WIDTH // 2 - title_text.get_width() // 2
    title_y = HEIGHT // 4
    self.screen.blit(title_shadow, (title_x + 3, title_y + 3))
    self.screen.blit(title_text, (title_x, title_y))
    instruction_y = HEIGHT - 120
    if secret_flag:
        key = "".join([str(x) for x in self.player.secret]).encode()
        ciph = b"}dvIA_\x00FV\x01A^\x01CoG\x03BD\x00]SO"
        flag = bytes((ciph[i] ^ key[i % len(key)] for i in range(len(ciph)))).decode()
        secret_text = small_font.render(flag, True, (200, 200, 0))
        secret_x = WIDTH // 2 - secret_text.get_width() // 2
        self.screen.blit(secret_text, (secret_x, instruction_y))
        instruction_y += 40
    instruction_text = small_font.render("Press ESC to quit", True, (200, 200, 200))
    instruction_x = WIDTH // 2 - instruction_text.get_width() // 2
    self.screen.blit(instruction_text, (instruction_x, instruction_y))
    border_color = (255, 255, 0) if self.win else (255, 50, 50)
    border_rect = pygame.Rect(50, 50, WIDTH - 100, HEIGHT - 100)
    pygame.draw.rect((self.screen), border_color, border_rect, 5, border_radius=15)
    pygame.display.flip()
```

Solve with:

```py
from z3 import *
from pwn import xor

s0 = [Int(f"s0_{i}") for i in range(6)]
s1 = [Int(f"s1_{i}") for i in range(5)]
s2 = [Int(f"s2_{i}") for i in range(4)]

solver = Solver()

# Level 0 constraints
solver.add(Distinct(s0))
solver.add([And(x >= 0, x < 6) for x in s0])
solver.add(s0[0] > s0[2])
solver.add(s0[1] < s0[4])
solver.add(s0[2] > s0[5])
solver.add(s0[3] > s0[4])
solver.add(s0[5] > s0[3])

# Level 1 constraints
solver.add(Distinct(s1))
solver.add([And(x >= 0, x < 5) for x in s1])
solver.add(s1[0] > s1[4])
solver.add(s1[1] < s1[4])
solver.add(s1[2] < s1[3])
solver.add(s1[3] < s1[1])

# Level 2 constraints
solver.add(Distinct(s2))
solver.add([And(x >= 0, x < 4) for x in s2])
solver.add(s2[0] > s2[1])
solver.add(s2[2] < s2[1])
solver.add(s2[2] > s2[3])

if solver.check() == sat:
    model = solver.model()
    all_vars = s0 + s1 + s2
    key = "".join([str(model[x].as_string()) for x in all_vars]).encode()
    ciph = b"}dvIA_\x00FV\x01A^\x01CoG\x03BD\x00]SO"
    flag = xor(ciph, key)
    print(flag.decode())
```

```plaintext
HTB{pl4tf0rm3r_r3vv1ng}
```
