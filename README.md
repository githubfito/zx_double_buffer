# zx_double_buffer

Para gestionar el doble buffer y los bancos en el ZX Spectrum 128K,
se utiliza principalmente el puerto 32765 ($7FFD).
Este puerto es "el guardián" que decide qué banco se ve y qué banco está conectado
en la ventana superior.

Comandos OUT y cómo se estructuran los valores:
1. El Puerto $7FFD (32765)
   Este puerto funciona enviando un byte donde cada bit controla una característica distinta:
     Bits 0-2: Seleccionan el banco de RAM (0 al 7) que aparecerá en el rango 49152-65535.
     Bit 3: Selecciona qué banco se envía a la televisión (Pantalla).
         - 0: Muestra el Banco 5 (Pantalla normal).
         - 1: Muestra el Banco 7 (Pantalla Shadow).
     Bit 4: Selecciona la ROM (normalmente no lo tocarás).
     Bit 5: BLOQUEO. Si pones este bit a 1, el Spectrum se "bloquea" en modo 128K y no podrás cambiar más de banco hasta resetear. ¡Cuidado con este bit!
   
--------------------------------------------------------------------------------------------------
2. Comandos para cambiar la Pantalla Visible
   Si solo quieres cambiar qué pantalla ve el usuario sin cambiar el banco que tienes "mapeado"
   arriba (suponiendo que usas el Banco 0 para datos):

      Acción                Comando ZX Basic                  Explicación
      Ver Pantalla Normal    OUT 32765, 16                    Bit 4 (ROM) activo, Bit 3 (Pantalla) en 0.
      Ver Pantalla Shadow    OUT 32765, 24                    Bit 4 activo, Bit 3 en 1 ($16 + 8 = 24$).
--------------------------------------------------------------------------------------------------

3. Comandos para cambiar el Banco de Trabajo ($C000)
   Para pintar tus sprites en la zona alta, primero debes conectar el banco adecuado.
   Aquí es donde debes tener cuidado de no machacar tus datos si tu código está por encima de 49151.

       Acción              Comando ZX Basic           Resultado en 49152-65535
       Conectar Banco 0    OUT 32765, 16              Es el estado por defecto (datos/BASIC).
       Conectar Banco 7    OUT 32765, 23              Conecta la Shadow Screen para poder escribir en ella ($16 + 7$).
       Conectar Banco 1    OUT 32765, 17              Ideal para guardar Sprites extra ($16 + 1$).

4. Combinando ambos (El Doble Buffer real)
   Lo normal es que quieras cambiar la pantalla visible y, a la vez,
   el banco de RAM para poder limpiar la pantalla que acaba de quedar oculta.

      * Si estás viendo la Pantalla 5 y quieres pintar en la 7:
            OUT 32765, 16 + 7 (Valor 23)
            --> Sigues viendo la 5, pero en la dirección 49152 ahora "está" la pantalla 7 lista para recibir píxeles.
      * Si estás viendo la Pantalla 7 y quieres pintar en la 5:
            OUT 32765, 24 + 5 (Valor 29)
            -->El usuario ve la 7, pero en la dirección 49152 "aparece" la pantalla 5 para que la prepares.

   No uses la memoria por encima de 49151 para tu programa si vas a ejecutar estos OUT
   o podrías sufrir un "crash" al desaparecer tu propio código de la vista del procesador.

* Utilizar directiva --org o org para aprender a limitar tu código por debajo de la dirección 49152 y evitar colisiones con el banco 7.
