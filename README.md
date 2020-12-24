import pygame
import random

class Ball(pygame.sprite.Sprite):
    def __init__(self, paddle):
        pygame.sprite.Sprite.__init__(self)
        self.paddle = paddle
        self.size = 17
        self.end_game_flag = False
        self.image = pygame.Surface((self.size, self.size))
        pygame.draw.circle(self.image, GREEN, (self.size//2 , self.size//2), self.size // 2)
        #self.image.fill(GREEN)
        self.rect = self.image.get_rect()
        self.rect.center = (WIDTH // 2, HEIGHT // 2)
        self.rect.center = (random.randint(int(0.2*WIDTH), int(0.8*WIDTH)), random.randint(HEIGHT//2, int(HEIGHT*0.8)))
        self.step = 5
        self.x_direction = 'right'
        self.y_direction = 'top'
        self.message = ''


    def update(self):
        if self.rect.y + self.size >= HEIGHT:
            self.message = "Game Over"
            self.end_game_flag = True

        if self.rect.colliderect(paddle.rect):
            self.y_direction = 'top'
        # if self.rect.y + self.size >= self.paddle.rect.y:
        #     if self.rect.x < self.paddle.rect.x +self.paddle.rect.w and self.rect.x + self.size > self.paddle.rect.x:
                

        if self.rect.x + self.size >= WIDTH:
            self.x_direction = 'left'
        elif self.rect.x <= 0:
            self.x_direction = 'right'

        if self.x_direction == 'right':
            self.rect.x = self.rect.x + self.step
        elif self.x_direction == 'left':
            self.rect.x = self.rect.x - self.step

        if self.rect.y <= 0:
            self.y_direction = 'down'

        if self.y_direction == 'down':
            self.rect.y = self.rect.y + self.step
        elif self.y_direction == 'top':
            self.rect.y = self.rect.y - self.step 

    def change_y_direction(self):
        if self.y_direction == 'down':
            self.y_direction = 'top'
        else:
            self.y_direction = 'down'
    
    def change_x_direction(self):
        if self.x_direction == 'right':
            self.x_direction = 'left'
        else:
            self.x_direction = 'right'
    
    def win(self):
        self.message = "!!!! YOU WIN !!!!"
        self.end_game_flag = True
        
            
class Paddle(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.h = 20
        self.w = 100
        self.image = pygame.Surface((self.w, self.h))
        self.image.fill((255,0,0))
        self.rect = self.image.get_rect()
        self.rect.x = WIDTH // 2
        self.rect.y = HEIGHT - self.h
        self.step = 15
        self.x_direction = 'right'

    def update(self):
        step = self.step
        if self.rect.x + self.w >= WIDTH and self.x_direction == 'right':
            step = 0
        elif self.rect.x <= 0 and self.x_direction == 'left':
            step = 0

        if self.x_direction == 'right':
            self.rect.x = self.rect.x + step
        elif self.x_direction == 'left':
            self.rect.x = self.rect.x - step
        self.x_direction = ''


class Block(pygame.sprite.Sprite):
    def __init__(self, x, y, size):
        pygame.sprite.Sprite.__init__(self)
        self.size = size
        self.image = pygame.Surface((self.size, self.size))
        self.image.fill((127,0,127))
        self.rect = self.image.get_rect()
        self.rect.center = (x + size // 2, y + size // 2)
        self.mortal = False
    
    def update(self):

            self.mortal = True
    
def get_score_text(score):
    return "Score: " +str(score)

def drawTextInCenter(screen, text, COLOR = (255,255,255), font_size = 24):
    font = pygame.font.SysFont(None, font_size)
    textSurface = font.render(text, True, COLOR)
    x = int(screen.get_width()*0.5 - textSurface.get_width()*0.5)
    y = int(screen.get_height()*0.5 - textSurface.get_height()*0.5)
    return textSurface, (x,y)
    

WIDTH = 1500
HEIGHT= 800
BASE_FPS = 60

BLACK=(0,0,0)
GREEN = (0,255,0)
SCORE_COLOR= (255,255,0)

## GET FROM https://www.pygame.org/wiki/SettingWindowPosition
x = 100
y = 50
import os
os.environ['SDL_VIDEO_WINDOW_POS'] = "%d,%d" % (x,y)

pygame.init()
pygame.mixer.init()
pygame.font.init()

screen = pygame.display.set_mode((WIDTH, HEIGHT))

pygame.display.set_caption("MEGA ARCANOID by Dan 'helper' Kuznetsov")
clock = pygame.time.Clock()
game_quit = False
while not game_quit:
    all_sprites = pygame.sprite.Group()
    blocks_sprites = pygame.sprite.Group()
    paddle = Paddle()
    all_sprites.add(paddle)
    ball = Ball(paddle)
    all_sprites.add(ball)

    block_size = 21

    block_start_y = block_size*6
    block_end_y = block_size*6+(block_size+1)*5

    for block_y in range(block_start_y, block_end_y, block_size+1):
        empty_blocks = random.randint(0, 10)
        block_start_x = empty_blocks * (block_size+1)
        block_end_x = WIDTH - empty_blocks * (block_size+1)
        for block_x in range(block_start_x, block_end_x, block_size+1):
            blockN = Block(block_x, block_y, block_size)
            blocks_sprites.add(blockN)

    screen.fill(BLACK)
    game_score = 0
    font = pygame.font.SysFont(None, 24)
    FPS = BASE_FPS
    screen.blit(font.render(get_score_text(game_score), True, SCORE_COLOR),(0,0))

    while not ball.end_game_flag:
        clock.tick(FPS)
        for event in pygame.event.get():
            # проверить закрытие окна
            if event.type == pygame.QUIT:
                game_quit = True
                break
        keys = pygame.key.get_pressed()
        if  keys[pygame.K_ESCAPE]:
            game_quit = True
            break
        if  keys[pygame.K_LEFT]:
            paddle.x_direction = 'left'
        if keys[pygame.K_RIGHT]:
            paddle.x_direction = 'right'
        
        #UPDATE ALL SPRITES POSITIONS
        all_sprites.update()
        blocks_sprites.update()
        
        blocks_hit_list = pygame.sprite.spritecollide(ball, blocks_sprites, True)
        game_score = game_score + len(blocks_hit_list)
        if len(blocks_hit_list) > 0:
            block_hitted = blocks_hit_list[0]
            dx = abs(block_hitted.rect.centerx - ball.rect.centerx)
            dy = abs(block_hitted.rect.centery - ball.rect.centery)
            if dx < dy:
                ball.change_y_direction()
            elif dy < dx:
                ball.change_x_direction()
            elif dy == dx:
                ball.change_x_direction()
                ball.change_y_direction()
        
        #DRAW ALL SPRITES
        screen.fill(BLACK)
        screen.blit(font.render(get_score_text(game_score), True, SCORE_COLOR),(0,0))
        all_sprites.draw(screen)
        blocks_sprites.draw(screen)

        #SHOW ALL UPDATES
        pygame.display.flip()
        
        if game_score > 0 and game_score % 5 == 0:
            FPS = BASE_FPS + (game_score // 5) * 6
            print(FPS)

        if len(blocks_sprites.sprites()) == 0:
            ball.win()
            break
        
    textImg, textCoordinate = drawTextInCenter(screen, ball.message, (255,127,255), 50)
    screen.blit(textImg,textCoordinate)

    screen.blit(font.render("Press N for new game", True, (255,0,255)),(WIDTH // 2 - 60,0))
    pygame.display.flip()
    while True:
        clock.tick(FPS)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                game_quit = True
            break
        keys = pygame.key.get_pressed()
        if  keys[pygame.K_ESCAPE]:
            game_quit = True
            break
        if  keys[pygame.K_n]:
            game_quit = False
            break
pygame.quit()
