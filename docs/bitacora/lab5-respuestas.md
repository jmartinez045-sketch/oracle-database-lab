# Laboratorio 5 - Respuestas

## Punto de partida y banco de medida

1. En el paso 6 de la copia de lab3-final había cinco datafiles fuera del volumen, y en mi copia de lab4-final (20261101T092146Z_lab4-final) el paso 6 dice "ninguno: el volumen contiene toda la base de datos". Esto demuestra que la reubicación de datafiles del Lab 4 funcionó y que ahora el volumen contiene toda la base.

2. Con las estadísticas de mi banco, CLIENTE tiene 100 000 filas en 1517 bloques, 102 bytes por fila y un bfr de 66. PEDIDO tiene 1 000 000 filas en 5320 bloques, 32 bytes por fila y un bfr de 188. PEDIDO cabe casi el triple de filas por bloque (188 frente a 66) porque su fila es unas tres veces más corta, así que el bfr depende del tamaño de la fila.

3. La primera ejecución puede incluir trabajo que solo se hace una vez, como el análisis de la sentencia y la limpieza diferida de bloques, y eso inflaría las lecturas. Por eso se descarta y se mide la segunda, ya en caliente. En la prueba prueba_pk mi segunda ejecución midió 4 lecturas lógicas, que es la altura del índice (3) más el bloque de la tabla.

## Anatomía y tipos de índice

4. IX_PED_CLIENTE tiene altura 3, 2216 hojas y 6 ramas. El factor de ramificación real es (2216 + 6 - 1) / 6 = 370 y el máximo es 543. Con mil millones de filas saldrían unas 2 208 969 hojas, altura 4 y 5 lecturas para encontrar una fila.

5. La primera entrada de la hoja de PK_CLIENTE guarda data:(6): 00 00 03 1b 00 00. Los cuatro primeros bytes son el bloque (0x31B = 795) y los dos últimos la ranura (0), que corresponden al cliente 1 (clave c1 02). No aparece el número de archivo porque el tablespace es bigfile y solo tiene uno.

6. Tres pruebas de la Parte D. En la búsqueda por DNI (Q13) el índice único baja de 1486 lecturas lógicas a 3, y además impide duplicados con ORA-00001. En el historial del cliente (Q7) se pasa de 15 lecturas con ordenación a 7 con el índice compuesto (id_cliente, fecha) y a 3 con el cubriente, que ya no toca la tabla. En la consulta con cuatro filtros (Q14) los bitmaps bajan de 4073 a 528 lecturas porque BITMAP AND combina los cuatro índices.

## Agrupamiento, equilibrio y sargabilidad

7. IX_PED_FECHA tiene un clustering factor de 5220, casi los 5320 bloques de la tabla, porque los pedidos se cargaron ordenados por fecha. IX_PED_CLIENTE tiene 999 041, casi el millón de filas, porque el cliente se reparte al azar. Un REBUILD no lo arregla: el factor es propiedad de la tabla y al reconstruir el índice el orden de las filas en disco sigue igual. Habría que reordenar la tabla.

8. En mi curva el optimizador pasa del índice al recorrido completo entre el 0,1 % y el 0,2 % de las filas, y la fórmula con lectura aleatoria cinco veces más cara da (1/188)·(1/5) = 0,11 %. En lecturas lógicas la fórmula da 0,53 %. El índice con factor malo ya lee más que la tabla a partir del 1 % (9920 lecturas frente a 5232), aunque al 0,5 % todavía lee menos (4875). El de factor bueno solo pierde al 100 %, con 7875 lecturas frente a 5232.

9. LIKE 'Gonz%' es sargable y usa el índice, pero devuelve 2056 filas, el 2 % de los clientes, y el factor de agrupamiento es malo, así que está por encima del punto de equilibrio. Lee 2024 lecturas lógicas frente a las 1523 del recorrido completo de CLIENTE (Q4a). Que una condición sea sargable no significa que el índice convenga.

## Coste de escritura y mantenimiento

10. Los bloques modificados por fila en el INSERT fueron 10,2 con 4 índices, 20,3 con 9 (propuesta del compañero) y 12,1 con 5 (V005). De la propuesta rechacé IX_PED_CANAL (3 valores, 33,3 % cada uno) e IX_PED_ESTADO (4 valores, 25 % cada uno), porque superan con mucho el 0,5 % de punto de equilibrio. Para los pendientes usé el índice parcial ix_ped_pendientes (54 hojas frente a 2933). También rechacé IX_PED_FEC_CLI, que salía sin uso y es redundante porque IX_PED_FECHA ya cubre su prefijo. Retiré IX_PED_IMPORTE tras dejarlo invisible: Q2 siguió leyendo 342 lecturas sin él.

11. En el índice con claves al azar hubo 595 divisiones, ninguna 90/10, y las hojas quedaron al 75,4 %. Con claves crecientes hubo 398 divisiones, todas 90/10, y las hojas quedaron al 99,9 %. En el patrón cola el índice tiene 425 hojas, 224 vacías y 70 497 entradas borradas (el 41 %), con la parte izquierda del mapa vacía y la derecha llena.

12. Elegiría COALESCE en producción, porque es en línea y sin bloqueo, y en mi medida bajó "dame el siguiente mensaje" de 226 lecturas a 2 y las hojas de 425 a 201. SHRINK SPACE haría lo mismo y además devuelve espacio (el segmento pasó de 4 a 1,69 MB), y REBUILD ONLINE lo dejaría en 223 hojas pero con espacio doble y más redo, así que solo lo usaría con una causa medida. La solución de fondo es de diseño: particionar por fecha.

## SCN, control files, redo y FRA

13. Oracle compara el SCN del último checkpoint guardado en el control file con el de la cabecera de cada datafile. En mi evidencia los dos valían 7264539 y quedaron como sincronizado, y tras el checkpoint forzado ambos pasaron a 7271105. Si coinciden no hace falta recuperación. El PDB$SEED se queda en 2098101 porque está en solo lectura.

14. Tener las tres copias en el mismo volumen protege del borrado o la corrupción de una sola copia: en el simulacro B borré control03.ctl, el arranque falló con ORA-00205 y lo arreglé copiando control01.ctl. No protege de perder el disco, porque las tres se van juntas. En producción cada copia se pondría en un disco o una controladora distinta.

15. CURRENT es el grupo en el que escribe LGWR. ACTIVE ya no es el actual, pero aún falta el checkpoint y se necesita para recuperar la instancia. INACTIVE tiene sus cambios ya en los datafiles. En mi evidencia el grupo 1 era INACTIVE y el 2 CURRENT, y tras el cambio de log pasaron a ACTIVE y CURRENT. Oracle rechaza los tres borrados del paso 4: ORA-01623 por borrar el grupo CURRENT, ORA-01609 por borrar un miembro del CURRENT y ORA-00350 al borrar otro grupo, porque en ARCHIVELOG ese grupo todavía necesitaba archivarse.

16. Al llenarse la FRA sale el aviso ORA-19815 y después ORA-19809 por superar el límite. ARCn no puede archivar (ORA-16038) y los grupos quedan sin archivar: en mi evidencia el grupo 2, secuencia 17, aparecía CURRENT con ARC = NO. LGWR no puede reutilizar un grupo sin archivar, las sesiones esperan en log file switch (archiving needed) y las conexiones nuevas reciben ORA-00257. Al final se detiene toda la actividad que genera redo. Lo resolví devolviendo espacio a la FRA, tras lo cual ARCn archivó y los grupos volvieron a ARC = YES.

## Undo, WAL y simulacros

17. La sesión A cambió la fila 101 sin confirmar y aun así B vio ACTIVO, porque Oracle reconstruye el valor confirmado con el undo de A (la transacción de A usaba 1 bloque y 1 registro de undo en el segmento 2). En el paso 4, B no pudo bloquear la fila: dio ORA-00054 a los 3 s, porque A tenía un bloqueo TX en modo 6. Leer, en cambio, no necesita bloqueo. Por eso los lectores no bloquean a los escritores.

18. Para deshacer un INSERT basta con guardar el ROWID, y 10 000 filas dejaron 143 bloques de undo (1144 KB). El DELETE tiene que guardar la fila entera y sus entradas de índice, y dejó 427 bloques (3416 KB), casi el triple. El ROLLBACK también genera redo porque deshacer es aplicar los cambios inversos: el del DELETE generó 4701 KB.

19. Justo después del COMMIT, la garantía está en el redo que LGWR ya escribió (1779 KB en 3 escrituras), mientras que DBWn había escrito 0 bloques y 231 bloques de la tabla seguían sucios en memoria. Después del checkpoint, DBWn escribe 355 bloques en los datafiles y los bloques sucios de la tabla bajan a 0.

20. Tras el SHUTDOWN ABORT, SMON empezó la recuperación en el checkpoint (logseq 18, bloque 42588) y leyó 13 KB de redo con 13 bloques que recuperar: ese es el roll forward. Después hizo el roll back de lo no confirmado con el undo. Por eso la fila 102, que estaba confirmada, conservó EGRESADO, y la 103, que no lo estaba, volvió a ACTIVO. Bastó un STARTUP normal y la base abrió en la secuencia de log 19.

21. En el simulacro C el RTO fue de 123 s y el RPO fueron los 3 s entre la copia y el fallo. En el D el RTO fue de 17 s (18 s desde el fallo) y el RPO de 0, porque el cambio de la fila 104, posterior a la copia, sobrevivió. Son tan distintos porque en el C se restaura la base entera desde una copia sin redo posterior, y en el D se restaura un solo archivo y se rehace con el redo archivado. En mi D Oracle cerró FREEPDB1 (ORA-63999) y hubo que abrirla de nuevo, y eso ya está dentro del RTO.

22. Clasifico los errores de mi bloque 5 del runbook. ORA-01113, ORA-01110 y ORA-63999 de las 10:28 son un efecto del laboratorio: vienen del simulacro D (Parte O), cuando borré el datafile 16 de academia y Oracle cerró FREEPDB1, por eso también pide recuperación el archivo 21, el de TiendaSur. ORA-00202 y ORA-205 de las 10:15 son también un efecto del laboratorio, del simulacro B, cuando borré control03.ctl y el arranque falló al montar. Los ORA-00800 Set Priority Failed de LGWR y VKTM (10:23 y 10:25) son ruido conocido del entorno, que ya clasifiqué en el Lab 4 y salen al arrancar la instancia en el contenedor. No hay ningún incidente real.
