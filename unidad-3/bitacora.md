# Unidad 3

## Bitácora de proceso de aprendizaje
### Actividad 03 
- En la terminal de comandos (Git Bash)
- pwd: Path working directory
- ls -al: Para abrir el directorio y archivos
- clear: Para borrar
- cd: change directory
  (Si se usa el cd el inicio del archivo al que se quiere y luego tab te sale los archviso con ese nombre disponibles)
- git clone: Para clonar repositorios
- git pull: Para hacer el pull y actualizar contenido
- code '.' : Abre el directorio en el que estoy parado
- code '..' : Abre el directorio padre 
   



con las flechas se navega en comandos anteriores
si escribes el inicio del comando y TAB muestra los codigos que se pueda tener ejemplo: pw + TAB






## Bitácora de aplicación 

### Actividad 4

```javascript
const ENTRY = "ENTRY";
const EXIT = "EXIT";

class Timer {
  constructor(owner, event, duration) {
    this.owner = owner;
    this.event = event;
    this.duration = duration;
    this.startTime = 0;
    this.active = false;
  }

  start() {
    this.startTime = millis();
    this.active = true;
  }

  stop() {
    this.active = false;
  }

  update() {
    if (this.active && millis() - this.startTime >= this.duration) {
      this.active = false;
      this.owner.postEvent(this.event);
    }
  }
}

class FSMTask {
  constructor() {
    this.state = null;
    this.timers = [];
    this.eventQueue = [];
  }

  addTimer(event, duration) {
    const t = new Timer(this, event, duration);
    this.timers.push(t);
    return t;
  }

  postEvent(ev) {
    this.eventQueue.push(ev);
  }

  transitionTo(newState) {
    if (this.state) this.state(EXIT);
    this.state = newState;
    this.state(ENTRY);
  }

  update() {
    for (let t of this.timers) {
      t.update();
    }

    while (this.eventQueue.length > 0) {
      const ev = this.eventQueue.shift();
      if (this.state) {
        this.state(ev);
      }
    }
  }
}

const TIMER_LIMITS = {
  min: 15,
  max: 25,
  defaultValue: 20,
};

const EVENTS = {
  DEC: "A",
  INC: "B",
  START: "S",
  TICK: "Timeout",
};

const UI = {
  dialSize: 250,
  ringWeight: 20,
  bigText: 100,
  configText: 120,
  helpText: 18,
};


class Temporizador extends FSMTask {
  constructor(minValue, maxValue, defaultValue) {
    super();

    this.minValue = minValue;
    this.maxValue = maxValue;
    this.defaultValue = defaultValue;
    this.configValue = defaultValue;
    this.totalSeconds = defaultValue;
    this.remainingSeconds = defaultValue;

    this.myTimer = this.addTimer(EVENTS.TICK, 1000);
    this.transitionTo(this.estado_config);

  }

  get currentState() {
    return this.state;
  }

  estado_config(ev) {
    if (ev === ENTRY) {
      this.configValue = this.defaultValue;
    }
    else if (ev === EVENTS.DEC) {
      if (this.configValue > this.minValue) this.configValue--;
    } else if (ev === EVENTS.INC) {
      if (this.configValue < this.maxValue) this.configValue++;
    } else if (ev === EVENTS.START) {
      this.totalSeconds = this.configValue;
      this.remainingSeconds = this.totalSeconds;
      this.transitionTo(this.estado_armed);
    }
  }


  estado_armed(ev) {
    if (ev === ENTRY) {
      this.myTimer.start();

    } else if (ev === EVENTS.TICK) {

      if (this.remainingSeconds > 0) {
        this.remainingSeconds--;

        if (this.remainingSeconds === 0) {
          this.transitionTo(this.estado_timeout);
        } else {
          this.myTimer.start();
        }
      }

    } else if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_paused1);

    } else if (ev === EXIT) {
      this.myTimer.stop();
    }
  }
  
  estado_paused1(ev) {
    if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_armed);

    } else if (ev === EVENTS.INC) {
      this.transitionTo(this.estado_paused2);
    }
  }
  
  estado_paused2(ev) {
    if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_config);

    } else if (ev === EVENTS.INC) {
      this.transitionTo(this.estado_paused3);
    }
  }
  
  estado_paused3(ev) {
    if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_armed);
    }
  }

  
  estado_timeout (ev)  {
    if (ev === ENTRY) {
      console.log("¡TIEMPO!");
    } else if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_config);
    }
  }
}

let temporizador;
const renderer = new Map();

function setup() {
  createCanvas(windowWidth, windowHeight);
  temporizador = new Temporizador(
    TIMER_LIMITS.min,
    TIMER_LIMITS.max,
    TIMER_LIMITS.defaultValue
  );
  textAlign(CENTER, CENTER);

  renderer.set(temporizador.estado_config, () => drawConfig(temporizador.configValue));
  renderer.set(temporizador.estado_armed, () => drawArmed(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_paused1, () => drawPaused1(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_paused2, () => drawPaused2(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_paused3, () => drawPaused3(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_timeout, () => drawTimeout());
}

function draw() {
  temporizador.update();
  renderer.get(temporizador.currentState)?.();
}

function drawConfig(val) {
  background(20, 40, 80);
  fill(255);
  textSize(120);
  text(val, width / 2, height / 2);
  textSize(18);
  fill(200);
  text("A(-)\nB(+)\nS(start)", width / 2, height / 2 + 100);
}

function drawArmed(val, total) {
  background(20, 20, 20);
  let pulse = sin(frameCount * 0.1) * 10;

  noFill();
  strokeWeight(20);
  stroke(255, 100, 0, 50);
  ellipse(width / 2, height / 2, 250);

  stroke(255, 150, 0);
  let angle = map(val, 0, total, 0, TWO_PI);
  arc(width / 2, height / 2, 250, 250, -HALF_PI, angle - HALF_PI);

  fill(255);
  noStroke();
  textSize(100 + pulse);
  text(val, width / 2, height / 2);
}

function drawPaused1(val, total) {
  background(30,0,0);

  fill(255, 200, 0);
  textSize(40);
  text("PAUSA", width / 2, height / 2 - 120);

  fill(255);
  textSize(100);
  text(val, width / 2, height / 2);
}
function drawPaused2(val, total) {
  background(0,30,0);

  fill(255, 200, 0);
  textSize(40);
  text("PAUSA", width / 2, height / 2 - 120);

  fill(255);
  textSize(100);
  text(val, width / 2, height / 2);
}
function drawPaused3(val, total) {
  background(0,0,30);

  fill(255, 200, 0);
  textSize(40);
  text("PAUSA", width / 2, height / 2 - 120);

  fill(255);
  textSize(100);
  text(val, width / 2, height / 2);
}

function drawTimeout() {
  let bg = frameCount % 20 < 10 ? color(150, 0, 0) : color(255, 0, 0);
  background(bg);
  fill(255);
  textSize(100);
  text("¡TIEMPO!", width / 2, height / 2);
}

function keyPressed() {
  if (key === "a" || key === "A") temporizador.postEvent("A");
  if (key === "b" || key === "B") temporizador.postEvent("B");
  if (key === "s" || key === "S") temporizador.postEvent("S");
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}
```

## Bitácora de reflexión
### Avtividad 05
- Codigo Microbit
``` Python
from microbit import *
import radio

radio.on()
radio.config(channel=7)  

while True:
    if button_a.was_pressed():
        radio.send("A")
        display.show("A")   
        display.clear()
    elif button_b.was_pressed():
        radio.send("B")
        display.show("B")  
        display.clear()
    elif accelerometer.was_gesture("shake"):
        radio.send("S")
        display.show("S")

        display.clear()

```

- P5.js
``` javascript
const ENTRY = "ENTRY";
const EXIT = "EXIT";

class Timer {
  constructor(owner, event, duration) {
    this.owner = owner;
    this.event = event;
    this.duration = duration;
    this.startTime = 0;
    this.active = false;
  }

  start() {
    this.startTime = millis();
    this.active = true;
  }

  stop() {
    this.active = false;
  }

  update() {
    if (this.active && millis() - this.startTime >= this.duration) {
      this.active = false;
      this.owner.postEvent(this.event);
    }
  }
}

class FSMTask {
  constructor() {
    this.state = null;
    this.timers = [];
    this.eventQueue = [];
  }

  addTimer(event, duration) {
    const t = new Timer(this, event, duration);
    this.timers.push(t);
    return t;
  }

  postEvent(ev) {
    this.eventQueue.push(ev);
  }

  transitionTo(newState) {
    if (this.state) this.state(EXIT);
    this.state = newState;
    this.state(ENTRY);
  }

  update() {
    for (let t of this.timers) {
      t.update();
    }

    while (this.eventQueue.length > 0) {
      const ev = this.eventQueue.shift();
      if (this.state) {
        this.state(ev);
      }
    }
  }
}

const TIMER_LIMITS = {
  min: 15,
  max: 25,
  defaultValue: 20,
};

const EVENTS = {
  DEC: "A",
  INC: "B",
  START: "S",
  TICK: "Timeout",
};

const UI = {
  dialSize: 250,
  ringWeight: 20,
  bigText: 100,
  configText: 120,
  helpText: 18,
};


class Temporizador extends FSMTask {
  constructor(minValue, maxValue, defaultValue) {
    super();

    this.minValue = minValue;
    this.maxValue = maxValue;
    this.defaultValue = defaultValue;
    this.configValue = defaultValue;
    this.totalSeconds = defaultValue;
    this.remainingSeconds = defaultValue;

    this.myTimer = this.addTimer(EVENTS.TICK, 1000);
    this.transitionTo(this.estado_config);

  }

  get currentState() {
    return this.state;
  }

  estado_config(ev) {
    if (ev === ENTRY) {
      this.configValue = this.defaultValue;
    }
    else if (ev === EVENTS.DEC) {
      if (this.configValue > this.minValue) this.configValue--;
    } else if (ev === EVENTS.INC) {
      if (this.configValue < this.maxValue) this.configValue++;
    } else if (ev === EVENTS.START) {
      this.totalSeconds = this.configValue;
      this.remainingSeconds = this.totalSeconds;
      this.transitionTo(this.estado_armed);
    }
  }


  estado_armed(ev) {
    if (ev === ENTRY) {
      this.myTimer.start();

    } else if (ev === EVENTS.TICK) {

      if (this.remainingSeconds > 0) {
        this.remainingSeconds--;

        if (this.remainingSeconds === 0) {
          this.transitionTo(this.estado_timeout);
        } else {
          this.myTimer.start();
        }
      }

    } else if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_paused1);

    } else if (ev === EXIT) {
      this.myTimer.stop();
    }
  }
  
  estado_paused1(ev) {
    if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_armed);

    } else if (ev === EVENTS.INC) {
      this.transitionTo(this.estado_paused2);
    }
  }
  
  estado_paused2(ev) {
    if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_config);

    } else if (ev === EVENTS.INC) {
      this.transitionTo(this.estado_paused3);
    }
  }
  
  estado_paused3(ev) {
    if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_armed);
    }
  }

  
  estado_timeout (ev)  {
    if (ev === ENTRY) {
      console.log("¡TIEMPO!");
    } else if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_config);
    }
  }
}

let temporizador;
const renderer = new Map();

function setup() {

port = createSerial();
connectBtn = createButton('Connect to micro:bit');
connectBtn.position(80, 300);
connectBtn.mousePressed(connectBtnClick);

  createCanvas(windowWidth, windowHeight);
  temporizador = new Temporizador(
    TIMER_LIMITS.min,
    TIMER_LIMITS.max,
    TIMER_LIMITS.defaultValue
  );
  textAlign(CENTER, CENTER);

  renderer.set(temporizador.estado_config, () => drawConfig(temporizador.configValue));
  renderer.set(temporizador.estado_armed, () => drawArmed(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_paused1, () => drawPaused1(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_paused2, () => drawPaused2(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_paused3, () => drawPaused3(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_timeout, () => drawTimeout());
  
}

function draw() {
  temporizador.update();
  renderer.get(temporizador.currentState)?.();

  if(port.availableBytes() > 0){
        let dataRx = port.read(1);
        if(dataRx == 'A'){
            temporizador.postEvent("A");
        }
        else if(dataRx == 'B'){
             temporizador.postEvent("B");
        }
        else if(dataRx== 'S'){
             temporizador.postEvent("S");
        }
    }


    if (!port.opened()) {
        connectBtn.html('Connect to micro:bit');
    }
    else {
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


function drawConfig(val) {
  background(20, 40, 80);
  fill(255);
  textSize(120);
  text(val, width / 2, height / 2);
  textSize(18);
  fill(200);
  text("A(-)\nB(+)\nS(start)", width / 2, height / 2 + 100);
}

function drawArmed(val, total) {
  background(20, 20, 20);
  let pulse = sin(frameCount * 0.1) * 10;

  noFill();
  strokeWeight(20);
  stroke(255, 100, 0, 50);
  ellipse(width / 2, height / 2, 250);

  stroke(255, 150, 0);
  let angle = map(val, 0, total, 0, TWO_PI);
  arc(width / 2, height / 2, 250, 250, -HALF_PI, angle - HALF_PI);

  fill(255);
  noStroke();
  textSize(100 + pulse);
  text(val, width / 2, height / 2);
}

function drawPaused1(val, total) {
  background(30,0,0);

  fill(255, 200, 0);
  textSize(40);
  text("PAUSA", width / 2, height / 2 - 120);

  fill(255);
  textSize(100);
  text(val, width / 2, height / 2);
}
function drawPaused2(val, total) {
  background(0,30,0);

  fill(255, 200, 0);
  textSize(40);
  text("PAUSA", width / 2, height / 2 - 120);

  fill(255);
  textSize(100);
  text(val, width / 2, height / 2);
}
function drawPaused3(val, total) {
  background(0,0,30);

  fill(255, 200, 0);
  textSize(40);
  text("PAUSA", width / 2, height / 2 - 120);

  fill(255);
  textSize(100);
  text(val, width / 2, height / 2);
}

function drawTimeout() {
  let bg = frameCount % 20 < 10 ? color(150, 0, 0) : color(255, 0, 0);
  background(bg);
  fill(255);
  textSize(100);
  text("¡TIEMPO!", width / 2, height / 2);
}

function keyPressed() {
  if (key === "a" || key === "A") temporizador.postEvent("A");
  if (key === "b" || key === "B") temporizador.postEvent("B");
  if (key === "s" || key === "S") temporizador.postEvent("S");
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}
```



