# Game

# Asteroids

#parameters - when closing
class Player:
    # Class constructor, ran when instantiating a class or creating an object of it
    
    # Self must be provided as a First argument to the Instance method and constructor.
    def __init__(self):
        # Self represents the current instance of the class
        # Instance attributes
        self.cor = PVector(width/2-15, height/2-15)

        # 2D vector
        # (x, y) - this vector will act on the player's coordinates every frame by a certain amount
        # player's x-coordinate would change by the x-coordinate of the self.vel tuple, and the player's y-coordinate would change by the y-coordinate of the self.vel tuple
        self.vel = PVector(0, 0)
        self.radius = 15
        
        # direction the player is facing
        self.heading = 0
        
    def show(self):
        # Simply put, stroke is the outline of whatever you draw
        stroke(255)
        # StrokeWeight is how wide the stroke (outline is)
        noFill() # Whatever's drawn is transparent inside
        
        pushMatrix() #saves the current coordinate system
        # Specifies an amount to displace objects within the display window
        translate(self.cor[0], self.cor[1])
        
        # we want to rotate the player spaceship so that it is facing the mouse pointer
        # every time this function is ran, so...
        # if you've taken geometry before, you may remember radians, degrees and main right triangle trigonmetric ratios
        # 2pi radians = 360 degrees
        # pi radians = 180
        # Sine = Opposite/Hypotenuse, Cosine = Adjacent/Hypotenuse, Tangent = Opposite/Adjacent
        # SOH CAH TOA
        # Tan = Opposite/Adjacent
        # Tan won't work for every quadrant, so you can use atan2 instead
        # Tan^-1, but it distinguishes between each quadrant so you can get the result that you want
        # self.heading is in radians in this case
        self.heading = atan2(mouseY - self.cor.y, mouseX - self.cor.x)
        #self.heading = atan2(mouseY - self.cor.y, mouseX - self.cor.x)
        #rotate(self.heading)
        # PI radians = 180 degrees
        # By the same logic, PI/2 radians = 180/2 degrees = 90 degrees
        rotate(self.heading + PI / 2)# + PI/2)
        # (self.cor.x, self.cor.y) reads as (0, 0)
        # triangle(x1, y1, x2, y2, x3, y3)
        # point(0, 0) (shows where the center is)
        triangle(-self.radius, self.radius, self.radius, self.radius, 0, -self.radius)
        popMatrix() # reverts to the previously saved coordinate system
        
    def move(self):
        self.cor.add(self.vel)
        
        
        """
        self.cor.x = self.cor.x + self.vel.x
        self.cor.y = self.cor.y + self.vel.y
        """
        # Slow down the player's velocity slightly each time this method is ran
        self.vel.x, self.vel.y = self.vel.x * .985, self.vel.y * .985
        

# we're also going to need a class for the Rock - or each asteroid
class Rock:
    def __init__(self, x, y, radius, total, vel_x, vel_y):
        self.cor = PVector(x, y)
        self.vel = PVector(vel_x, vel_y)
        self.radius = radius
        # self.total - the total numbe rof sides
        self.total = total
        
        # self.offset will store how much each of the verticies is offset from the radius
        # (how much closer or farther away from the radius it is)
        # We're gonna use a list comprehension to do this.
        self.offset = [randint(int(-self.radius * 0.5), int(self.radius * 0.5)) for i in range(self.total)]
        
    def show(self):
        stroke(255)
        noFill()
        strokeWeight(2)
        
        pushMatrix()
        translate(self.cor.x, self.cor.y)
        # Example - circle(0, 0, self.radius)
        
       # circle(0, 0, self.radius)
        
        # In processing you can also draw a shape by setting verticies and connecting polygons.
        # If you want the more jagged edges to the asteroid, you'd want to move some points further away from the center and some closer
        # Polar to Cartesian (x, y) coordinates
        # Polar coordinates - each point is an angle from the center and a distance from the center
        beginShape()
        # Let's say that the asteroid will have 10 sides
        for i in range(self.total): 
            # 2pi is 360 degrees. You want to map i - which is the current side from 0 to ten
            # to a degree measure
            # The map function Re-maps a value from one range to a new range- takes 5 arguments 
            # variable2 = map(variable1, min1, max1, min2, max2);
            # min1:variable1 = min2:variable2, min2:variable2 = min2:variable2
            # map establishes a proportion between two ranges
            # min1 is to min2 as max1 is to max2.
            # variable1 stores a value between the first range min1~max1.
            # variable2 gets a value between the second range min2~max2.
            
            # make more complex shapes in processing by connecting verticies
            
            angle = map(i, 0, self.total, 0, PI * 2)
            
            # adds the corresponding offset
            r = self.radius + self.offset[i]
            
            # Convert polar coordinates to cartesian coordinates
            # (x,y) -cartesian / polar coordinates- (r, angle) 
            x = r * cos(angle)
            y = r * sin(angle)
            
            # All shapes are constructed by connecting verticies
            
            vertex(x, y)
            
        endShape(CLOSE)
        popMatrix()
        
    def move(self):
        self.cor.add(self.vel)
        
        # If the player's x-coordinates are out of bounds:
        if self.cor.x > width + self.radius or self.cor.x < -self.radius:
            self.vel.x = -self.vel.x
        if self.cor.y > height + self.radius or self.cor.y < -self.radius:
            self.vel.y = -self.vel.y
            
    # Check whether a bullet has hit this instance of the Rock class (asteroid) and determine whether we want to break up the asteroid into two little pieces or just delete it from the asteroids list
    def check(self):
        global score 
        
        if dist(player.cor.x, player.cor.y, self.cor.x, self.cor.y) < player.radius + self.radius:
            exit()
        
        for bullet in bullets:
            # Distance formula or pythagorean theorem
            # (x1, y1, x2, y2)
            # If the bullet his hit this asteroid
            if dist(self.cor.x, self.cor.y, bullet.cor.x, bullet.cor.y) < self.radius:
                # Which means that if the distance between the coordintes of the bullet and the circle that is made by the asteroid's radius
                bullets.remove(bullet)
                score += 1

                for i in range(20):
                    particles.append(Particle(self.cor.x, self.cor.y, uniform(-.1, .1) * self.radius, uniform(-.1, .1) * self.radius, [color(6, 108, 156), color(168, 227, 255)], 150))
                
                # If the radius of the asteroid is greater than 15, we want to break it up into two new asteroids
                if self.radius > 15:
                    for i in range(-1, 1):
                        # 
                        rocks.append(Rock(self.cor.x - i * self.radius, self.cor.y + i * self.radius, self.radius / 2, randint(8, 15), randint(-100, 100) * .025, randint(-100, 100) * .025))
                        
                return True
        return False    
        
class Bullet:
    # Class constructor
    def __init__(self, x, y, vel_x, vel_y):
        self.cor = PVector(x, y)
        self.vel = PVector(vel_x, vel_y)
        
    def show(self):
        # circle takes 3 parameters: (x, y, radius)
        ellipseMode(CENTER)
        circle(self.cor.x, self.cor.y, 5)
        
    def move(self):
        self.cor.add(self.vel)
        
        if self.cor.x > width or self.cor.x < 0 or self.cor.y > height or self.cor.y < 0:
            bullets.remove(self)
    
class Particle:
    def __init__(self, x, y, vel_x, vel_y, colors, alpha = 255):
        self.cor = PVector(x, y)
        self.vel = PVector(vel_x, vel_y)
        self.color = choice(colors)
        self.alpha = alpha
        
    def show(self):
        noStroke()
        fill(self.color, self.alpha)
        
        circle(self.cor.x, self.cor.y, 12)
        
    # We're going to reduce self.alpha so that the particle becomes more and more transparent frame by frame.
    def update(self):
        self.cor.add(self.vel)
        
        if self.alpha < 0:
            # This way we can remove any particles that are transparent (or can no longer be seen in the window)
            particles.remove(self)
        
        self.alpha -= 4
         
# Runs once when the program starts
def setup():
    # global namespace - variable player can be changed everywhere in the program
    global player
    
    # window the size of width height
    size(width, height)
    
    # object of player = instance of class player with (x, y) at (width/2, height/2) (center of screen), radius, 15
    player = Player()
    
    for i in range(5):
        rocks.append(Rock(randint(0, width), randint(0, height), randint(15, 50), randint(8, 15), randint(-50, 50) * .025, randint(-50, 50) * .025))
 
# Runs continuously every frame from top to bottom until the program is stopped.
def draw(): 
    global game_started, spawn_time
       
    background(0)
    
    if game_started:
        player.show()
        player.move()
        
        # If the w key is currently being pressed:
        if keys[87]:
            particles.append(Particle(player.cor.x, player.cor.y, cos(player.heading) + uniform(-1, 1), sin(player.heading) + uniform(-1, 1), [color(241, 80, 44), color(255, 204, 0)]))
            # COS - Adjacent/Hypotenuse
            # player.heading is the current direction that the player is pointing represented in radians
            
            # Show the mathematical calculations
            player.vel.x = .85 * (player.vel.x + cos(player.heading))
            player.vel.y = .85 * (player.vel.y + sin(player.heading))
            
        rock_indexes_to_remove = []
        
        for i in range(len(rocks)):
            rocks[i].show()
            rocks[i].move()
        
            if rocks[i].check() == True:
                rock_indexes_to_remove.append(i)
                
        for index in rock_indexes_to_remove:
            del rocks[index]
            
        for bullet in bullets:
            bullet.show()
            bullet.move()
            
        for particle in particles:
            if dist(particle.cor.x, particle.cor.y, player.cor.x, player.cor.y) > player.radius * 2:
                particle.show()
                particle.update()
                
        if frameCount % spawn_time == 0 and len(rocks) < 12:
            rock = Rock(randint(0, width), randint(20, height), randint(15, 50), randint(8, 15), randint(-50, 50) * .025, randint(-50, 50) * .025)
            
            while dist(rock.cor.x, rock.cor.y, player.cor.x, player.cor.y) < player.radius * 5:
                rock.cor.x, rock.cor.y = randint(0, width), randint(0, height)
                
            rocks.append(rock)
            
        if frameCount % 50 == 0:
            # This makes it so the spawn_time doesn't get inhumanely low
            if spawn_time > 50:
                spawn_time -=1
                
        stroke(255)
        fill(255)
        textSize(25)
        text("Score: {}".format(score), 75, 50)
    else:
        textAlign(CENTER)
        textSize(25)
        text("Press space to start!", width/2, height/2)
        
        if keys[32]:
            game_started = True
            
            
# Processing uses the functions keyPressed and keyReleased to see if a key is pressed by the user.

"""
Since 87 (the w key) is the only key that we have in the 'keys' dictionary,
it will only be changed to 'True' or 'False' if they key pressed is w
"""
# mousePressed function
def mousePressed():
    if game_started:
        bullets.append(Bullet(player.cor.x, player.cor.y, 16 * cos(player.heading + randint(-5, 5) * .004), 16 * sin(player.heading + randint (-5, 5) * .004)))

# Runs if any key is pressed
def keyPressed():
    keys[keyCode] = True
    
# Runs if any key is released
def keyReleased():
    keys[keyCode] = False
