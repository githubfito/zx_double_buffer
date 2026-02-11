# zx_double_buffer
<sub>

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
Gestión de Doble Buffer y Shadow Screen (ZX Spectrum 128K)Este documento detalla la configuración de memoria y los comandos necesarios para implementar un sistema de doble buffer utilizando la Shadow Screen del ZX Spectrum 128K en Boriel Basic.1. Los Bancos de PantallaEl hardware del ZX Spectrum 128K permite alternar entre dos bancos físicos para la salida de vídeo:BancoFunciónDirección de Memoria FijaBanco 5Pantalla Principal (Estándar 48K)16384 ($4000)Banco 7Pantalla Shadow (Extra 128K)49152 ($C000)2. El Puerto de Control $7FFD (32765)Para gestionar qué pantalla se muestra y en qué banco se escribe, se utiliza el puerto 32765. El bit más importante para el doble buffer es el Bit 3.Bit 3 = 0: El televisor muestra el Banco 5.Bit 3 = 1: El televisor muestra el Banco 7.Tabla de Conmutación (Doble Buffer Simétrico)Para evitar parpadeos, mientras el usuario ve una pantalla, el programa debe estar dibujando en la otra a través de la "ventana" de memoria superior (49152 - 65535).Estado del JuegoPantalla VisibleBanco para Dibujar (en $C000)Comando OUT 32765, VEscena ABanco 5 (Normal)Banco 7 (Shadow)OUT 32765, 23Escena BBanco 7 (Shadow)Banco 5 (Normal)OUT 32765, 293. Direccionamiento en Funciones de DibujoAl utilizar la ventana superior ($C000) para pintar, las direcciones base para tus funciones de ensamblador (fastChars, paint) deben cambiar según el tipo de datos:Tipo de DatosBase High Byte (Decimal)Dirección HexadecimalPíxeles192$C000Atributos216$D800Nota: Estas bases (192 para píxeles y 216 para atributos) funcionan tanto si el banco paginado en la ventana superior es el 5 como si es el 7.4. Reglas de Oro y LimitacionesUbicación del Código: El código ejecutable y las variables críticas NO deben residir por encima de 49151. Al cambiar de banco con OUT, el código en esa zona desaparecería, provocando un cuelgue del sistema.Sincronización: Utiliza siempre HALT antes de cambiar el banco visible para asegurar que el cambio ocurra durante el retrazado vertical y evitar el efecto de "rayado" (tearing).Comentarios: En Boriel Basic/ugBasic, los comentarios REM deben ir en líneas separadas.Variables: No utilices el símbolo $ al declarar strings ni uses k como nombre de

* Utilizar directiva --org o org para aprender a limitar tu código por debajo de la dirección 49152 y evitar colisiones con el banco 7.

</sub>
