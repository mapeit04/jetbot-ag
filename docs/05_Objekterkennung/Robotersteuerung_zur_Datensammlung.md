# Bilder mit dem Gamepad sammeln

Anstatt den JetBot für jedes Foto per Hand umzustellen, wird er mit dem Gamepad gesteuert. Die Bilder werden direkt per Knopfdruck gespeichert.

Die folgenden Zellen werden im Notebook `collision_avoidance/data_collection.ipynb` **nach der Zelle mit `display(image)`** eingefügt.

## Tastenbelegung

| Taste | Funktion |
|---|---|
| Taste 7 | Gas geben |
| Linker Stick (links/rechts) | Lenken |
| **A** | Bild als `free` speichern |
| **B** | Bild als `blocked` speichern |

> **Hinweis:** Das Gamepad wird am **Laptop** angeschlossen, nicht am JetBot.
> Ist das Notebook `teleoperation` noch geöffnet, dort vorher den Kernel beenden (*Kernel → Shut Down Kernel*).

---

## Zelle 1: Roboter und Gamepad initialisieren

```python
from jetbot import Robot

robot = Robot()
controller = widgets.Controller(index=0)
display(controller)
```

Nach dem Ausführen **eine beliebige Taste am Gamepad drücken**, bis im Widget die Balken erscheinen.

---

## Zelle 2: Steuerung aktivieren

```python
GAS_BUTTON = 7    # Taste fürs Gasgeben
STEER_AXIS = 0    # Achse des linken Sticks (links/rechts)
SPEED = 0.5       # Höchstgeschwindigkeit (0.0 bis 1.0)
TURN = 0.3        # Stärke der Lenkung

def drive(change):
    gas   = controller.buttons[GAS_BUTTON].value     # 0.0 (los) bis 1.0 (voll)
    steer = controller.axes[STEER_AXIS].value        # -1.0 (links) bis 1.0 (rechts)
    robot.left_motor.value  = max(-1.0, min(1.0, gas * SPEED + steer * TURN))
    robot.right_motor.value = max(-1.0, min(1.0, gas * SPEED - steer * TURN))

def on_a(change):
    if change['new']:          # nur beim Drücken, nicht beim Loslassen
        save_free()

def on_b(change):
    if change['new']:
        save_blocked()

controller.buttons[GAS_BUTTON].observe(drive, names='value')
controller.axes[STEER_AXIS].observe(drive, names='value')
controller.buttons[0].observe(on_a, names='pressed')
controller.buttons[1].observe(on_b, names='pressed')

display(image)
display(widgets.HBox([free_count, free_button]))
display(widgets.HBox([blocked_count, blocked_button]))
```

> **Wichtig:** Diese Zelle nur **einmal** ausführen. Wird sie mehrfach ausgeführt, speichert ein Tastendruck mehrere Bilder. In diesem Fall *Kernel → Restart Kernel* und alle Zellen erneut von oben ausführen.

### Tipps

- Vor jedem Foto kurz **anhalten**, sonst werden die Bilder verwackelt.
- Mindestens **100 Bilder pro Klasse** sammeln, beide Klassen etwa **gleich viele**.
- Lenkt der JetBot spiegelverkehrt, `TURN = -0.3` setzen.
- Mit `SPEED` und `TURN` lassen sich Geschwindigkeit und Lenkung anpassen.

---

## Zelle 3: Steuerung beenden

```python
controller.buttons[GAS_BUTTON].unobserve(drive, names='value')
controller.axes[STEER_AXIS].unobserve(drive, names='value')
robot.stop()
```

Danach im Notebook wie gewohnt mit `camera.stop()` weitermachen.
