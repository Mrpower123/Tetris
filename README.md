#Tetris

import pygame
import random
import sys

# Inicializar Pygame
pygame.init()

# Dimensiones de la pantalla
WIDTH, HEIGHT = 300, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Tetris")

# Colores
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
CYAN = (0, 255, 255)
MAGENTA = (255, 0, 255)
YELLOW = (255, 255, 0)
ORANGE = (255, 165, 0)

# Variables del juego
clock = pygame.time.Clock()
font = pygame.font.SysFont("Arial", 30)
grid_width, grid_height = 10, 20
cell_size = 30
grid = [[0 for _ in range(grid_width)] for _ in range(grid_height)]
current_block = None
block_pos = [0, 0]
fall_time = 0
level = 1
score = 0
lines_cleared = 0
rounds = 3
current_round = 1

# Definir bloques de Tetris
blocks = [
    [[1, 1, 1, 1]],  # I
    [[1, 1], [1, 1]],  # O
    [[0, 1, 0], [1, 1, 1]],  # T
    [[1, 0, 0], [1, 1, 1]],  # L
    [[0, 0, 1], [1, 1, 1]],  # J
    [[0, 1, 1], [1, 1, 0]],  # S
    [[1, 1, 0], [0, 1, 1]],  # Z
]

# Colores de los bloques
block_colors = [CYAN, YELLOW, MAGENTA, ORANGE, BLUE, GREEN, RED]

# Función para dibujar texto en pantalla
def draw_text(text, font, color, surface, x, y):
    textobj = font.render(text, True, color)
    textrect = textobj.get_rect()
    textrect.center = (x, y)
    surface.blit(textobj, textrect)

# Función para dibujar la cuadrícula
def draw_grid(surface, grid):
    for y in range(grid_height):
        for x in range(grid_width):
            if grid[y][x] == 0:
                pygame.draw.rect(surface, BLACK, (x * cell_size, y * cell_size, cell_size, cell_size))
            else:
                pygame.draw.rect(surface, block_colors[grid[y][x] - 1], (x * cell_size, y * cell_size, cell_size, cell_size))
            pygame.draw.rect(surface, WHITE, (x * cell_size, y * cell_size, cell_size, cell_size), 1)

# Función para rotar el bloque
def rotate_block(block):
    return [[block[y][x] for y in range(len(block))] for x in range(len(block[0]) - 1, -1, -1)]

# Función para comprobar colisiones
def check_collision(grid, block, offset):
    off_x, off_y = offset
    for y, row in enumerate(block):
        for x, cell in enumerate(row):
            if cell:
                try:
                    if grid[y + off_y][x + off_x]:
                        return True
                except IndexError:
                    return True
    return False

# Función para unir el bloque con la cuadrícula
def join_grid(grid, block, offset):
    off_x, off_y = offset
    for y, row in enumerate(block):
        for x, cell in enumerate(row):
            if cell:
                grid[y + off_y][x + off_x] = cell

# Función para eliminar filas completas
def clear_rows(grid):
    global score, lines_cleared
    lines_to_clear = [i for i, row in enumerate(grid) if all(row)]
    for i in lines_to_clear:
        del grid[i]
        grid.insert(0, [0 for _ in range(grid_width)])
    lines_cleared += len(lines_to_clear)
    score += len(lines_to_clear) * 100

# Función para generar un nuevo bloque
def new_block():
    block = random.choice(blocks)
    color = block_colors[blocks.index(block)]
    return [[color if cell else 0 for cell in row] for row in block]

# Pantallas de victoria y derrota
def victory_screen():
    screen.fill(BLACK)
    draw_text("¡Victoria!", font, WHITE, screen, WIDTH // 2, HEIGHT // 2)
    pygame.display.update()
    pygame.time.wait(3000)

def game_over_screen():
    screen.fill(BLACK)
    draw_text("Game Over", font, RED, screen, WIDTH // 2, HEIGHT // 2)
    pygame.display.update()
    pygame.time.wait(3000)

# Función para reiniciar el juego para la siguiente ronda
def reset_game():
    global grid, current_block, block_pos, fall_time, level, lines_cleared
    grid = [[0 for _ in range(grid_width)] for _ in range(grid_height)]
    current_block = new_block()
    block_pos = [grid_width // 2 - len(current_block[0]) // 2, 0]
    fall_time = 0
    level += 1
    lines_cleared = 0

# Bucle principal del juego
running = True
game_over = False
current_block = new_block()
block_pos = [grid_width // 2 - len(current_block[0]) // 2, 0]
while running:
    screen.fill(BLACK)
    draw_grid(screen, grid)
    
    # Eventos
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.KEYDOWN and not game_over:
            if event.key == pygame.K_LEFT:
                new_pos = [block_pos[0] - 1, block_pos[1]]
                if not check_collision(grid, current_block, new_pos):
                    block_pos = new_pos
            if event.key == pygame.K_RIGHT:
                new_pos = [block_pos[0] + 1, block_pos[1]]
                if not check_collision(grid, current_block, new_pos):
                    block_pos = new_pos
            if event.key == pygame.K_DOWN:
                new_pos = [block_pos[0], block_pos[1] + 1]
                if not check_collision(grid, current_block, new_pos):
                    block_pos = new_pos
            if event.key == pygame.K_UP:
                rotated_block = rotate_block(current_block)
                if not check_collision(grid, rotated_block, block_pos):
                    current_block = rotated_block

    if not game_over:
        # Mover bloque hacia abajo
        fall_time += clock.get_rawtime()
        clock.tick()
        if fall_time / 1000 > (0.5 - (level - 1) * 0.05):
            fall_time = 0
            new_pos = [block_pos[0], block_pos[1] + 1]
            if not check_collision(grid, current_block, new_pos):
                block_pos = new_pos
            else:
                join_grid(grid, current_block, block_pos)
                clear_rows(grid)
                current_block = new_block()
                block_pos = [grid_width // 2 - len(current_block[0]) // 2, 0]
                if check_collision(grid, current_block, block_pos):
                    game_over_screen()
                    game_over = True

    # Dibujar bloque actual
    for y, row in enumerate(current_block):
        for x, cell in enumerate(row):
            if cell:
                pygame.draw.rect(screen, cell, (block_pos[0] * cell_size + x * cell_size, block_pos[1] * cell_size + y * cell_size, cell_size, cell_size))

    # Dibujar puntaje y nivel
    draw_text(f"Score: {score}", font, WHITE, screen, WIDTH // 2, 20)
    draw_text(f"Level: {level}", font, WHITE, screen, WIDTH // 2, 50)

    # Verificar victoria de ronda
    if lines_cleared >= 10 and not game_over:
        current_round += 1
        if current_round > rounds:
            victory_screen()
            game_over = True
        else:
            reset_game()

    # Actualizar pantalla
    pygame.display.flip()

pygame.quit()
sys.exit()
