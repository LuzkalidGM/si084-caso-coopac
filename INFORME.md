# Informe de auditoría · Core financiero de la COOPAC Santa Rosa

**SI-084 · Auditoría de Sistemas** · Examen práctico de Unidad I

| | |
|---|---|
| **Apellidos y nombres** | Gutierrez Mamani Gabriela Luzkalid |
| **Código de estudiante** | 2022074263 |
| **URL del repositorio** | `[https://github.com/LuzkalidGM/si084-caso-coopac/tree/examen-u1](https://github.com/LuzkalidGM/si084-caso-coopac.git)` |
| **Fecha** | 30/09/2026 |

## 1. Resultados de los procedimientos

| Regla | Resultado, con cifras | ¿Cumple? | Archivo de evidencia |
|---|---|---|---|
| R1 | El servidor `sr_bd` publica PostgreSQL en `0.0.0.0:55432` y `[::]:55432`, dirigido al puerto interno `5432/tcp`. La base de datos está expuesta a todas las interfaces de red. | No | `evidencias/P1_puertos.txt` |
| R2 | La contraseña del administrador está escrita en texto plano en la línea 10 de `docker-compose.yml` y tiene 10 caracteres. La política exige que no esté en texto plano y que tenga como mínimo 12 caracteres. | No | `evidencias/P2_credenciales.txt` |
| R3 | La cuenta `app_core`, además de `postgres`, tiene el atributo `Superuser`. La política solo permite este privilegio a `postgres`. | No | `evidencias/P3_roles.txt` |
| R4 | Existen 16 cuentas activas pertenecientes a 10 personas cesadas. El cese más antiguo es del 18/12/2015. También existen 22 cuentas activas sin documento ni responsable, de las cuales 4 tienen perfil `ADMIN`. | No | `evidencias/P4_cesados.txt` · `evidencias/P4_genericas.txt` |
| R5 | Se identificaron 23 desembolsos superiores al umbral aprobados por el mismo usuario que los registró. El monto total es S/ 709,370.47 y participaron 21 usuarios distintos. | No | `evidencias/P5_segregacion.txt` |
| R6 | `log_connections` tiene el valor `off` y `log_statement` tiene el valor `none`. Para cumplir la política deberían tener los valores `on` y `mod`, respectivamente. | No | `evidencias/P6_registro.txt` |
| R7 | El último respaldo disponible es del 14/11/2025. Hasta el corte del 31/12/2025 transcurrieron 47 días sin un respaldo correcto debido a falta de espacio. Al restaurarlo solo se recuperaron las tablas `empleados` y `usuarios`; falta `desembolsos`. | No | `evidencias/P7_respaldos.txt` · `evidencias/P7_restauracion.txt` |

## 2. Hallazgo 1

| Elemento | Contenido |
|---|---|
| **Título** | Desembolsos superiores al umbral sin segregación entre registro y aprobación |
| **Condición** | Se identificaron 23 desembolsos superiores al umbral de aprobación que fueron registrados y aprobados por el mismo usuario. Estos desembolsos suman S/ 709,370.47 y fueron realizados por 21 usuarios distintos, según `evidencias/P5_segregacion.txt`. |
| **Criterio** | La regla R5 de la Política de Seguridad establece que un desembolso que supera el umbral no puede ser aprobado por quien lo registró. Esta regla corresponde al control A.5.3 Segregación de funciones de la NTP-ISO/IEC 27001:2022. |
| **Causa** | La base de datos no cuenta con una restricción, validación o mecanismo de aprobación que impida que `usuario_registra` y `usuario_aprueba` sean iguales cuando el monto supera el umbral. |
| **Efecto** | La falta de segregación permite que una sola cuenta controle el registro y la aprobación de operaciones por S/ 709,370.47. Esto aumenta el riesgo de desembolsos no autorizados, fraude y dificultad para establecer responsabilidades. |
| **Recomendación** | El Jefe de Sistemas debe implementar, en un plazo máximo de 15 días calendario, una validación que bloquee la aprobación cuando el registrador y el aprobador sean el mismo usuario en operaciones superiores al umbral. También debe revisar los 23 casos identificados y separar los permisos de registro y aprobación. |

## 3. Hallazgo 2

| Elemento | Contenido |
|---|---|
| **Título** | Respaldo incompleto y falta de copias recientes del core financiero |
| **Condición** | El último respaldo disponible corresponde al 14/11/2025. Hasta el corte del 31/12/2025 transcurrieron 47 días sin un respaldo correcto. Al restaurar el último archivo solo se recuperaron las tablas `empleados` y `usuarios`; la tabla `desembolsos` no estaba incluida, según `evidencias/P7_respaldos.txt` y `evidencias/P7_restauracion.txt`. |
| **Criterio** | La regla R7 de la Política de Seguridad exige un respaldo diario completo que incluya la tabla de desembolsos y una prueba de restauración trimestral. Esta regla corresponde al control A.8.13 Respaldo de la información de la NTP-ISO/IEC 27001:2022. |
| **Causa** | El script `respaldos/respaldo.sh` excluye expresamente la tabla `desembolsos`. Además, desde el 15/11/2025 la tarea de respaldo falla porque el dispositivo no tiene espacio disponible. |
| **Efecto** | Ante una falla del servidor, la cooperativa no podría recuperar los desembolsos y podría perder hasta 47 días de información posterior al último respaldo. Esto podría afectar la continuidad operativa, la conciliación de operaciones y la integridad de la información financiera. |
| **Recomendación** | El Jefe de Sistemas debe liberar o ampliar el almacenamiento y retirar inmediatamente la exclusión de la tabla `desembolsos`. En un plazo máximo de 5 días hábiles debe generar un respaldo completo y demostrar su restauración. También debe configurar alertas ante fallas diarias y documentar pruebas de restauración trimestrales. |
