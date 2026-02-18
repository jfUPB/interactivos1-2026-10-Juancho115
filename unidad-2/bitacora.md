# Unidad 2

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 
## Actividad 02
<img width="489" height="485" alt="image" src="https://github.com/user-attachments/assets/d990636a-b624-486c-81c0-805da531ff97" />


### Actividad 3
```python
from microbit import *
import utime

class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration

        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)

class Semaforo:
   def __init__(self,_x,_y): 
       self.redX = _x
       self.redY = _y
       self.event_queue = []
       self.timers = []

       self.myTimer = self.createTimer("Timeout",2000)   

       self.estado_actual = None
       self.transicion_a(self.estado_WaitRed)
   
   def createTimer(self,event,duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t
   def post_event(self, ev):
            self.event_queue.append(ev)

   def update(self):
        # 1. Actualizar todos los timers internos automáticamente
        for t in self.timers:
            t.update()

        # 2. Procesar la cola de eventos resultante
        
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

   def transicion_a(self, nuevo_estado):
       if self.estado_actual: self.estado_actual("EXIT")
       self.estado_actual = nuevo_estado
       self.estado_actual("ENTRY")

   def estado_WaitRed(self,ev):
        if ev == "ENTRY":
            display.set_pixel(self.redX,self.redY,9)
            self.myTimer.start(2000)
        if ev == "Timeout":
            display.set_pixel(self.redX,self.redY,0)
            self.transicion_a(self.estado_WaitGreen)
       
            
       
   def estado_WaitGreen(self,ev):
        if ev == "ENTRY":
            display.set_pixel(self.redX,self.redY+2,9)
            self.myTimer.start(1000)
        if ev == "Timeout":
            display.set_pixel(self.redX,self.redY+2,0)
            self.transicion_a(self.estado_WaitYellow)
        if ev == "A":
            self.myTimer.stop()
            display.set_pixel(self.redX,self.redY+2,0)
            self.transicion_a(self.estado_WaitYellow)
    
   def estado_WaitYellow(self,ev):
        if ev == "ENTRY":
            display.set_pixel(self.redX,self.redY+1,9)
            self.myTimer.start(500)
        if ev == "Timeout":
            display.set_pixel(self.redX,self.redY+1,0)
            self.transicion_a(self.estado_WaitRed)


       
semaforo1 = Semaforo(0,0)

while True:

    if button_a.was_pressed():
        semaforo1.post_event("A")

        
    semaforo1.update()
    utime.sleep_ms(20)
```

### Actividad 04 
```python
from microbit import *
import utime
import music


def make_fill_images(on='9', off='0'):
    imgs = []
    for n in range(26):
        rows = []
        k = 0
        for _ in range(5):
            row = []
            for _ in range(5):
                row.append(on if k < n else off)
                k += 1
            rows.append(''.join(row))
        imgs.append(Image(':'.join(rows)))
    return imgs


class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration
        self.start_time = 0
        self.active = False

    def start(self):
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)


class Task:
    def __init__(self):
        self.event_queue = []
        self.timers = []

        self.timer = self.createTimer("Timeout", 1000)

        self.imgs = make_fill_images()
        self.tiempo = 20   # 20 pixeles = 20 segundos

        self.estado_actual = None
        self.transicion_a(self.estado_config)

    def createTimer(self, event, duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    def post_event(self, ev):
        self.event_queue.append(ev)

    def update(self):
        for t in self.timers:
            t.update()

        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual:
            self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")

    def estado_config(self, ev):
        if ev == "ENTRY":
            display.show(self.imgs[self.tiempo])

        if ev == "A" and self.tiempo > 15:
            self.tiempo -= 1
            display.show(self.imgs[self.tiempo])

        if ev == "B" and self.tiempo < 25:
            self.tiempo += 1
            display.show(self.imgs[self.tiempo])

        if ev == "S":
            self.transicion_a(self.estado_cuenta)
            music.play(music.ENTERTAINER)

    def estado_cuenta(self, ev):
        if ev == "ENTRY":
            self.timer.start()

        if ev == "Timeout":
            self.tiempo -= 1
            display.show(self.imgs[self.tiempo])

            if self.tiempo <= 0:
                self.transicion_a(self.estado_explosion)
            else:
                self.timer.start()

        if ev == "EXIT":
            self.timer.stop()

    def estado_explosion(self, ev):
        if ev == "ENTRY":
            display.show(Image.SKULL)
            music.play(music.DADADADUM)

        if ev == "A" or ev == "B" or ev == "S":
            self.tiempo = 20
            self.transicion_a(self.estado_config)


task = Task()

while True:
    if button_a.was_pressed():
        task.post_event("A")
    if button_b.was_pressed():
        task.post_event("B")
    if accelerometer.was_gesture("shake"):
        task.post_event("S")

    task.update()
    utime.sleep_ms(20)
```
<img width="492" height="396" alt="image" src="https://github.com/user-attachments/assets/ff6ec8fc-0bb6-4c9d-a551-35673ee0560e" />


## Bitácora de reflexión

### Actividad 05

- El reto lo resolví basandome en actividades anteriores en la que se uso el p5.js y agregué en el codigo del microbit una parte para que el p5.js reconozca los inputs que equivalen a lo que se necesita y en el p5.js ademas de agregar un canva de instrucciones de como usar el microbit con el teclado hay una parte dedicada a la lectura del teclado y la interpretación de las teclas que se usan

- Codigo en microbit
```python
uart.init(baudrate=115200)
```

  ```python
   data = uart.read()

    if data is not None:
        for char in data.decode():
            if char == "A":
              task.post_event("A")
            elif char == "B":
              task.post_event("B")
            elif char == "S":
                task.post_event("S")
  ``` 
- Codigo p5.js

```JAVASCRIPT
let port;
let connectBtn;

function setup() {
  createCanvas(400, 200);
  textAlign(CENTER, CENTER);
  textSize(16);
  port = createSerial();
  connectBtn = createButton('Connect to micro:bit');
  connectBtn.position(130, 180);
  connectBtn.mousePressed(connectToSerial);
}

function draw() {
  background(30);
  fill(255);

  if (!port.opened()) {
    text("Presiona el botón para conectar\n\nLuego usa:\nA = UP\nB = DOWN\nS = ARM", width/2, height/2);
  } else {
    text("Conectado ✅\n\nPresiona A, B o S", width/2, height/2);
  }
}

function connectToSerial() {
  if (!port.opened()) {
    port.open(115200);
  } else {
    port.close();
  }
}


function keyPressed() {
  if (!port.opened()) return;

  let k = key.toUpperCase();

  if (k === "A" || k === "B" || k === "S") {
    port.write(k);
    console.log("Enviado:", k);
  }
}

```



