8.1.1. Docker
1. ¿Qué diferencia hay entre una imagen y un contenedor? Usa como ejemplo lo que hiciste en los ejercicios
G2 y G4.
Una imagen es una plantilla que solamente sirve para su lectura, un contenedor es una instancia de esa plantilla, puedes crear multiples contenedores usando la imagen como molde.
2. En el Ejercicio G5 el archivo nota.txt desapareció y en el G6 no. Explica por qué.
En G5 al terminar borramos el contenedor junto con todo el sistema de archivos que contenía este, en el ejercicio G6 el dato estaba almacenado en un volumen externo independiente al contenedor.
3. ¿Qué diferencia hay entre docker ps y docker ps -a, y qué significa STATUS = Exited (0)?
Docker ps nos muestra los contenedores que están ejecutandose ahora mismo, mientras que al añadirle -a se nos muestran también los que estan creados pero no ejecutandose. Exited (0) nos dice que ha terminado correctamente y sin errores.
4. En -p 8181:8181, ¿qué número corresponde a tu equipo y cuál al contenedor? ¿Qué pasaría con -p 80:8080
en el ejercicio de nginx?
En -p 8181:8181, el primer numero corresponde al del equipo host y el segundo al conetendor. Con -p 80:8080, nginx escucharía en el 8080 dentro del contenedor, pero como nginx escucha en el 80 la página no cargaría.
5. ¿Por qué un contenedor de Oracle se queda en marcha y el de hello-world termina solo?
El contenedor de oracle tiene un proceso principal que se mantiene activo (el motor de BD), sin embargo el hello-world simplemente termina en cuanto se imprime el mensaje y el proceso principal termina.
6. ¿Qué es el digest de una imagen y por qué lo registramos si ya sabemos que usamos :latest?
Digest es la huella sha256 exacta de la imagen, se registra porque :lastest va cambiando con el tiempo, digest no
7. ¿Qué comando borraría realmente los datos de Oracle? ¿Por qué docker rm oralab-26ai no lo hace?
docker volume rm oralab-26ai-data borra los datos, docker rm no los borra por lo que hemos explicado antes, eliminar el contenedor no eliminará los datos si estos están alojados en un volumen separado.


8.1.2. Git, organización y evidencia
8. ¿Por qué este laboratorio se hace dentro del repositorio oracle-database-lab, con Issue, branch y Pull Request, en vez de en una carpeta aparte?
Para mantener Issue, historial y la protección de main del repositorio del curso, fuera de este entorno no tenemos esas características. 
9. ¿Qué diferencia hay entre source 00-config.sh y bash 00-config.sh? ¿Por qué usamos source?
source ejecuta el script nuestro shell actual (las variables quedan disponibles); bash lo ejecuta en un proceso hijo que desaparece al terminar.
10. Explica cada parte del nombre 20260915T091230Z_02-docker.script.log.
20260915T091230Z: marca de tiempo UTC ISO 8601
02: número del paso
docker: descripción
.script.log: salida de la terminal
11. ¿Para qué sirve .gitattributes y qué error evita?
Nos sirve para normalizar los saltos de linea para en caso de ejecutar sobre otro SO no tener problemas, ya que cada uno los representa a su manera.
12. ¿Por qué en este Pull Request elegimos Create a merge commit en lugar de Squash and merge?
Porque de esta manera conservamos el historial completo de comits individuales en vez de aplastarlos todos en uno solo


8.1.3. Seguridad
13. Describe las cuatro capas de la estrategia de contraseñas (Parte D) y qué pasaría si te saltas la primera.
Capa 1: .gitignore
Capa 2 Plantilla .env.example
Capa 3: .env con los datos reales
Capa 4: Cargar con source sin necesidad de teclear la contraseña.
Si nos saltamos la capa 1, a la hora de hacer un commit se nos añadirá al repositorio el archivo .env con todas las contraseñas y constantes, dando acceso a nuestra información sensible.
14. ¿Por qué no escribimos la contraseña directamente en el comando docker run, aunque el script no se suba a Git?
Porque todo lo que usemos por comandos queda registrado en ~/bash_history en texto plano para siempre.
15. Si descubres tu contraseña en un commit ya publicado, ¿basta con borrarla en un commit nuevo? ¿Qué debes hacer?
No, ya que una vez esta ha sido usada en un commit esta queda registrada para siempre, la solución es cambiarla y dar ese repositorio como comprometido.


8.1.4. Oracle y herramientas
16. ¿Por qué no usamos SPOOL ni @archivo.sql con sqlplus dentro del contenedor, y qué hicimos en su lugar?
Porque sqlplus corre dentro del contenedor: un SPOOL o @archivo buscaría rutas que no existen en nuestro equipo. Usamos redireccion < y tee desde el host.
17. ¿Qué hace WHENEVER SQLERROR EXIT SQL.SQLCODE al inicio de V000 y V001, y qué pasaría sin esa línea?
Para la ejecución en el momento en el que ocurre un error, sin esto al encontrar un error seguiría ejecutando el resto de las sentencias en un estado incorrecto, podiendo llevar a errores.
18. ¿Qué es una migración y por qué V000 y V001 no se deben editar una vez aplicadas?
Es un script versionado y numerado que lleva la BD de un estado al siguiente; no se editan porque herramientas de tipo Flyway/Liquibase asumen que una vez aplicadas sin inmutables - los cambios van en una migración nueva.
19. ¿Por qué en SQL Developer se usa el servicio FREEPDB1 y no FREE ni un SID?
FREEPDB1 es la base de datos conectable donde trabajamos realmente; FREE es el contenedor raiz el cual no está destinado al trabajo diario.
20. ¿Qué aporta SQLcl frente a SQL*Plus, y por qué un DBA debe dominar ambas?
SQLcl añade autocompletado, historial, salida en formatos modernos (JSON) y soporte de migraciones tipo Liquibase; un DBA debe dominar ambas porque SQLPlus esta en todos los servidores de Oracle.


8.1.5. Entorno de trabajo
21. ¿Por qué el curso pasa de Git Bash a Ubuntu en WSL 2? Da al menos dos problemas concretos de Git Bash que desaparecen en Ubuntu.
Porque Git Bash es una emulacion: rompe rutas y argumentos de Docker, necesita winpty para terminales interactivas y le faltan utilidades como free/ss/htop.
22. ¿Por qué clonamos el repositorio en ~/oracle-database-lab y no trabajamos sobre la carpeta de Windows (/mnt/c/...)? ¿Y por qué recomendamos bash frente a zsh para los scripts del curso?
Porque /mnt/c/... cruza dos sistemas de archivos distintos, loq que ralentiza Git/Docker y rompe permisos de ejecución. Se recomienda usar bash porque es la shell existente por defecto en todos los servidores de Linux.
