# wiimote-airmouse

Wii Remote Plus (o con MotionPlus) como air mouse con giroscopio, pensado para Hyprland en Arch.

Lee los evdev que crea el driver `hid-wiimote` del kernel (botones, acelerómetro y MotionPlus) y
escribe en un ratón+teclado virtual (uinput). Incluye:

- **Puntero con giroscopio**: suavizado con filtro 1€, el movimiento se emite a ritmo fijo (250 Hz) y
  el offset se recalibra solo en reposo. Con `span`, la velocidad se ajusta al ancho del escritorio.
- **Perfiles de botones** (Home mantenido cambia de perfil; los LED del mando muestran cuál está activo),
  acciones al mantener, combinaciones de dos botones (`a+b`, o mantener B y pulsar otro), embrague y borrar texto.
- **Modo juego**: si la ventana enfocada es de un juego de la lista, los botones van a un mando
  virtual ("Wiimote Gamepad").
- **Ahorro de batería**: apaga los sensores si está quieto, lo desconecta tras un rato y avisa
  cuando le queda poca batería.
- **Teclado en pantalla** (`wii-keyboard`): un input method de Wayland detecta cuándo un campo pide
  texto. La interfaz gráfica no viene incluida: la pone cada uno ([ui], ver abajo).
- **Cursor propio** mientras se usa el mando (`[cursor] theme`, cualquier tema instalado).

## Instalar

```
git clone https://github.com/PauloBenvenuto/wiimote-airmouse.git ~/src/wiimote-airmouse
cd ~/src/wiimote-airmouse && makepkg -si
systemctl --user enable --now wiimote-airmouse
```

El paquete instala:
- `/usr/bin/wiimote-airmouse` y `/usr/bin/wii-keyboard`;
- el servicio de usuario;
- la regla udev, que da acceso a `/dev/uinput` y al mando sin sudo ni grupo `input`;
- el ejemplo de config en `/usr/share/doc/wiimote-airmouse/`.

Para actualizar: `git pull && makepkg -si`. La versión es `r<commits>.<hash>` del repo.

## Emparejar

Hazlo con `bluetoothctl`; los asistentes gráficos no responden a la petición de PIN. Lanza esto y
pulsa a la vez el botón rojo **SYNC**, que está bajo la tapa de las pilas:

```
{ echo 'scan on'; sleep 10; echo 'pair <MAC>'; sleep 8; echo 'trust <MAC>'; echo 'connect <MAC>'; sleep 5; echo 'scan off'; echo quit; } | bluetoothctl --agent NoInputNoOutput
```

Si ya estaba emparejado a medias, antes `bluetoothctl remove <MAC>`. Después, cualquier botón lo
reconecta. Al encenderlo, déjalo quieto un momento para que calibre el giroscopio.

## Config

`~/.config/wiimote/airmouse.toml`: copia `airmouse.example.toml`, que comenta cada opción. Tras
editarlo, `systemctl --user reload wiimote-airmouse`.

Para ver los valores crudos:

```
systemctl --user stop wiimote-airmouse && wiimote-airmouse --debug
```

El registro se ve con `journalctl --user -u wiimote-airmouse -f`.

`wiimote-airmouse --describe` imprime en JSON qué hace cada botón en cada perfil, para que una
interfaz muestre el mapa de botones. Formato: `{"current", "profiles": [{"name", "buttons": [{"button", "press", "hold"}]}]}`.

## Interfaz externa ([ui])

El teclado en pantalla y la acción `wsdrag` (arrastrar una ventana a otro escritorio) necesitan una
interfaz gráfica, por ejemplo una config de Quickshell. `[ui] ipc` es el prefijo de la orden, por
ejemplo `"qs -c <config> ipc call"`, y se le añade el objetivo y la función:

| Orden | Cuándo |
|---|---|
| `wiikbd open` / `wiikbd close` | un campo de texto pide texto tras un clic del mando / lo pierde |
| `wsdrag start <dirección>` / `drop` / `cancel` | se pulsa / suelta / interrumpe la acción `wsdrag` |

La interfaz del teclado le responde a `wii-keyboard` por el socket
`$XDG_RUNTIME_DIR/wii-keyboard.sock`, con una línea JSON `{"cmd": …, "text": …}` por orden:

- `commit`: escribe `text` en el campo;
- `backspace` y `enter`: pulsan esas teclas;
- `closed`: el usuario cerró el teclado.

`wii-keyboard` necesita que la app use text-input-v3. Firefox, GTK y Qt lo hacen; Chromium solo con
`--enable-wayland-ime --wayland-text-input-version=3`.

## Desarrollo

Para probar un cambio sin reinstalar:

```
systemctl --user stop wiimote-airmouse && ./wiimote-airmouse --debug
```

Cuando funcione, `makepkg -si` y `systemctl --user restart wiimote-airmouse`.

`protocols/input-method-unstable-v2.xml` es de wlroots, con licencia MIT; su aviso va dentro del archivo.

## Licencia

MIT (ver `LICENSE`).
