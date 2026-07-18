---
tags: [data-science, machine-learning, roles, carrera, mercado-laboral]
aliases: [Roles de Datos, Carreras en Ciencia de Datos]
---
# Parte 2 — Roles y Carreras en Datos, ML e IA

> Ver primero [[01-Fundamentos-y-Panorama-General]] para el mapa conceptual. Esta nota conecta cada rol con la etapa del ciclo (sección 13 de la Parte 1) de la que es responsable. Los rangos salariales son **orientativos e internacionales** (principalmente USA/Europa Occidental como referencia alta, con nota sobre LatAm/remoto) — varían mucho por país, empresa y experiencia; trátalos como orden de magnitud, no como cifra exacta.

---

## Tabla de Contenidos

1. [[#Cómo Leer Esta Nota|Cómo Leer Esta Nota]]
2. [[#Data Analyst|Data Analyst]]
3. [[#Data Scientist|Data Scientist]]
4. [[#Machine Learning Engineer (ML Engineer)|Machine Learning Engineer (ML Engineer)]]
5. [[#MLOps Engineer|MLOps Engineer]]
6. [[#Data Engineer|Data Engineer]]
7. [[#Analytics Engineer|Analytics Engineer]]
8. [[#Data Architect|Data Architect]]
9. [[#Deep Learning / AI Research Engineer|Deep Learning / AI Research Engineer]]
10. [[#BI Developer / BI Analyst|BI Developer / BI Analyst]]
11. [[#Statistician / Applied Scientist|Statistician / Applied Scientist]]
12. [[#Tabla comparativa rápida|Tabla comparativa rápida]]
13. [[#Nota sobre tu contexto (consultor de ingeniería, WFM)|Nota sobre tu contexto (consultor de ingeniería, WFM)]]
14. [[#Ver también|Ver también]]

---
## Cómo Leer Esta Nota

Para cada rol: **qué hace**, **herramientas**, **conocimientos necesarios**, **salario internacional aproximado**, **quién contrata**, y cuatro ejes comparables — programación, matemáticas, nube, y en qué etapa del ciclo de [[01-Fundamentos-y-Panorama-General]] participa.

---

## Data Analyst

**Qué hace:** Explora datos existentes para responder preguntas de negocio, construye dashboards y reportes, identifica tendencias. Rara vez construye modelos predictivos; su output principal son **insights y visualizaciones**, no modelos.

- **Herramientas:** SQL (imprescindible), Excel/Google Sheets, Power BI, Tableau, Data Studio, Python/R básico (pandas, matplotlib).
- **Conocimientos:** Estadística descriptiva, modelado de datos básico, storytelling con datos, el negocio específico de la empresa.
- **Salario internacional:** ~USD 55,000–90,000 (USA/Europa Occidental, junior-mid). En LatAm remoto para empresas extranjeras, con frecuencia 30-50% de esa banda.
- **Empresas que contratan:** Prácticamente todas — retail, banca, telecom, startups. Es el rol de entrada más común al mundo de datos.
- **Programación:** Baja-media (SQL fuerte, Python opcional).
- **Matemáticas:** Media (estadística descriptiva).
- **Nube:** Baja (a veces consulta datos en un Data Warehouse en la nube, pero no la administra).
- **Etapa del ciclo:** EDA y comunicación de insights (paso 3).

---

## Data Scientist

**Qué hace:** Formula hipótesis, diseña experimentos, construye modelos predictivos/estadísticos para resolver problemas de negocio específicos (predicción de churn, pricing, forecasting, segmentación). Es el rol "clásico" que combina estadística + programación + negocio.

- **Herramientas:** Python (pandas, scikit-learn, statsmodels), SQL, Jupyter, a veces R, herramientas de visualización, Git.
- **Conocimientos:** Estadística inferencial y probabilidad, ML clásico, diseño experimental (A/B testing), comunicación de resultados a no técnicos.
- **Salario internacional:** ~USD 90,000–150,000 (mid-senior, USA); Europa Occidental algo menor; LatAm remoto frecuentemente USD 30,000-70,000.
- **Empresas que contratan:** Tech, fintech, e-commerce, consultoría de datos, banca, telecom.
- **Programación:** Media-alta.
- **Matemáticas:** Alta (estadística, probabilidad, álgebra lineal básica).
- **Nube:** Media (usa servicios de ML gestionados, no siempre los administra).
- **Etapa del ciclo:** EDA, feature engineering, entrenamiento y validación (pasos 3-6).

---

## Machine Learning Engineer (ML Engineer)

**Qué hace:** Toma los modelos que un Data Scientist prototipa (a menudo en un notebook) y los convierte en **sistemas de software robustos, escalables y mantenibles**. Es la bisagra entre ciencia de datos e ingeniería de software.

- **Herramientas:** Python avanzado, Docker, APIs (FastAPI/Flask), frameworks de ML (scikit-learn, PyTorch, TensorFlow), Git, CI/CD básico, a veces Kubernetes.
- **Conocimientos:** Ingeniería de software (patrones de diseño, testing), fundamentos de ML, sistemas distribuidos básicos, APIs.
- **Salario internacional:** ~USD 110,000–170,000 (USA, mid-senior). Suele pagar más que Data Scientist puro por el componente de ingeniería.
- **Empresas que contratan:** Tech grande y mediana, cualquier empresa con modelos en producción a escala.
- **Programación:** Alta (el rol con más código "de verdad" dentro del área de ML).
- **Matemáticas:** Media (entiende los modelos, pero no siempre los diseña desde cero).
- **Nube:** Alta (despliega en la nube constantemente).
- **Etapa del ciclo:** Del modelo entrenado al despliegue y monitoreo (pasos 5-9).

---

## MLOps Engineer

**Qué hace:** Construye y mantiene la **infraestructura y automatización** que permite que los modelos de ML se entrenen, desplieguen, versionen y monitoreen de forma confiable y repetible — el equivalente a DevOps pero para el ciclo de vida de modelos de ML.

- **Herramientas:** MLflow, Kubeflow, Airflow, Docker, Kubernetes, Terraform, herramientas de CI/CD (GitHub Actions, Jenkins, GitLab CI), plataformas cloud de ML (SageMaker, Vertex AI, Azure ML), feature stores (Feast, Tecton).
- **Conocimientos:** DevOps/infraestructura, contenedores y orquestación, fundamentos de ML (para saber qué automatizar), monitoreo y observabilidad, versionado de datos y modelos.
- **Salario internacional:** ~USD 120,000–180,000 (USA, mid-senior) — uno de los roles mejor pagados por su escasez relativa.
- **Empresas que contratan:** Empresas con madurez de ML alta — tech grande, fintech, empresas con equipos de ML establecidos (varias plataformas y bancos en LatAm ya lo están construyendo, típicamente dentro de consultorías o áreas de datos como la tuya).
- **Programación:** Alta (pero orientada a infraestructura/automatización, no a modelado).
- **Matemáticas:** Baja-media (no diseña modelos, los opera).
- **Nube:** Muy alta — es prácticamente el eje central del rol.
- **Etapa del ciclo:** Despliegue, monitoreo, reentrenamiento (pasos 8-10) — y en general, hacer que _todo el ciclo_ sea automático.

---

## Data Engineer

**Qué hace:** Diseña, construye y mantiene los sistemas que **mueven y transforman datos** desde las fuentes (apps, sensores, bases de datos operacionales) hasta los sistemas donde analistas y científicos de datos pueden usarlos (data warehouse, data lake). Es quien construye las "tuberías" que todos los demás roles dan por sentadas.

- **Herramientas:** SQL avanzado, Python/Scala, Apache Spark, Airflow, Kafka, dbt, herramientas de Data Warehouse (Snowflake, BigQuery, Redshift), Docker.
- **Conocimientos:** Modelado de datos (dimensional, normalización), sistemas distribuidos, arquitectura de datos, optimización de queries.
- **Salario internacional:** ~USD 100,000–160,000 (USA, mid-senior).
- **Empresas que contratan:** Cualquier empresa con volumen de datos serio — es de los roles con mayor demanda sostenida en los últimos años.
- **Programación:** Alta (SQL + Python/Scala/Java).
- **Matemáticas:** Baja-media.
- **Nube:** Alta.
- **Etapa del ciclo:** Ingesta y transformación de datos (paso 2) — la base de todo lo demás. Ver [[04-Ingenieria-de-Datos]] para el detalle técnico completo.

---

## Analytics Engineer

**Qué hace:** Un rol relativamente nuevo (popularizado por dbt) que vive entre Data Engineer y Data Analyst: toma datos ya cargados en el Data Warehouse y los **modela, limpia y documenta** para que sean fáciles de consumir de forma confiable por analistas y dashboards, aplicando buenas prácticas de ingeniería de software (control de versiones, testing) a las transformaciones SQL.

- **Herramientas:** dbt (su herramienta insignia), SQL avanzado, Git, Data Warehouses (Snowflake/BigQuery), Looker/similar.
- **Conocimientos:** Modelado dimensional, SQL avanzado, fundamentos de ingeniería de software aplicados a analítica.
- **Salario internacional:** ~USD 90,000–140,000 (USA).
- **Empresas que contratan:** Empresas data-driven medianas/grandes que adoptaron el enfoque moderno "ELT + dbt".
- **Programación:** Media-alta (SQL principalmente, algo de Python/Jinja).
- **Matemáticas:** Baja.
- **Nube:** Alta (vive dentro del Data Warehouse en la nube).
- **Etapa del ciclo:** Transformación de datos ya cargados (parte del paso 2, específicamente la "T" de ELT).

---

## Data Architect

**Qué hace:** Diseña la **arquitectura general** de cómo fluyen y se almacenan los datos en toda la organización — decide si usar Data Warehouse, Data Lake o Lakehouse, qué herramientas encajan entre sí, cómo se gobierna el dato. Es un rol más senior y estratégico, menos "manos en el código" día a día.

- **Herramientas:** Conocimiento amplio de todo el stack (warehouses, lakes, ETL, gobernanza), diagramas de arquitectura, a veces herramientas de catálogo de datos (Collibra, Atlan).
- **Conocimientos:** Arquitectura de sistemas, gobernanza y seguridad de datos, costos de infraestructura cloud, todo lo cubierto en [[03-Arquitectura-Empresarial-de-Datos-y-ML]].
- **Salario internacional:** ~USD 140,000–200,000 (USA, senior).
- **Empresas que contratan:** Empresas grandes con arquitecturas de datos complejas, consultoras.
- **Programación:** Media (diseña, no siempre implementa).
- **Matemáticas:** Baja.
- **Nube:** Muy alta.
- **Etapa del ciclo:** Diseña el "mapa" completo de la sección 13 de [[01-Fundamentos-y-Panorama-General]].

---

## Deep Learning / AI Research Engineer

**Qué hace:** Diseña, entrena y optimiza arquitecturas de redes neuronales para problemas de visión por computadora, NLP, audio, o investigación de nuevas arquitecturas. El rol más matemáticamente intensivo del área.

- **Herramientas:** PyTorch (dominante en investigación), TensorFlow/Keras, CUDA, Hugging Face, clusters de GPU/TPU, herramientas de experiment tracking (Weights & Biases, MLflow).
- **Conocimientos:** Álgebra lineal y cálculos avanzados, arquitecturas de redes neuronales (CNNs, Transformers, etc.), optimización, papers de investigación recientes.
- **Salario internacional:** ~USD 150,000–300,000+ (USA, empresas de IA de punta) — el rol con el rango salarial más alto y más disperso del área.
- **Empresas que contratan:** Labs de IA (OpenAI, Anthropic, Google DeepMind, Meta AI), tech grande, empresas de visión/NLP especializadas.
- **Programación:** Alta.
- **Matemáticas:** Muy alta.
- **Nube:** Alta (entrenamiento distribuido en clusters).
- **Etapa del ciclo:** Entrenamiento de modelos (paso 5), con foco en arquitecturas de Deep Learning específicamente.

---

## BI Developer / BI Analyst

**Qué hace:** Construye y mantiene reportes y dashboards de Business Intelligence de forma más especializada que un Data Analyst genérico — a menudo dueño de todo el proceso desde el modelo de datos semántico hasta la visualización final.

- **Herramientas:** Power BI, Tableau, Data Studio, SQL, a veces DAX (Power BI) o LookML (Looker).
- **Conocimientos:** Modelado dimensional, diseño de visualizaciones efectivas, el negocio.
- **Salario internacional:** ~USD 65,000–110,000 (USA).
- **Empresas que contratan:** Prácticamente todas las medianas/grandes.
- **Programación:** Baja-media.
- **Matemáticas:** Baja.
- **Nube:** Media.
- **Etapa del ciclo:** Comunicación de insights (paso 3), similar a Data Analyst pero más especializado en la capa de visualización.

---

## Statistician / Applied Scientist

**Qué hace:** Diseña experimentos rigurosos (A/B tests, ensayos clínicos, estudios causales), construye modelos estadísticos formales, valida supuestos matemáticos que otros roles a veces dan por sentados. Muy presente en tech (experimentación a gran escala), farmacéuticas, seguros.

- **Herramientas:** R, Python (statsmodels, scipy), SAS (en industrias reguladas como farma/seguros), herramientas de experimentación.
- **Conocimientos:** Estadística inferencial avanzada, inferencia causal, diseño experimental, probabilidad.
- **Salario internacional:** ~USD 100,000–170,000 (USA).
- **Empresas que contratan:** Tech grande (equipos de experimentación), farmacéuticas, seguros, banca, gobierno/investigación.
- **Programación:** Media.
- **Matemáticas:** Muy alta.
- **Nube:** Baja-media.
- **Etapa del ciclo:** Validación rigurosa (paso 6) y diseño experimental previo al modelado.

---

## Tabla comparativa rápida

|Rol|Programación|Matemáticas|Nube|Etapa principal del ciclo|
|---|---|---|---|---|
|Data Analyst|Baja-media|Media|Baja|EDA / comunicación|
|Data Scientist|Media-alta|Alta|Media|EDA → validación|
|ML Engineer|Alta|Media|Alta|Modelo → despliegue|
|MLOps Engineer|Alta|Baja-media|Muy alta|Despliegue → reentrenamiento|
|Data Engineer|Alta|Baja-media|Alta|Ingesta / transformación|
|Analytics Engineer|Media-alta|Baja|Alta|Transformación (ELT)|
|Data Architect|Media|Baja|Muy alta|Diseño de todo el flujo|
|DL/AI Research Engineer|Alta|Muy alta|Alta|Entrenamiento (DL)|
|BI Developer|Baja-media|Baja|Media|Comunicación de insights|
|Statistician|Media|Muy alta|Baja-media|Validación / diseño experimental|

---

## Nota sobre tu contexto (consultor de ingeniería, WFM)

Un rol de **consultor de ingeniería** que toca análisis de datos, ciencia de datos y MLOps sobre proyectos de _Workforce Management_ (como los que llevas para Claro RD) típicamente combina piezas de **Data Scientist** (forecasting, modelos de predicción de demanda/dotación), **Data Engineer** (pipelines que alimentan esos modelos) y **MLOps** (si esos modelos corren de forma recurrente en producción). Vale la pena, cuando documentes tus proyectos reales, anotar explícitamente **qué "sombrero" de esta lista estás usando en cada tarea** — te ayuda a identificar en qué área quieres profundizar más.

---

## Ver también

- [[01-Fundamentos-y-Panorama-General]]
- [[03-Arquitectura-Empresarial-de-Datos-y-ML]]
- [[04-Ingenieria-de-Datos]]
