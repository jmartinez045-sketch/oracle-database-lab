Pregunta 1
Salieron 4 errores ORA- en 24 h por arranques del contenedor como ORA-00800. Sin línea base no es alarma.

Pregunta 2
WEDNESDAY_WINDOW empieza a las 22:00 y dura 4 h. Corren auto optimizer stats collection, auto space advisor y sql tuning advisor.

Pregunta 3
Con SCOPE=SPFILE, VSPPARAMETERtenía250yVPARAMETER seguía en 200 porque db_files es estático. Sin RESET habría arrancado en 250.

Pregunta 4
RESET quita el ajuste del SPFILE y deja el valor por defecto; en db_files no había línea y en processes sí estaba *.processes=200.

Pregunta 5
C##OPS_RO dio ORA-01109 porque FREEPDB1 estaba MOUNTED sin estado guardado; el aviso de abierta aplica a instancia y CDB$ROOT.

Pregunta 6
SID 225, SPID 2033, PL/SQL lock timer. MODULE matriculas, ACTION cierre-periodo y CLIENT_IDENTIFIER ana.garcia@ucam concretan operación y usuario frente al USERNAME genérico.

Pregunta 7
Tras CANCEL SQL queda 1 transacción abierta por el SLEEP; tras KILL SESSION quedan 0 sesiones y la fila 101 vuelve a ACTIVO por rollback.

Pregunta 8
common_user_prefix es C##, así que los usuarios comunes en CDB$ROOT deben empezar por C##.

Pregunta 9
C##OPS_RO en CDBROOT:ALL_T ABLES0,DBA_U SERS37,CDB_U SERS37enCON_I D1y47enCON_I D3,VDATAFILE 17. En FREEPDB1: ALL_TABLES 0, 
DBA_USERS 47, CDB_USERS 47 en CON_ID 3, V$DATAFILE 10. APP_ACADEMIA en FREEPDB1: ALL_TABLES 1 y ORA-00942 en el resto. 
En FREEPDB1 se ven solo los 10 datafiles de la PDB y no los 17 de la CDB.

Pregunta 10
5377 lecturas lógicas por recorrido, DB time 0.68 s y DB CPU 0.58 s; casi todo el tiempo es CPU en caché sin apenas esperas.

Pregunta 11
En TBS_ACADEMIA_HIST antes de la carga había 10 MB asignados, 3.9 MB libres y 40 MB máximos; alertaría sobre el máximo de 40 MB por autoextensión.

Pregunta 12
Insertó SYS y la cuota la paga el dueño de CARGA_HIST; ampliar el tablespace no arreglaba el ORA-01536 porque faltaba cuota de 2 MB.

Pregunta 13
Unos 12 s de pausa de 17:41:04 a 17:41:17 con SUSPEND_TIME a las 17:41:03; en el alert log salen los mensajes de suspended y resumed de carga-historico.

Pregunta 14
Antes del DELETE: 20000 filas, 2944 bloques, 23 MB; tras DELETE: 2000 filas, 2944 bloques, 23 MB; tras SHRINK SPACE: 2000 filas, 
304 bloques, 2.4 MB. DELETE no baja la marca de agua alta y SHRINK SPACE sí.

Pregunta 15
DEFAULT tenía FAILED_LOGIN_ATTEMPTS 10, INACTIVE_ACCOUNT_TIME 365, PASSWORD_LIFE_TIME 180, PASSWORD_LOCK_TIME 1 y PASSWORD_VERIFY_FUNCTION NULL; 
PRF_PERSONAS pone 3 intentos, 90 días y ora12c_verify_function para frenar ataques y contraseñas flojas.

Pregunta 16
Ana leyó ACA_MATRICULAS por SELECT ANY TABLE ON SCHEMA admin_academia vía ROL_LECTURA_ACADEMIA, y en NOMBRES dio ORA-41900 porque solo tenía UPDATE en ESTADO.

Pregunta 17
Con telefono no se invalidó nada; con email_institucional solo V_CONTACTO_LAB6; con nombres se invalidaron V_ACTIVOS_LAB6 y F_TOTAL_ACTIVOS_LAB6 por dependencias a nivel de columna.

Pregunta 18
Duplicar la SGA a 3072 MB mantiene ESTD_DB_TIME_FACTOR en 1; la PGA tiene OVERALLOC 2 por debajo de 384 MB en 256 MB y 128 MB.

Pregunta 19
247 lecturas lógicas, con cr=247 en la línea FETCH de la traza y query = 247 en tkprof.

Pregunta 20
INC-601: SID 226 bloqueado por 220,3540 en enq: TX - row lock contention, resuelto con ALTER SYSTEM KILL SESSION '220,3540' IMMEDIATE; 
y 0 bloqueos con fila 101 ACTIVO. INC-602: academia_app en NO, arrancado con DBMS_SERVICE.START_SERVICE y SAVE STATE, 
validado en SI y app.sh. INC-603: APP_ACADEMIA LOCKED, desbloqueado con ALTER USER app_academia ACCOUNT UNLOCK;, 
validado en OPEN y app.sh.
