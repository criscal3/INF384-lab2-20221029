# Diagnóstico
1.1. 
Los cuatros defectos son:
a) Falta incluir un needs: validar en el job Publicar (luego de la línea 41 del archivo .github/workflows/pipeline.yml). 
Esto trae como consecuencia que se pueda publicar el artefacto, aun cuando puede no haber pasado la validación del análisis de calidad del SonarCloud.

