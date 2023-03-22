---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---

<script src="https://cdn.tailwindcss.com"></script>

![](imagenes/portada.png)
<h1 class="text-center text-3xl">INSTITUTO TECNOLOGICO DE TIJUANA</h1>
<h1 class="text-center text-3xl">PANTOJA REYES BENY SAMUEL</h1>
<h1 class="text-center text-3xl">19211703</h1>
<h1 class="text-center text-3xl">Rotary Encoder</h1>

### ¿Qué es?
El `Rotary Encoder` es un dispositivo que mide la posición angular (rotación) o eje. Lo convierte en una señal para encontrar la posición y dirección de la rotación. Es principalmente usado en motoress para tener un mejor control y en interfaces de usuarios para reemplazar potenciómetros. Algunos de estos dispositivos vienen con un botón para presionar.

### Diferencias entre el Rotary Encoder y el potenciómetro
Para la mayoría, el rotary encoder `KY-40` es lo mismo que un potenciómetro. En realidad, un potenciómetro es un sensor analógico y el `Rotary Encoder`  es digital.

El `Rotary Encoder`  es un dispositivo que puede detectar cuantas veces rota y manda una señal cada vez que esto pasa. Puede rotar una cantidad infinita de veces. El tacto de ambos en sentido físico es distinto, mientras que el `Rotary Encoder` tiene un movimiento un poco más brusco, el potenciómetro es más suave.

### Rotary Encoder en la Pi Pico
El `Rotary Encoder` tiene dos señales para saber la posicion `CLK` y `DT`. El pin `SW` está conectado al botón integrado.

Tabla de las conexiones:

| Rotary Encoder | Raspberry Pi Pico |
|----------------|-------------------|
| `CLK`          | `GP14`            |
| `DT`           | `GP13`            |
| `SW`           | No se usa         |
| `+`            | `3v3`             |
| `GND`          | `GND`             |

Queda de la siguiente forma el circuito:
![](imagenes/circuit.png)

### Uso
Para hacer uso del dispositivo usaremos la siguiente clase:
```python
import machine
import utime as time
from machine import Pin
import micropython

class Rotary:

    ROT_CW = 1
    ROT_CCW = 2
    SW_PRESS = 4
    SW_RELEASE = 8

    def __init__(self,dt,clk,sw):
        self.dt_pin = Pin(dt, Pin.IN, Pin.PULL_DOWN)
        self.clk_pin = Pin(clk, Pin.IN, Pin.PULL_DOWN)
        self.sw_pin = Pin(sw, Pin.IN, Pin.PULL_DOWN)
        self.last_status = (self.dt_pin.value() << 1) | self.clk_pin.value()
        self.dt_pin.irq(handler=self.rotary_change, trigger=Pin.IRQ_FALLING | Pin.IRQ_RISING )
        self.clk_pin.irq(handler=self.rotary_change, trigger=Pin.IRQ_FALLING | Pin.IRQ_RISING )
        self.sw_pin.irq(handler=self.switch_detect, trigger=Pin.IRQ_FALLING | Pin.IRQ_RISING )
        self.handlers = []
        self.last_button_status = self.sw_pin.value()

    def rotary_change(self, pin):
        new_status = (self.dt_pin.value() << 1) | self.clk_pin.value()
        if new_status == self.last_status:
            return
        transition = (self.last_status << 2) | new_status
        if transition == 0b1110:
            micropython.schedule(self.call_handlers, Rotary.ROT_CW)
        elif transition == 0b1101:
            micropython.schedule(self.call_handlers, Rotary.ROT_CCW)
        self.last_status = new_status

    def switch_detect(self,pin):
        if self.last_button_status == self.sw_pin.value():
            return
        self.last_button_status = self.sw_pin.value()
        if self.sw_pin.value():
            micropython.schedule(self.call_handlers, Rotary.SW_RELEASE)
        else:
            micropython.schedule(self.call_handlers, Rotary.SW_PRESS)

    def add_handler(self, handler):
        self.handlers.append(handler)

    def call_handlers(self, type):
        for handler in self.handlers:
            handler(type)
```
Esta clase servira como driver entre el dispositivo y la pico.

Y el codigo será el siguiente:
```python
rotary = Rotary(13, 14, 0)
val = 0

def rotary_changed(change):
    global val
    if change == Rotary.ROT_CW:
        val = val + 1
        print(val)
    elif change == Rotary.ROT_CCW:
        val = val - 1
        print(val)
    elif change == Rotary.SW_PRESS:
        print('PRESS')
    elif change == Rotary.SW_RELEASE:
        print('RELEASE')

rotary.add_handler(rotary_changed)

while True:
    time.sleep(0.1)
```
Aquí hay una [demostración](https://wokwi.com/projects/359922595891764225).
### Fuentes
[upesy](https://www.upesy.com/blogs/tutorials/rotary-encoder-raspberry-pi-pico-on-micro-python)
[MicroPython for Kids](https://www.coderdojotc.org/micropython/sensors/10-rotary-encoder/)


<iframe class="w-full h-screen" src="https://wokwi.com/projects/359922595891764225" />
