# Diagnóstico

1.1. 

Los cuatros defectos son:

a) Falta incluir un needs: validar en el job Publicar (luego de la línea 41 del archivo .github/workflows/pipeline.yml). 
Esto trae como consecuencia que se pueda publicar el artefacto, aun cuando puede no haberse ejecutado el job con la validación del análisis de calidad del SonarCloud.

b) Falta especificar la instalación de dependencias desde un archivo de bloqueo (por ejemplo, un requirements.txt con versiones y hashes fijados, generado con pip-compile o similar) en los pasos "Instalar dependencias" (líneas 25-26 y 55-56 del archivo .github/workflows/pipeline.yml). Esto trae como consecuencia que pip resuelva las versiones en cada ejecución, por lo que "Validar" y "Publicar" pueden instalar árboles de dependencias distintos entre sí, perdiendo la garantía de que el paquete publicado corresponde exactamente a lo que fue probado y analizado.

c) Falta incluir el cacheo de dependencias (por ejemplo, cache: 'pip' en el paso "Preparar Python", líneas 18-21 y 48-51 del archivo .github/workflows/pipeline.yml). Esto trae como consecuencia que cada ejecución del pipeline vuelva a descargar e instalar todas las dependencias desde cero, aumentando el tiempo de ejecución y la dependencia de la disponibilidad de PyPI en cada corrida.

d) Falta un paso que espere y verifique el resultado del Quality Gate de SonarCloud después del análisis dentro del mismo job (luego de la línea 39 del archivo .github/workflows/pipeline.yml). Esto trae como consecuencia que el job "Analisis de calidad" solo envíe el código a analizar sin detener el pipeline si SonarCloud determina que no se cumplen los umbrales de calidad, perdiendo la garantía de que el pipeline realmente valida la calidad del código antes de continuar.

1.2. 

Es la falta de cacheo de dependencias. Sin cache: 'pip' (u otro mecanismo de cache) en el paso "Preparar Python", cada ejecución descarga e instala todas las dependencias desde PyPI en lugar de reutilizar una cache entre corridas, lo cual explica por qué el tiempo registrado como línea base (~1 minuto) es más alto de lo que podría ser con cache habilitado.

1.3.

El defecto que ataca a la casuística vista en nuestro grupo en clase es la ausencia de un paso que verifique y detenga el pipeline según el Quality Gate de SonarCloud. A diferencia de los otros tres defectos, el control de calidad automatizado es transversal porque actúa como filtro antes de que el cambio llegue a las etapas de revisión de código y pruebas funcionales, que son justamente las que concentran el mayor retrabajo del proceso descrito (35% de PRs rechazados, 45% de elementos devueltos por QA con defectos). Al no detener el pipeline cuando el código no cumple los umbrales de calidad, se permite que defectos evitables sigan avanzando manualmente por el resto del flujo, sobrecargando esas etapas y contribuyendo a que sean el cuello de botella del sistema completo.

1.4.

La métrica DORA que se espera mover es la tasa de fallos en cambios (Change Failure Rate). El pipeline ya corregido impide que artefactos con bugs, vulnerabilidades o baja cobertura lleguen a producción.

1.5. 

El número concreto a medir es el porcentaje de despliegues a producción que fallan o requieren rollback/hotfix. Tras la intervención, se debe recalcular este mismo indicador (despliegues fallidos ÷ total de despliegues en un período determinado) y compararlo contra resultados previos. Una reducción sostenida de ese porcentaje es la evidencia concreta de que la Change Failure Rate efectivamente se movió.

2.

Desde el tag v1.2.0 se identifican tres tipos de cambio: dos feat sobre el código de negocio (0b981ca, b7e44ce) y cambios de tipo ci/chore sobre .github/workflows/pipeline.yml. 
Según SemVer, la versión del paquete se determina únicamente por cambios que afectan su comportamiento o API pública. Los cambios de CI no inciden en esa versión. Por lo tanto, el bump correspondiente es minor: 1.2.0 → 1.3.0.



# Declaración de uso de IA generativa

Prompt utilizados: 

1.

En el siguiente pipeline, se presentan cuatro defectos:
name: pipeline

on:
  push:
  workflow_dispatch:

jobs:

  validar:
    name: Validar
    runs-on: ubuntu-latest
    steps:
      - name: Descargar el codigo
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Preparar Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Instalar dependencias
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Ejecutar pruebas
        run: pytest --cov=src --cov-report=xml

      - name: Analisis de calidad
        uses: SonarSource/sonarqube-scan-action@v8
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: https://sonarcloud.io
        with:
          args: >
            -Dsonar.organization=${{ vars.SONAR_ORG }}
            -Dsonar.projectKey=${{ vars.SONAR_PROJECT_KEY }}

  publicar:
    name: Publicar artefacto
    runs-on: ubuntu-latest
    steps:
      - name: Descargar el codigo
        uses: actions/checkout@v4

      - name: Preparar Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Instalar dependencias
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Construir el paquete
        run: python -m build

      - name: Publicar el paquete
        uses: actions/upload-artifact@v4
        with:
          name: paquete
          path: dist/
          overwrite: true
He podido identificar uno:
a) Falta incluir un needs: validar en el job Publicar (luego de la línea 41 del archivo .github/workflows/pipeline.yml). 
Esto trae como consecuencia que se pueda publicar el artefacto, aun cuando puede no haber pasado la validación del análisis de calidad del SonarCloud.
Verifica si ese error está presente e indica los otros 3. 
Se debe seguir las siguientes indicaciones: 
1.1 Los cuatro defectos. Para cada uno: qué está mal, en qué archivo y en qué líneas se manifiesta, y qué consecuencia tiene. Un defecto no es "falta una línea": es qué garantía se pierde por no tenerla.
Enumera con una letra cada defecto.

2.

De los cuatro, cuál explica el tiempo que registraron en docs/linea-base.md (de aproximadamente 1 minuto). Respuesta en párrafo.

3. 

Definición del requerimiento El analista redacta la historia de usuario: unas 4 horas de trabajo, con una espera previa de 16 horas. De cada 10 historias, 4 son devueltas después por el desarrollador o por QA porque el criterio de aceptación era ambiguo o estaba incompleto. Desarrollo El desarrollador implementa: alrededor de 16 horas efectivas, con 8 horas de espera en la cola del sprint. 3 de cada 10 implementaciones deben rehacerse parcialmente por un malentendido funcional o por conflictos al integrar. Revisión de código La revisión toma unas 2 horas y el pull request espera cerca de 24 horas a ser atendido. 35 de cada 100 pull requests son rechazados y devueltos al autor. Pruebas funcionales QA prueba el cambio: 8 horas de trabajo, precedidas de 20 horas de espera. Casi la mitad de los elementos —45 de cada 100— regresan a desarrollo con defectos. Aceptación del usuario El área de siniestros valida en un ambiente de aceptación: 6 horas de trabajo del usuario y 2 días de espera hasta que consigue agenda. 2 de cada 10 validaciones terminan en rechazo y vuelven atrás. Despliegue a producción El despliegue está semiautomatizado y toma unas 2 horas, con 1 día de espera hasta la ventana. 1 de cada 10 despliegues falla y debe repetirse.
Cuál de los cuatro defectos ataca la restricción del caso transversal? Respuesta breve y en párrafo

4.

Qué métrica DORA se espera mover con la intervención (el pipeline corregido) y por qué. Respuesta breve y en párrafo

5.

Qué número concreto se va a medir para sustentar que la métrica se movió. Respuesta breve en párrafo

6.

Ahora, se deben corregir los cuatro defectos en .github/workflows/pipeline.yml. Genera el nuevo contenido del archivo. El pipeline resultante debe cumplir: # Condición 1 Las dependencias se instalan desde el archivo de bloqueo, no resolviendo versiones 2 Las dependencias se cachean entre ejecuciones 3 El pipeline se detiene si el análisis de calidad no cumple el quality gate 4 El artifact publicado debe llamarse despachos-, solo desde main, y solo si la validación pasó Sobre el punto 4. La versión no se inventa: se deriva del historial de commits desde el tag v1.2.0. Revisen qué tipo de cambios hay desde ese tag y determinen si corresponde mayor, menor o parche. Actualicen VERSION y pyproject.toml con el valor que corresponda, y justifíquenlo en el entregable. Sobre el quality gate. Configuren sonar-project.properties con su organization key y su project key antes de la primera ejecución. Verificación. Ejecuten el pipeline y confirmen que pasa en verde y que el artefacto publicado lleva la versión en el nombre.




