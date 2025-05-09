# Jogo-Python-Thiago-

import os
import random
import time

# Mapa do jogo ('.' = chão, '#' = parede, 'E' = inimigo, 'P' = player, 'T' = tesouro)
WIDTH = 20
HEIGHT = 10

TILE_EMPTY = '.'
TILE_WALL = '#'
TILE_PLAYER = 'P'
TILE_ENEMY = 'E'
TILE_TREASURE = 'T'

def clear():
    os.system('cls' if os.name == 'nt' else 'clear')

class Entity:
    def __init__(self, x, y, symbol, hp):
        self.x = x
        self.y = y
        self.symbol = symbol
        self.hp = hp

    def move(self, dx, dy, game_map):
        new_x = self.x + dx
        new_y = self.y + dy
        if 0 <= new_x < WIDTH and 0 <= new_y < HEIGHT:
            if game_map[new_y][new_x] == TILE_EMPTY or game_map[new_y][new_x] == TILE_TREASURE:
                game_map[self.y][self.x] = TILE_EMPTY
                self.x = new_x
                self.y = new_y
                game_map[self.y][self.x] = self.symbol

def generate_map():
    game_map = [[TILE_EMPTY for _ in range(WIDTH)] for _ in range(HEIGHT)]
    for y in range(HEIGHT):
        for x in range(WIDTH):
            if random.random() < 0.1:
                game_map[y][x] = TILE_WALL
    return game_map

def place_entity(game_map, symbol, hp=1):
    while True:
        x = random.randint(0, WIDTH - 1)
        y = random.randint(0, HEIGHT - 1)
        if game_map[y][x] == TILE_EMPTY:
            game_map[y][x] = symbol
            return Entity(x, y, symbol, hp)

def draw(game_map, player, enemies):
    clear()
    for y in range(HEIGHT):
        for x in range(WIDTH):
            print(game_map[y][x], end='')
        print()
    print(f'HP: {player.hp}')
    print(f'Enemies remaining: {len(enemies)}')

def enemy_turns(enemies, player, game_map):
    for enemy in enemies[:]:
        dx = player.x - enemy.x
        dy = player.y - enemy.y
        step_x = (1 if dx > 0 else -1) if dx != 0 else 0
        step_y = (1 if dy > 0 else -1) if dy != 0 else 0

        # Try to move closer or attack
        if abs(dx) + abs(dy) == 1:
            # Attack
            player.hp -= 1
            print("Enemy attacked you! HP -1")
            time.sleep(0.5)
        else:
            enemy.move(step_x, step_y, game_map)

def main():
    game_map = generate_map()
    player = place_entity(game_map, TILE_PLAYER, hp=10)
    treasure = place_entity(game_map, TILE_TREASURE)
    enemies = [place_entity(game_map, TILE_ENEMY, hp=1) for _ in range(5)]

    while True:
        draw(game_map, player, enemies)

        if player.hp <= 0:
            print("Você morreu! Fim de jogo.")
            break

        if player.x == treasure.x and player.y == treasure.y:
            print("Você encontrou o tesouro! Vitória!")
            break

        move = input("Mover (WASD): ").lower()
        dx = dy = 0
        if move == 'w':
            dy = -1
        elif move == 's':
            dy = 1
        elif move == 'a':
            dx = -1
        elif move == 'd':
            dx = 1
        else:
            continue

        target_x = player.x + dx
        target_y = player.y + dy

        if 0 <= target_x < WIDTH and 0 <= target_y < HEIGHT:
            target_tile = game_map[target_y][target_x]
            if target_tile == TILE_ENEMY:
                # attack enemy
                for enemy in enemies:
                    if enemy.x == target_x and enemy.y == target_y:
                        enemies.remove(enemy)
                        game_map[target_y][target_x] = TILE_EMPTY
                        print("Você derrotou um inimigo!")
                        time.sleep(0.5)
                        break
            elif target_tile in (TILE_EMPTY, TILE_TREASURE):
                player.move(dx, dy, game_map)

        enemy_turns(enemies, player, game_map)

if __name__ == '__main__':
    main()

