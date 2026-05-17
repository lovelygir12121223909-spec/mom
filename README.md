import pygame
import sys
import random

pygame.init()

# Screen
WIDTH, HEIGHT = 900, 500
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Bendy and the Ink Machine - Simple Prototype")
clock = pygame.time.Clock()
font = pygame.font.SysFont(None, 36)

# Colors (Ink theme)
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
INK = (40, 40, 60)
YELLOW = (255, 220, 0)
RED = (180, 0, 0)

# Player (Henry)
player_size = 40
player_pos = [50, 380]
player_speed = 6
player_vel_y = 0
gravity = 0.8
on_ground = True

# Bendy (Enemy)
bendy_size = 45
bendy_pos = [700, 360]
bendy_speed = 3.5

# Platforms / Studio floors
platforms = [
    pygame.Rect(0, 420, WIDTH, 80),      # ground
    pygame.Rect(150, 320, 200, 20),      # platform 1
    pygame.Rect(500, 250, 250, 20),      # platform 2
    pygame.Rect(100, 180, 180, 20),      # high platform
]

# Ink collectibles
ink_items = [
    pygame.Rect(200, 280, 25, 25),
    pygame.Rect(580, 210, 25, 25),
    pygame.Rect(250, 140, 25, 25),
]

collected = 0
total_ink = len(ink_items)

caught = False
won = False

def draw_text(text, color, x, y):
    img = font.render(text, True, color)
    screen.blit(img, (x, y))

running = True
while running:
    screen.fill(INK)
    
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    if not caught and not won:
        # Player movement
        keys = pygame.key.get_pressed()
        new_x = player_pos[0]
        if keys[pygame.K_LEFT] or keys[pygame.K_a]:
            new_x -= player_speed
        if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
            new_x += player_speed

        # Gravity & jump
        player_vel_y += gravity
        new_y = player_pos[1] + player_vel_y

        # Horizontal collision
        player_rect = pygame.Rect(new_x, player_pos[1], player_size, player_size)
        if not any(player_rect.colliderect(p) for p in platforms):
            player_pos[0] = new_x

        # Vertical collision
        player_rect = pygame.Rect(player_pos[0], new_y, player_size, player_size)
        on_ground = False
        for p in platforms:
            if player_rect.colliderect(p):
                if player_vel_y > 0:   # falling
                    new_y = p.top - player_size
                    player_vel_y = 0
                    on_ground = True
                elif player_vel_y < 0: # jumping up
                    new_y = p.bottom
                    player_vel_y = 0
                break

        player_pos[1] = new_y

        # Jump
        if (keys[pygame.K_SPACE] or keys[pygame.K_w] or keys[pygame.K_UP]) and on_ground:
            player_vel_y = -15

        # Keep player in bounds
        player_pos[0] = max(0, min(WIDTH - player_size, player_pos[0]))

        # Bendy movement (simple chase)
        if abs(player_pos[0] - bendy_pos[0]) > 30:
            direction = 1 if player_pos[0] > bendy_pos[0] else -1
            bendy_pos[0] += direction * bendy_speed

        # Collect ink
        player_rect = pygame.Rect(player_pos[0], player_pos[1], player_size, player_size)
        for ink in ink_items[:]:
            if player_rect.colliderect(ink):
                ink_items.remove(ink)
                collected += 1

        # Caught by Bendy?
        bendy_rect = pygame.Rect(bendy_pos[0], bendy_pos[1], bendy_size, bendy_size)
        if player_rect.colliderect(bendy_rect):
            caught = True

        # Win condition
        if collected >= total_ink and player_pos[0] > WIDTH - 100:
            won = True

    # Drawing
    # Platforms (old studio look)
    for p in platforms:
        pygame.draw.rect(screen, (80, 60, 40), p)
        pygame.draw.rect(screen, YELLOW, p, 3)

    # Ink items
    for ink in ink_items:
        pygame.draw.rect(screen, YELLOW, ink)
        pygame.draw.circle(screen, WHITE, (ink.centerx, ink.centery), 8)

    # Player (simple rectangle with hat)
    pygame.draw.rect(screen, WHITE, (*player_pos, player_size, player_size))
    pygame.draw.rect(screen, BLACK, (player_pos[0]+10, player_pos[1]+8, 20, 12))  # hat

    # Bendy
    pygame.draw.rect(screen, BLACK, (*bendy_pos, bendy_size, bendy_size))
    pygame.draw.circle(screen, WHITE, (bendy_pos[0]+15, bendy_pos[1]+15), 8)  # eye
    pygame.draw.circle(screen, WHITE, (bendy_pos[0]+30, bendy_pos[1]+15), 8)

    # UI
    draw_text(f"Ink collected: {collected}/{total_ink}", WHITE, 10, 10)
    draw_text("A/D - Move   SPACE - Jump", WHITE, 10, 40)

    if caught:
        draw_text("BENDY GOT YOU!", RED, WIDTH//2 - 140, HEIGHT//2)
    if won:
        draw_text("YOU ESCAPED THE INK MACHINE!", YELLOW, WIDTH//2 - 220, HEIGHT//2)

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
sys.exit()# mom