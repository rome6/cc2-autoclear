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
  1. aparca el cabezal sobre el cubilete (X 51, Y 264), sube la cama hasta dejar las piezas a 5 mm de la boquilla
     y enfría con el ventilador de capa y el auxiliar (`M106 P2`) hasta la temperatura de despegue;
  2. dobla la placa contra el doblador (baja la cama, la mantiene y repite);
  3. barre de arriba abajo por columnas. Primero sube la cama con el cabezal aún sobre el cubilete,
     luego el cabezal cruza a su columna y la cama hace el último tramo. Cada columna solo recibe las pasadas
     que llegan a sus propias piezas (según las etiquetas de objeto de Orca) y empieza justo detrás de ellas;
  4. sopla sobre la placa limpia y apaga los motores.
- **Rechaza** archivos que no son de la CC2, que ya tienen un ciclo, con piezas en la franja del fondo que la
  pala no alcanza, o cuyos ajustes harían chocar el pórtico.
- **Copias:** puede repetir la impresión N veces en un solo archivo; cada copia imprime, limpia y empieza la siguiente.
- **Prueba de solo doblado:** enfría y dobla sin barrer, para probar el doblador.

## La página

- Deslizadores para la altura de la primera pasada, la separación y el número de pasadas (enlazados),
  el doblador y el enfriado. Solo dejan elegir valores seguros para la pieza cargada.
- Vista lateral de las pasadas y un visor 3D que reproduce el G-code real del ciclo: con la animación parada,
  cada pasada es un anillo numerado alrededor de la pieza a la altura donde la golpea la pala.

## Sin probar en una impresora

Las medidas por defecto (pala, pórtico, doblador) vienen de FarmLoop y no están comprobadas en la impresora.
La posición del cubilete sale de `printer_dsp.cfg` de la CC2. Empieza siempre por la **prueba de solo doblado**
con una pieza pequeña, mirando la impresora y con la mano cerca de la parada de emergencia.

En OrcaSlicer activa **Etiquetar objetos** para que el barrido por columnas use la posición y la altura de
cada pieza; sin etiquetas barre todas las columnas a la altura de la pieza más alta.
