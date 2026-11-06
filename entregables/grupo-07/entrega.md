# Entregable — TP PHP2Shell / Node2Shell

**Grupo:** _completar_ · **Fork del grupo:** _completar_ · **Rama:** `grupo-XX`
**Fecha:** jueves 29/10/2026

| Integrante | Usuario de GitHub | Rol en el TP |
|---|---|---|
| | | |
| | | |
| | | |
| | | |

> Roles sugeridos: inventario, cambios/parche, detección, contención/comunicación.
> Todos tienen que poder explicar su parte y una parte ajena.

---

## 1. Inventario

| Activo | Versión | ¿Expuesto a Internet? | ¿Procesa contenido no confiable? | ¿Afectado? | Evidencia |
|--------|---------|:---------------------:|:--------------------------------:|:----------:|-----------|
| Portal PHP | PHP 8.3.35, Apache con mod_php, `session.upload_progress.enabled=1` | No | Sí | Sí | `capturas/01-arranque-inventario.txt` y `capturas/02-apache-modulos.txt` |
| API Node.js | Node.js 22.23.3, Brotli integrado 1.1.0 | No | Sí | Sí | `capturas/01-arranque-inventario.txt` |
| Servicio Python | Python 3.11.18, Brotli 1.0.9 | No | Sí | Sí | `capturas/01-arranque-inventario.txt` |

**Supuestos de exposición:**

En el laboratorio, ninguno de los tres servicios está expuesto directamente a Internet, ya que `docker-compose.yml` utiliza una red interna (`internal: true`) y no publica puertos hacia el exterior.

El portal PHP procesa solicitudes HTTP y permite la recepción de archivos. La API Node.js recibe contenido mediante el endpoint `/decode`, que utiliza el decodificador Brotli. El servicio Python también puede recibir y descomprimir contenido Brotli mediante solicitudes HTTP.

Los tres servicios utilizan versiones afectadas por las vulnerabilidades analizadas. Sin embargo, estar afectados no implica que hayan sido comprometidos.

Para analizar un posible entorno de producción, suponemos que el portal PHP y la API Node.js podrían estar publicados detrás de un proxy inverso, mientras que el servicio Python permanecería accesible únicamente desde la red interna. Estos son supuestos de arquitectura, no condiciones comprobadas en el laboratorio.

---

## 2. Priorización y decisión

| Activo | CVSS | Explotación real | Exposición | Criticidad | Decisión y plazo |
|--------|:----:|:----------------:|:----------:|:----------:|------------------|
| | | | | | |
| | | | | | |
| | | | | | |

**Justificación:** _¿por qué ese orden? ¿qué evidencia nueva cambiaría la decisión?_

---

## 3. Mitigación, parche y reversión

| Fecha/hora | Activo | Cambio | Antes | Después | Reversión |
|------------|--------|--------|-------|---------|-----------|
| | | | | | |
| | | | | | |

**Prueba funcional mínima:** _¿qué verificaron para saber que la app sigue viva?_

**Qué no se pudo probar:** _declararlo explícitamente._

---

## 4. Detección y caza

**Hipótesis:** _una frase: proceso, padre, horario, evidencia esperada._

**Hallazgos en `audit.log`:**

| Línea / timestamp | Host | Padre → hijo | ¿Anómalo? | Por qué |
|-------------------|------|--------------|:---------:|---------|
| | | | | |
| | | | | |

**¿Hay compromiso?** _sí/no/indeterminado, con evidencia._

**Regla propuesta para Node.js:** _qué proceso observa, qué condición y qué falso
negativo podría tener._

**Limitaciones del log:** _usuario, IP de origen, retención, ruido, etc._

---

## 5. Contención

| Control | Configuración | Evidencia | ¿Funciona? |
|---------|---------------|-----------|:----------:|
| Red sin salida | `internal: true` | | |
| Usuario sin privilegios | | | |
| Filesystem read-only | | | |
| Capabilities mínimas | | | |
| Límites de memoria/PIDs | | | |
| Rechazo de Brotli innecesario | | | |

**Prueba de salida a Internet:** _resultado en los tres servicios._

---

## 6. Conclusión (5 líneas)

_¿Qué harían distinto en las primeras 2 horas?_

---

## 7. Checklist antes de entregar

- [ ] El laboratorio arrancó y los 3 servicios quedaron `healthy`.
- [ ] Inventario con evidencia (versión, SAPI, configuración).
- [ ] Matriz con decisión y plazo.
- [ ] Parche aplicado y verificado; reversión documentada.
- [ ] Al menos 3 controles de contención probados.
- [ ] `deteccion.md` completo.
- [ ] `informe.md` individual completo.
- [ ] Sin claves, tokens ni placeholders.
- [ ] Repo pusheado y PR abierto.

Checklist completa: [`../docs/checklist-entrega.md`](../docs/checklist-entrega.md).
