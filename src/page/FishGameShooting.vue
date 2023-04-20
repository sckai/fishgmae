<script setup lang="ts">
import { onMounted } from 'vue'

let canvas: HTMLCanvasElement
let ctx: CanvasRenderingContext2D
let game: Game
let lastTime: number

onMounted(() => {
  Init()
})

// Init
const Init = () => {
  canvas = document.querySelector('.canvasGame') as HTMLCanvasElement
  ctx = canvas.getContext('2d') as CanvasRenderingContext2D
  canvas.width = 1000
  canvas.height = 500
  game = new Game(canvas.width, canvas.height)
  lastTime = 0
  Animate(0)
}

// function
const Animate = (timeStamp: number) => {
  const deltaTime = timeStamp - lastTime
  lastTime = timeStamp
  ctx.clearRect(0, 0, canvas.width, canvas.height)
  game.draw(ctx)
  game.update(deltaTime)
  requestAnimationFrame(Animate)
}

class InputHandler {
  private game

  constructor(game: any) {
    this.game = game
    window.addEventListener('keydown', (e) => {
      if ((e.key === 'ArrowUp' || e.key === 'ArrowDown') && this.game.keys.indexOf(e.key) === -1) {
        this.game.keys.push(e.key)
      } else if (e.key === 'd') {
        this.game.debug = !this.game.debug
      }
      else if (e.key === ' ') {
        this.game.player.shootTop()
      }  
    })
    window.addEventListener('keyup', (e) => {
      if (this.game.keys.indexOf(e.key) > -1) this.game.keys.splice(this.game.keys.indexOf(e.key), 1)
    })
  }
}

class SoundController {
  private powerUpSound: any
  private powerDownSound: any
  private explosionSound: any
  private shotSound:any
  private hitSound: any
  private shieldSound: any

  constructor() {
    this.powerUpSound = document.querySelector('.powerup') as HTMLCanvasElement
    this.powerDownSound = document.querySelector('.powerdown') as HTMLCanvasElement
    this.explosionSound = document.querySelector('.explosion') as HTMLCanvasElement
    this.shotSound = document.querySelector('.shot') as HTMLCanvasElement
    this.hitSound = document.querySelector('.hit') as HTMLCanvasElement
    this.shieldSound = document.querySelector('.shieldSound') as HTMLCanvasElement
  }
  public powerUp() {
    this.powerUpSound.currentTime = 0
    this.powerUpSound.play()
  }
  public powerDown() {
    this.powerDownSound.currentTime = 0
    this.powerDownSound.play()
  }
  public explosion() {
    this.explosionSound.currentTime = 0
    this.explosionSound.play()
  }
  public shot() {
    this.shotSound.currentTime = 0
    this.shotSound.play()
  }
  public hit() {
    this.hitSound.currentTime = 0
    this.hitSound.play()
  }
  public shield() {
    this.shieldSound.currentTime = 0
    this.shieldSound.play()
  }
}

class Shield {
  private game
  private width: number
  private height: number
  private frameX: number
  private maxFrame: number
  private image
  private fps: number
  private timer: number
  private interval: number

  constructor(game: any) {
    this.game = game
    this.width = this.game.player.width
    this.height = this.game.player.height
    this.frameX = 0
    this.maxFrame = 24
    this.image = document.querySelector('.shield') as HTMLCanvasElement
    this.fps = 30
    this.timer = 0
    this.interval = 1000/30
  }
  public update(deltaTime: number) {
    if (this.frameX <= this.maxFrame) {
      if (this.timer > this.interval) {
        this.frameX++
        this.timer = 0
      } else {
        this.timer += deltaTime
      }
    }
  }
  public draw(context: { drawImage: (arg0: any, arg1: number, arg2: number, arg3: number, arg4: number, arg5: number, arg6: number,
    arg7: number, arg8: number) => void }) {
    context.drawImage(this.image, this.frameX * this.width, 0, this.width, this.height, this.game.player.x, this.game.player.y, this.width, this.height)
  }
  public reset(){
    this.frameX = 0
    this.game.sound.shield()
  }
}

class Projectile {
  private game
  private x: number
  private y: number
  private width: number
  private height: number
  private speed: number
  private markedForDeletion: boolean
  private image
  private frameX: number
  private maxFrame: number
  private fps: number
  private timer: number
  private interval: number

  constructor(game: any, x: number, y: number) {
    this.game = game
    this.x = x
    this.y = y
    this.width = 36.25
    this.height = 20
    this.speed = Math.random() * 0.2 + 2.8
    this.markedForDeletion = false
    this.image = document.querySelector('.fireball') as HTMLCanvasElement
    this.frameX = 0
    this.maxFrame = 3
    this.fps = 20
    this.timer = 0
    this.interval = 1000/this.fps
  }
  public update(deltaTime: number): void {
    this.x += this.speed
    if (this.timer > this.interval) {
      if (this.frameX < this.maxFrame) this.frameX++
      else this.frameX = 0
      this.timer = 0
    } else {
      this.timer += deltaTime
    }
    if (this.x > this.game.width * 0.8) this.markedForDeletion = true
  }
  public draw(context: { fillStyle: string, drawImage: (arg0: any, arg1: number, arg2: number, arg3: number, arg4: number, arg5: number, arg6: number,
    arg7: number, arg8: number) => void }): void {
    context.drawImage(this.image, this.frameX * this.width, 0, this.width, this.height, this.x, this.y, this.width, this.height)
  }
}

class Particle {
  private game
  private x: number
  private y: number
  private image
  private frameX: number
  private frameY: number
  private spriteSize: number
  private sizeModifier: any
  private size: number
  private speedX: number
  private speedY: number
  private gravity: number
  public markedForDeletion: boolean
  private angle: number
  private va: number
  private bounced: number
  private bottomBounceBoundary: number

  constructor(game: any, x: number, y: number) {
    this.game = game
    this.x = x
    this.y = y
    this.image = document.querySelector('.gears') as HTMLCanvasElement
    this.frameX = Math.floor(Math.random() * 3)
    this.frameY = Math.floor(Math.random() * 3)
    this.spriteSize = 50
    this.sizeModifier = parseFloat((Math.random() * 0.5 + 0.5).toFixed(1))
    this.size = this.spriteSize * this.sizeModifier
    this.speedX = Math.random() * 6 - 3
    this.speedY = Math.random() * -15
    this.gravity = 0.5
    this.markedForDeletion = false
    this.angle = 0
    this.va = Math.random() * 0.2 - 0.1
    this.bounced = 0
    this.bottomBounceBoundary = Math.random() * 80 + 60
  }
  public update(): void {
    this.angle += this.va
    this.speedY += this.gravity
    this.x -= this.speedX
    this.y += this.speedY
    if (this.y > this.game.height + this.size || this.x < 0 - this.size) this.markedForDeletion = true
    if (this.y > this.game.height - this.bottomBounceBoundary && this.bounced < 2) {
      this.bounced++
      this.speedY *= -0.7
    }
  }

  public draw(context: { save: any, translate: (arg0: number, arg1: number) => void, drawImage: (arg0: any, arg1: number, arg2: number, arg3: number, arg4: number, arg5: number, arg6: number, arg7: number, arg8: number) => void,
    rotate: (arg0: number) => void, restore: any }): void {
    context.save()
    context.translate(this.x, this.y)
    context.rotate(this.angle)
    context.drawImage(this.image, this.frameX * this.spriteSize, this.frameY * this.spriteSize, this.spriteSize, this.spriteSize, this.size * -0.5, this.size * - 0.5, this.size, this.size)
    context.restore()
  }
}

class Player {
  private game
  public width: number
  public height: number
  public x: number
  public y: number
  private frameX: number
  private frameY: number
  private maxFrame: number
  private speedY: number
  private maxSpeed: number
  public projectiles: (number|any)[]
  private image
  private powerUp: boolean
  private powerUpTimer: number
  private powerUpLimit: number

  constructor(game: any) {
    this.game = game
    this.width = 120
    this.height = 190
    this.x = 20
    this.y = 100
    this.frameX = 0
    this.frameY = 0
    this.maxFrame = 37
    this.speedY = 0.2
    this.maxSpeed = 3
    this.projectiles = []
    this.image = document.querySelector('.player') as HTMLCanvasElement
    this.powerUp = false
    this.powerUpTimer = 0
    this.powerUpLimit = 10000
  }
  public update(deltaTime: number) {
    if (this.game.keys.includes('ArrowUp')) this.speedY = -this.maxSpeed
    else if (this.game.keys.includes('ArrowDown')) this.speedY = this.maxSpeed
    else this.speedY = 0
    this.y = this.speedY + this.y
    
    // vertical boundaries
    if (this.y > this.game.height - this.height * 0.5) this.y = this.game.height - this.height * 0.5
    else if (this.y < -this.height * 0.5) this.y = -this.height * 0.5

    // handle projectiles
    this.projectiles.forEach(projectile => {
      projectile.update(deltaTime)
    })
    this.projectiles = this.projectiles.filter(projectile => !projectile.markedForDeletion)

    // sprite animation
    if (this.frameX < this.maxFrame) this.frameX++
    else this.frameX = 0

    // power up
    if (this.powerUp) {
      if (this.powerUpTimer > this.powerUpLimit) {
        this.powerUpTimer = 0
        this.powerUp = false
        this.frameY = 0
        this.game.sound.powerDown()
      } else {
        this.powerUpTimer += deltaTime
        this.frameY = 1
        this.game.ammo += 0.1
      }
    }
  }
  public draw(context: { strokeRect: (arg0: number, arg1: number, arg2: number, arg3: number) => void,
    drawImage: (arg0: any, arg1: number, arg2: number, arg3: number, arg4: number, arg5: number, arg6: number, arg7: number, arg8: number) => void }) {
    if (this.game.debug) context.strokeRect(this.x, this.y, this.width, this.height)
    this.projectiles.forEach(projectile => {
      projectile.draw(context)
    })
    context.drawImage(this.image, this.frameX * this.width, this.frameY * this.height, this.width, this.height, this.x, this.y, this.width, this.height)
  }
  public shootTop(): void {
    if (this.game.ammo > 0) {
      this.projectiles.push(new Projectile(this.game, this.x + 80, this.y + 30))
      this.game.ammo--
    }
    this.game.sound.shot()
    if (this.powerUp) this.shootBottom()
  }
  private shootBottom(): void {
    if (this.game.ammo > 0) this.projectiles.push(new Projectile(this.game, this.x + 80, this.y + 175))
  }
  public enterPowerUp(): void {
    this.powerUpTimer = 0
    this.powerUp = true
    if (this.game.ammo < this.game.maxAmmo) this.game.ammo = this.game.maxAmmo
    this.game.sound.powerUp()
  }
}

class Enemy {
  private game
  public width!: number
  public height!: number
  private x: number
  public y!: number
  public speedX: number
  private markedForDeletion: boolean
  public lives!: number
  public score!: number
  public image!: any
  private frameX: number
  public frameY!: number
  private maxFrame: number

  constructor(game: any) {
    this.game = game
    this.x = this.game.width
    this.speedX = Math.random() * -1.5 - 0.5
    this.markedForDeletion = false
    this.frameX = 0
    this.maxFrame = 37

  }
  public update(): void {
    this.x += this.speedX - this.game.speed
    if (this.x + this.width < 0) this.markedForDeletion = true
    if (this.frameX < this.maxFrame) this.frameX++
    else this.frameX = 0
  }
  public draw(context: { fillStyle: string, strokeRect: (arg0: number, arg1: number, arg2: number, arg3: number) => void, font: string, fillText: (arg0: number, arg1: number, arg2: number) => void,
    drawImage: (arg0: any, arg1: number, arg2: number, arg3: number, arg4: number, arg5: number, arg6: number, arg7: number, arg8: number) => void }): void {
    if (this.game.debug) context.strokeRect(this.x, this.y, this.width, this.height)
    context.drawImage(this.image, this.frameX * this.width, this.frameY * this.height, this.width, this.height, this.x, this.y, this.width, this.height)
    if (this.game.debug) {
      context.font = '20px Helvetica'
      context.fillText(this.lives, this.x, this.y)
    }
  }
}

class Angler1 extends Enemy {
  public width: number
  public height: number
  public y: number
  public image
  public lives: number
  public score: number
  
  constructor(game: any) {
    super(game)
    this.width = 228
    this.height = 169
    this.y = Math.random() * (game.height * 0.9 - this.height)
    this.image = document.querySelector('.angler1') as HTMLCanvasElement
    this.frameY = Math.floor(Math.random() * 3)
    this.lives = 5
    this.score = this.lives
  }
}

class Angler2 extends Enemy {
  public width: number
  public height: number
  public y: number
  public image
  public lives: number
  public score: number
  
  constructor(game: any) {
    super(game)
    this.width = 213
    this.height = 165
    this.y = Math.random() * (game.height * 0.9 - this.height)
    this.image = document.querySelector('.angler2') as HTMLCanvasElement
    this.frameY = Math.floor(Math.random() * 2)
    this.lives = 6
    this.score = this.lives
  }
}

class LuckyFish extends Enemy {
  public width: number
  public height: number
  public y: number
  public image
  public frameY: number
  public lives: number
  public score: number
  public type: string
  
  constructor(game: any) {
    super(game)
    this.width = 99
    this.height = 95
    this.y = Math.random() * (game.height * 0.95 - this.height)
    this.image = document.querySelector('.luckyFish') as HTMLCanvasElement
    this.frameY = Math.floor(Math.random() * 2)
    this.lives = 5
    this.score = 15
    this.type = 'lucky'
  }
}

class HiveWhale extends Enemy {
  public width: number
  public height: number
  public y: number
  public image
  public frameY: number
  public lives: number
  public score: number
  public type: string
  public speedX: number
  
  constructor(game: any) {
    super(game)
    this.width = 400
    this.height = 227
    this.y = Math.random() * (game.height * 0.95 - this.height)
    this.image = document.querySelector('.hivewhale') as HTMLCanvasElement
    this.frameY = 0
    this.lives = 20
    this.score = this.lives
    this.type = 'hive'
    this.speedX = Math.random() * -1.2 - 0.2
  }
}

class BulbWhale extends Enemy {
  public width: number
  public height: number
  public y: number
  public image
  public frameY: number
  public lives: number
  public score: number
  public speedX: number
  
  constructor(game: any) {
    super(game)
    this.width = 270
    this.height = 219
    this.y = Math.random() * (game.height * 0.95 - this.height)
    this.image = document.querySelector('.bulbwhale') as HTMLCanvasElement
    this.frameY = Math.floor(Math.random() * 2)
    this.lives = 20
    this.score = this.lives
    this.speedX = Math.random() * -1.2 - 0.2
  }
}

class MoonFish extends Enemy {
  public width: number
  public height: number
  public y: number
  public image
  public frameY: number
  public lives: number
  public score: number
  public speedX: number
  public type: string
  
  constructor(game: any) {
    super(game)
    this.width = 227
    this.height = 240
    this.y = Math.random() * (game.height * 0.95 - this.height)
    this.image = document.querySelector('.moonfish') as HTMLCanvasElement
    this.frameY = 0
    this.lives = 10
    this.score = this.lives
    this.speedX = Math.random() * -1.2 - 2
    this.type = 'moon'
  }
}

class Drone extends Enemy {
  public width: number
  public height: number
  public y: number
  public image
  public frameY: number
  public lives: number
  public score: number
  public type: string
  public speedX: number
  
  constructor(game: any, x: number, y: number) {
    super(game)
    this.width = 115
    this.height = 95
    this.y = y
    this.image = document.querySelector('.drone') as HTMLCanvasElement
    this.frameY = Math.floor(Math.random() * 2)
    this.lives = 3
    this.score = this.lives
    this.type = 'drone'
    this.speedX = Math.random() * -4.2 - 0.5
  }
}

class Layer {
  private game
  private image: string
  private speedModifier: number
  private width: number
  private height: number
  private x: number
  private y: number

  constructor(game: any, image: any, speedModifier: number) {
    this.game = game
    this.image = image
    this.speedModifier = speedModifier
    this.width = 1768
    this.height = 500
    this.x = 0
    this.y =0
  }
  public update() {
    if (this.x <= -this.width) this.x = 0
    else this.x -= this.game.speed * this.speedModifier
  }
  public draw(context: { drawImage: (arg0: string, arg1: number, arg2: number) => void }) {
    context.drawImage(this.image, this.x, this.y)
    context.drawImage(this.image, this.x + this.width, this.y)
  }
}

class Background {
  private game
  private image1
  private image2
  private image3
  private image4
  private layer1
  private layer2
  private layer3
  public layer4
  private layers

  constructor(game: any) {
    this.game = game
    this.image1 = document.querySelector('.layer1') as HTMLCanvasElement
    this.image2 = document.querySelector('.layer2') as HTMLCanvasElement
    this.image3 = document.querySelector('.layer3') as HTMLCanvasElement
    this.image4 = document.querySelector('.layer4') as HTMLCanvasElement
    this.layer1 = new Layer(this.game, this.image1, 0.2)
    this.layer2 = new Layer(this.game, this.image2, 0.4)
    this.layer3 = new Layer(this.game, this.image3, 1)
    this.layer4 = new Layer(this.game, this.image4, 1.5)
    this.layers = [this.layer1, this.layer2, this.layer3]
  }
  public update() {
    this.layers.forEach(layer => layer.update())
  }
  public draw(context: any) {
    this.layers.forEach(layer => layer.draw(context))
  }
}

class Explosion {
  private game
  private x: number
  private y: number
  private frameX: number
  private spriteHeight: number
  private spriteWidth: number
  private fps:number
  private timer: number
  private interval: number
  private markedForDeletion: boolean
  private maxFrame: number
  public image!: any
  private width: number
  private height: number

  constructor(game: any, x: number , y:number) {
    this.game = game
    this.spriteWidth = 200
    this.spriteHeight = 200
    this.width = this.spriteWidth
    this.height = this.spriteHeight
    this.x = x - this.width * 0.5
    this.y = y - this.height * 0.5
    this.frameX = 0
    this.fps = 5
    this.timer = 0
    this.interval = 1000/this.fps
    this.markedForDeletion = false
    this.maxFrame = 8
  }
  public update(deltaTime: number) {
    this.x -= this.game.speed
    if (this.timer > this.interval) {
      this.frameX++
      this.timer = 0
    }
    else this.timer += deltaTime

    if (this.frameX > this.maxFrame) this.markedForDeletion = true
  }
  public draw(context: { drawImage: (arg0: any, arg1: number, arg2: number, arg3: number, arg4: number, arg5: number, arg6: number,
    arg7: number, arg8: number) => void }) {
    context.drawImage(this.image, this.frameX * this.spriteWidth, 0, this.spriteWidth, this.spriteHeight, this.x,
    this.y, this.width, this.height)
  }
}

class SomkeExplosion extends Explosion {
  public image: any

  constructor(game: any, x: number, y: number) {
    super(game, x, y)
    this.image = document.querySelector('.smokeExplosion') as HTMLCanvasElement
  }
}

class FireExplosion extends Explosion {
  public image: any

  constructor(game: any, x: number, y: number) {
    super(game, x, y)
    this.image = document.querySelector('.fireExplosion') as HTMLCanvasElement
  }
}

class UI {
  private game
  private fontSize: number
  private fontFamily: string
  private color: string

  constructor(game: any) {
    this.game = game
    this.fontSize = 25
    this.fontFamily = 'Bangers'
    this.color = 'white'

  }
  public draw(context: { fillStyle: string, fillRect: (arg0: number, arg1: number, arg2: number, arg3: number) => void, font: string,
    fillText: (arg0: any, arg1: number, arg2: number) => void, shadowOffsetX: number, shadowOffsetY: number, shadowColor: string,
    save: () => void, restore: () => void, textAlign: string  }
    ) {
    context.save()

    // score
    context.fillStyle = this.color
    context.shadowOffsetX = 2
    context.shadowOffsetY = 2
    context.shadowColor = 'black'
    context.font = `${this.fontSize}px ${this.fontFamily}`
    context.fillText(`Score: ${this.game.score}`, 20, 40)
    // timer
    const formattedTime = (this.game.gameTime * 0.001).toFixed(1)
    context.fillText(`Timer: ${formattedTime}`, 20, 100)
    // game over messages
    if (this.game.gameOver) {
      context.textAlign = 'center'
      let message1
      let message2
      if (this.game.score > this.game.winningScroe) {
        message1 = 'Most Wondrous!'
        message2 = 'Well done explorer!'
      } else {
        message1 = 'Blazes!'
        message2 = 'Get my repair kit and try again!'        
      }
      context.font = `70px ${this.fontFamily}`
      context.fillText(message1, this.game.width * 0.5, this.game.height * 0.5 - 40)
      context.font = `25px ${this.fontFamily}`
      context.fillText(message2, this.game.width * 0.5, this.game.height * 0.5 + 40)
    }

    // ammo
    if (this.game.player.powerUp) context.fillStyle = '#ffffbd'
    for (let index = 0; index < this.game.ammo; index++) {
      context.fillRect(20 + 5 * index, 50, 3, 20)
    }

    context.restore()
  }
}

class Game {
  private width: number
  private height: number
  private background
  private player
  private input
  private ui
  private sound
  private shield
  private keys: string[]
  private enemies: (number|any)[]
  private particles: any[]
  private explosions: any[]
  private enemyTimer: number
  private enemyInterval: number
  private ammo: number
  private maxAmmo: number
  private ammoTimer: number
  private ammoInterval: number
  private gameOver: boolean
  private score: number
  private winningScroe: number
  private gameTime: number
  private timeLimit: number
  private speed: number
  public debug: boolean

  constructor(width: number, height: number) {
    this.width = width
    this.height = height
    this.background = new Background(this)
    this.player = new Player(this)
    this.input = new InputHandler(this)
    this.ui = new UI(this)
    this.sound = new SoundController()
    this.shield = new Shield(this)
    this.keys = []
    this.enemies = []
    this.particles = []
    this.explosions = []
    this.enemyTimer = 0
    this.enemyInterval = 2000
    this.ammo = 20
    this.maxAmmo = 30
    this.ammoTimer = 0
    this.ammoInterval = 350
    this.gameOver = false
    this.score = 0
    this.winningScroe = 300
    this.gameTime = 0
    this.timeLimit = 3000000
    this.speed = 1
    this.debug = true
  }
  public update(deltaTime: number): void {
    if (!this.gameOver) this.gameTime += deltaTime
    if (this.gameTime > this.timeLimit) this.gameOver = true
    this.background.update()
    this.background.layer4.update()
    this.player.update(deltaTime)
    if (this.ammoTimer > this.ammoInterval) {
      if (this.ammo < this.maxAmmo) this.ammo++
      this.ammoTimer = 0
    } else {
      this.ammoTimer += deltaTime
    }

    this.shield.update(deltaTime)

    this.particles.forEach(particle => particle.update())
    this.particles = this.particles.filter(particle => !particle.markedForDeletion)

    this.explosions.forEach(explosion => explosion.update(deltaTime))
    this.explosions = this.explosions.filter(explosion => !explosion.markedForDeletion)

    this.enemies.forEach(enemy => {
      enemy.update()
      if (this.checkCollision(this.player, enemy)) {
        enemy.markedForDeletion = true
        this.addExplosion(enemy)
        this.sound.hit()
        this.shield.reset()
        for (let index = 0; index < enemy.score; index++) {
          this.particles.push(new Particle(this, enemy.x + enemy.width * 0.5, enemy.y + enemy.height * 0.5))
        }
        if (enemy.type === 'lucky') this.player.enterPowerUp()
        else if (!this.gameOver) this.score
      }

      this.player.projectiles.forEach(projectile =>{
        if (this.checkCollision(projectile, enemy)) {
          enemy.lives--
          projectile.markedForDeletion = true
          this.particles.push(new Particle(this, enemy.x + enemy.width * 0.5, enemy.y + enemy.height * 0.5))
          if (enemy.lives <= 0) {
            for (let index = 0; index < enemy.score; index++) {
              this.particles.push(new Particle(this, enemy.x + enemy.width * 0.5, enemy.y + enemy.height * 0.5))
            }
            enemy.markedForDeletion = true
            this.addExplosion(enemy)
            this.sound.explosion()
            if (enemy.type === 'moon') this.player.enterPowerUp()
            if (enemy.type === 'hive') {
              for(let index = 0; index < 5; index++) {
                this.enemies.push(new Drone(this, enemy.x + Math.random() * enemy.width, enemy.y + Math.random() * enemy.height * 0.5))
              }
            }
            if (!this.gameOver) this.score += enemy.score
            // if (this.score > this.winningScroe) this.gameOver = true
          }
        }
      })
    })
    this.enemies = this.enemies.filter(enemy  => !enemy.markedForDeletion)
    if (this.enemyTimer > this.enemyInterval && !this.gameOver) {
      this.addEnemy()
      this.enemyTimer = 0
    } else {
      this.enemyTimer += deltaTime
    }
  }
  public draw(context: any): void {
    this.background.draw(context)
    this.ui.draw(context)
    this.player.draw(context)
    this.shield.draw(context)
    this.particles.forEach(particle => particle.draw(context))
    this.enemies.forEach(enemy => {
      enemy.draw(context)
    })
    this.explosions.forEach(explosion => {
      explosion.draw(context)
    })
    this.background.layer4.draw(context)
  }
  public addEnemy() {
    const randomize = Math.random()
    if (randomize < 0.3) this.enemies.push(new Angler1(this))
    else if (randomize < 0.6) this.enemies.push(new Angler2(this))
    else if (randomize < 0.7) this.enemies.push(new HiveWhale(this))
    else if (randomize < 0.8) this.enemies.push(new BulbWhale(this))
    else if (randomize < 0.9) this.enemies.push(new MoonFish(this))
    else this.enemies.push(new LuckyFish(this))
  }
  public addExplosion(enemy: any) {
    const randomize = Math.random()
    if (randomize < 0.5) this.explosions.push(new SomkeExplosion(this, enemy.x + enemy.width * 0.5, enemy.y + enemy.height * 0.5))
    else  this.explosions.push(new FireExplosion(this, enemy.x + enemy.width * 0.5, enemy.y + enemy.height * 0.5))
  }
  public checkCollision(rect1: { x: number, y: number, width: number, height: number }, rect2: { x: number, y: number, width: number, height: number }): boolean {
    return (rect1.x < rect2.x + rect2.width &&
        rect1.x + rect1.width > rect2.x &&
        rect1.y < rect2.y + rect2.height &&
        rect1.height + rect1.y > rect2.y)
  }
}
</script>

<template lang="pug">
#FishGameShooting
  canvas(class="canvasGame")

  //- characters
  img(class="player" src="@/assets/images/characters/player.png", alt="layer1")
  img(class="angler1" src="@/assets/images/characters/angler1.png", alt="angler1")
  img(class="angler2" src="@/assets/images/characters/angler2.png", alt="angler2")
  img(class="luckyFish" src="@/assets/images/characters/lucky.png", alt="luckyFish")
  img(class="hivewhale" src="@/assets/images/characters/hivewhale.png", alt="hivewhale")
  img(class="drone" src="@/assets/images/characters/drone.png", alt="drone")
  img(class="bulbwhale" src="@/assets/images/characters/bulbwhale.png", alt="bulbwhale")
  img(class="moonfish" src="@/assets/images/characters/moonfish.png", alt="moonfish")

  //- props
  //- img(class="projectile" src="@/assets/images/props/projectile.png", alt="projectile")
  img(class="gears" src="@/assets/images/props/gears.png", alt="gears")
  img(class="smokeExplosion" src="@/assets/images/props/smokeExplosion.png", alt="smokeExplosion")
  img(class="fireExplosion" src="@/assets/images/props/fireExplosion.png", alt="fireExplosion")
  img(class="shield" src="@/assets/images/props/shield.png", alt="shield")
  img(class="fireball" src="@/assets/images/props/fireball.png", alt="fireball")

  //- environment
  img(class="layer1" src="@/assets/images/environment/layer1.png", alt="layer1")
  img(class="layer2" src="@/assets/images/environment/layer2.png", alt="layer2")
  img(class="layer3" src="@/assets/images/environment/layer3.png", alt="layer3")
  img(class="layer4" src="@/assets/images/environment/layer4.png", alt="layer4")

  //- sounds
  audio(class="powerup" src="src/assets/sounds/powerup.wav")
  audio(class="powerdown" src="src/assets/sounds/powerdown.wav")
  audio(class="explosion" src="src/assets/sounds/explosion.wav")
  audio(class="shot" src="src/assets/sounds/shot.wav")
  audio(class="hit" src="src/assets/sounds/hit.wav")
  audio(class="shieldSound" src="src/assets/sounds/shield.wav")
</template>

<style lang="scss" scoped>
#FishGameShooting {
  font-family: 'Bangers', cursive;
  & .canvasGame {
    border: 4px solid black;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background: #4d79bc;
    max-width: 100%;
    max-height: 100%;
    box-sizing: border-box;
  }
  & .layer1, .layer2, .layer3, .layer4, .player, .angler1, .angler2,
  .luckyFish, .projectile, .gears, .hivewhale, .drone, .smokeExplosion, .fireExplosion,
  .bulbwhale, .moonfish, .shield, .fireball {
    display: none;
  }
}
</style>