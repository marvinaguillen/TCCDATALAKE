# Data Lakehouse en Streaming para el Censo de Conductores de la DGT 🚀

Este repositorio contiene el diseño, automatización y despliegue de una arquitectura de **Data Lakehouse distribuido y orientado a streaming** para procesar, limpiar y analizar el censo mensual de conductores de la Dirección General de Tráfico (DGT) de España.

A diferencia de las soluciones tradicionales basadas en lotes (Batch), este ecosistema implementa un paradigma moderno que separa por completo el almacenamiento del cómputo (**Compute vs Storage**) y proporciona soporte transaccional ACID en el Data Lake.

---

## 🏛️ Arquitectura del Sistema

La solución está diseñada bajo un enfoque estrictamente distribuido, aislando los recursos en tres niveles independientes de servidores para evitar cuellos de botella y garantizar la tolerancia a fallos:

1. **Capa de Ingestión (Streaming):** Un script automatizado en Python realiza *Web Scraping* sobre el portal oficial de la DGT para detectar nuevos ficheros mensuales `.txt`. Los registros se extraen y se envían fila por fila como eventos JSON a un clúster de **Apache Kafka**.
2. **Capa de Almacenamiento y Transaccionalidad (Lakehouse):** El framework **Kafka Connect (Hudi Sink)** consume los eventos en streaming y los persiste en **MinIO** (Object Storage compatible con la API de Amazon S3). Los datos se estructuran utilizando **Apache Hudi** en formato columnar Apache Parquet. Esto permite ejecutar operaciones **Upsert** (actualizaciones en caliente) ante correcciones de datos históricos.
3. **Capa de Cómputo Analítico y Consumo:** El motor de consultas distribuidas MPP en memoria **Trino (PrestoSQL)** actúa como el cerebro analítico. Lee el catálogo de Hudi de manera ultra rápida para servir los datos bajo demanda a un **Dashboard interactivo**.

---

## 🛠️ Tecnologías Utilizadas

* **Ingestión:** Apache Kafka, Kafka Connect, Python (BeautifulSoup & Requests).
* **Almacenamiento y Capa ACID:** MinIO Server, Apache Hudi, Apache Parquet.
* **Motor SQL:** Trino SQL (Presto).
* **Infraestructura como Código (IaC):** Ansible (Roles modularizados).
* **Automatización:** Linux Cron Jobs.

---

## 📂 Estructura del Proyecto

```bash
├── ansible/
│   ├── inventory.ini             # Definición de IPs de los servidores distribuidos
│   ├── deploy-lakehouse.yml      # Playbook maestro de Ansible
│   └── roles/
│       ├── common/               # Configuración base (Java 17, Python)
│       ├── minio/                # Despliegue del Object Storage
│       ├── kafka/                # Configuración de Kafka KRaft y Kafka Connect
│       └── trino/                # Instalación de Trino y catálogo Hudi
├── ingestion/
│   ├── productor_dgt.py          # Script de scraping e ingesta hacia Kafka
│   └── censo_cron.sh             # Automatización del disparador mensual
└── dashboard/
    └── app.js                    # Backend/Frontend para la visualización analítica

```
## 🚀 Despliegue Automático
El entorno se despliega de forma agnóstica a la infraestructura mediante Ansible, ya sea en máquinas locales o en instancias de Amazon EC2.

Configura las direcciones IP de tus servidores especializados en ansible/inventory.ini.

Ejecuta la receta maestra desde la carpeta ansible:
```bash
Bash
ansible-playbook -i inventory.ini deploy-lakehouse.yml
```

Configura el script de ingesta en el planificador del sistema (crontab -e) para automatizar la descarga mensual:

```bash

Bash
0 0 5 * * /usr/bin/python3 /opt/ingestion/productor_dgt.py
```
## 📈 Beneficios Técnicos Clave
Aislamiento del "Efecto Dominó": Las consultas pesadas en memoria por parte de Trino no comprometen el rendimiento ni la disponibilidad de Kafka para seguir recibiendo datos.

Soporte de Upserts: Rompe la limitación de inmutabilidad de los Data Lakes tradicionales, permitiendo actualizar registros específicos sin reescribir todo el histórico.

Escalabilidad Elástica: Permite apagar o escalar los nodos de cómputo (Trino) de forma independiente al almacenamiento (MinIO), optimizando drásticamente los costes operativos.

Autores: Juan Diego López & Marvin Antonio Guillén

Institución: Universitat Politècnica de València (UPV)

Asignatura: Tecnologías para el Continuo Computacional (TCC)

Profesor: Germán Moltó
