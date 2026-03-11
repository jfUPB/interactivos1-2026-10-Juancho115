# Unidad 4

## Bitácora de proceso de aprendizaje
### Gitbash
-se pone node bridgeServer.js luego saca error y luego 
- npm es node package manager
- npm install para instalar los paquetes del proyecto
- ya despues de esos pasos ahora si node bridgeServer.js
- para terminarl el proceso del servidor es ctrl+C (Exit)
- code. es para abrir el proyecto en el visual
- luego en el gitbash hay que póner el node bridgeServer.js y ahora si conecta a la pagina
- para conectar con el microbit $ node bridgeServer.js --device microbit



## Bitácora de aplicación 
- Para el **`microbit v2`**
```java
// adapters/MicrobitV2Adapter.js
const { SerialPort } = require("serialport");
const BaseAdapter = require("./BaseAdapter");

class ParseError extends Error {}

function parseV2Line(line) {
  // Línea esperada: $T:45020|X:-245|Y:12|A:1|B:0|CHK:258
  if (!line.startsWith("$")) throw new ParseError("Line does not start with $");
  // Eliminar $ al inicio si existe
  const body = line.slice(1).trim();
  const parts = body.split("|").map(p => p.trim()).filter(Boolean);
  const obj = {};
  for (const p of parts) {
    const [k, v] = p.split(":");
    if (typeof v === "undefined") throw new ParseError(`Malformed pair: ${p}`);
    obj[k] = v;
  }
  // Campos requeridos
  const keys = ["T","X","Y","A","B","CHK"];
  for (const k of keys) if (!(k in obj)) throw new ParseError(`Missing ${k}`);

  const x = Number(obj["X"]);
  const y = Number(obj["Y"]);
  const a = Number(obj["A"]);
  const b = Number(obj["B"]);
  const chk = Number(obj["CHK"]);

  if (!Number.isFinite(x) || !Number.isFinite(y) || !Number.isFinite(a) || !Number.isFinite(b) || !Number.isFinite(chk)) {
    throw new ParseError("Non-numeric value in data");
  }
  // Rango esperado para acelerómetros (según spec)
  if (x < -2048 || x > 2047 || y < -2048 || y > 2047) {
    throw new ParseError("Accelerometer value out of expected range");
  }
  if (![0,1].includes(a) || ![0,1].includes(b)) {
    throw new ParseError("Button values must be 0 or 1");
  }
  // Checksum: suma de absolutos de X, Y, A y B
  const computed = Math.abs(x) + Math.abs(y) + a + b;
  if (computed !== chk) {
    // Debe descartarse silenciosamente, pero notificar en consola.
    const err = new ParseError("Checksum mismatch");
    err.isChecksumMismatch = true;
    err.computed = computed;
    err.received = chk;
    throw err;
  }

  return { x: x | 0, y: y | 0, btnA: a === 1, btnB: b === 1 };
}

class MicrobitV2Adapter extends BaseAdapter {
  constructor({ path, baud = 115200, verbose = false } = {}) {
    super();
    this.path = path;
    this.baud = baud;
    this.port = null;
    this.buf = "";
    this.verbose = verbose;
  }

  async connect() {
    if (this.connected) return;
    if (!this.path) throw new Error("serialPort is required for microbit device mode");
    this.port = new SerialPort({ path: this.path, baudRate: this.baud, autoOpen: false });
    await new Promise((resolve, reject) => {
      this.port.open((err) => (err ? reject(err) : resolve()));
    });
    this.connected = true;
    this.onConnected?.(`serial open ${this.path} @${this.baud}`);
    this.port.on("data", (chunk) => this._onChunk(chunk));
    this.port.on("error", (err) => this._fail(err));
    this.port.on("close", () => this._closed());
  }

  async disconnect() {
    if (!this.connected) return;
    this.connected = false;
    if (this.port && this.port.isOpen) {
      await new Promise((resolve, reject) => {
        this.port.close((err) => (err ? reject(err) : resolve()));
      });
    }
    this.port = null;
    this.buf = "";
    this.onDisconnected?.("serial closed");
  }

  getConnectionDetail() {
    return `serial open ${this.path}`;
  }

  _onChunk(chunk) {
    
    this.buf += chunk.toString("utf8");
    let idx;
    while ((idx = this.buf.indexOf("\n")) >= 0) {
      const rawLine = this.buf.slice(0, idx).replace("\r", "").trim();
      this.buf = this.buf.slice(idx + 1);
      if (!rawLine) continue;
      try {
        const parsed = parseV2Line(rawLine);
        // Emitir exactamente el JSON que espera el servidor/bridge:
        this.onData?.(parsed);
      } catch (e) {
        if (e instanceof ParseError) {
          if (e.isChecksumMismatch) {
            // comportamiento requerido: descartar trama corrupta pero advertir en consola
            console.warn(`[Adapter] checksum mismatch: computed=${e.computed} recv=${e.received} raw="${rawLine}"`);
            // NO actualizar vista / NO emitir datos
          } else {
            if (this.verbose) console.log("Bad data:", e.message, "raw:", rawLine);
          }
        } else {
          this._fail(e);
        }
      }
    }
    if (this.buf.length > 8192) this.buf = ""; // evitar crecimiento indefinido
  }

  _fail(err) {
    this.onError?.(String(err?.message || err));
    this.disconnect();
  }

  _closed() {
    if (!this.connected) return;
    this.connected = false;
    this.port = null;
    this.buf = "";
    this.onDisconnected?.("serial closed (event)");
  }

  async writeLine(line) {
    if (!this.port || !this.port.isOpen) return;
    await new Promise((resolve, reject) => {
      this.port.write(line, (err) => (err ? reject(err) : resolve()));
    });
  }

  // Mantener el mismo contrato de comandos (por ejemplo setLed)
  async handleCommand(cmd) {
    if (cmd?.cmd === "setLed") {
      const x = Math.max(0, Math.min(4, Math.trunc(cmd.x)));
      const y = Math.max(0, Math.min(4, Math.trunc(cmd.y)));
      const v = Math.max(0, Math.min(9, Math.trunc(cmd.value)));
      // Si el dispositivo acepta un comando ASCII: "LED,x,y,v\n"
      await this.writeLine(`LED,${x},${y},${v}\n`);
    }
  }
}

module.exports = MicrobitV2Adapter;
```


## Bitácora de reflexión

