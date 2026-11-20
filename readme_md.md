# 🏋️‍♂️ Base de Datos Relacional: Gimnasio Cuerpo Activo

**Proyecto Académico:** Bases de Datos Relacionales (Fases 1 a 4 — Entrega Final)  
**Autor:** Alexis Rodríguez

---

## 📋 Descripción del Proyecto

Este repositorio contiene el diseño, implementación, refuerzo de reglas de negocio y consultas analíticas de la base de datos **`gimnasio_cuerpo_activo`**. El sistema administra de forma integral los socios, planes comerciales, historial real de pagos de membresías, fichas médicas, instructores, programación de clases grupales y control de cupos.

---

## 🗂️ Estructura del Repositorio

```text
taller-gimnasio/
├── README.md                        # Documentación completa del proyecto y respuestas
├── gimnasio_cuerpo_activo.sql       # Respaldo completo (Backup Plain UTF-8)
├── informe_final.pdf                # Informe ejecutivo y evidencias visuales de pgAdmin
├── diagramas/                       # Diagramas Entidad-Relación (Modelo Conceptual y Relacional)
├── 01_estructura.sql                # Creación de tablas base y llaves primarias/foráneas
├── 02_datos_iniciales.sql           # Poblamiento inicial de tablas
├── 03_consultas_basicas.sql         # Consultas exploratorias iniciales
├── 04_modificaciones.sql            # Ajustes estructurales de Fase 2
├── 05_actualizaciones.sql           # Actualizaciones de registros de Fase 2
├── 06_tipos.sql                     # Adición de valor_pagado y acepta_whatsapp (Parte H)
├── 07_restricciones.sql             # Reglas CHECK, NOT NULL y UNIQUE compuesto (Parte I)
├── 08_claves_foraneas.sql           # Políticas ON DELETE CASCADE, SET NULL y RESTRICT (Parte J)
├── 09_mas_datos.sql                 # Carga masiva de datos de prueba (Parte K)
├── 10_consultas.sql                 # Consultas operativas con filtros y ordenamiento (Parte K)
├── 11_reportes.sql                  # Métricas globales con funciones de agregación (Parte L)
├── 12_por_grupos.sql                # Reportes agrupados con GROUP BY y HAVING (Parte M)
├── 13_joins.sql                     # Cruce de tablas con INNER JOIN (Parte O)
├── 14_sin_pareja.sql                # Detección de registros sin relación con LEFT JOIN (Parte P)
├── 15_horarios.sql                  # Consultas multi-tabla a través de tablas intermedias (Parte Q)
├── 16_informe.sql                   # Reportes ejecutivos combinando JOIN y GROUP BY (Parte R)
├── 17_subconsultas.sql              # Consultas anidadas con IN, NOT IN y promedios (Parte S)
└── 18_vistas.sql                    # Vistas operativas para recepción (Parte T)
```
---

## 🧠 Documentación Técnica y Respuestas (Fase 3)

### Parte H — Tipos de Datos (`06_tipos.sql`)
* **H1. ¿Por qué no basta con el precio en `planes` y hace falta `valor_pagado` en `membresias`?**  
  Porque las tarifas de los planes cambian cada año y se otorgan descuentos personalizados a ciertos socios. Guardar `valor_pagado` en cada membresía preserva la exactitud financiera histórica de la transacción real.
* **H2. ¿Qué tipo se eligió para `valor_pagado` y por qué no `INTEGER` ni `DOUBLE PRECISION`?**  
  Se eligió `NUMERIC(10, 2)`. No se usa `INTEGER` porque se requieren decimales exactos para descuentos porcentuales, ni `DOUBLE PRECISION` porque los tipos de punto flotante introducen errores de redondeo binario inaceptables en contabilidad.
* **H3. ¿Por qué el documento y el teléfono siguen siendo `VARCHAR`?**  
  Porque son identificadores y cadenas de contacto, no magnitudes matemáticas destinadas a operaciones aritméticas; además, evita la pérdida de ceros a la izquierda y admite prefijos internacionales.
* **H4. Operador usado en el `UPDATE` de `acepta_whatsapp`:**  
  Se utilizó el operador `IN` junto a una subconsulta de selección para marcar múltiples socios en una sola sentencia.

### Parte I — Restricciones de Negocio (`07_restricciones.sql`)
* **I1 (`membresias_fecha_fin_check`):** Bloquea el registro de membresías cuya `fecha_fin` sea menor o igual a `fecha_inicio`.
* **I2 (`clases_entrenador_id_dia_semana_hora_inicio_key`):** Impide programar dos clases para el mismo entrenador en el mismo día y hora.
* **I3 (`membresias_valor_pagado_check`):** Rechaza pagos iguales o inferiores a `0`.
* **I4 (`NOT NULL` en `valor_pagado`):** Impide registrar membresías sin un monto pagado asociado.
* **I5. ¿Por qué el límite de cupo no se puede escribir como un `CHECK` en `inscripciones`?**  
  Porque una restricción `CHECK` evalúa únicamente los valores de la fila individual que se inserta en ese momento; no puede consultar ni contar de forma agregada cuántas filas previas ya existen registradas para esa misma clase.

### Parte J — Integridad Referencial (`08_claves_foraneas.sql`)

| Clave Foránea | Decisión | Justificación del Negocio |
| :--- | :--- | :--- |
| `fichas_medicas.socio_id` | `CASCADE` | La ficha médica es información personal vinculada exclusivamente al socio; sin él carece de utilidad. |
| `membresias.socio_id` | Proteger (`NO ACTION`) | Representa historial financiero que no debe eliminarse mientras existan pagos registrados. |
| `membresias.plan_id` | Proteger (`NO ACTION`) | Un plan con historial de ventas debe conservarse para auditoría contable. |
| `clases.entrenador_id` | `SET NULL` | Si un entrenador se retira, la clase continúa activa en la programación para asignarle otro instructor. |
| `inscripciones.socio_id` | `CASCADE` | Una reserva de cupo pierde sentido si el socio titular es eliminado del sistema. |
| `inscripciones.clase_id` | Proteger (`NO ACTION`) | Conserva el historial de asistencia; por ello una clase con inscritos se suspende en lugar de borrarse. |

### Partes K, L y M — Consultas, Reportes y Agrupamientos (`10` a `12`)
* **K4 (Precedencia de operadores):** En SQL, `AND` tiene mayor precedencia que `OR`. Sin paréntesis en `estado = 'activa' AND (dia_semana = 'lunes' OR dia_semana = 'jueves')`, la consulta traería todas las clases del jueves sin importar si están suspendidas.
* **L1 (`COUNT(*)` vs `COUNT(correo)`):** `COUNT(*)` cuenta el total de filas de la tabla, mientras que `COUNT(correo)` ignora los registros donde el correo es `NULL`.
* **L5 (Recurrencia del negocio):** La diferencia entre el total de membresías y los socios distintos demuestra fidelización: varios clientes han realizado renovaciones múltiples.
* **M7 (`WHERE` vs `HAVING`):** El filtro de `COUNT(*) >= 4` debe ir en `HAVING` porque evalúa el resultado de una función de agregación después de agrupar (`GROUP BY`), mientras que `WHERE` filtra filas individuales antes de agrupar.

---

## 🔗 Documentación Técnica y Respuestas (Fase 4)

### Parte O — INNER JOIN (`13_joins.sql`)
* **O3:** Se usaron **2 JOIN** (`ON m.socio_id = s.id` y `ON m.plan_id = p.id`). Si se cambia por `m.id = s.id`, PostgreSQL no arroja error porque ambos campos son enteros, pero cruza erróneamente los consecutivos de membresías con los IDs de socios, generando reportes financieros falsos sin levantar alertas.

### Parte P — LEFT JOIN (`14_sin_pareja.sql`)
* **P4:** En P1 se colocó la tabla `entrenadores` en el `FROM` (a la izquierda) para conservar a todos los instructores, y se filtró con `WHERE c.id IS NULL` (clave primaria de la tabla derecha) para identificar a quienes no tienen clases asignadas.

### Parte Q — Cruce Multi-tabla (`15_horarios.sql`)
* **Camino relacional (Q1):** `socios (id)` $\rightarrow$ `inscripciones (socio_id / clase_id)` $\rightarrow$ `clases (id / entrenador_id)` $\rightarrow$ `entrenadores (id)`.
* **Q3:** Lleva **3 JOIN**. No se puede ir de `socios` a `clases` con un solo `JOIN` porque mantienen una relación de muchos a muchos resuelta físicamente por la tabla intermedia `inscripciones`.

### Parte R — Informe Agrupado (`16_informe.sql`)
* **R5 (`COUNT(i.id)` vs `COUNT(*)`):** Se cuenta `COUNT(i.id)` para que Pilates aparezca con `0` inscritos. Si se usa `COUNT(*)`, el `LEFT JOIN` cuenta la fila propia de Pilates en `clases` (con inscripción en `NULL`) y reportaría erróneamente `1` inscrito.
* **R6 (`WHERE` vs `ON`):** El filtro `c.estado = 'activa'` va en el `WHERE` porque pertenece a la tabla principal (`clases`). Si se pusiera en el `ON`, el `LEFT JOIN` seguiría trayendo las clases suspendidas mostrando `0` inscritos.

### Parte S — Subconsultas (`17_subconsultas.sql`)
* **S4 (Riesgo de `NULL` en `NOT IN`):** La columna `inscripciones.socio_id` tiene restricción `NOT NULL`, por lo que no puede traer vacíos. Sin embargo, si se colara un `NULL` en la lista de un `NOT IN`, toda la consulta devolvería `0` filas debido a la lógica trivalente de SQL (`valor != NULL` evalúa a `UNKNOWN`).

### Parte T — Vistas Operativas (`18_vistas.sql`)
* **T4:** Al inscribir un socio en Pilates y volver a consultar `ocupacion_clases`, los inscritos pasan automáticamente de `0` a `1` porque una vista no guarda datos estáticos, sino que ejecuta su consulta subyacente en tiempo real sobre las tablas base.

---

## 📊 Parte U — Informe Ejecutivo para Marcela

| Pregunta del Negocio | Respuesta en Lenguaje Claro | Fuente SQL | Decisión Técnica Tomada |
| :--- | :--- | :--- | :--- |
| **1. ¿Qué socios tienen la membresía vencida?** | Se listan los socios cuya última membresía ya expiró junto con su teléfono y autorización de WhatsApp (ej. Julián Arango). | `16_informe.sql` (R1) / Vista `socios_vencidos` | Se evaluó únicamente `MAX(fecha_fin) < CURRENT_DATE` por socio para no marcar como vencidos a quienes ya renovaron. |
| **2. ¿Cuál es el plan más vendido?** | Se muestra el ranking de planes ordenados por cantidad de ventas e ingresos reales generados. | `16_informe.sql` (R2) | Se usó `LEFT JOIN` desde `planes` para incluir todos los planes del catálogo aunque alguno tuviera cero ventas. |
| **3. ¿Cuánto dinero entró por membresías este mes?** | Entrega el recaudo exacto correspondiente a las membresías iniciadas en noviembre de 2026. | `11_reportes.sql` (L2) | Se sumó `valor_pagado` en lugar del precio base del plan para reflejar descuentos reales aplicados. |
| **4. ¿Cuántos inscritos tiene cada clase y cuáles ya llenaron el cupo?** | Muestra la ocupación de todas las clases activas (incluyendo Pilates con `0` inscritos) y filtra las que alcanzaron su `cupo`. | `16_informe.sql` (R3, R4) / Vista `ocupacion_clases` | Se contó `COUNT(i.id)` sobre un `LEFT JOIN` para reflejar correctamente las clases nuevas sin alumnos. |
| **5. ¿Qué entrenadores no tienen ninguna clase asignada?** | Identifica los instructores disponibles en plantilla sin carga horaria asignada actualmente. | `14_sin_pareja.sql` (P1) | Se aplicó `LEFT JOIN` filtrando por `c.id IS NULL`. |
| **6. ¿Qué socios nunca se han inscrito a una clase grupal?** | Lista los nombres y teléfonos de socios que utilizan zona general y nunca han reservado clases grupales. | `14_sin_pareja.sql` (P2) / `17_subconsultas.sql` (S3) | Se validó la ausencia de registros en la tabla intermedia `inscripciones`. |

### 💡 Recomendación Operativa
> A partir de hoy, el equipo de recepción puede consultar directamente las vistas **`socios_vencidos`** y **`ocupacion_clases`** cada mañana para gestionar renovaciones por WhatsApp y controlar los cupos disponibles en tiempo real sin depender de reportes manuales. Como siguiente paso analítico, se sugiere evaluar qué días y franjas horarias presentan menor ocupación para optimizar la programación de entrenadores y lanzar campañas promocionales en horarios valle.