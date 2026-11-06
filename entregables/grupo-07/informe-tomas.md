
# Informe individual — Grupo 07

**Integrante:** Tomás Terruli  
**Fecha:** 06/11/2026  
**Destinatario:** Gerente de Operaciones / Comité de Crisis

---

## 1. Situación y riesgo

Se analizaron tres servicios de un entorno de laboratorio: un portal web basado en PHP, una API desarrollada en Node.js y un servicio interno en Python. El objetivo fue determinar su exposición a vulnerabilidades críticas, definir prioridades de intervención y comprobar que las actualizaciones no afectaran su funcionamiento básico.

El portal utilizaba PHP 8.3.35 con Apache y mod_php, una configuración afectada por las vulnerabilidades CVE-2026-109618 y CVE-2026-109619, conocidas como PHP2Shell. Estas fallas pueden permitir ejecución remota de código sin autenticación bajo determinadas condiciones. Su gravedad es elevada y existe evidencia pública de explotación activa.

La API utilizaba Node.js 22.23.3, afectado por CVE-2026-111497, vinculada con el decodificador Brotli. Este componente procesa información comprimida y, si recibe contenido especialmente diseñado, podría provocar fallos del servicio o comprometer su seguridad. El servicio Python también utilizaba una versión vulnerable de Brotli, la 1.0.9.

Aunque las tres vulnerabilidades requieren atención, ninguno de los servicios estaba expuesto directamente a Internet en el laboratorio. Por lo tanto, el riesgo observado debe distinguirse de un posible entorno de producción, donde la exposición pública, los datos procesados y la criticidad comercial podrían aumentar considerablemente el impacto.

## 2. Plan de acción priorizado

| Acción | Activo | Plazo propuesto | Responsable | Reversión |
|---|---|---|---|---|
| Aplicar mitigación temporal y actualizar PHP a 8.3.36. | Portal PHP | Primeras 2 horas | Administrador de infraestructura | Restaurar la imagen anterior únicamente con autorización y controles compensatorios. |
| Actualizar Node.js a 22.23.4 y reducir la exposición al contenido Brotli no necesario. | API Node.js | Primeras 4 horas | Equipo de desarrollo e infraestructura | Volver temporalmente a la imagen anterior si existe una falla funcional grave, documentando el riesgo. |
| Actualizar Brotli a 1.2.1 y restringir el acceso al servicio. | Servicio Python | Primeras 24 horas | Administrador del servicio | Restaurar la versión anterior solo bajo un procedimiento de emergencia. |
| Verificar funcionamiento, registrar evidencias y mantener monitoreo. | Todos los servicios | Después de cada cambio | Equipo de operaciones y seguridad | Detener el cambio y aplicar el procedimiento de recuperación aprobado. |

El portal PHP se priorizó por la combinación de una vulnerabilidad crítica, explotación activa conocida y una configuración de Apache especialmente sensible. La API Node.js se ubicó en segundo lugar por su capacidad de procesar contenido HTTP mediante Brotli y por la explotación conocida de la vulnerabilidad asociada.

El servicio Python quedó en tercer lugar porque se consideró interno en el escenario de producción supuesto. Este orden debería modificarse si se detectara exposición pública, información especialmente sensible o indicadores de compromiso en cualquiera de los servicios.

En el laboratorio se aplicó primero una mitigación temporal sobre PHP, deshabilitando `session.upload_progress.enabled`. Posteriormente, se actualizaron los tres servicios a las versiones corregidas.

Las verificaciones posteriores mostraron que todos los contenedores permanecieron funcionando. PHP respondió correctamente a una solicitud HTTP, Node.js devolvió información válida de su API y Python respondió con HTTP 200. Las pruebas fueron básicas y no reemplazan una validación integral de producción.

## 3. Indicadores de compromiso

Para investigar un posible compromiso se deben correlacionar los registros de solicitudes HTTP, los procesos ejecutados y los eventos de seguridad.

Una señal relevante sería encontrar que un proceso del servidor web Apache inicia inesperadamente una consola de comandos o una herramienta para descargar contenido. En Node.js también resultaría sospechoso que el proceso de la aplicación ejecutara programas como `sh`, `bash`, `curl` o `wget` sin una justificación operativa.

En el registro de auditoría del laboratorio se incluyen ejemplos de relaciones entre procesos que podrían indicar actividad anómala. Sin embargo, estas señales por sí solas no permiten determinar con certeza que exista un ataque exitoso. Es necesario relacionarlas con horarios, usuarios, solicitudes y comportamiento esperado de cada servicio.

Para descartar razonablemente un incidente se necesitaría comprobar la ausencia de actividad sospechosa en registros suficientemente completos, verificar la integridad de los sistemas y revisar los accesos realizados durante el período de exposición. La ausencia de alertas no demuestra por sí sola que nunca haya ocurrido un compromiso.

Una limitación importante es que el registro de ejemplo no incluye toda la información necesaria para atribuir acciones a un usuario o identificar la IP de origen. Tampoco se realizaron análisis forenses, pruebas de explotación ni verificaciones sobre servidores reales.

Por lo tanto, la conclusión adecuada es que se identificaron versiones vulnerables y se verificó su actualización, pero no se puede afirmar que hayan existido ataques efectivos ni descartar incidentes históricos con la información disponible.

## 4. Transferencia

El procedimiento utilizado puede aplicarse a otra vulnerabilidad, por ejemplo CVE-2026-115560, mencionada en el material de la cátedra.

Ante una alerta sobre una vulnerabilidad diferente, el primer paso sería consultar su aviso oficial para determinar el componente afectado, las versiones vulnerables, el tipo de impacto y las correcciones disponibles.

Después se realizaría un inventario de los activos que utilizan ese componente, considerando no solo las dependencias declaradas por las aplicaciones, sino también los runtimes, bibliotecas incorporadas e imágenes de contenedores.

La prioridad se establecería según la gravedad técnica, la existencia de explotación real, la exposición de los servicios y la importancia de los datos o procesos del negocio. Una vulnerabilidad con menor puntuación podría requerir una respuesta más urgente si afecta a un servicio público explotado activamente.

Una vez definido el plan, se aplicaría una mitigación temporal cuando fuera necesaria, seguida del parche correspondiente. Cada cambio debería registrar su estado anterior, el resultado posterior, una prueba funcional y un procedimiento de reversión.

Lo que cambiaría respecto del ejercicio sería la evaluación específica del componente y de sus condiciones de explotación. No sería correcto reutilizar automáticamente la misma mitigación de PHP o las mismas versiones de Node.js y Brotli para una vulnerabilidad diferente.

## 5. Lecciones y procesos

El ejercicio permitió comprobar que gestionar una vulnerabilidad no consiste solamente en instalar una actualización. Primero es necesario conocer qué servicios existen, qué versiones utilizan, cómo están expuestos y qué impacto tendría una interrupción.

Una mejora prioritaria sería mantener un inventario actualizado de aplicaciones, runtimes, dependencias e imágenes Docker. Esto permitiría identificar rápidamente los activos afectados cuando se publica una nueva alerta.

También resulta fundamental establecer un procedimiento de cambios que contemple prioridades, responsables, evidencia previa, aplicación del parche, pruebas posteriores y posibilidad de reversión.

Desde el punto de vista de detección, sería conveniente centralizar los registros, mejorar la visibilidad de los procesos y relacionar los eventos del sistema con las solicitudes recibidas por las aplicaciones.

Por último, la comunicación con gerencia debería centrarse en explicar qué riesgo existe, qué decisiones se tomaron, cuáles fueron los resultados comprobados y qué incertidumbres permanecen.

La principal conclusión es que una respuesta efectiva combina inventario, priorización, mitigación, actualización, verificación y comunicación. La evidencia obtenida debe respaldar las decisiones, y aquello que no pudo comprobarse debe declararse expresamente.
