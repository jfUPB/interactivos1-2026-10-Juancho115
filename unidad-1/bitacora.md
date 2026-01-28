# Unidad 1

## Bitácora de proceso de aprendizaje

### Actividad 01
- Un sistema físico interactivo es un input que combina lo digital con el mundo físico, permitiendo que estos 2 mundos se encuentren y permitan la inmersión en tiempo real
- Esto lo podría aplicar a mi perfil profesional es usando estos sistemas para entornos diferentes, especialmente en el mundo de la música, he estado muy interesado en la creación de visuales para tanto música en vivo como algo pre grabado, además de esto me interesa mucho el mundo del branding y la publicidad

### Avtividad 02

- Diseño / arte generativo es una forma de crear por medio de procesos sitematicos y procedurales imagenes, sonidos o cualquier forma de expresion por medio de herramientas digitales
- Lo prodria aplicar en mi perfil prosefional usando esto para poder integrarlo en animaciones especificas, esto siento que me ayudaria a crear una estetica un poco mas unica.

## Bitácora de aplicación 
### Actividad 03
- El programa no funciona con el `was_pressed()` debido a que esta función, solo le indica las veces que fue presionado el botón, mientras que con el `is_pressed()` procesa la información del microbit de sí el botón está siendo presionado o no, y la intención del programa era que se mostrara constantemente el input en el microbit mientras el botón está presionado, no si el botón fue presionado alguna vez.

### Actividad 04

- Programa Pjs.5

```javascript
let port;
let connectBtn;

let CIRCLE_RADIUS = 50;
let circleX;
let circleY;
let speed = 5;

function setup() {
  createCanvas(400, 400);
  port = createSerial();

  connectBtn = createButton('Connect to micro:bit');
  connectBtn.position(80, 300);
  connectBtn.mousePressed(connectBtnClick);

  let sendBtn = createButton('Send Love');
  sendBtn.position(220, 300);
  sendBtn.mousePressed(sendBtnClick);

  circleX = width / 2;
  circleY = height / 2;
}

function draw() {
  background(220);

  if (port.availableBytes() > 0) {
    let dataRx = port.read(1);

    if (dataRx === 'A') {
      circleX -= speed;
      fill('red');
    } 
    else if (dataRx === 'B') {
      circleX += speed;
      fill('yellow');
    } 
    else {
      fill('green');
    }

    circleX = constrain(circleX, CIRCLE_RADIUS, width - CIRCLE_RADIUS);

    fill('black');
    textAlign(CENTER, CENTER);
    text(dataRx, width / 2, height / 2 + 70);
  }

  ellipse(circleX, circleY, CIRCLE_RADIUS*2, CIRCLE_RADIUS*2);

  if (!port.opened()) {
    connectBtn.html('Connect to micro:bit');
  } else {
    connectBtn.html('Disconnect');
  }
}

function connectBtnClick() {
  if (!port.opened()) {
    port.open('MicroPython', 115200);
  } else {
    port.close();
  }
}

function sendBtnClick() {
  port.write('h');
}
```

- Programa microbit
```python
from microbit import *

uart.init(baudrate=115200)
display.show(Image.BUTTERFLY)

while True:
    if button_a.is_pressed():
        uart.write('A')
        sleep(500)
    if button_b.is_pressed():
        uart.write('B')
        sleep(500)
    if accelerometer.was_gesture('shake'):
        uart.write('C')
        sleep(500)
    if uart.any():
        data = uart.read(1)
        if data:
            if data[0] == ord('h'):
                display.show(Image.HEART)
                sleep(500)
                display.show(Image.HAPPY)
```

- El sistema fisico interactivo permite al usuario mover un circulo con botones fisicos, ademas de esto permite cambiar de color el circulo y que en el microbit se proyecten diferentes formas 



## Bitácora de reflexión

