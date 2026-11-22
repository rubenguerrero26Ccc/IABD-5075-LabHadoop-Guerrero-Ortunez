# IABD-5075-LabHadoop-Guerrero-Ortunez

Despliegue de un clúster multi-nodo Apache Hadoop con Docker, auditoría física en HDFS y simulación de tolerancia a fallos.

Laboratorio (AEE) · Módulo 5074/5075

## Autoría

- **Alumno:** Ruben Guerrero Ortunez
- **Curso / Módulo:** Big Data Aplicado · Módulo 5074/5075
- **Fecha:** 22 de noviembre de 2026

## Arquitectura desplegada

Seis contenedores sobre la red puente aislada `hadoop-net`, todos con la imagen `apache/hadoop:3` y la configuración común de `docker/config`.

| Contenedor | Rol | Puerto publicado |
|---|---|---|
| `namenode` | Maestro de HDFS (metadatos) | 9870 (interfaz web) |
| `datanode1` | Trabajador de HDFS (bloques) | — |
| `datanode2` | Trabajador de HDFS (bloques) | — |
| `datanode3` | Trabajador de HDFS (bloques) | — |
| `resourcemanager` | Maestro de YARN (recursos) | 8088 (interfaz web) |
| `nodemanager` | Trabajador de YARN (contenedores) | — |

**Parámetros clave** (`docker/config`):

- Tamaño de bloque: 64 MiB (`dfs.blocksize=67108864`)
- Factor de replicación: 2 (`dfs.replication=2`)
- NodeManager: 2048 MB de memoria y 2 vCores

## Requisitos

- Docker Desktop (o Docker Engine con Compose v2) en ejecución
- Al menos 6 GB de RAM libres
- Git y una terminal (PowerShell o Bash)

## Instrucciones de arranque

```bash
cd docker
docker compose up -d
docker compose ps
docker exec -it namenode bash
hdfs dfsadmin -report
```

Interfaces web: NameNode en http://localhost:9870 y ResourceManager de YARN en http://localhost:8088.

Para apagar el clúster y liberar los volúmenes:

```bash
docker compose down -v
```

## Resumen de resultados

- **Fragmentación:** un fichero de 150 MiB (157.286.400 B) se divide en 3 bloques de 64, 64 y 22 MiB (el último ocupa solo los bytes reales). Con R=2 hay 6 réplicas físicas repartidas entre los DataNodes.
- **Tolerancia a fallos:** al detener `datanode2`, el NameNode lo mantuvo como activo, pasó por una fase *stale* y lo declaró *Dead* tras unos 10-11 minutos de silencio. Los datos siguieron siendo legibles con una réplica viva por bloque, y el NameNode re-replicó los bloques afectados.
- **MapReduce sobre YARN:** el WordCount procesó un corpus de 26 palabras y generó 21 palabras distintas. Con el límite de 2048 MB del NodeManager hubo que reducir la memoria de los contenedores de MapReduce mediante parámetros `-D` para que el trabajo avanzara. El detalle está en el informe.

## Estructura del repositorio

```
.
├── .gitignore
├── README.md
├── docker/
│   ├── config
│   └── docker-compose.yml
├── evidencias/
│   ├── 01_namenode_ui_9870.png
│   ├── 02_hdfs_fsck_bloques.png
│   ├── 03_simulacion_fallo_datanode2.png
│   └── 04_yarn_wordcount_8088.png
└── informe/
    └── Guerrero_AOrtunez_InformeLabHadoop.pdf
```
