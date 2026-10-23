8.1.1. Respaldo

1. La copia es consistente porque se hace tras un SHUTDOWN IMMEDIATE y con el contenedor parado, así que los archivos están en el mismo punto. Con la base abierta sería inconsistente y solo serviría con ARCHIVELOG y RMAN (Lab 5). Evidencia: script/20261022T155047Z_00-backup-lab3.script.log, sección 4.

2. El manifiesto describe el contenido de la copia (versión, base, SCN, archivos) y SHA256SUMS guarda la suma SHA-256 de cada archivo. Antes de tocar nada, restaurar-estado.sh comprueba con sha256sum --check que la copia está íntegra; si falla, se detiene sin modificar nada. Después pide escribir el nombre del volumen para confirmar.

3. RPO: todo lo hecho desde la copia 20261022T155047Z_lab3-final (SCN 4154316). RTO: el tiempo que tarda restaurar-estado.sh en restaurar esa copia de 513 MB; se medirá en el simulacro del Lab 5.

8.1.2. Diagnóstico y correcciones

4. V000 usó rutas relativas y, sin destino OMF, Oracle creó los datafiles en $ORACLE_HOME/dbs. Lo habría evitado definir db_create_file_dest. En mi caso dbs es un enlace al volumen (/opt/oracle/oradata/dbconfig/FREE/dbs). Evidencia: script/20261022T160146Z_01-diagnostico.script.log.

5. Porque RENAME DATAFILE solo cambia el control file, no mueve el archivo; además exige el tablespace OFFLINE. Por eso se copia el archivo y se usa el método offline (sin Online Move Datafile en Free, ORA-00439). Evidencia: script/20261022T160412Z_03-reubicar-datafiles.script.log.

6. Porque V000 ya está aplicado y en el historial; se crea V003 como cambio nuevo. Borrar la contraseña del archivo no basta, ya que sigue en el historial de Git; invalidarla (cuentas de solo esquema) la deja inservible. Evidencia: spool/20261022T160650Z_04-V003__cuentas_esquema_sin_password.spool.log.

7. Demuestra que la base sigue al borrar el contenedor porque los datos están en el volumen. Se hace después de reubicar porque, si los datafiles estuvieran fuera del volumen, el docker rm los perdería. Evidencia: script/20261022T162012Z_03-prueba-persistencia.script.log.

8.1.3. Instancia y arranque

8. Instancia: INSTANCE_NAME = FREE, HOST_NAME = 00c7fe56d583, STATUS = OPEN. Base de datos: NAME = FREE, LOG_MODE = ARCHIVELOG, CDB = YES. Evidencia: script/20261022T160146Z_01-diagnostico.script.log, secciones 1 y 2.

9. NOMOUNT: instancia (SPFILE, SGA y procesos). MOUNT: lee el control file. OPEN: abre datafiles y redo, y el diccionario. ORA-01507 sale en NOMOUNT al consultar v$database (no hay control file leído). ORA-01219 sale en MOUNT al consultar dba_tables (el diccionario está en SYSTEM, aún cerrado). Evidencia: script/20261023T085257Z_06-arranque-por-etapas.script.log.

10. Porque tiene guardado su estado con SAVE STATE (dba_pdb_saved_states). Si no se abriera, ejecutaría ALTER PLUGGABLE DATABASE FREEPDB1 OPEN; y luego ALTER PLUGGABLE DATABASE FREEPDB1 SAVE STATE;.

11. Ambos son ruido conocido. ORA-00800 [Set Priority Failed]: el contenedor no tiene la capacidad CAP_SYS_NICE y Oracle no puede subir la prioridad de VKTM/LGWR; es un aviso "soft". ORA-20013 (DBMS_QOPATCH): el inventario de parches no se puede leer en una imagen de contenedor. La base funciona; se documentan y se excluyen de las alertas. Evidencia: script/20261023T085447Z_06-alert-log.script.log.

8.1.4. Parámetros

12. v$parameter muestra el valor en memoria y v$spparameter el del SPFILE. Con db_files: tras SCOPE = SPFILE salió 200 en memoria y 300 en SPFILE (pendiente); tras RESET, 200 y "(no especificado)". Evidencia: script/20261023T085958Z_07-parametros-cambios.script.log.

13. SCOPE=BOTH cambia el valor en memoria y en el SPFILE, y sobrevive al reinicio. El valor por defecto con SPFILE es BOTH.

14. Porque PROCESSES dimensiona estructuras de la SGA que se reservan al arrancar, así que en la práctica es estático (ORA-02095). Hay que usar SCOPE=SPFILE y reiniciar.

15. El PFILE versionado es una foto de la configuración; con git diff se detectan cambios no documentados. Eso es la deriva de configuración (configuration drift). Evidencia: script/20261023T090140Z_07-pfile.script.log.

8.1.5. Memoria y procesos

16. ASMM. memory_target = 0 (no es AMM), sga_target = 1536M (Oracle reparte la SGA) y pga_aggregate_target = 512M (la PGA va aparte). Evidencia: spool/20261023T090521Z_08-memoria.spool.log.

17. Ronda 1 (caché fría): 359 lecturas lógicas y 254 físicas. Ronda 2 (caché caliente): 247 lógicas y 0 físicas. Tasa de acierto de la ronda 2: 1 − 0/247 = 100 %. Evidencia: script/20261023T091149Z_08-experimento-cache.script.log.

18. Porque el RSS de cada proceso incluye la SGA compartida, que aparece repetida en todos; sumarlos daría una cifra mucho mayor que la real. Para la memoria real se usa v$sga, v$pgastat o docker stats.

19. El SPID de LGWR en v$process es 1649 (spool/20261023T091347Z_09-procesos.spool.log) y es el mismo PID que sale en /proc dentro del contenedor (script/20261023T091535Z_09-procesos-so.script.log). docker top muestra el PID del anfitrión: es el mismo proceso en dos espacios de nombres de PID distintos.

20. PMON detecta un proceso de usuario huérfano, deshace su transacción y libera sus bloqueos (p. ej., un portátil que se apaga a mitad de un UPDATE). SMON hace la recuperación de la instancia (p. ej., tras un fallo, al arrancar).

8.1.6. Seguridad y trabajo profesional

21. Por mínimo privilegio: para mirar no hace falta poder romper. C##OPS_RO es de solo lectura, así que no puede modificar datos ni parámetros. Evidencia: script/20261022T163158Z_02-cuenta-observabilidad.script.log.

22. La contraseña de SYS en la línea de comandos (visible con ps y en el historial de la shell), corregida con autenticación del sistema operativo; y las contraseñas de los propietarios de esquema en V000, en el historial de Git, corregidas con V003.
