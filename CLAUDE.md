# CLAUDE.md — Auditoría de Artes/Calidad (lives-auditoria-artes)

Las reglas generales de trabajo están en el repo privado `lives-instrucciones` (CLAUDE.md). Este archivo solo describe esta herramienta.

## Qué hace
Auditoría de Artes/Calidad de un lote (hoja FO-256): estampado, bordado y lavandería, con AQL. Herramienta nueva (28/10/2026), aparte del formulario de Auditoría Final pero hecha lo más parecida posible a él. Aún no está en el portal: se usa con su link.
- `index.html` — **entrada** (consulta "Buscar"): filtros (fechas, servicio, auditora, resultado, búsqueda), lista con regla 500, Excel LIVES (`LS_XLS`) e Imprimir. Sin fecha muestra los últimos 3 meses; si se elige un "Desde" anterior, vuelve a pedir a la base. En la barra: **Registrar** (abre `formulario.html` en una ventana dentro de la página) y **Dashboard** (abre `dashboard.html` igual). Cada fila tiene **Ver PDF** (abre `ver.html` en la ventana).
- `formulario.html` — **formulario** (también link directo para el celular). Desde el 28/10/2026 va por pestañas, como Tintorería: arriba fijo "¿Corregir una auditoría?" (en una línea) y la OP (elegida, queda en una línea con "Cambiar"; un resumen OP · grupo · color · servicio queda pegado sobre las pestañas), y debajo las pestañas Lote (datos de la OP, total recibido, servicio, auditora, fecha y AQL), Defectos, Medidas, Fotos y Cierre (pase al rechazo y comentarios), con ‹ Anterior / Siguiente › y Guardar siempre a la vista; al guardar con faltantes salta a la primera pestaña con error y marca las demás con un punto rojo. Contenido igual que el de Auditoría Final: "¿Necesitas corregir una auditoría que ya hiciste?" (por OP: corregir o anular), datos que salen de la OP (base de OPs guardada en el equipo), total recibido y servicio, nivel AQL que elige la auditora (1.0/1.5/2.5) con Lote/Muestra/Acepta/Rechaza, defectos con buscador en 3 grupos con 📷 por defecto (cámara o galería) y resumen, medidas, pase al rechazo (quién autoriza y motivo), fotos de hojas de medidas (filtro escáner) y de la prenda/arte, comentarios. Barra fija abajo con el veredicto y Guardar. Sin señal: queda en cola en el equipo (IndexedDB `lives_artes_cola`) y se envía sola; borrador en el equipo.
- `ver.html?id=` — **reporte** con el formato de la hoja FO-256 (A4 vertical): datos, muestreo de calidad y medidas, casillas Aprobado/Rechazado, las 3 tablas de defectos completas, tabla AQL con la fila y el nivel marcados, comentarios, firmas, y las fotos aparte. **Compartir PDF** (html2canvas + jsPDF: hoja en la página 1, fotos en las siguientes), Imprimir, Corregir y Anular.
- `dashboard.html` — **dashboard**: filtros año/mes/servicio/cliente/auditora; KPI % lotes rechazados, lotes auditados, defectos por cada 100 prendas de muestra, % rechazados por medidas; rechazo por mes (líneas por año), defectos por tipo de arte, defectos más frecuentes, rechazo por servicio, por grupo y por cliente (mínimo 8 lotes; sin meta: se compara contra el promedio ponderado ± 1 desviación). Excel LIVES.

Resultado automático: Calidad = total de defectos ≤ "acepta" de la tabla AQL (por lote y nivel). Medidas (si se llena la muestra) = defectos de medidas ≤ "acepta" de la fila de la tabla con esa muestra (o la inmediata inferior), mismo nivel. Final = RECHAZADO si cualquiera rechaza; con pase al rechazo queda APROBADO y se guarda `resultado_aql` = RECHAZADO. La base recalcula todo al guardar.

## Fuera del portal
Sin `?usuario=` en el link aparece **Identificarme** (cuadro "¿Quién eres?" con "Quienes registran" de `art_cat` y las auditoras del catálogo, o un usuario escrito); ya identificada, "Cambiar de persona" queda bajo el campo Auditora. Se guarda en el equipo (`localStorage` `lives_art_quien`) y firma lo registrado.

## Base de datos (Supabase lives-datos) — registro `Ar01 auditoria artes fo256`
- Puerta única `art_api(p_accion, p)`: `inicio` (permiso, nombre, quienes registran, catálogo de defectos, servicios ya usados, auditoras, clientes, tabla AQL), `ops` (base de OPs de `publico_ref_ops`), `guardar` (nueva o corrección; no duplica si se reintenta, por `token`), `anular`, `ver`, `datos` (lista para buscar y dashboard; sin fechas = últimos 92 días, `todo:true` = todo).
- Tablas: `art_auditorias` (cabecera, código ART-00001 con la secuencia `art_seq`), `art_defectos`, `art_fotos`, `art_envios` (envíos ya procesados), `art_cat` (45 defectos de la hoja, `AJUSTE` `REGISTRO_LIBRE` y `REGISTRA`). El catálogo de defectos se edita en `art_cat` sin tocar el código.
- Funciones: `art_aql_tabla_`, `art_aql_`, `art_aql_med_`, `art_nombre_`, `art_supervision_`, `art_puede`, `art_foto_permitida`.
- Fotos: almacén privado `artes-fotos`, ruta `AAAA-MM-DD/<token del envío>/<nombre>.jpg`. Se suben sin sesión (política `artes-fotos: subir` con `art_foto_permitida`: solo esa forma de ruta y fecha de hoy ±). Se ven con enlaces firmados de 1 hora.
- Visor de fotos nuevas: `fotos_listar_artes` (lo llama `lives-auditorias-visor`, tipo "AUDITORIA DE ARTES", con enlace al reporte).
- Usa también `cat_personal`, `cat_clientes`, `perfiles`, `nombre_sistemas`, `mi_nombre`.

## Quién puede registrar
- Registro libre durante las pruebas: `art_cat` AJUSTE `REGISTRO_LIBRE` = `SI`. Para cerrar: ponerlo en `NO`; entonces registran solo los de `art_cat` tipo `REGISTRA` (hoy `ctalepcio`). Christian siempre puede. Rodrigo (rcamogliano): supervisión, ve todo y no guarda.
- A Sistemas: `art_puede` es la pieza que reemplazan con su seguridad; en SQL Server la herramienta exige usuario para abrir y para leer. Exclusivo de Postgres: jsonb en la puerta, `auth.uid()` (solo para supervisión), lectura libre por RLS y Storage para las fotos.

## Quién la usa
Auditoras de Artes de Calidad Manufactura (desde el 28/10/2026, en prueba).
