# TOM-AND-JERRY-GAME
USING PYTHON

import pygame, sys, random

# Init Pygame
pygame.init()

# Constants & Setup
WIDTH, HEIGHT = 800, 600
TOM_SIZE, JERRY_SIZE = 180, 150 # These now define the image size
SPEED_FACTOR = 5
BLACK = (0,0,0) # Only need black for text
SCREEN = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Tom & Jerry: The Uncatchable Chase!")

# Game States
START_SCREEN = 0
PLAYING = 1
# You can add a GAME_OVER_SCREEN = 2 if you want a separate screen for end-game

# Fonts
GAME_FONT = pygame.font.Font(None, 48) # For game messages
START_FONT = pygame.font.Font(None, 60) # Larger font for start screen message

CLOCK = pygame.time.Clock()
FPS = 60

# --- Load and Scale Images ---
try:
    # Load original images
    tom_image_orig = pygame.image.load('tom.png').convert_alpha() # .convert_alpha() for transparency
    jerry_image_orig = pygame.image.load('jerry.png').convert_alpha()

    # Scale images to desired character sizes
    tom_image = pygame.transform.scale(tom_image_orig, (TOM_SIZE, TOM_SIZE))
    jerry_image = pygame.transform.scale(jerry_image_orig, (JERRY_SIZE, JERRY_SIZE))

    # Load and Scale Background Image
    background_image_orig = pygame.image.load('background.png').convert() # .convert() is fine for non-transparent backgrounds
    background_image = pygame.transform.scale(background_image_orig, (WIDTH, HEIGHT))

except pygame.error as e:
    print(f"Error loading image: {e}")
    print("Make sure 'tom.png', 'jerry.png', AND 'background.png' are in the same directory as the script.")
    pygame.quit()
    sys.exit()

class Character:
    def __init__(self, x, y, size, speed, image):
        self.image = image
        self.rect = self.image.get_rect(topleft=(x, y)) # Get rect from image
        self.speed = speed

    def draw(self):
        SCREEN.blit(self.image, self.rect) # Blit the image instead of drawing ellipse
        # Update text position to stay centered with the character's rect

    def move(self, dx, dy):
        self.rect.x += dx * self.speed; self.rect.y += dy * self.speed
        self.rect.left = max(0, self.rect.left); self.rect.right = min(WIDTH, self.rect.right)
        self.rect.top = max(0, self.rect.top); self.rect.bottom = min(HEIGHT, self.rect.bottom)

# Game Objects (using images now) - Initial positions for playing state
tom = Character(WIDTH // 4, HEIGHT // 2, TOM_SIZE, SPEED_FACTOR, tom_image)
jerry = Character(WIDTH * 3 // 4, HEIGHT // 2, JERRY_SIZE, SPEED_FACTOR + 1, jerry_image) # Jerry is faster

# --- Game State Variables ---
current_game_state = START_SCREEN # Start the game in the start screen
game_over = False # This variable is for when the chase itself ends
blink_timer = 0
blink_interval = 500 # milliseconds

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT: pygame.quit(); sys.exit()
        if event.type == pygame.USEREVENT + 1 and game_over: pygame.quit(); sys.exit() # For game over screen auto-quit

        if current_game_state == START_SCREEN:
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_RETURN: # Check for Enter key press
                    current_game_state = PLAYING # Change state to playing
                    # Reset character positions for new game start
                    tom.rect.topleft = (WIDTH // 4, HEIGHT // 2)
                    jerry.rect.topleft = (WIDTH * 3 // 4, HEIGHT // 2)


    # --- Game Logic based on State ---
    if current_game_state == START_SCREEN:
        # Blinking logic for start message
        current_time = pygame.time.get_ticks()
        if current_time - blink_timer > blink_interval:
            blink_timer = current_time
            # This toggles visibility by changing alpha or a boolean (we'll just use a boolean)
            # For simplicity, we'll draw/not draw the text based on current_time's parity with interval
            pass # Blinking will be handled in drawing section

    elif current_game_state == PLAYING and not game_over:
        # Tom's Movement (Player Control)
        keys = pygame.key.get_pressed(); dx, dy = 0, 0
        if keys[pygame.K_LEFT]: dx = -1
        if keys[pygame.K_RIGHT]: dx = 1
        if keys[pygame.K_UP]: dy = -1
        if keys[pygame.K_DOWN]: dy = 1
        tom.move(dx, dy)

        # Jerry's AI Movement
        jerry_dx = 1 if tom.rect.centerx < jerry.rect.centerx else -1 if tom.rect.centerx > jerry.rect.centerx else 0
        jerry_dy = 1 if tom.rect.centery < jerry.rect.centery else -1 if tom.rect.centery > jerry.rect.centery else 0
        jerry.move(jerry_dx, jerry_dy)

        # Jerry's Escape (Teleport)
        if tom.rect.colliderect(jerry.rect.inflate(50, 50)): # Inflate rect for a "safe zone"
            while True:
                # Find a new random position for Jerry far from Tom
                new_rect = pygame.Rect(random.randint(0, WIDTH - JERRY_SIZE), random.randint(0, HEIGHT - JERRY_SIZE), JERRY_SIZE, JERRY_SIZE)
                if not tom.rect.colliderect(new_rect.inflate(150, 150)): # Ensure teleport is far enough
                    jerry.rect.topleft = new_rect.topleft; break

        # Check if Tom "almost" caught Jerry
        if tom.rect.colliderect(jerry.rect):
            game_over = True
            pygame.time.set_timer(pygame.USEREVENT + 1, 3000) # Auto-quit after 3s


    # --- Drawing ---
    SCREEN.blit(background_image, (0, 0)) # Always draw background first

    if current_game_state == START_SCREEN:
        # Draw "Press Enter to Play" text, making it blink
        current_time = pygame.time.get_ticks()
        if current_time % (blink_interval * 2) < blink_interval: # Toggle visibility every 'blink_interval' ms
            start_text = START_FONT.render("Press ENTER to Play", True, BLACK)
            start_text_rect = start_text.get_rect(center=(WIDTH // 2, HEIGHT // 2 + 100))
            SCREEN.blit(start_text, start_text_rect)

    elif current_game_state == PLAYING:
        tom.draw(); jerry.draw() # Draw characters only when playing

        if game_over:
            msg = "Tom almost caught Jerry! But Jerry is too quick!"
            text_surface = GAME_FONT.render(msg, True, BLACK)
            SCREEN.blit(text_surface, text_surface.get_rect(center=(WIDTH // 2, HEIGHT // 2)))

    pygame.display.flip()
    CLOCK.tick(FPS)
