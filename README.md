# zx_double_buffer

Para gestionar el doble buffer y los bancos en el ZX Spectrum 128K, se utiliza principalmente el puerto 32765 ($7FFD).

Este puerto es "el guardián" que decide qué banco se ve y qué banco está conectado en la ventana superior.

Comandos OUT y cómo se estructuran los valores:

# 1. El Puerto $7FFD (32765)
   
   Este puerto funciona enviando un byte donde cada bit controla una característica distinta:
   
     * Bits 0-2: Seleccionan el banco de RAM (0 al 7) que aparecerá en el rango 49152-65535.   
     * Bit 3: Selecciona qué banco se envía a la televisión (Pantalla).
         - 0: Muestra el Banco 5 (Pantalla normal).
         - 1: Muestra el Banco 7 (Pantalla Shadow).   
     * Bit 4: Selecciona la ROM (normalmente no lo tocarás).   
     * Bit 5: BLOQUEO. Si pones este bit a 1, el Spectrum se "bloquea" en modo 128K y no podrás cambiar más de banco hasta resetear. ¡Cuidado con este bit!   
--------------------------------------------------------------------------------------------------
# 2. Comandos para cambiar la Pantalla Visible
   Si solo quieres cambiar qué pantalla ve el usuario sin cambiar el banco que tienes "mapeado" arriba (suponiendo que usas el Banco 0 para datos):

| Acción | Comando ZX Basic | Explicación |
| ------------ | ------------ | ------------ |
| Ver Pantalla Normal | OUT 32765, 16 | Bit 4 (ROM) activo, Bit 3 (Pantalla) en 0. |
| Ver Pantalla Shadow | OUT 32765, 24 | Bit 4 activo, Bit 3 en 1 ($16 + 8 = 24$). |
--------------------------------------------------------------------------------------------------
# 3. Comandos para cambiar el Banco de Trabajo ($C000)
   Para pintar tus sprites en la zona alta, primero debes conectar el banco adecuado.
   Aquí es donde debes tener cuidado de no machacar tus datos si tu código está por encima de 49151.

|Acción               |Comando ZX Basic           |Resultado en 49152-65535 |
| ------------ | ------------ | ------------ |
|Conectar Banco 0      |OUT 32765, 16              |Es el estado por defecto (datos/BASIC).|
|Conectar Banco 7      |OUT 32765, 23              |Conecta la Shadow Screen para poder escribir en ella ($16 + 7$).|
|Conectar Banco 1      |OUT 32765, 17              |Ideal para guardar Sprites extra ($16 + 1$).|
---
# 4. Combinando ambos (El Doble Buffer real)
   Lo normal es que quieras cambiar la pantalla visible y, a la vez,
   el banco de RAM para poder limpiar la pantalla que acaba de quedar oculta.

      * Si estás viendo la Pantalla 5 y quieres pintar en la 7:
            OUT 32765, 16 + 7 (Valor 23)
            --> Sigues viendo la 5, pero en la dirección 49152 ahora "está" la pantalla 7 lista para recibir píxeles.
      * Si estás viendo la Pantalla 7 y quieres pintar en la 5:
            OUT 32765, 24 + 5 (Valor 29)
            -->El usuario ve la 7, pero en la dirección 49152 "aparece" la pantalla 5 para que la prepares.

   No uses la memoria por encima de 49151 para tu programa si vas a ejecutar estos OUT o podrías sufrir un "crash" al desaparecer tu propio código de la vista del procesador.
   
---
# Gestión de Doble Buffer y Shadow Screen (ZX Spectrum 128K)

Este repositorio contiene implementaciones de bajo nivel para la manipulación de gráficos en el ZX Spectrum 128K utilizando **Boriel Basic** (ZX Basic). El objetivo principal es lograr un movimiento fluido de sprites sin parpadeos (flicker) mediante el uso del **Shadow Screen** (Banco 7).

## 1. Conceptos Fundamentales

El hardware del ZX Spectrum 128K permite alternar entre dos bancos de memoria física para la salida de vídeo. Mientras una pantalla es visible para el usuario, el procesador puede dibujar en la otra de forma oculta.

| Banco de RAM | Función | Dirección de Memoria Fija |
| :--- | :--- | :--- |
| **Banco 5** | Pantalla Principal (Estándar 48K) | `16384` ($4000) |
| **Banco 7** | Pantalla Shadow (Extra 128K) | `49152` ($C000) |



## 2. Control del Puerto `$7FFD` (32765)

La gestión de las pantallas y la paginación de memoria se realiza mediante el puerto `32765`. El control se basa principalmente en el **Bit 3** (Selector de pantalla visible) y los **Bits 0-2** (Banco de RAM en la ventana superior).

### Tabla de Conmutación para Doble Buffer
Para un sistema simétrico, mapeamos el banco donde queremos dibujar en la ventana superior (`$C000 - $FFFF`):

| Estado | Pantalla Visible | Banco de Dibujo (Mapeado en $C000) | Valor `OUT 32765, V` |
| :--- | :--- | :--- | :--- |
| **Escena A** | Banco 5 (Normal) | **Banco 7 (Shadow)** | `23` ($16 + 7) |
| **Escena B** | Banco 7 (Shadow) | **Banco 5 (Normal)** | `29` ($24 + 5) |

## 3. Direccionamiento de Funciones ASM

Al utilizar la ventana superior (`$C000`) para las operaciones de dibujado, las direcciones base para las funciones de píxeles y atributos deben ser constantes, independientemente de si estamos en el Banco 5 o 7.

| Capa de Pantalla | Base High Byte (Decimal) | Dirección Memoria |
| :--- | :--- | :--- |
| **Píxeles** | `192` | `$C000` |
| **Atributos** | `216` | `$D800` |

### Ejemplo de integración en `fastChars`:
```asm
; Cálculo de dirección de pantalla usando base dinámica
LD A, (IX+7)    ; Coordenada Y
AND 24
ADD A, (IX+15)  ; Suma 192 (Píxeles en ventana $C000)
LD H, A

* Utilizar directiva --org o org para aprender a limitar tu código por debajo de la dirección 49152 y evitar colisiones con el banco 7.


