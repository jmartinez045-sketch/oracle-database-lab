## INC-601 · Bloqueo de fila en ACA_ESTUDIANTES por cierre-periodo
- Detección: 17:47 UTC · Ana García de secretaría por ticket INC-601; pantalla colgada al guardar la ficha del alumno 101 en FREEPDB1.
- Impacto: Secretaría no puede guardar el alumno 101; SID 226 lleva bloqueada al menos 109 s en la primera captura y sigue esperando en las siguientes.
- Diagnóstico: Bloque 1: SID 226 en enq: TX - row lock contention bloqueada por SID 220 de APP_ACADEMIA / cierre-periodo. Bloque 2: fila en ACA_ESTUDIANTES, archivo 1024, bloque 783, fila 5. Bloque 3: SID 220 tiene TX con LMODE=6 y BLOCK=1; SID 226 lo pide con REQUEST=6. Bloque 4: USED_UBLK=1 y USED_UREC=1. Bloque 5: ASH confirma la espera de 226 por 220.
- Causa raíz: La sesión 220 dejó abierta una transacción con bloqueo exclusivo de fila TX modo 6 sin hacer commit ni rollback.
- Intervención: ALTER SYSTEM KILL SESSION '220,3540' IMMEDIATE; en FREEPDB1. Es la mínima posible porque solo corta al bloqueador y deshace 1 bloque y 1 registro.
- Validación: Relanzar 08-diagnostico.sql para comprobar que desaparece el bloqueo TX y probar a guardar la ficha del alumno 101 desde secretaría.
- Acción preventiva: Revisar los commits en el proceso cierre-periodo y configurar alertas para bloqueos TX largos.

## INC-602 · Servicio academia_app detenido en FREEPDB1
- Detección: 17:49 UTC · Equipo de matrículas por ticket INC-602; error ORA-12514 al conectar la aplicación.
- Impacto: Ninguna instancia de la aplicación que usa el servicio academia_app puede conectar desde las 17:49 UTC hasta su arranque.
- Diagnóstico: Bloque 6 en la ejecución de las 17:49:37 UTC: CON_ID 3, academia_app con ARRANCADO = NO, mientras FREEPDB1 sigue en SI.
- Causa raíz: El servicio academia_app está detenido en FREEPDB1, por lo que el listener rechaza las conexiones con ORA-12514.
- Intervención: EXEC DBMS_SERVICE.START_SERVICE('academia_app'); en FREEPDB1. Es la mínima posible porque levanta solo ese servicio sin reiniciar la PDB ni la instancia.
- Validación: Comprobar en el bloque 6 de 08-diagnostico.sql que academia_app pasa a ARRANCADO = SI y verificar la conexión de la aplicación.
- Acción preventiva: Monitorizar el estado del servicio academia_app y alertar si pasa a NO o aparecen errores ORA-12514.

## INC-603 · Usuario APP_ACADEMIA bloqueado
- Detección: 17:49 UTC · Equipo de matrículas por ticket INC-603; error ORA-28000 tras el despliegue de las 21:00 y la ejecución del script trimestral de bajas.
- Impacto: La aplicación no puede iniciar sesión con APP_ACADEMIA por ORA-28000 desde el bloqueo a las 17:49 UTC hasta su desbloqueo.
- Diagnóstico: Bloque 7: CON_ID 3, APP_ACADEMIA con ACCOUNT_STATUS = LOCKED, BLOQUEO = 2026-11-25 17:49, PROFILE = DEFAULT e INTENTOS_PERMITIDOS = 10. Al ser LOCKED y no LOCKED(TIMED), fue un bloqueo manual o por script y no por intentos fallidos.
- Causa raíz: La cuenta APP_ACADEMIA quedó en estado LOCKED en FREEPDB1 durante tareas operativas o el script trimestral de bajas.
- Intervención: ALTER USER APP_ACADEMIA ACCOUNT UNLOCK; en FREEPDB1. Es la mínima posible porque solo desbloquea la cuenta sin modificar su contraseña ni su perfil.
- Validación: Comprobar en el bloque 7 de 08-diagnostico.sql que APP_ACADEMIA ya no figura bloqueada y verificar el acceso de la aplicación.
- Acción preventiva: Revisar el script trimestral de bajas y el procedimiento de despliegue para no bloquear cuentas de aplicación, y alertar ante cambios en ACCOUNT_STATUS.
