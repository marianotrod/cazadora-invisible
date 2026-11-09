# Roadmap — La Cazadora Invisible (versión propia)

Documento vivo. Acá se anotan las ideas y decisiones de diseño a medida que aparecen.
Con el tiempo, lo que esté acá se pasa al GDD.

Estados: **Decidido** (se va a hacer así) · **Idea** (para pensar o definir) · **Hecho**.

---

## Principio general: todo funciona por sistemas

Nada de comportamientos armados a mano para un caso puntual. Cada cosa del juego
(cazadora, guardias, animales, objetos) se arma con **sistemas genéricos** que se
combinan entre sí, para que aparezcan interacciones que no se programaron una por una.

- **Decidido.** La percepción (vista y oído), los ruidos, los estados (dormido,
  desmayado, alerta…) y las reacciones son sistemas que cualquier entidad puede usar.
- **Decidido.** Lo que se calibra (alcances, tiempos, velocidades, costos) va en un lugar
  central de configuración, no en números sueltos por el código.
- **Ejemplo que el sistema tiene que permitir sin código especial:** dormir a un animal
  con una flecha anestésica, cargarlo hasta cerca de los guardias, dejarlo ahí, despertarlo
  con una distracción y que, al ver a los guardias, los ataque según su propio comportamiento.

---

## Niveles y editor

- **Decidido.** Editor de niveles externo, hecho en Phaser, más un servidor Python local.
  Guardar escribe el nivel en el proyecto; Probar abre Godot directo en ese nivel.
  Es más simple que el editor de Godot, del que no hacen falta la mayoría de las opciones.
- **Decidido.** Cada nivel es un archivo JSON con capas separadas:
  - `logica`: cómo funciona cada celda. Es la única fuente de verdad para la física y la IA.
  - `detras`, `principal`, `frente`: cómo se ve. `frente` se dibuja delante de la cazadora (pasto, arbustos).
  - `objetos`: guardias, carcajes, entradas, salidas, etc., con sus propiedades.
  - Cabecera: nombre, música, capas de parallax, etc.
- **Decidido.** La lógica y la skin son independientes. Se pinta el tipo en `logica` y,
  aparte, el dibujo en la capa visual. Ejemplo: varias escaleras que funcionan igual pero
  se ven distintas (soga, madera, piedra). Lo mismo para todos los tipos.
  - Idea: un pincel opcional que pinte lógica y skin a la vez para combinaciones frecuentes.
- **Decidido.** Una sola hoja de tiles de ancho fijo que crece solo hacia abajo: agregar
  tiles nuevos no cambia los que ya se usan.
- **Decidido.** No se conserva el mapa actual (`mundo.tscn`). Los niveles se rehacen con el editor.
  Al principio, un nivel de prueba chico que use todas las funcionalidades.
- **Decidido.** Sin generador automático de niveles.
- **Decidido.** Red de niveles:
  - Cada nivel tiene varias **entradas** (con nombre) y **salidas**.
  - Una salida puede ser una **puerta** dentro del nivel o un **cartel** cerca del borde.
  - Cada salida indica a qué nivel y a qué entrada lleva.
  - Idea: agrupar niveles en **misiones** (nombre a definir).

### Tipos de lógica (propuesta inicial)

| Tipo | Cazadora | Guardias |
|---|---|---|
| vacío | pasa | pasa |
| sólido | no pasa | no pasa |
| pasarela | se pisa, se atraviesa saltando desde abajo, se baja | suben a través y bajan |
| escalera | sube y baja | suben y bajan |
| pasto | pasa, la oculta un poco | pasa |
| arbusto | pasa, la oculta más | pasa |
| muerte | muere | la evitan |

---

## Movimiento de la cazadora

- **Decidido.** Las pasarelas son una pieza central del diseño de niveles, no solo las
  ramas: con ellas se arman paredones que se suben y niveles de varias alturas.
- **Idea.** Animación de trepar de frente, para cuando sube por una pasarela sin agarrarse
  de un costado.
- **Idea.** Shimmy: colgada de una pasarela, se puede desplazar de costado.

---

## Sigilo: percepción y ruido

- **Decidido.** La detección (por vista y por ruido) funciona por sistemas, no armada a mano
  para cada caso. Hoy está muy hardcodeada y hay que rehacerla.
- **Decidido.** Los guardias ven a través de las pasarelas.
- **Idea.** Colgada de una pasarela y quieta, no la ven desde arriba. Si hace shimmy, hace
  algo de ruido: la pueden ver desde abajo, pero no desde arriba, salvo que llame la atención.
- **Decidido.** Cada ruido tiene su alcance ajustable.

---

## Enemigos y otras entidades

- **Decidido.** Los guardias actuales solo saltan una celda. Subir paredones es una ventaja
  de la cazadora.
- **Idea.** Algún enemigo nuevo podría saltar o trepar más.
- **Decidido.** El pathfinding actual funciona bien: se adapta (pasarelas, escaleras desde
  la lógica), no se reescribe.
- **Idea.** Animales con su propio comportamiento: se pueden dormir, cargar, soltar y
  despertar; reaccionan a lo que perciben (ver el principio general).

---

## Objetos y eventos del nivel

- **Idea.** Puente que cae, como cutscene:
  - El resto del juego se congela y se reproduce la caída animada cuadro a cuadro
    (archivo reemplazable).
  - Antes de caer, sus celdas son impasables; después, son pasarelas con la skin del puente caído.
  - Se arma de forma general, para reusarlo en otras cosas que cambien el mapa (puertas, derrumbes).

---

## Pendientes de definir

- Nombre y estructura de las "misiones".
- Lista de ruidos con sus alcances.
- Detalle de los sistemas de percepción y estados (qué tiene cada entidad y cómo se combinan).
