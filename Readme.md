# Pygame을 이용한 저승사자 뱀서라이크 게임 개발 수업

## 수업 개요
이 수업에서는 Python의 Pygame 라이브러리를 사용하여 2D 액션 게임을 단계별로 개발합니다. 저승사자가 주인공인 뱀서라이크 게임을 만들어보겠습니다.

## 필요한 라이브러리 설치
```bash
pip install pygame
pip install pillow  # 이미지 처리용 (선택사항)
```

---

## 1단계: 기본 Pygame 창 설정

먼저 게임의 기본 틀을 만들어보겠습니다.

**새 파일 생성: game.py**

```python
import pygame
import sys

# 초기화
pygame.init()

# 게임 설정
WINDOW_WIDTH = 1000
WINDOW_HEIGHT = 700
FPS = 60

# 색상 정의
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
RED = (220, 20, 60)
GOLD = (255, 215, 0)
DARK_BLUE = (25, 25, 112)
GRAY = (50, 50, 50)

class Game:
    def __init__(self):
        self.screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
        pygame.display.set_caption("저승사자 뱀서라이크 게임")
        self.clock = pygame.time.Clock()
        
    def handle_events(self):
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                return False
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_ESCAPE:
                    return False
        return True
    
    def update(self, dt):
        pass
    
    def draw(self):
        self.screen.fill(GRAY)
        
        # 테스트용 텍스트
        font = pygame.font.Font(None, 36)
        text = font.render("저승사자 게임 - 1단계", True, WHITE)
        text_rect = text.get_rect(center=(WINDOW_WIDTH//2, WINDOW_HEIGHT//2))
        self.screen.blit(text, text_rect)
        
        pygame.display.flip()
    
    def run(self):
        running = True
        while running:
            dt = self.clock.tick(FPS) / 1000.0
            
            running = self.handle_events()
            self.update(dt)
            self.draw()
        
        pygame.quit()
        sys.exit()

if __name__ == "__main__":
    game = Game()
    game.run()
```

**실행해보기**: 이 코드를 실행하면 회색 배경에 텍스트가 표시된 창이 나타납니다.

---

## 2단계: 플레이어 캐릭터(저승사자) 추가

이제 움직일 수 있는 플레이어 캐릭터를 추가해보겠습니다.

**Game 클래스 위에 GrimReaper 클래스 추가:**

```python
import math

class GrimReaper:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.size = 15
        self.speed = 200
        self.hp = 100
        self.max_hp = 100
        
    def update(self, dt, keys):
        # 이동 처리
        dx = dy = 0
        if keys[pygame.K_LEFT] or keys[pygame.K_a]:
            dx -= self.speed * dt
        if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
            dx += self.speed * dt
        if keys[pygame.K_UP] or keys[pygame.K_w]:
            dy -= self.speed * dt
        if keys[pygame.K_DOWN] or keys[pygame.K_s]:
            dy += self.speed * dt
            
        # 대각선 이동 시 속도 조정
        if dx != 0 and dy != 0:
            dx *= 0.707
            dy *= 0.707
            
        self.x += dx
        self.y += dy
        
        # 화면 경계 체크
        self.x = max(self.size, min(WINDOW_WIDTH - self.size, self.x))
        self.y = max(self.size, min(WINDOW_HEIGHT - self.size, self.y))
    
    def draw(self, screen):
        # 저승사자를 원으로 그리기
        pygame.draw.circle(screen, DARK_BLUE, (int(self.x), int(self.y)), self.size)
        pygame.draw.circle(screen, GOLD, (int(self.x), int(self.y)), self.size - 3)
        pygame.draw.circle(screen, BLACK, (int(self.x), int(self.y)), self.size, 2)
        
        # 눈 그리기
        pygame.draw.circle(screen, GOLD, (int(self.x - 4), int(self.y - 2)), 2)
        pygame.draw.circle(screen, RED, (int(self.x + 4), int(self.y - 2)), 2)
```

**Game 클래스의 __init__ 메서드 수정:**

```python
def __init__(self):
    self.screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
    pygame.display.set_caption("저승사자 뱀서라이크 게임")
    self.clock = pygame.time.Clock()
    
    # 플레이어 생성
    self.player = GrimReaper(WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2)
```

**Game 클래스의 update 메서드 수정:**

```python
def update(self, dt):
    keys = pygame.key.get_pressed()
    self.player.update(dt, keys)
```

**Game 클래스의 draw 메서드 수정:**

```python
def draw(self):
    self.screen.fill(GRAY)
    
    # 플레이어 그리기
    self.player.draw(self.screen)
    
    # HP 바 그리기
    hp_bar_width = 200
    hp_bar_height = 20
    hp_percentage = self.player.hp / self.player.max_hp
    pygame.draw.rect(self.screen, RED, (10, 10, hp_bar_width, hp_bar_height))
    pygame.draw.rect(self.screen, GREEN, (10, 10, hp_bar_width * hp_percentage, hp_bar_height))
    pygame.draw.rect(self.screen, WHITE, (10, 10, hp_bar_width, hp_bar_height), 2)
    
    # 조작법 표시
    font = pygame.font.Font(None, 24)
    text = font.render("WASD or Arrow Keys: Move, ESC: Exit", True, WHITE)
    self.screen.blit(text, (10, WINDOW_HEIGHT - 30))
    
    pygame.display.flip()
```

**GREEN 색상도 추가:**

```python
GREEN = (0, 255, 0)
```

**실행해보기**: 이제 WASD나 방향키로 움직일 수 있는 저승사자가 나타납니다.

---

## 3단계: 적(악귀) 추가

이제 플레이어와 전투할 적을 추가해보겠습니다.

**GrimReaper 클래스 뒤에 EvilSpirit 클래스 추가:**

```python
import random

class EvilSpirit:
    def __init__(self, x, y, spirit_type="normal"):
        self.x = x
        self.y = y
        self.spirit_type = spirit_type
        
        if spirit_type == "boss":
            self.size = 20
            self.hp = 150
            self.max_hp = 150
            self.speed = 30
            self.damage = 25
            self.exp_reward = 50
            self.color = (128, 0, 128)  # PURPLE
        elif spirit_type == "strong":
            self.size = 12
            self.hp = 60
            self.max_hp = 60
            self.speed = 45
            self.damage = 15
            self.exp_reward = 20
            self.color = (255, 165, 0)  # ORANGE
        else:  # normal
            self.size = 10
            self.hp = 30
            self.max_hp = 30
            self.speed = 60
            self.damage = 10
            self.exp_reward = 10
            self.color = RED
            
        self.alive = True
    
    def update(self, dt, player):
        if not self.alive:
            return
            
        # 플레이어 쪽으로 이동
        dx = player.x - self.x
        dy = player.y - self.y
        distance = math.sqrt(dx * dx + dy * dy)
        
        if distance > 0:
            dx /= distance
            dy /= distance
            
            self.x += dx * self.speed * dt
            self.y += dy * self.speed * dt
    
    def take_damage(self, amount):
        self.hp -= amount
        if self.hp <= 0:
            self.alive = False
            return True
        return False
    
    def draw(self, screen):
        if not self.alive:
            return
            
        # 악귀 그리기
        pygame.draw.circle(screen, self.color, (int(self.x), int(self.y)), self.size)
        pygame.draw.circle(screen, BLACK, (int(self.x), int(self.y)), self.size, 1)
        
        # 눈 그리기
        eye_size = max(1, self.size // 5)
        eye_offset = max(2, self.size // 3)
        pygame.draw.circle(screen, WHITE, (int(self.x - eye_offset), int(self.y - eye_size)), eye_size)
        pygame.draw.circle(screen, WHITE, (int(self.x + eye_offset), int(self.y - eye_size)), eye_size)
        
        # HP 바
        bar_width = self.size * 2
        bar_height = 3
        bar_x = self.x - bar_width // 2
        bar_y = self.y - self.size - 8
        
        pygame.draw.rect(screen, RED, (bar_x, bar_y, bar_width, bar_height))
        pygame.draw.rect(screen, GREEN, (bar_x, bar_y, (self.hp / self.max_hp) * bar_width, bar_height))
        pygame.draw.rect(screen, WHITE, (bar_x, bar_y, bar_width, bar_height), 1)
```

**Game 클래스 __init__ 메서드에 적 관련 변수 추가:**

```python
def __init__(self):
    self.screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
    pygame.display.set_caption("저승사자 뱀서라이크 게임")
    self.clock = pygame.time.Clock()
    
    # 플레이어 생성
    self.player = GrimReaper(WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2)
    
    # 적 관련
    self.enemies = []
    self.spawn_timer = 0
    self.spawn_rate = 2.0
    self.game_time = 0
```

**Game 클래스에 적 생성 메서드 추가:**

```python
def spawn_enemy(self):
    # 화면 가장자리에서 생성
    side = random.randint(0, 3)
    if side == 0:  # 위
        x = random.randint(0, WINDOW_WIDTH)
        y = -50
    elif side == 1:  # 오른쪽
        x = WINDOW_WIDTH + 50
        y = random.randint(0, WINDOW_HEIGHT)
    elif side == 2:  # 아래
        x = random.randint(0, WINDOW_WIDTH)
        y = WINDOW_HEIGHT + 50
    else:  # 왼쪽
        x = -50
        y = random.randint(0, WINDOW_HEIGHT)
    
    # 적 타입 결정
    rand = random.random()
    if self.game_time > 30 and rand < 0.05:  # 5% 확률로 보스
        enemy_type = "boss"
    elif self.game_time > 15 and rand < 0.2:  # 20% 확률로 강한 적
        enemy_type = "strong"  
    else:
        enemy_type = "normal"
    
    self.enemies.append(EvilSpirit(x, y, enemy_type))
```

**Game 클래스 update 메서드 수정:**

```python
def update(self, dt):
    self.game_time += dt
    
    keys = pygame.key.get_pressed()
    self.player.update(dt, keys)
    
    # 적 생성
    self.spawn_timer += dt
    if self.spawn_timer >= 1.0 / self.spawn_rate:
        self.spawn_enemy()
        self.spawn_timer = 0
    
    # 적 업데이트
    for enemy in self.enemies[:]:
        enemy.update(dt, self.player)
        
        # 플레이어와 충돌 체크
        distance = math.sqrt((enemy.x - self.player.x) ** 2 + (enemy.y - self.player.y) ** 2)
        if distance < self.player.size + enemy.size:
            self.player.hp -= enemy.damage
            enemy.alive = False
        
        # 죽은 적 제거
        if not enemy.alive:
            self.enemies.remove(enemy)
```

**Game 클래스 draw 메서드에 적 그리기 추가:**

```python
def draw(self):
    self.screen.fill(GRAY)
    
    # 적 그리기
    for enemy in self.enemies:
        enemy.draw(self.screen)
    
    # 플레이어 그리기
    self.player.draw(self.screen)
    
    # HP 바 그리기
    hp_bar_width = 200
    hp_bar_height = 20
    hp_percentage = self.player.hp / self.player.max_hp
    pygame.draw.rect(self.screen, RED, (10, 10, hp_bar_width, hp_bar_height))
    pygame.draw.rect(self.screen, GREEN, (10, 10, hp_bar_width * hp_percentage, hp_bar_height))
    pygame.draw.rect(self.screen, WHITE, (10, 10, hp_bar_width, hp_bar_height), 2)
    
    # 게임 정보 표시
    font = pygame.font.Font(None, 24)
    time_text = font.render(f"Time: {int(self.game_time)}s", True, WHITE)
    enemy_text = font.render(f"Enemies: {len(self.enemies)}", True, WHITE)
    
    self.screen.blit(time_text, (10, 40))
    self.screen.blit(enemy_text, (10, 65))
    
    # 조작법 표시
    text = font.render("WASD or Arrow Keys: Move, ESC: Exit", True, WHITE)
    self.screen.blit(text, (10, WINDOW_HEIGHT - 30))
    
    pygame.display.flip()
```

**실행해보기**: 이제 화면 가장자리에서 적들이 생성되어 플레이어를 추적합니다.

---

## 4단계: 공격 시스템 추가

플레이어가 적을 공격할 수 있는 시스템을 추가해보겠습니다.

**GrimReaper 클래스에 공격 관련 속성 추가 (__init__ 메서드 수정):**

```python
def __init__(self, x, y):
    self.x = x
    self.y = y
    self.size = 15
    self.speed = 200
    self.hp = 100
    self.max_hp = 100
    
    # 공격 관련
    self.attack_damage = 25
    self.attack_range = 80
    self.attack_cooldown = 0
    self.attack_speed = 30
    self.attacking = False
    self.attack_animation_timer = 0
```

**GrimReaper 클래스에 공격 메서드 추가:**

```python
def attack(self, enemies):
    if self.attack_cooldown <= 0:
        # 가장 가까운 적 찾기
        closest_enemy = None
        closest_distance = float('inf')
        
        for enemy in enemies:
            distance = math.sqrt((enemy.x - self.x) ** 2 + (enemy.y - self.y) ** 2)
            if distance <= self.attack_range and distance < closest_distance:
                closest_distance = distance
                closest_enemy = enemy
        
        if closest_enemy:
            closest_enemy.take_damage(self.attack_damage)
            self.attack_cooldown = self.attack_speed
            self.attacking = True
            self.attack_animation_timer = 0.3
            return True
    return False
```

**GrimReaper 클래스 update 메서드에 공격 쿨다운 업데이트 추가:**

```python
def update(self, dt, keys):
    # 이동 처리 (기존 코드 유지)
    dx = dy = 0
    if keys[pygame.K_LEFT] or keys[pygame.K_a]:
        dx -= self.speed * dt
    if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
        dx += self.speed * dt
    if keys[pygame.K_UP] or keys[pygame.K_w]:
        dy -= self.speed * dt
    if keys[pygame.K_DOWN] or keys[pygame.K_s]:
        dy += self.speed * dt
        
    if dx != 0 and dy != 0:
        dx *= 0.707
        dy *= 0.707
        
    self.x += dx
    self.y += dy
    
    self.x = max(self.size, min(WINDOW_WIDTH - self.size, self.x))
    self.y = max(self.size, min(WINDOW_HEIGHT - self.size, self.y))
    
    # 공격 쿨다운 업데이트
    if self.attack_cooldown > 0:
        self.attack_cooldown -= 1
    
    if self.attack_animation_timer > 0:
        self.attack_animation_timer -= dt
        if self.attack_animation_timer <= 0:
            self.attacking = False
```

**GrimReaper 클래스 draw 메서드에 공격 이펙트 추가:**

```python
def draw(self, screen):
    # 공격 범위 표시
    if not self.attacking:
        attack_surface = pygame.Surface((self.attack_range * 2, self.attack_range * 2))
        attack_surface.set_alpha(20)
        pygame.draw.circle(attack_surface, GOLD, (self.attack_range, self.attack_range), self.attack_range)
        screen.blit(attack_surface, (self.x - self.attack_range, self.y - self.attack_range))
    
    # 공격 이펙트
    if self.attacking:
        attack_surface = pygame.Surface((self.attack_range * 2, self.attack_range * 2))
        attack_surface.set_alpha(100)
        pygame.draw.circle(attack_surface, GOLD, (self.attack_range, self.attack_range), self.attack_range)
        screen.blit(attack_surface, (self.x - self.attack_range, self.y - self.attack_range))
    
    # 저승사자 그리기 (기존 코드 유지)
    pygame.draw.circle(screen, DARK_BLUE, (int(self.x), int(self.y)), self.size)
    pygame.draw.circle(screen, GOLD, (int(self.x), int(self.y)), self.size - 3)
    pygame.draw.circle(screen, BLACK, (int(self.x), int(self.y)), self.size, 2)
    
    pygame.draw.circle(screen, GOLD, (int(self.x - 4), int(self.y - 2)), 2)
    pygame.draw.circle(screen, RED, (int(self.x + 4), int(self.y - 2)), 2)
```

**Game 클래스 update 메서드에 공격 처리 추가:**

```python
def update(self, dt):
    self.game_time += dt
    
    keys = pygame.key.get_pressed()
    self.player.update(dt, keys)
    
    # 플레이어 자동 공격
    self.player.attack(self.enemies)
    
    # 적 생성 (기존 코드 유지)
    self.spawn_timer += dt
    if self.spawn_timer >= 1.0 / self.spawn_rate:
        self.spawn_enemy()
        self.spawn_timer = 0
    
    # 적 업데이트 (기존 코드 유지)
    for enemy in self.enemies[:]:
        enemy.update(dt, self.player)
        
        distance = math.sqrt((enemy.x - self.player.x) ** 2 + (enemy.y - self.player.y) ** 2)
        if distance < self.player.size + enemy.size:
            self.player.hp -= enemy.damage
            enemy.alive = False
        
        if not enemy.alive:
            self.enemies.remove(enemy)
```

**실행해보기**: 이제 플레이어가 자동으로 범위 내의 적을 공격합니다.

---

## 5단계: 레벨 시스템과 특수 공격 추가

플레이어의 성장 시스템과 스페이스바로 발동하는 특수 공격을 추가해보겠습니다.

**GrimReaper 클래스 __init__ 메서드에 레벨과 특수 공격 관련 속성 추가:**

```python
def __init__(self, x, y):
    self.x = x
    self.y = y
    self.size = 15
    self.speed = 200
    self.hp = 100
    self.max_hp = 100
    self.level = 1
    self.exp = 0
    self.exp_to_next = 100
    
    # 공격 관련
    self.attack_damage = 25
    self.attack_range = 80
    self.attack_cooldown = 0
    self.attack_speed = 30
    self.attacking = False
    self.attack_animation_timer = 0
    
    # 특수 공격 관련
    self.ultimate_skill_cooldown = 0
    self.ultimate_skill_max_cooldown = 300  # 5초 (60 FPS 기준)
    self.ultimate_skill_active = False
    self.ultimate_skill_timer = 0
    self.ultimate_skill_duration = 180  # 3초
    self.ultimate_skill_damage = 100
    self.ultimate_skill_range = 200
```

**GrimReaper 클래스에 레벨업과 특수 공격 메서드 추가:**

```python
def gain_exp(self, amount):
    self.exp += amount
    if self.exp >= self.exp_to_next:
        self.level_up()

def level_up(self):
    self.level += 1
    self.exp = 0
    self.exp_to_next = int(self.exp_to_next * 1.2)
    # 레벨업 보상
    self.attack_damage += 5
    self.max_hp += 10
    self.hp = self.max_hp

def activate_ultimate_skill(self):
    self.ultimate_skill_active = True
    self.ultimate_skill_timer = self.ultimate_skill_duration
    self.ultimate_skill_cooldown = self.ultimate_skill_max_cooldown

def ultimate_skill_attack(self, enemies):
    if not self.ultimate_skill_active:
        return 0
    
    damaged_enemies = 0
    for enemy in enemies:
        distance = math.sqrt((enemy.x - self.x) ** 2 + (enemy.y - self.y) ** 2)
        if distance <= self.ultimate_skill_range:
            damage_modifier = max(0.5, 1 - (distance / self.ultimate_skill_range) * 0.5)
            final_damage = int(self.ultimate_skill_damage * damage_modifier)
            enemy.take_damage(final_damage)
            damaged_enemies += 1
    
    return damaged_enemies

def take_damage(self, amount):
    self.hp -= amount
    if self.hp <= 0:
        self.hp = 0
        return True  # 죽음
    return False
```

**GrimReaper 클래스 update 메서드에 특수 공격 관련 업데이트 추가:**

```python
def update(self, dt, keys):
    # 이동 처리 (기존 코드 유지)
    speed_modifier = 0.3 if self.ultimate_skill_active else 1.0
    dx = dy = 0
    if keys[pygame.K_LEFT] or keys[pygame.K_a]:
        dx -= self.speed * dt * speed_modifier
    if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
        dx += self.speed * dt * speed_modifier
    if keys[pygame.K_UP] or keys[pygame.K_w]:
        dy -= self.speed * dt * speed_modifier
    if keys[pygame.K_DOWN] or keys[pygame.K_s]:
        dy += self.speed * dt * speed_modifier
        
    if dx != 0 and dy != 0:
        dx *= 0.707
        dy *= 0.707
        
    self.x += dx
    self.y += dy
    
    self.x = max(self.size, min(WINDOW_WIDTH - self.size, self.x))
    self.y = max(self.size, min(WINDOW_HEIGHT - self.size, self.y))
    
    # 특수 공격 입력 체크
    if keys[pygame.K_SPACE] and self.ultimate_skill_cooldown <= 0 and not self.ultimate_skill_active:
        self.activate_ultimate_skill()
    
    # 쿨다운 업데이트
    if self.attack_cooldown > 0:
        self.attack_cooldown -= 1
    
    if self.attack_animation_timer > 0:
        self.attack_animation_timer -= dt
        if self.attack_animation_timer <= 0:
            self.attacking = False
    
    if self.ultimate_skill_cooldown > 0:
        self.ultimate_skill_cooldown -= 1
    
    if self.ultimate_skill_active:
        self.ultimate_skill_timer -= 1
        if self.ultimate_skill_timer <= 0:
            self.ultimate_skill_active = False
```

**GrimReaper 클래스 draw 메서드에 특수 공격 이펙트 추가:**

```python
def draw(self, screen):
    # 특수 공격 이펙트
    if self.ultimate_skill_active:
        progress = 1 - (self.ultimate_skill_timer / self.ultimate_skill_duration)
        
        # 파동 효과
        for i in range(3):
            wave_progress = max(0, min(1, progress * 3 - i))
            if wave_progress > 0:
                wave_radius = int(wave_progress * self.ultimate_skill_range)
                wave_alpha = int(100 * (1 - wave_progress))
                
                wave_surface = pygame.Surface((wave_radius * 2, wave_radius * 2))
                wave_surface.set_alpha(wave_alpha)
                pygame.draw.circle(wave_surface, (135, 206, 235), (wave_radius, wave_radius), wave_radius)
                screen.blit(wave_surface, (self.x - wave_radius, self.y - wave_radius))
        
        # 범위 표시
        pygame.draw.circle(screen, (135, 206, 235), (int(self.x), int(self.y)), self.ultimate_skill_range, 2)
    
    # 일반 공격 범위 표시 (기존 코드)
    elif not self.attacking:
        attack_surface = pygame.Surface((self.attack_range * 2, self.attack_range * 2))
        attack_surface.set_alpha(20)
        pygame.draw.circle(attack_surface, GOLD, (self.attack_range, self.attack_range), self.attack_range)
        screen.blit(attack_surface, (self.x - self.attack_range, self.y - self.attack_range))
    
    # 공격 이펙트 (기존 코드)
    if self.attacking:
        attack_surface = pygame.Surface((self.attack_range * 2, self.attack_range * 2))
        attack_surface.set_alpha(100)
        pygame.draw.circle(attack_surface, GOLD, (self.attack_range, self.attack_range), self.attack_range)
        screen.blit(attack_surface, (self.x - self.attack_range, self.y - self.attack_range))
    
    # 저승사자 그리기 (기존 코드 유지)
    current_size = self.size + (5 if self.ultimate_skill_active else 0)
    
    pygame.draw.circle(screen, DARK_BLUE, (int(self.x), int(self.y)), current_size)
    pygame.draw.circle(screen, GOLD, (int(self.x), int(self.y)), current_size - 3)
    pygame.draw.circle(screen, BLACK, (int(self.x), int(self.y)), current_size, 2)
    
    left_eye_color = GOLD if not self.ultimate_skill_active else (255, 255, 0)
    right_eye_color = RED if not self.ultimate_skill_active else (255, 0, 0)
    
    pygame.draw.circle(screen, left_eye_color, (int(self.x - 4), int(self.y - 2)), 2)
    pygame.draw.circle(screen, right_eye_color, (int(self.x + 4), int(self.y - 2)), 2)
```

**Game 클래스 __init__ 메서드에 점수 변수 추가:**

```python
def __init__(self):
    self.screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
    pygame.display.set_caption("저승사자 뱀서라이크 게임")
    self.clock = pygame.time.Clock()
    
    # 플레이어 생성
    self.player = GrimReaper(WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2)
    
    # 적 관련
    self.enemies = []
    self.spawn_timer = 0
    self.spawn_rate = 2.0
    self.game_time = 0
    self.score = 0
    self.game_over = False
```

**Game 클래스 update 메서드 수정 (경험치 획득과 특수 공격 처리 추가):**

```python
def update(self, dt):
    if self.game_over:
        return
        
    self.game_time += dt
    
    keys = pygame.key.get_pressed()
    self.player.update(dt, keys)
    
    # 플레이어 자동 공격
    self.player.attack(self.enemies)
    
    # 특수 공격 처리
    if self.player.ultimate_skill_active:
        damaged_count = self.player.ultimate_skill_attack(self.enemies)
        if damaged_count > 0:
            self.score += damaged_count * 5
    
    # 적 생성
    self.spawn_timer += dt
    if self.spawn_timer >= 1.0 / self.spawn_rate:
        self.spawn_enemy()
        self.spawn_timer = 0
    
    # 적 업데이트
    for enemy in self.enemies[:]:
        enemy.update(dt, self.player)
        
        distance = math.sqrt((enemy.x - self.player.x) ** 2 + (enemy.y - self.player.y) ** 2)
        if distance < self.player.size + enemy.size:
            if self.player.take_damage(enemy.damage):
                self.game_over = True
            enemy.alive = False
        
        if not enemy.alive:
            self.player.gain_exp(enemy.exp_reward)
            self.score += enemy.exp_reward
            self.enemies.remove(enemy)
```

**Game 클래스 draw 메서드에 UI 추가:**

```python
def draw(self):
    self.screen.fill(GRAY)
    
    if not self.game_over:
        # 적 그리기
        for enemy in self.enemies:
            enemy.draw(self.screen)
        
        # 플레이어 그리기
        self.player.draw(self.screen)
    
    # UI 그리기
    # HP 바
    hp_bar_width = 200
    hp_bar_height = 20
    hp_percentage = self.player.hp / self.player.max_hp
    pygame.draw.rect(self.screen, RED, (10, 10, hp_bar_width, hp_bar_height))
    pygame.draw.rect(self.screen, GREEN, (10, 10, hp_bar_width * hp_percentage, hp_bar_height))
    pygame.draw.rect(self.screen, WHITE, (10, 10, hp_bar_width, hp_bar_height), 2)
    
    # 경험치 바
    exp_bar_width = 200
    exp_bar_height = 10
    exp_percentage = self.player.exp / self.player.exp_to_next
    pygame.draw.rect(self.screen, GRAY, (10, 35, exp_bar_width, exp_bar_height))
    pygame.draw.rect(self.screen, GOLD, (10, 35, exp_bar_width * exp_percentage, exp_bar_height))
    pygame.draw.rect(self.screen, WHITE, (10, 35, exp_bar_width, exp_bar_height), 2)
    
    # 특수 공격 쿨다운 바
    skill_bar_width = 200
    skill_bar_height = 15
    if self.player.ultimate_skill_active:
        skill_percentage = self.player.ultimate_skill_timer / self.player.ultimate_skill_duration
        pygame.draw.rect(self.screen, (50, 50, 50), (10, 50, skill_bar_width, skill_bar_height))
        pygame.draw.rect(self.screen, (135, 206, 235), (10, 50, skill_bar_width * skill_percentage, skill_bar_height))
        skill_text = "Ultimate Active!"
    elif self.player.ultimate_skill_cooldown > 0:
        skill_percentage = 1 - (self.player.ultimate_skill_cooldown / self.player.ultimate_skill_max_cooldown)
        pygame.draw.rect(self.screen, (50, 50, 50), (10, 50, skill_bar_width, skill_bar_height))
        pygame.draw.rect(self.screen, (100, 100, 100), (10, 50, skill_bar_width * skill_percentage, skill_bar_height))
        remaining_seconds = int(self.player.ultimate_skill_cooldown / 60) + 1
        skill_text = f"Ultimate Cooldown: {remaining_seconds}s"
    else:
        pygame.draw.rect(self.screen, (50, 50, 50), (10, 50, skill_bar_width, skill_bar_height))
        pygame.draw.rect(self.screen, (135, 206, 235), (10, 50, skill_bar_width, skill_bar_height))
        skill_text = "Ultimate Ready! (SPACE)"
    
    pygame.draw.rect(self.screen, WHITE, (10, 50, skill_bar_width, skill_bar_height), 2)
    
    # 정보 텍스트
    font = pygame.font.Font(None, 24)
    level_text = font.render(f"Level: {self.player.level}", True, WHITE)
    time_text = font.render(f"Time: {int(self.game_time)}s", True, WHITE)
    score_text = font.render(f"Score: {self.score}", True, WHITE)
    enemy_text = font.render(f"Enemies: {len(self.enemies)}", True, WHITE)
    skill_status = font.render(skill_text, True, WHITE)
    
    self.screen.blit(level_text, (10, 75))
    self.screen.blit(time_text, (10, 100))
    self.screen.blit(score_text, (10, 125))
    self.screen.blit(enemy_text, (10, 150))
    self.screen.blit(skill_status, (220, 50))
    
    if self.game_over:
        # 게임 오버 화면
        overlay = pygame.Surface((WINDOW_WIDTH, WINDOW_HEIGHT))
        overlay.set_alpha(128)
        overlay.fill(BLACK)
        self.screen.blit(overlay, (0, 0))
        
        font_big = pygame.font.Font(None, 72)
        game_over_text = font_big.render("GAME OVER", True, RED)
        final_score_text = font.render(f"Final Score: {self.score}", True, WHITE)
        restart_text = font.render("Press R to Restart, ESC to Exit", True, WHITE)
        
        go_rect = game_over_text.get_rect(center=(WINDOW_WIDTH//2, WINDOW_HEIGHT//2 - 50))
        fs_rect = final_score_text.get_rect(center=(WINDOW_WIDTH//2, WINDOW_HEIGHT//2))
        rs_rect = restart_text.get_rect(center=(WINDOW_WIDTH//2, WINDOW_HEIGHT//2 + 50))
        
        self.screen.blit(game_over_text, go_rect)
        self.screen.blit(final_score_text, fs_rect)
        self.screen.blit(restart_text, rs_rect)
    
    # 조작법 표시
    control_text = font.render("WASD: Move, SPACE: Ultimate, ESC: Exit", True, WHITE)
    self.screen.blit(control_text, (10, WINDOW_HEIGHT - 30))
    
    pygame.display.flip()
```

**Game 클래스에 게임 리셋 메서드 추가:**

```python
def reset_game(self):
    self.player = GrimReaper(WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2)
    self.enemies = []
    self.spawn_timer = 0
    self.spawn_rate = 2.0
    self.game_time = 0
    self.score = 0
    self.game_over = False
```

**Game 클래스 handle_events 메서드에 재시작 처리 추가:**

```python
def handle_events(self):
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            return False
        elif event.type == pygame.KEYDOWN:
            if self.game_over:
                if event.key == pygame.K_r:
                    self.reset_game()
                elif event.key == pygame.K_ESCAPE:
                    return False
            else:
                if event.key == pygame.K_ESCAPE:
                    return False
    return True
```

**실행해보기**: 이제 완전한 게임플레이가 가능합니다. 스페이스바로 특수 공격을 사용하고, 적을 처치해 레벨업할 수 있습니다.

---

## 6단계: 맵 확장과 카메라 시스템

게임 맵을 확장하고 플레이어를 따라다니는 카메라 시스템을 추가해보겠습니다.

**파일 상단의 맵 크기 상수 추가:**

```python
MAP_WIDTH = 4000
MAP_HEIGHT = 3000
```

**Camera 클래스 추가 (Game 클래스 위에):**

```python
class Camera:
    def __init__(self):
        self.x = 0
        self.y = 0
        self.target_x = 0
        self.target_y = 0
        self.smoothing = 0.1
    
    def update(self, target_x, target_y):
        # 카메라가 타겟을 화면 중앙에 유지하도록 계산
        self.target_x = target_x - WINDOW_WIDTH // 2
        self.target_y = target_y - WINDOW_HEIGHT // 2
        
        # 맵 경계 제한
        self.target_x = max(0, min(MAP_WIDTH - WINDOW_WIDTH, self.target_x))
        self.target_y = max(0, min(MAP_HEIGHT - WINDOW_HEIGHT, self.target_y))
        
        # 부드러운 카메라 이동
        self.x += (self.target_x - self.x) * self.smoothing
        self.y += (self.target_y - self.y) * self.smoothing
    
    def apply(self, x, y):
        """월드 좌표를 화면 좌표로 변환"""
        return (x - self.x, y - self.y)
    
    def reverse_apply(self, screen_x, screen_y):
        """화면 좌표를 월드 좌표로 변환"""
        return (screen_x + self.x, screen_y + self.y)
    
    def is_visible(self, x, y, size=50):
        """객체가 화면에 보이는지 확인"""
        screen_x, screen_y = self.apply(x, y)
        return (-size <= screen_x <= WINDOW_WIDTH + size and
                -size <= screen_y <= WINDOW_HEIGHT + size)
```

**GrimReaper 클래스 update 메서드의 경계 체크 수정:**

```python
def update(self, dt, keys):
    # 이동 처리 (기존 코드 유지)
    speed_modifier = 0.3 if self.ultimate_skill_active else 1.0
    dx = dy = 0
    if keys[pygame.K_LEFT] or keys[pygame.K_a]:
        dx -= self.speed * dt * speed_modifier
    if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
        dx += self.speed * dt * speed_modifier
    if keys[pygame.K_UP] or keys[pygame.K_w]:
        dy -= self.speed * dt * speed_modifier
    if keys[pygame.K_DOWN] or keys[pygame.K_s]:
        dy += self.speed * dt * speed_modifier
        
    if dx != 0 and dy != 0:
        dx *= 0.707
        dy *= 0.707
        
    self.x += dx
    self.y += dy
    
    # 맵 경계 체크 (확장된 맵)
    self.x = max(self.size, min(MAP_WIDTH - self.size, self.x))
    self.y = max(self.size, min(MAP_HEIGHT - self.size, self.y))
    
    # 나머지 코드는 기존과 동일...
    if keys[pygame.K_SPACE] and self.ultimate_skill_cooldown <= 0 and not self.ultimate_skill_active:
        self.activate_ultimate_skill()
    
    if self.attack_cooldown > 0:
        self.attack_cooldown -= 1
    
    if self.attack_animation_timer > 0:
        self.attack_animation_timer -= dt
        if self.attack_animation_timer <= 0:
            self.attacking = False
    
    if self.ultimate_skill_cooldown > 0:
        self.ultimate_skill_cooldown -= 1
    
    if self.ultimate_skill_active:
        self.ultimate_skill_timer -= 1
        if self.ultimate_skill_timer <= 0:
            self.ultimate_skill_active = False
```

**GrimReaper 클래스 draw 메서드에 카메라 매개변수 추가:**

```python
def draw(self, screen, camera):
    # 화면 좌표로 변환
    screen_x, screen_y = camera.apply(self.x, self.y)
    
    # 화면에 보이지 않으면 그리지 않음
    if not camera.is_visible(self.x, self.y, self.ultimate_skill_range):
        return
    
    # 특수 공격 이펙트
    if self.ultimate_skill_active:
        progress = 1 - (self.ultimate_skill_timer / self.ultimate_skill_duration)
        
        for i in range(3):
            wave_progress = max(0, min(1, progress * 3 - i))
            if wave_progress > 0:
                wave_radius = int(wave_progress * self.ultimate_skill_range)
                wave_alpha = int(100 * (1 - wave_progress))
                
                wave_surface = pygame.Surface((wave_radius * 2, wave_radius * 2))
                wave_surface.set_alpha(wave_alpha)
                pygame.draw.circle(wave_surface, (135, 206, 235), (wave_radius, wave_radius), wave_radius)
                screen.blit(wave_surface, (screen_x - wave_radius, screen_y - wave_radius))
        
        pygame.draw.circle(screen, (135, 206, 235), (int(screen_x), int(screen_y)), self.ultimate_skill_range, 2)
    
    # 일반 공격 범위 표시
    elif not self.attacking:
        attack_surface = pygame.Surface((self.attack_range * 2, self.attack_range * 2))
        attack_surface.set_alpha(20)
        pygame.draw.circle(attack_surface, GOLD, (self.attack_range, self.attack_range), self.attack_range)
        screen.blit(attack_surface, (screen_x - self.attack_range, screen_y - self.attack_range))
    
    # 공격 이펙트
    if self.attacking:
        attack_surface = pygame.Surface((self.attack_range * 2, self.attack_range * 2))
        attack_surface.set_alpha(100)
        pygame.draw.circle(attack_surface, GOLD, (self.attack_range, self.attack_range), self.attack_range)
        screen.blit(attack_surface, (screen_x - self.attack_range, screen_y - self.attack_range))
    
    # 저승사자 그리기
    current_size = self.size + (5 if self.ultimate_skill_active else 0)
    
    pygame.draw.circle(screen, DARK_BLUE, (int(screen_x), int(screen_y)), current_size)
    pygame.draw.circle(screen, GOLD, (int(screen_x), int(screen_y)), current_size - 3)
    pygame.draw.circle(screen, BLACK, (int(screen_x), int(screen_y)), current_size, 2)
    
    left_eye_color = GOLD if not self.ultimate_skill_active else (255, 255, 0)
    right_eye_color = RED if not self.ultimate_skill_active else (255, 0, 0)
    
    pygame.draw.circle(screen, left_eye_color, (int(screen_x - 4), int(screen_y - 2)), 2)
    pygame.draw.circle(screen, right_eye_color, (int(screen_x + 4), int(screen_y - 2)), 2)
```

**EvilSpirit 클래스 draw 메서드에 카메라 매개변수 추가:**

```python
def draw(self, screen, camera):
    if not self.alive:
        return
    
    # 화면 좌표로 변환
    screen_x, screen_y = camera.apply(self.x, self.y)
    
    # 화면에 보이지 않으면 그리지 않음
    if not camera.is_visible(self.x, self.y, self.size * 2):
        return
    
    # 악귀 그리기
    pygame.draw.circle(screen, self.color, (int(screen_x), int(screen_y)), self.size)
    pygame.draw.circle(screen, BLACK, (int(screen_x), int(screen_y)), self.size, 1)
    
    # 눈 그리기
    eye_size = max(1, self.size // 5)
    eye_offset = max(2, self.size // 3)
    pygame.draw.circle(screen, WHITE, (int(screen_x - eye_offset), int(screen_y - eye_size)), eye_size)
    pygame.draw.circle(screen, WHITE, (int(screen_x + eye_offset), int(screen_y - eye_size)), eye_size)
    
    # HP 바
    bar_width = self.size * 2
    bar_height = 3
    bar_x = screen_x - bar_width // 2
    bar_y = screen_y - self.size - 8
    
    pygame.draw.rect(screen, RED, (bar_x, bar_y, bar_width, bar_height))
    pygame.draw.rect(screen, GREEN, (bar_x, bar_y, (self.hp / self.max_hp) * bar_width, bar_height))
    pygame.draw.rect(screen, WHITE, (bar_x, bar_y, bar_width, bar_height), 1)
```

**Game 클래스 __init__ 메서드에 카메라 추가:**

```python
def __init__(self):
    self.screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
    pygame.display.set_caption("저승사자 뱀서라이크 게임")
    self.clock = pygame.time.Clock()
    
    # 플레이어를 맵 중앙에 생성
    self.player = GrimReaper(MAP_WIDTH // 2, MAP_HEIGHT // 2)
    self.camera = Camera()
    
    # 적 관련
    self.enemies = []
    self.spawn_timer = 0
    self.spawn_rate = 2.0
    self.game_time = 0
    self.score = 0
    self.game_over = False
```

**Game 클래스 spawn_enemy 메서드 수정 (맵 확장에 맞춤):**

```python
def spawn_enemy(self):
    # 플레이어 주변 범위에서 화면 밖에 생성
    spawn_distance = 400
    
    angle = random.uniform(0, 2 * math.pi)
    distance = spawn_distance + random.uniform(0, 200)
    
    x = self.player.x + math.cos(angle) * distance
    y = self.player.y + math.sin(angle) * distance
    
    # 맵 경계 안에 위치하도록 조정
    x = max(50, min(MAP_WIDTH - 50, x))
    y = max(50, min(MAP_HEIGHT - 50, y))
    
    # 적 타입 결정 (기존 코드와 동일)
    rand = random.random()
    if self.game_time > 30 and rand < 0.05:
        enemy_type = "boss"
    elif self.game_time > 15 and rand < 0.2:
        enemy_type = "strong"  
    else:
        enemy_type = "normal"
    
    self.enemies.append(EvilSpirit(x, y, enemy_type))
```

**Game 클래스 update 메서드에 카메라 업데이트 추가:**

```python
def update(self, dt):
    if self.game_over:
        return
        
    # 카메라 업데이트 (플레이어 따라가기)
    self.camera.update(self.player.x, self.player.y)
    
    self.game_time += dt
    
    keys = pygame.key.get_pressed()
    self.player.update(dt, keys)
    
    # 플레이어 자동 공격
    self.player.attack(self.enemies)
    
    # 특수 공격 처리
    if self.player.ultimate_skill_active:
        damaged_count = self.player.ultimate_skill_attack(self.enemies)
        if damaged_count > 0:
            self.score += damaged_count * 5
    
    # 적 생성
    self.spawn_timer += dt
    if self.spawn_timer >= 1.0 / self.spawn_rate:
        self.spawn_enemy()
        self.spawn_timer = 0
    
    # 적 업데이트 (최적화: 화면에 보이는 적들만)
    for enemy in self.enemies[:]:
        # 플레이어와 너무 멀리 떨어진 적은 제거
        distance_to_player = math.sqrt((enemy.x - self.player.x) ** 2 + (enemy.y - self.player.y) ** 2)
        if distance_to_player > 1000:
            self.enemies.remove(enemy)
            continue
        
        enemy.update(dt, self.player)
        
        if distance_to_player < self.player.size + enemy.size:
            if self.player.take_damage(enemy.damage):
                self.game_over = True
            enemy.alive = False
        
        if not enemy.alive:
            self.player.gain_exp(enemy.exp_reward)
            self.score += enemy.exp_reward
            self.enemies.remove(enemy)
```

**Game 클래스 draw 메서드에 카메라 시스템과 미니맵 추가:**

```python
def draw(self):
    # 배경 그리기
    self.screen.fill(GRAY)
    
    # 큰 맵용 격자 그리기
    grid_size = 100
    start_x = int(self.camera.x // grid_size) * grid_size
    start_y = int(self.camera.y // grid_size) * grid_size
    
    for x in range(start_x, start_x + WINDOW_WIDTH + grid_size, grid_size):
        screen_x, _ = self.camera.apply(x, 0)
        if 0 <= screen_x <= WINDOW_WIDTH:
            pygame.draw.line(self.screen, (70, 70, 70), (screen_x, 0), (screen_x, WINDOW_HEIGHT))
    
    for y in range(start_y, start_y + WINDOW_HEIGHT + grid_size, grid_size):
        _, screen_y = self.camera.apply(0, y)
        if 0 <= screen_y <= WINDOW_HEIGHT:
            pygame.draw.line(self.screen, (70, 70, 70), (0, screen_y), (WINDOW_WIDTH, screen_y))
    
    if not self.game_over:
        # 게임 오브젝트 그리기 (카메라 적용)
        for enemy in self.enemies:
            enemy.draw(self.screen, self.camera)
        
        self.player.draw(self.screen, self.camera)
    
    # UI는 고정 위치 (카메라 영향 없음)
    # HP 바 (기존 코드와 동일)
    hp_bar_width = 200
    hp_bar_height = 20
    hp_percentage = self.player.hp / self.player.max_hp
    pygame.draw.rect(self.screen, RED, (10, 10, hp_bar_width, hp_bar_height))
    pygame.draw.rect(self.screen, GREEN, (10, 10, hp_bar_width * hp_percentage, hp_bar_height))
    pygame.draw.rect(self.screen, WHITE, (10, 10, hp_bar_width, hp_bar_height), 2)
    
    # 경험치 바 (기존 코드와 동일)
    exp_bar_width = 200
    exp_bar_height = 10
    exp_percentage = self.player.exp / self.player.exp_to_next
    pygame.draw.rect(self.screen, GRAY, (10, 35, exp_bar_width, exp_bar_height))
    pygame.draw.rect(self.screen, GOLD, (10, 35, exp_bar_width * exp_percentage, exp_bar_height))
    pygame.draw.rect(self.screen, WHITE, (10, 35, exp_bar_width, exp_bar_height), 2)
    
    # 특수 공격 쿨다운 바 (기존 코드와 동일)
    skill_bar_width = 200
    skill_bar_height = 15
    if self.player.ultimate_skill_active:
        skill_percentage = self.player.ultimate_skill_timer / self.player.ultimate_skill_duration
        pygame.draw.rect(self.screen, (50, 50, 50), (10, 50, skill_bar_width, skill_bar_height))
        pygame.draw.rect(self.screen, (135, 206, 235), (10, 50, skill_bar_width * skill_percentage, skill_bar_height))
        skill_text = "Ultimate Active!"
    elif self.player.ultimate_skill_cooldown > 0:
        skill_percentage = 1 - (self.player.ultimate_skill_cooldown / self.player.ultimate_skill_max_cooldown)
        pygame.draw.rect(self.screen, (50, 50, 50), (10, 50, skill_bar_width, skill_bar_height))
        pygame.draw.rect(self.screen, (100, 100, 100), (10, 50, skill_bar_width * skill_percentage, skill_bar_height))
        remaining_seconds = int(self.player.ultimate_skill_cooldown / 60) + 1
        skill_text = f"Ultimate Cooldown: {remaining_seconds}s"
    else:
        pygame.draw.rect(self.screen, (50, 50, 50), (10, 50, skill_bar_width, skill_bar_height))
        pygame.draw.rect(self.screen, (135, 206, 235), (10, 50, skill_bar_width, skill_bar_height))
        skill_text = "Ultimate Ready! (SPACE)"
    
    pygame.draw.rect(self.screen, WHITE, (10, 50, skill_bar_width, skill_bar_height), 2)
    
    # 정보 텍스트
    font = pygame.font.Font(None, 24)
    level_text = font.render(f"Level: {self.player.level}", True, WHITE)
    time_text = font.render(f"Time: {int(self.game_time)}s", True, WHITE)
    score_text = font.render(f"Score: {self.score}", True, WHITE)
    enemy_text = font.render(f"Enemies: {len(self.enemies)}", True, WHITE)
    pos_text = font.render(f"Pos: ({int(self.player.x)}, {int(self.player.y)})", True, WHITE)
    skill_status = font.render(skill_text, True, WHITE)
    
    self.screen.blit(level_text, (10, 75))
    self.screen.blit(time_text, (10, 100))
    self.screen.blit(score_text, (10, 125))
    self.screen.blit(enemy_text, (10, 150))
    self.screen.blit(pos_text, (10, 175))
    self.screen.blit(skill_status, (220, 50))
    
    # 미니맵 (우상단)
    minimap_size = 150
    minimap_x = WINDOW_WIDTH - minimap_size - 10
    minimap_y = 10
    
    # 미니맵 배경
    pygame.draw.rect(self.screen, (30, 30, 30), (minimap_x, minimap_y, minimap_size, minimap_size))
    pygame.draw.rect(self.screen, WHITE, (minimap_x, minimap_y, minimap_size, minimap_size), 2)
    
    # 플레이어 위치 (미니맵에서)
    player_minimap_x = minimap_x + (self.player.x / MAP_WIDTH) * minimap_size
    player_minimap_y = minimap_y + (self.player.y / MAP_HEIGHT) * minimap_size
    pygame.draw.circle(self.screen, GOLD, (int(player_minimap_x), int(player_minimap_y)), 3)
    
    # 적들 위치 (미니맵에서)
    for enemy in self.enemies:
        enemy_minimap_x = minimap_x + (enemy.x / MAP_WIDTH) * minimap_size
        enemy_minimap_y = minimap_y + (enemy.y / MAP_HEIGHT) * minimap_size
        color = (128, 0, 128) if enemy.spirit_type == "boss" else (255, 165, 0) if enemy.spirit_type == "strong" else RED
        pygame.draw.circle(self.screen, color, (int(enemy_minimap_x), int(enemy_minimap_y)), 1)
    
    if self.game_over:
        # 게임 오버 화면 (기존 코드와 동일)
        overlay = pygame.Surface((WINDOW_WIDTH, WINDOW_HEIGHT))
        overlay.set_alpha(128)
        overlay.fill(BLACK)
        self.screen.blit(overlay, (0, 0))
        
        font_big = pygame.font.Font(None, 72)
        game_over_text = font_big.render("GAME OVER", True, RED)
        final_score_text = font.render(f"Final Score: {self.score}", True, WHITE)
        restart_text = font.render("Press R to Restart, ESC to Exit", True, WHITE)
        
        go_rect = game_over_text.get_rect(center=(WINDOW_WIDTH//2, WINDOW_HEIGHT//2 - 50))
        fs_rect = final_score_text.get_rect(center=(WINDOW_WIDTH//2, WINDOW_HEIGHT//2))
        rs_rect = restart_text.get_rect(center=(WINDOW_WIDTH//2, WINDOW_HEIGHT//2 + 50))
        
        self.screen.blit(game_over_text, go_rect)
        self.screen.blit(final_score_text, fs_rect)
        self.screen.blit(restart_text, rs_rect)
    
    # 조작법 표시
    control_text = font.render("WASD: Move, SPACE: Ultimate, ESC: Exit", True, WHITE)
    self.screen.blit(control_text, (10, WINDOW_HEIGHT - 30))
    
    pygame.display.flip()
```

**Game 클래스 reset_game 메서드 수정:**

```python
def reset_game(self):
    self.player = GrimReaper(MAP_WIDTH // 2, MAP_HEIGHT // 2)
    self.camera = Camera()
    self.enemies = []
    self.spawn_timer = 0
    self.spawn_rate = 2.0
    self.game_time = 0
    self.score = 0
    self.game_over = False
```

**실행해보기**: 이제 넓은 맵에서 카메라가 플레이어를 따라다니며, 우상단에 미니맵이 표시됩니다.

---

## 완성된 게임의 주요 기능

1. **플레이어 캐릭터**: WASD로 움직이는 저승사자
2. **자동 공격**: 범위 내 가장 가까운 적을 자동으로 공격
3. **특수 공격**: 스페이스바로 발동하는 범위 공격
4. **적 AI**: 플레이어를 추적하는 다양한 타입의 악귀
5. **레벨 시스템**: 경험치 획득과 레벨업
6. **카메라 시스템**: 플레이어를 따라다니는 부드러운 카메라
7. **넓은 맵**: 4000x3000 크기의 게임 월드
8. **미니맵**: 전체 맵과 적들의 위치를 한눈에 확인
9. **UI 시스템**: HP, 경험치, 특수공격 쿨다운 바
10. **게임 오버**: 죽음과 재시작 시스템

이 게임을 기반으로 더 많은 기능을 추가할 수 있습니다:
- 다양한 무기와 아이템
- 더 복잡한 적 AI 패턴
- 보스 몬스터
- 사운드 효과와 배경음악
- 이미지와 애니메이션 시스템
