# CC2 Auto Clear

Página web de un solo archivo que añade la **retirada automática de piezas** a un G-code de la
**ELEGOO Centauri Carbon 2**, para impresoras con la cama inclinada, la puerta abierta, una pala en el
cabezal y un doblador de placa (el montaje de FarmLoop).

Funciona sin conexión: abre `index.html` en el navegador, arrastra tu `.gcode` y descarga el procesado.
El archivo no sale de tu ordenador.

## Qué hace con el G-code

- **Línea de purga:** si el inicio de ELEGOO dibuja la línea de purga en el borde delantero (`G180 S7` … `G180 S8`),
  la cambia por la misma cantidad purgada en el cubilete. Si ya la quitaste en OrcaSlicer, no toca nada.
- **Final:** cambia el `M84` del final de ELEGOO por el ciclo:
  1. aparca el cabezal sobre el cubilete (X 51, Y 264), sube la cama arriba del todo (Z 3), o hasta donde deje
     el espacio libre que indiques por encima de la boquilla si la pieza es alta, y enfría con el ventilador de
     capa y el auxiliar (`M106 P2`) hasta la temperatura de despegue. El firmware de la CC2 ignora `TEMPERATURE_WAIT`
     y su `M190` solo espera calentando, así que la espera es un `G4` calculado con la temperatura de cama de la
     impresión: medido en una CC2, de 70 °C baja a 40 °C en unos 4 min y a 30 °C en unos 8. Se puede poner un mínimo;
  2. dobla la placa contra el doblador (baja la cama, la mantiene y repite);
  3. barre de arriba abajo por columnas. Primero sube la cama con el cabezal aún sobre el cubilete,
     luego el cabezal cruza a su columna y la cama hace el último tramo. Cada columna solo recibe las pasadas
     que llegan a sus propias piezas (según las etiquetas de objeto de Orca) y empieza justo detrás de ellas;
  4. vuelve sobre el cubilete, sube la cama igual que al enfriar, sopla sobre la placa limpia y apaga los motores.
- **Rechaza** archivos que no son de la CC2, que ya tienen un ciclo, con piezas en la franja del fondo que la
  pala no alcanza, o cuyos ajustes harían chocar el pórtico.
- **Copias:** puede repetir la impresión N veces en un solo archivo; cada copia imprime, limpia y empieza la siguiente.
- **Prueba de solo doblado:** enfría y dobla sin barrer, para probar el doblador.
- **Solo el ciclo:** descarga un archivo aparte con únicamente el enfriado y la retirada de esa impresión, para
  ponerlo en la cola justo detrás de ella. No hace home (al hacer home de Z la boquilla bajaría sobre las piezas):
  hay que lanzarlo en cuanto termina la impresión, con los motores aún encendidos, así que el G-code de acabado
  no debe llevar `M84`. Si la impresora ha perdido la posición, debería rechazar el primer movimiento.

## Firmware: de serie o COSMOS

Un selector elige el firmware del archivo; al cargar uno se elige solo (los perfiles de COSMOS empiezan con
`PRINT_START`) y se puede cambiar a mano.

- **De serie (ELEGOO o Community):** todo lo de arriba, tal cual.
- **[COSMOS](https://github.com/OpenCentauri/cosmos)** (Klipper/Kalico), con sus perfiles de OrcaSlicer:
  - **Enfriado por temperatura:** COSMOS sí respeta `TEMPERATURE_WAIT`, así que el ciclo espera a que la cama
    llegue a la temperatura de despegue, sin `G4` calculado. El mínimo de enfriado, si se pone, sigue siendo un `G4`.
  - **Final:** el ciclo va justo antes de `PRINT_END`, que se queda: aparca, limpia la boquilla y apaga los motores.
    El ciclo no lleva `M84`.
  - **Purga:** después de `PRINT_START` se añade `PURGE`, la purga de COSMOS en el cubilete (y limpieza de boquilla).
    Pon `adaptive_purge = False` en `cosmos.conf`; si no, `PRINT_START` también purga sobre la placa.
  - **Cubilete:** la posición de `MOVE_TO_TRAY` de COSMOS (X 52,5, Y 264, entrando desde Y 255).
  - **Solo el ciclo** no está disponible: `PRINT_END` apaga los motores y un archivo aparte no podría empezar sin
    hacer home. Usa el archivo procesado (o las copias).

## La página

- Deslizadores para la altura de la primera pasada, la separación y el número de pasadas (enlazados),
  el doblador y el enfriado. Se puede bajar a 2 pasadas (o 1 si el pórtico pasa por encima de la pieza a ras de
  placa); si la separación deja al pórtico sin margen ante una pieza que no se suelte, lo avisa en naranja.
- La pieza más alta la marca el doblador: antes de doblar, la cama baja rápido hasta 10 mm antes del punto de
  contacto de la pestaña, y la pieza tiene que caber ahí bajo la boquilla con el margen.
- **Tres modos de barrido:**
  - **Completo:** todas las columnas a todas las alturas; limpia la placa entera (el modo de siempre).
  - **Automático:** las mismas alturas de los deslizadores, pero por encima de la placa cada pasada empuja una sola
    vez en el centro de la capa de cada pieza a esa altura; la última, a ras de placa, cubre las columnas con piezas.
  - **Manual:** tu propia lista de empujes («Empezar desde Automático» la rellena con lo que haría ese modo).
- **Empujes a medida (modo Manual):** una lista de empujes que se ejecutan en orden.
  Cada uno tiene su altura de la pala sobre la placa, dónde empuja (las columnas donde llega a piezas, el centro de la capa
  de cada pieza a esa altura, toda la placa o una sola posición X), cuántas veces y a qué velocidad. Una pieza que ya ha golpeado un empuje anterior se da por
  retirada; solo rechaza un empuje cuyo pórtico chocaría con una pieza que ningún empuje anterior ha tocado. 
- **Prueba de espera de temperatura:** descarga un G-code que no mueve nada; calienta la cama a 45 °C y la deja
  enfriar probando `TEMPERATURE_WAIT` y `M190`, para ver en el log de la impresora cuál espera de verdad.
  Resultado en una CC2: `TEMPERATURE_WAIT` se ignora siempre y `M190` solo espera al calentar.
- Velocidades por perfiles (lento, conservador, rápido), uno para los movimientos que no empujan y otro para los
  que empujan. Las medidas de la pala y del pórtico son fijas.
- Vista lateral de las pasadas y un visor 3D que reproduce el G-code real del ciclo. Las piezas se dibujan con su
  forma real (los perímetros exteriores de cada capa, leídos del G-code). Con la animación parada, cada pasada es un
  anillo con el contorno real de la pieza a la altura donde la golpea la pala, y una flecha a esa misma altura muestra por dónde
  empuja; al mover un empuje se resalta con el ancho de la pala, y en rojo los que se rechazan.

## Sin probar en una impresora

Las medidas por defecto (pala, pórtico, doblador) vienen de FarmLoop y no están comprobadas en la impresora.
La posición del cubilete sale de `printer_dsp.cfg` de la CC2. Empieza siempre por la **prueba de solo doblado**
con una pieza pequeña, mirando la impresora y con la mano cerca de la parada de emergencia. El botón de cancelar
no basta: la impresora ya tiene los movimientos del ciclo en cola y los termina igualmente.

En OrcaSlicer activa **Etiquetar objetos** para que el barrido por columnas use la posición y la altura de
cada pieza; sin etiquetas barre todas las columnas a la altura de la pieza más alta.
