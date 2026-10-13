# Unstable Underworld v3.4.0 — Rendimiento (Minecraft Java 26.3)

Proyecto fuente para **Fabric**. Contiene el código completo, `gradle.properties`,
`build.gradle`, recursos, pruebas locales y GitHub Actions. No incluye el `.jar` compilado.

## Optimización

La v3.3.2 generaba el terreno completo de cada chunk en el evento `CHUNK_LOAD`,
lo que podía saturar el hilo del servidor al explorar rápidamente. En v3.4.0:

- **Cola de generación en el hilo principal**: como máximo 2 chunks terminados por tick,
  con un presupuesto **orientativo** de 8 ms entre trabajos. Un chunk individual no
  se puede interrumpir: si tarda más, **aún podría producir un tick lento**.
- Al descargarse un chunk se cancela el trabajo pendiente para ese chunk.
- Las posiciones se deduplican: no se genera dos veces un chunk cargado.
- Al entrar desde el Nether, se genera inmediatamente el chunk de destino,
  para mantener una superficie real mientras cae la entidad.
- Los cambios diferidos en bloques usan `ServerLevel.setBlock` para que lleguen
  al cliente aunque el chunk se haya enviado antes de terminar la decoración.
- Las estructuras omiten operaciones sobre el aire ya vacío y reutilizan
  resultados de comprobación de emplazamientos con caché LRU de 512 entradas.
- Se elimina una evaluación redundante de la altura por columna y se evita
  la generación fuera del radio de 6.400 bloques.
- Los portales se comprueban cada 5 ticks, sin retardar la detección de caídas.

**Limitación intencionada:** cuando se explora rápidamente, algunos chunks
pueden visualizarse vacíos durante un breve periodo hasta que su turno se procese.
Esto sacrifica inmediatez a cambio de reducir ráfagas de trabajo. Si aparecen
huecos persistentes o el servidor se bloquea de nuevo, se necesitan los registros
`logs/latest.log` y `crash-reports` para encontrar el origen concreto.

## Características conservadas

- Dimensión independiente del Nether: **el Nether original no se modifica**.
- Entrada al caer bajo el Nether, con las mismas X/Z y movimiento descendente.
- Regreso mediante `/underworld return` a **Y = −7** y las mismas X/Z.
- Modo Aventura en el Underworld, restaurando el modo anterior al salir.
- Muerte por daño normal del vacío cuando se cae fuera del Underworld.
- Terreno de **30 bloques**, pequeñas montañas, grandes zonas de soul sand,
  soul soil, blackstone, grietas y vacío completo a más de 6.400 bloques.
- Ocho tipos de ruinas detalladas, cofres con botín temático y portales
  extremadamente raros al Overworld.
- Cielo tipo Nether, sin sol ni nubes.

## Comandos

- `/underworld enter`: entra cayendo desde arriba del Underworld.
- `/underworld return`: sale al Nether, Y = −7, X/Z intactas.
- `/underworld queue`: muestra cuántos chunks esperan generación; útil al
  explorar rápidamente o diagnosticar retrasos.

## Requisitos

- Minecraft Java **26.3**
- Fabric Loader **0.19.5**
- Fabric API **0.162.0+26.3**
- Java **25** (para compilación y ejecución)

## Compilación en GitHub

1. Crea una copia de seguridad completa de tu mundo y del servidor.
2. Sube **todos** los archivos y carpetas de este ZIP a la raíz del repositorio.
3. Asegúrate de borrar los JAR y los JSON duplicados de versiones anteriores.
4. Ve a **Actions → Build Minecraft 26.3 mod → Run workflow**.
5. Descarga el artefacto `unstable-underworld-26.3` y extrae
   `unstable-underworld-3.4.0.jar`.
6. Instala **el mismo JAR** en cliente y servidor Fabric, junto con Fabric API.
7. Prueba primero **un mundo nuevo o una copia del original**.

El workflow realiza: pruebas de geometría, distribución, regresión matemática,
validación de recursos y botín, `gradle build` e inspección del JAR.

## Migración y seguridad

- Los chunks del Underworld de v3.3.2 que tengan el marcador de finalización
  **no se regeneran** y conservan sus estructuras.
- Un chunk antiguo sin ese marcador puede regenerarse, eliminando construcciones.
- Si aparecen avisos de datapacks o modo seguro, no guardes el único mundo
  original en modo seguro. Extrae la causa exacta de `logs/latest.log`.
- No recomendamos migrar el único mundo existente sin una copia de seguridad.

## Verificación y límites

Las pruebas matemáticas, los validadores JSON y la integridad del ZIP pueden
realizarse sin Minecraft. **No son una compilación ni una prueba de arranque**:
el proyecto requiere una ejecución real de GitHub Actions y prueba en el juego.
La cola de trabajos reduce los picos por ráfagas de chunks pero no ofrece una
cota dura al tiempo de un único chunk. Tampoco garantiza que la causa de los
crashes previos fuera exclusivamente la generación: podría ser memoria,
conflictos de mods o datos de mundo.

Los diseños de ruinas son originales y no copias de mapas de videos.
Licencia: MIT; consulta `LICENSE`.
