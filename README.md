# ShooterGame
from pygame import *
from random import randint
import time as t
#If we want to use time, we should use t.time()


class GameSprite(sprite.Sprite):
   def __init__(self, sprite_image, sprite_x, sprite_y, sprite_speed):
       super().__init__()
       self.image = transform.scale(image.load(sprite_image), (65, 65))
       self.speed = sprite_speed
       self.rect = self.image.get_rect()
       self.rect.x = sprite_x
       self.rect.y = sprite_y

   def reset(self):
       window.blit(self.image, (self.rect.x, self.rect.y))


class Player(GameSprite):
   def update(self):
       keys_pressed = key.get_pressed()
       if keys_pressed[K_LEFT] and self.rect.x > 0:
           self.rect.x -= self.speed
       if keys_pressed[K_RIGHT] and self.rect.x < 700 - self.rect.width:
           self.rect.x += self.speed

   def fire(self):
       global bullet_count
       x = self.rect.centerx - 40
       y = self.rect.y
       bullet = Bullet("bullet.png", x, y, 5)
       bullets.add(bullet)
       bullet_count -= 1


class Enemy(GameSprite):
   def update(self):
       global missed
       self.rect.y += self.speed
       if self.rect.y >= 500:
           x = randint(0, 635)
           y = -65
           self.rect.x = x
           self.rect.y = y
           self.speed = randint(2, 4)
           missed += 1



class Bullet(GameSprite):
   def update(self):
       self.rect.y -= self.speed
       if self.rect.y <= 0:
           self.kill()




class Powerup(GameSprite):
   def __init__(self, sprite_image, sprite_x, sprite_y, sprite_speed, power):
       super().__init__(sprite_image, sprite_x, sprite_y, sprite_speed)
       self.power = power

   def goodBarrage(self):
       global missed
       missed = 0
       x = 15
       for i in range(10):
           bullet = Bullet("bullet.png", x, 500, 5)  # CHANGE IMAGE FILE NAME HERE
           bullets.add(bullet)
           x += 70

   def badBarrage(self):
       for enemy in monsters:
           enemy.speed = 8

   def update(self):
       self.rect.y += self.speed
       if self.rect.y >= 500:
           self.kill()


window = display.set_mode((700, 500))
display.set_caption("Shooter Game")
clock = time.Clock()
background = transform.scale(image.load("galaxy.jpg"), (700, 500))  # CHANGE IMAGE FILE NAME HERE
mixer.init()
font.init()
style = font.SysFont(Arial, 36)
bigText = font.SysFont(Arial, 60)


win_text = bigText.render("You Win", True, (0, 255, 0))
lose_text = bigText.render("Game Over", True, (255, 0, 0))
reload_text = style.render("Reloading...", True, (255, 0, 0))

player = Player("rocket.png", 320, 415, 5)  # CHANGE IMAGE FILE NAME HERE

monsters = sprite.Group()

for i in range(5):
   enemy = Enemy("ufo.png", randint(0, 635), randint(-60, -10), randint(2, 4))  # CHANGE IMAGE FILE NAME HERE
   monsters.add(enemy)
bullets = sprite.Group()
powerups = sprite.Group()
missed = 0
score = 0
bullet_count = 5
RELOAD_TIME = 3
isRunning = True
finished = False
start_time = 0
while isRunning:
   if not finished:
       events = event.get()
       for e in events:
           if e.type == QUIT:
               isRunning = False
           if e.type == KEYDOWN:
               if e.key == K_SPACE:
                   if bullet_count > 0:
                       player.fire()
                   if bullet_count == 0:
                       start_time = t.time()


       window.blit(background, (0, 0))
       bullets.draw(window)
       player.reset()
       monsters.draw(window)
       powerups.draw(window)
       text_lose = style.render("Missed: " + str(missed), 1, (255, 255, 255))
       score_lose = style.render("score: " + str(score), 1, (255, 255, 255))
       window.blit(text_lose, (10, 20))
       window.blit(score_lose, (10, 0))

       player.update()
       monsters.update()
       bullets.update()
       powerups.update()

       if randint(1, 100) == 23:
           powerup = Powerup("bubble bullet.png", randint(0, 700), 0, 5, "goodBarrage")  # CHANGE IMAGE FILE NAME HERE
           powerups.add(powerup)

       if randint(1, 100) == 13:
           powerup = Powerup("fire asteroid.png", randint(0, 700), 0, 5,
                             "badBarrage")  # CHANGE IMAGE FILE NAME HERE
           powerups.add(powerup)

       # Collision
       collided_powerups = sprite.spritecollide(player, powerups, True)
       for power in collided_powerups:
           if power.power == "goodBarrage":
               power.goodBarrage()
           elif power.power == "badBarrage":
               power.badBarrage()

       sprites_list = sprite.groupcollide(monsters, bullets, True, True)
       for enemy in sprites_list:
           enemy.rect.x = randint(0, 635)
           enemy.rect.y = randint(-60, -10)
           enemy.speed = randint(2, 4)
           monsters.add(enemy)
           score += 1
       #Reloading
       if t.time() - start_time >= RELOAD_TIME and start_time != 0:
           bullet_count = 5
           start_time = 0
       if bullet_count == 0:
           window.blit(reload_text, (200, 470))
       # Winning and Losing
       if score >= 10:
           window.blit(win_text, (200, 100))
           finished = True
       
       elif missed >= 300 or sprite.spritecollide(player, monsters, False):
           window.blit(lose_text, (200, 100))
           finished = True
       
   else:
       for e in event.get():
           if e.type == QUIT:
               isRunning = False

   display.update()
   clock.tick(60)
