import pygame
import sys




# -----------------------------
# Initialize Pygame
# -----------------------------
pygame.init()




# -----------------------------
# Window settings
# -----------------------------
WIDTH = 900
HEIGHT = 900




screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Combat Adventure")




clock = pygame.time.Clock()




# -----------------------------
# World settings
# -----------------------------
WORLD_WIDTH = 2800
WORLD_HEIGHT = 1800




# -----------------------------
# Player
# -----------------------------
player_size = 30
player_x = 200
player_y = 200
player_speed = 6




player_health = 10
max_health = 5




# Direction the player is facing
# "right", "left", "up", or "down"
facing = "right"




# -----------------------------
# Weapons
# -----------------------------
weapons = ["RAM", "KICK"]
current_weapon = 0




ram_damage = 2
kick_damage = 1.5




# -----------------------------
# Projectiles
# -----------------------------
projectiles = []




# -----------------------------
# Walls
# -----------------------------
walls = [pygame.Rect(500, 200, 400, 50),
pygame.Rect(500, 600, 50, 400),
pygame.Rect(1000, 300, 500, 50),
pygame.Rect(1300, 700, 50, 500),
pygame.Rect(1700, 400, 500, 50),
pygame.Rect(2100, 800, 50, 500),
pygame.Rect(700, 1300, 600, 50),
pygame.Rect(1800, 1400, 600, 50),
]




# -----------------------------
# Enemies
# -----------------------------
enemies = [
    {
        "rect": pygame.Rect(700, 400, 50, 50),
        "health": 3,
        "max_health": 3,
        "speed": 3
    },
    {
        "rect": pygame.Rect(1200, 500, 50, 50),
        "health": 3,
        "max_health": 3,
        "speed": 3
    },
    {
        "rect": pygame.Rect(1800, 600, 50, 50),
        "health": 4,
        "max_health": 4,
        "speed": 3
    },
    {
        "rect": pygame.Rect(2300, 1100, 50, 50),
        "health": 5,
        "max_health": 5,
        "speed": 3
    },
]




# -----------------------------
# Camera
# -----------------------------
camera_x = 0
camera_y = 0




# -----------------------------
# Fonts
# -----------------------------
font = pygame.font.Font(None, 36)
big_font = pygame.font.Font(None, 70)




# -----------------------------
# Shooting cooldown
# -----------------------------
enemy_shoot_cooldown = 0




# -----------------------------
# Attack cooldown
# -----------------------------
attack_cooldown = 0




# -----------------------------
# Game states
# -----------------------------
game_over = False
game_won = False








# -----------------------------
# Collision movement
# -----------------------------
def move_with_walls(rect, dx, dy):




    # Horizontal movement
    rect.x += dx




    for wall in walls:
        if rect.colliderect(wall):




            if dx > 0:
                rect.right = wall.left




            elif dx < 0:
                rect.left = wall.right




    # Vertical movement
    rect.y += dy




    for wall in walls:
        if rect.colliderect(wall):




            if dy > 0:
                rect.bottom = wall.top




            elif dy < 0:
                rect.top = wall.bottom








# -----------------------------
# Attack enemy
# -----------------------------
def attack_enemy(enemy, damage):




    enemy["health"] -= damage




    if enemy["health"] <= 0:




        if enemy in enemies:
            enemies.remove(enemy)








# -----------------------------
# Get attack direction
# -----------------------------
def get_direction_vector():




    if facing == "right":
        return pygame.Vector2(1, 0)




    if facing == "left":
        return pygame.Vector2(-1, 0)




    if facing == "up":
        return pygame.Vector2(0, -1)




    if facing == "down":
        return pygame.Vector2(0, 1)




    return pygame.Vector2(1, 0)








# -----------------------------
# Check if enemy is in front
# -----------------------------
def enemy_in_front(enemy, attack_range):




    player_center = pygame.Vector2(
        player_x + player_size / 2,
        player_y + player_size / 2
    )




    enemy_center = pygame.Vector2(
        enemy["rect"].centerx,
        enemy["rect"].centery
    )




    difference = enemy_center - player_center




    distance = difference.length()




    if distance > attack_range:
        return False




    if distance == 0:
        return True




    direction = difference.normalize()
    facing_direction = get_direction_vector()




    # Dot product tells us if enemy is in front
    dot = facing_direction.dot(direction)




    return dot > 0.35








# -----------------------------
# Game loop
# -----------------------------
running = True




while running:




    # -------------------------
    # Events
    # -------------------------
    for event in pygame.event.get():




        if event.type == pygame.QUIT:
            running = False




        if event.type == pygame.KEYDOWN:




            # Weapon selection
            if event.key == pygame.K_1:
                current_weapon = 0




            if event.key == pygame.K_2:
                current_weapon = 1




            # -----------------
            # Attack
            # -----------------
            if (
                event.key == pygame.K_SPACE
                and not game_over
                and not game_won
                and attack_cooldown <= 0
            ):




                weapon = weapons[current_weapon]




                # RAM: damages enemies only when touching them.
                if weapon == "RAM":
                    player_rect = pygame.Rect(
                        int(player_x), int(player_y), player_size, player_size
                    )
                    for enemy in enemies[:]:
                        if player_rect.colliderect(enemy["rect"]):
                            attack_enemy(enemy, ram_damage)
                    attack_cooldown = 20


                # KICK: also damages enemies only when touching them.
                elif weapon == "KICK":
                    player_rect = pygame.Rect(
                        int(player_x), int(player_y), player_size, player_size
                    )
                    for enemy in enemies[:]:
                        if player_rect.colliderect(enemy["rect"]):
                            attack_enemy(enemy, kick_damage)
                    attack_cooldown = 15


    # -------------------------
    # Main game
    # -------------------------
    if not game_over and not game_won:




        # -------------------------
        # Keyboard movement
        # -------------------------
        keys = pygame.key.get_pressed()




        dx = 0
        dy = 0




        if keys[pygame.K_LEFT] or keys[pygame.K_a]:




            dx -= player_speed
            facing = "left"




        if keys[pygame.K_RIGHT] or keys[pygame.K_d]:




            dx += player_speed
            facing = "right"




        if keys[pygame.K_UP] or keys[pygame.K_w]:




            dy -= player_speed
            facing = "up"




        if keys[pygame.K_DOWN] or keys[pygame.K_s]:




            dy += player_speed
            facing = "down"




        # Diagonal movement
        if dx != 0 and dy != 0:




            dx *= 0.707
            dy *= 0.707




        # Player rectangle
        player_rect = pygame.Rect(
            int(player_x),
            int(player_y),
            player_size,
            player_size
        )




        # Move player
        move_with_walls(
            player_rect,
            int(dx),
            int(dy)
        )




        player_x = player_rect.x
        player_y = player_rect.y




        # Keep player inside world
        player_x = max(
            0,
            min(
                WORLD_WIDTH - player_size,
                player_x
            )
        )




        player_y = max(
            0,
            min(
                WORLD_HEIGHT - player_size,
                player_y
            )
        )




        # -------------------------
        # Enemy movement
        # -------------------------
        player_center = pygame.Vector2(
            player_x + player_size / 2,
            player_y + player_size / 2
        )




        for enemy in enemies:




            enemy_rect = enemy["rect"]




            enemy_center = pygame.Vector2(
                enemy_rect.centerx,
                enemy_rect.centery
            )




            direction = player_center - enemy_center




            if direction.length() > 0:




                direction = direction.normalize()




                enemy_dx = int(
                    direction.x * enemy["speed"]
                )




                enemy_dy = int(
                    direction.y * enemy["speed"]
                )




                move_with_walls(
                    enemy_rect,
                    enemy_dx,
                    enemy_dy
                )




            # ---------------------
            # Enemy damages player
            # ---------------------
            player_rect = pygame.Rect(
                player_x,
                player_y,
                player_size,
                player_size
            )




            if enemy_rect.colliderect(player_rect):
                if pygame.time.get_ticks() % 30 == 0:
                    player_health -= 1


        # Chasing cubes shoot aimed projectiles at the player.
        if enemy_shoot_cooldown <= 0 and enemies:
            for enemy in enemies:
                enemy_center = pygame.Vector2(enemy["rect"].center)
                direction_to_player = player_center - enemy_center
                if direction_to_player.length() > 0:
                    direction_to_player = direction_to_player.normalize()
                    projectiles.append({
                        "x": enemy_center.x,
                        "y": enemy_center.y,
                        "dx": direction_to_player.x * 6,
                        "dy": direction_to_player.y * 6,
                        "damage": 1,
                        "from_enemy": True
                    })
            enemy_shoot_cooldown = 75




        # -------------------------
        # Projectiles
        # -------------------------
        for projectile in projectiles[:]:




            projectile["x"] += projectile["dx"]
            projectile["y"] += projectile["dy"]




            projectile_rect = pygame.Rect(
                int(projectile["x"] - 6),
                int(projectile["y"] - 6),
                12,
                12
            )




            # Outside world
            if (
                projectile["x"] < 0
                or projectile["x"] > WORLD_WIDTH
                or projectile["y"] < 0
                or projectile["y"] > WORLD_HEIGHT
            ):




                if projectile in projectiles:
                    projectiles.remove(projectile)




                continue




            # Wall collision
            hit_wall = False




            for wall in walls:




                if projectile_rect.colliderect(wall):




                    hit_wall = True
                    break




            if hit_wall:




                if projectile in projectiles:
                    projectiles.remove(projectile)




                continue




            # Enemy shots hurt the player; player shots (if any) hurt enemies.
            if projectile.get("from_enemy", False):
                player_rect = pygame.Rect(
                    int(player_x), int(player_y), player_size, player_size
                )
                if projectile_rect.colliderect(player_rect):
                    player_health -= projectile.get("damage", 1)
                    if projectile in projectiles:
                        projectiles.remove(projectile)
                    continue


            else:
                for enemy in enemies[:]:
                    if projectile_rect.colliderect(enemy["rect"]):




                        attack_enemy(enemy, projectile["damage"])
                        if projectile in projectiles:
                            projectiles.remove(projectile)
                        break




        # -------------------------
        # Cooldowns
        # -------------------------
        if enemy_shoot_cooldown > 0:
            enemy_shoot_cooldown -= 1




        if attack_cooldown > 0:
            attack_cooldown -= 1




        # -------------------------
        # Camera
        # -------------------------
        camera_x = (
            player_x
            - WIDTH // 2
            + player_size // 2
        )




        camera_y = (
            player_y
            - HEIGHT // 2
            + player_size // 2
        )




        camera_x = max(
            0,
            min(
                WORLD_WIDTH - WIDTH,
                camera_x
            )
        )




        camera_y = max(
            0,
            min(
                WORLD_HEIGHT - HEIGHT,
                camera_y
            )
        )




        # -------------------------
        # Lose condition
        # -------------------------
        if player_health <= 0:




            player_health = 0
            game_over = True




        # -------------------------
        # WIN CONDITION
        # -------------------------
        if len(enemies) == 0:




            game_won = True




    # -----------------------------
    # Draw background
    # -----------------------------
    screen.fill((100, 180, 100))




    # -----------------------------
    # Draw grid
    # -----------------------------
    grid_size = 100




    for x in range(
        0,
        WORLD_WIDTH,
        grid_size
    ):




        pygame.draw.line(
            screen,
            (90, 160, 90),
            (
                x - camera_x,
                0
            ),
            (
                x - camera_x,
                HEIGHT
            )
        )




    for y in range(
        0,
        WORLD_HEIGHT,
        grid_size
    ):




        pygame.draw.line(
            screen,
            (90, 160, 90),
            (
                0,
                y - camera_y
            ),
            (
                WIDTH,
                y - camera_y
            )
        )




    # -----------------------------
    # Draw walls
    # -----------------------------
    for wall in walls:




        pygame.draw.rect(
            screen,
            (70, 70, 70),
            (
                wall.x - camera_x,
                wall.y - camera_y,
                wall.width,
                wall.height
            )
        )




    # -----------------------------
    # Draw enemies
    # -----------------------------
    for enemy in enemies:




        rect = enemy["rect"]




        pygame.draw.rect(
            screen,
            (180, 40, 40),
            (
                rect.x - camera_x,
                rect.y - camera_y,
                rect.width,
                rect.height
            )
        )




        # Enemy health bar
        health_width = 50




        pygame.draw.rect(
            screen,
            (0, 0, 0),
            (
                rect.x - camera_x,
                rect.y - camera_y - 10,
                health_width,
                6
            )
        )




        pygame.draw.rect(
            screen,
            (0, 255, 0),
            (
                rect.x - camera_x,
                rect.y - camera_y - 10,
                health_width
                * (
                    enemy["health"]
                    / enemy["max_health"]
                ),
                6
            )
        )




    # -----------------------------
    # Draw projectiles
    # -----------------------------
    for projectile in projectiles:




        pygame.draw.circle(
            screen,
            (255, 255, 0),
            (
                int(
                    projectile["x"]
                    - camera_x
                ),
                int(
                    projectile["y"]
                    - camera_y
                )
            ),
            7
        )




    # -----------------------------
    # Draw player
    # -----------------------------
    pygame.draw.rect(
        screen,
        (40, 80, 220),
        (
            player_x - camera_x,
            player_y - camera_y,
            player_size,
            player_size
        )
    )




    # -----------------------------
    # Draw direction indicator
    # -----------------------------
    player_center_x = (
        player_x
        - camera_x
        + player_size // 2
    )




    player_center_y = (
        player_y
        - camera_y
        + player_size // 2
    )




    direction = get_direction_vector()




    pygame.draw.line(
        screen,
        (255, 255, 255),
        (
            player_center_x,
            player_center_y
        ),
        (
            player_center_x
            + direction.x * 30,
            player_center_y
            + direction.y * 30
        ),
        5
    )




    # -----------------------------
    # HUD
    # -----------------------------




    # Hearts
    heart_text = "♥ " * player_health




    health_display = font.render(
        heart_text,
        True,
        (255, 50, 50)
    )




    screen.blit(
        health_display,
        (20, 20)
    )




    # Weapon
    weapon_text = font.render(
        "Action: " + weapons[current_weapon],
        True,
        (255, 255, 255)
    )




    screen.blit(
        weapon_text,
        (20, 60)
    )




    # Controls
    controls = font.render(
        "1 Ram   2 Kick   SPACE Attack (only while touching a cube)",
        True,
        (255, 255, 255)
    )




    screen.blit(
        controls,
        (20, HEIGHT - 40)
    )




    # -----------------------------
    # WIN SCREEN
    # -----------------------------
    if game_won:




        overlay = pygame.Surface(
            (WIDTH, HEIGHT)
        )




        overlay.set_alpha(190)
        overlay.fill((0, 0, 0))




        screen.blit(
            overlay,
            (0, 0)
        )




        win_text = big_font.render(
            "YOU WIN!",
            True,
            (255, 255, 255)
        )




        screen.blit(
            win_text,
            (
                WIDTH // 2
                - win_text.get_width() // 2,
                HEIGHT // 2 - 50
            )
        )




        sub_text = font.render(
            "You defeated all the enemies!",
            True,
            (255, 255, 255)
        )




        screen.blit(
            sub_text,
            (
                WIDTH // 2
                - sub_text.get_width() // 2,
                HEIGHT // 2 + 30
            )
        )




    # -----------------------------
    # GAME OVER SCREEN
    # -----------------------------
    if game_over:




        overlay = pygame.Surface(
            (WIDTH, HEIGHT)
        )




        overlay.set_alpha(190)
        overlay.fill((0, 0, 0))




        screen.blit(
            overlay,
            (0, 0)
        )




        game_over_text = big_font.render(
            "GAME OVER",
            True,
            (255, 255, 255)
        )




        screen.blit(
            game_over_text,
            (
                WIDTH // 2
                - game_over_text.get_width() // 2,
                HEIGHT // 2 - 50
            )
        )




    # -----------------------------
    # Update screen
    # -----------------------------
    pygame.display.flip()




    # -----------------------------
    # FPS
    # -----------------------------
    clock.tick(60)




# -----------------------------
# Quit
# -----------------------------
pygame.quit()
sys.exit()
