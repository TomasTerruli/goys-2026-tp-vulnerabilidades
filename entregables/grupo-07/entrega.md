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
| Portal PHP | 9.2 | Sí. CVE-2026-109618 y CVE-2026-109619 incluidos en CISA KEV desde el 26/10/2026. | No expuesto en el laboratorio. Se supone acceso público mediante proxy en producción. | Alta: portal con sesiones y recepción de archivos, que podría permitir ejecución remota de código sin autenticación. | Prioridad 1: aplicar mitigación inmediata y parchear PHP 8.3.35 a 8.3.36 en una ventana de emergencia dentro de las primeras 2 horas. |
| API Node.js | 9.2 | Sí. CVE-2026-111497 incluido en CISA KEV desde el 27/10/2026. | No expuesta en el laboratorio. Se supone acceso público mediante proxy en producción. | Alta: API que procesa contenido Brotli y podría sufrir ejecución remota de código o denegación de servicio. | Prioridad 2: reducir temporalmente la exposición a Brotli y actualizar Node.js 22.23.3 a 22.23.4 dentro de las primeras 4 horas. |
| Servicio Python | 9.2 | Sí, para la vulnerabilidad de Brotli CVE-2026-111497, aunque no hay evidencia específica de explotación de este servicio. | No expuesto en el laboratorio. Se supone que permanece en una red interna en producción. | Media: servicio interno que utiliza Brotli 1.0.9 para procesar contenido recibido mediante HTTP. | Prioridad 3: restringir el acceso al servicio y actualizar Brotli 1.0.9 a 1.2.1 dentro de las primeras 24 horas. |

**Justificación:**

El orden de prioridad se definió considerando la gravedad de las vulnerabilidades, la explotación real conocida, la exposición potencial a Internet y la criticidad de cada servicio.

El portal PHP se considera la primera prioridad porque utiliza una versión vulnerable del núcleo de PHP y Apache con mod_php, configuración identificada como especialmente riesgosa. Además, las vulnerabilidades CVE-2026-109618 y CVE-2026-109619 tienen explotación activa confirmada.

La API Node.js ocupa la segunda prioridad porque su runtime contiene un decodificador Brotli vulnerable y procesa contenido HTTP. La vulnerabilidad CVE-2026-111497 también figura en CISA KEV, por lo que requiere una actualización urgente.

El servicio Python se ubica en tercer lugar porque, aunque utiliza una versión afectada de Brotli, en el escenario de producción supuesto permanece en una red interna y tiene menor exposición que los otros servicios.

Los plazos indicados son objetivos propuestos por el equipo para un escenario de producción. No representan tiempos efectivamente medidos durante el laboratorio.

**Evidencia que podría modificar el orden:**

La priorización se revisaría si se comprobara que el servicio Python es accesible desde Internet, procesa información crítica o presenta indicadores de compromiso. En ese caso podría aumentar su prioridad.

También cambiaría si aparecieran indicadores de explotación activa en la API Node.js, si alguno de los servicios procesara datos especialmente sensibles o si un control de mitigación resultara ineficaz.

**Fuentes utilizadas:**

- `recursos/contexto.md`: vulnerabilidades, CVSS, versiones corregidas y antecedentes de explotación.
- `capturas/01-arranque-inventario.txt`: versiones reales del laboratorio.
- `capturas/02-apache-modulos.txt`: confirmación del módulo PHP en Apache.
- `laboratorio/docker-compose.yml`: configuración de la red y exposición de los servicios.

---

## 3. Mitigación, parche y reversión


| Fecha/hora | Activo | Cambio | Antes | Después | Reversión |
|------------|--------|--------|-------|---------|-----------|
| 06/11/2026 | Portal PHP | Mitigación temporal: se modificó `laboratorio/php/php.ini`, deshabilitando `session.upload_progress.enabled`, y se reinició `php-lab`. | `session.upload_progress.enabled=1`. Evidencia: `capturas/03-php-antes-mitigacion.txt`. | `session.upload_progress.enabled=0`. Los tres contenedores permanecieron `healthy`. Evidencias: `capturas/04-php-despues-mitigacion.txt` y `capturas/05-php-post-mitigacion-ps.txt`. | Restaurar `session.upload_progress.enabled = 1` en `php/php.ini`, reiniciar con `docker compose restart php-lab` y verificar el valor. Esta reversión reactivaría la configuración vulnerable, por lo que solo se consideraría ante una necesidad justificada y con controles compensatorios. |


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
