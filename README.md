#TOM AND JERRY CHASE
USING PYTHON 

import pygame, sys, random

# Init Pygame
pygame.init()

# Constants & Setup
WIDTH, HEIGHT = 800, 600
TOM_SIZE, JERRY_SIZE = 70, 50
SPEED_FACTOR = 4
BLACK, WALL_COLOR = (0,0,0), (100, 50, 0)
SCREEN = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Tom & Jerry: The Maze Chase!")

# Game States
START_SCREEN, PLAYING, GAME_OVER_SCREEN = 0, 1, 2

# Fonts
GAME_FONT, NAME_FONT, START_FONT = pygame.font.Font(None, 48), pygame.font.Font(None, 20), pygame.font.Font(None, 60)
GAMEOVER_FONT = pygame.font.Font(None, 72) # New font for Game Over

# --- Load and Scale Images ---
try:
    tom_image = pygame.transform.scale(pygame.image.load('tom.png').convert_alpha(), (TOM_SIZE, TOM_SIZE))
    jerry_image = pygame.transform.scale(pygame.image.load('jerry.png').convert_alpha(), (JERRY_SIZE, JERRY_SIZE))
    background_image = pygame.transform.scale(pygame.image.load('background.png').convert(), (WIDTH, HEIGHT))
except pygame.error as e:
    print(f"Error loading image: {e}"); print("Make sure image files are in the same directory."); pygame.quit(); sys.exit()

# --- Maze Walls Definition ---
MAZE_WALLS = [
    pygame.Rect(50, 50, 700, 20), pygame.Rect(50, 530, 700, 20),
    pygame.Rect(50, 70, 20, 460), pygame.Rect(730, 70, 20, 460),
    pygame.Rect(150, 150, 200, 20), pygame.Rect(150, 170, 20, 150),
    pygame.Rect(300, 250, 200, 20), pygame.Rect(480, 270, 20, 150),
    pygame.Rect(300, 400, 200, 20), pygame.Rect(600, 100, 20, 150),
    pygame.Rect(500, 450, 200, 20), pygame.Rect(100, 400, 50, 20),
    pygame.Rect(200, 480, 100, 20), pygame.Rect(400, 100, 20, 100),
]

class Character:
    def __init__(self, x, y, size, speed, image, name=""):
        self.image, self.speed, self.name = image, speed, name
        self.rect = self.image.get_rect(topleft=(x, y))
        self.name_surface = NAME_FONT.render(self.name, True, BLACK)
        self.name_rect = self.name_surface.get_rect(center=self.rect.center)
    def draw(self):
        SCREEN.blit(self.image, self.rect)
        self.name_rect.center = self.rect.center
        SCREEN.blit(self.name_surface, self.name_rect)
    def move(self, dx, dy, walls):
        original_x, original_y = self.rect.x, self.rect.y
        self.rect.x += dx * self.speed; self.rect.left = max(0, self.rect.left); self.rect.right = min(WIDTH, self.rect.right)
        for wall in walls:
            if self.rect.colliderect(wall): self.rect.x = original_x; break
        self.rect.y += dy * self.speed; self.rect.top = max(0, self.rect.top); self.rect.bottom = min(HEIGHT, self.rect.bottom)
        for wall in walls:
            if self.rect.colliderect(wall): self.rect.y = original_y; break

# Game Objects (initialized for potential reset)
tom = Character(WIDTH // 4, HEIGHT // 2, TOM_SIZE, SPEED_FACTOR, tom_image, "Tom")
jerry = Character(WIDTH * 3 // 4, HEIGHT // 2, JERRY_SIZE, SPEED_FACTOR + 1, jerry_image, "Jerry")

# --- Game State Variables ---
current_game_state = START_SCREEN
blink_timer = 0
blink_interval = 500 # milliseconds

# --- Scoreboard Variables ---
current_score = 0
score_update_timer = 0
SCORE_INTERVAL = 1000 # Add 1 to score every 1000ms (1 second)

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT: pygame.quit(); sys.exit()
        # Handle state transitions based on timers/events
        if event.type == pygame.USEREVENT + 1 and current_game_state == GAME_OVER_SCREEN:
            current_game_state = START_SCREEN # Go back to start screen after game over message

        if current_game_state == START_SCREEN:
            if event.type == pygame.KEYDOWN and event.key == pygame.K_RETURN:
                current_game_state = PLAYING
                tom.rect.topleft = (WIDTH // 4, HEIGHT // 2); jerry.rect.topleft = (WIDTH * 3 // 4, HEIGHT // 2)
                current_score = 0; score_update_timer = pygame.time.get_ticks()

    # --- Game Logic ---
    if current_game_state == PLAYING:
        current_time = pygame.time.get_ticks()
        if current_time - score_update_timer > SCORE_INTERVAL: current_score += 1; score_update_timer = current_time

        keys = pygame.key.get_pressed(); dx, dy = 0, 0
        if keys[pygame.K_LEFT]: dx = -1
        if keys[pygame.K_RIGHT]: dx = 1
        if keys[pygame.K_UP]: dy = -1
        if keys[pygame.K_DOWN]: dy = 1
        tom.move(dx, dy, MAZE_WALLS)

        jerry_dx = 1 if tom.rect.centerx < jerry.rect.centerx else -1 if tom.rect.centerx > jerry.rect.centerx else 0
        jerry_dy = 1 if tom.rect.centery < jerry.rect.centery else -1 if tom.rect.centery > jerry.rect.centery else 0
        jerry.move(jerry_dx, jerry_dy, MAZE_WALLS)

        if tom.rect.colliderect(jerry.rect.inflate(50, 50)): # Jerry's escape
            while True:
                new_x, new_y = random.randint(0, WIDTH - JERRY_SIZE), random.randint(0, HEIGHT - JERRY_SIZE)
                new_rect = pygame.Rect(new_x, new_y, JERRY_SIZE, JERRY_SIZE)
                is_safe_from_tom = not tom.rect.colliderect(new_rect.inflate(150, 150))
                is_not_in_wall = not any(new_rect.colliderect(wall) for wall in MAZE_WALLS) # More compact wall check
                if is_safe_from_tom and is_not_in_wall:
                    jerry.rect.topleft = new_rect.topleft; break

        if tom.rect.colliderect(jerry.rect): # Actual "almost caught"
            current_game_state = GAME_OVER_SCREEN
            pygame.time.set_timer(pygame.USEREVENT + 1, 3000) # Show game over for 3s

    # --- Drawing ---
    SCREEN.blit(background_image, (0, 0)) # Always draw background

    for wall in MAZE_WALLS: pygame.draw.rect(SCREEN, WALL_COLOR, wall) # Draw maze

    current_time = pygame.time.get_ticks() # Get time once for blinking

    if current_game_state == START_SCREEN:
        if current_time % (blink_interval * 2) < blink_interval:
            start_text = START_FONT.render("Press ENTER to Play", True, BLACK)
            SCREEN.blit(start_text, start_text.get_rect(center=(WIDTH // 2, HEIGHT // 2 + 100)))
    elif current_game_state == PLAYING:
        tom.draw(); jerry.draw() # Draw characters
        score_text = GAME_FONT.render(f"Score: {current_score}", True, BLACK)
        SCREEN.blit(score_text, (WIDTH - score_text.get_width() - 20, 20))
    elif current_game_state == GAME_OVER_SCREEN:
        tom.draw(); jerry.draw() # Show characters in final position
        if current_time % (blink_interval * 2) < blink_interval:
            gameover_text = GAMEOVER_FONT.render("GAME OVER!", True, BLACK)
            msg_text = GAME_FONT.render("Jerry was too quick!", True, BLACK)
            SCREEN.blit(gameover_text, gameover_text.get_rect(center=(WIDTH // 2, HEIGHT // 2 - 30)))
            SCREEN.blit(msg_text, msg_text.get_rect(center=(WIDTH // 2, HEIGHT // 2 + 30)))


    pygame.display.flip()
    pygame.time.Clock().tick(FPS) # Use Pygame.time.Clock().tick for consistent FPS
