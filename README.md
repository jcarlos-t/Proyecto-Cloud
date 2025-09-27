# Proyecto-Cloud
Gestión de cursos, proyecto implementado para el curso de cloud computing

# MS4 — Microservicio de Analítica

Este microservicio expone una API REST construida con FastAPI que permite consultar métricas de uso y cursos desde registros almacenados en AWS Athena.

Endpoint

Top Courses: devuelve los cursos más utilizados en un rango de fechas.
(Se pueden añadir más consultas pero necesito sus tablas y los atributos de las tablas).
Yo propongo crear un S3 y con subcarpetas para que sea más facil el importe a Glue para las bd

Tecnologías
- Python 3.12
- FastAPI
- Boto3 / AWS SDK (ejecución de queries en Athena)
- Athena + S3 como motor de consulta y almacenamiento de logs

link: https://github.com/EV081/ms4.git


# MS3 - Evaluaciones & Progreso API
https://github.com/jcarlos-t/Evaluaciones-MS3.git


# MS2 - Cursos
https://github.com/J-D-Rosales/ms2_cursos.git
