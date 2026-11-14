## Recepción (robertogr)

- [ ] Registrar paciente
- [ ] Asignar caso
- [ ] Asignar cita
- [ ] Exportar base de pacientes

## Estudiante (luciazd y luisgm)

- [ ] Elegir paciente
- [ ] Expediente clínico del paciente
- [ ] Odontograma

## Docente (mariahz)

- [ ] Supervisión
- [ ] Validación
- [ ] Finalizar caso

## Admin (valeriamz)

- [ ] Indicadores globales
- [ ] Usuarios
- [ ] Roles
- [ ] Catálogos

## Soporte (samuelpr)

- [ ] Cuentas y catálogos sin acceso clínico

## Guion: seguir la vida de un paciente (~30 min)

Recomiendo contarlo como un solo paciente que pasa por todo el sistema. Es más fácil de seguir que un recorrido pantalla por pantalla.

**A. Login.** Muestra la pantalla de entrada y el enlace "¿Olvidaste tu contraseña?". Explica que cada rol ve un menú distinto según sus permisos.

**B. Recepción, robertogr**
- Dashboard con el título de su rol.
- Pacientes → Registro de paciente: registra uno nuevo con foto y WhatsApp. Pon a propósito un DUI mal escrito para que se vea la validación.
- Directorio: buscar y filtrar, y Exportar CSV (ábrelo en Excel; los acentos salen bien).
- Asignaciones: asígnalo a luciazd usando el buscador y escribe una nota.
- Calendario de citas: crea una cita para ese paciente y luego reprográmala.

**C. Estudiante, luciazd**
- La campana muestra el aviso de la asignación.
- El dashboard solo muestra sus casos. Si alguien pregunta, esa regla la decidiste el 11 de octubre.
- Asignaciones: elige el segundo paciente pendiente. Así se ve que el estudiante también puede tomar casos por su cuenta.
- En la ficha del paciente: odontograma (marca dientes y guarda), diagnóstico, tratamiento (desde el catálogo del tarifario) y evolución clínica.
- Calendario: aparece su cita. Puedes contar que el sistema le avisa el día anterior; el comando existe, pero falta programarlo para que corra solo.

**D. Docente, mariahz**
- La campana muestra que hay un diagnóstico por validar.
- Supervisión: abre la cola y valida el diagnóstico. El estudiante recibe la notificación (enséñalo en la otra ventana).
- Base de estudiantes: el listado con sus casos.
- Finaliza el caso. La ficha queda en solo lectura y conserva el estudiante y la nota. Si quieres, compáralo con el paciente #8.

**E. Admin, valeriamz**
- Dashboard con indicadores globales: digitalización, % de diagnósticos validados, % de casos con diagnóstico validado.
- Usuarios: crea una cuenta y restablece una contraseña.
- Roles y permisos (el momento fuerte de la demo): quítale un permiso a un rol, por ejemplo calendar.view a recepción. Entra como robertogr y el menú ya no muestra el calendario. Devuélvele el permiso al terminar. Así demuestras que el control de acceso se configura sin tocar código.
- Catálogos clínicos: áreas y tratamientos. Ortodoncia está vacía a propósito, porque falta definirla con Rodrigo, y se completa desde esta pantalla sin programar.

**F. Soporte, samuelpr.** Gestiona usuarios y catálogos, pero no tiene acceso al expediente clínico. Aquí dices directamente que el panel de soporte técnico está pendiente.

- Ortodoncia no
- Validación para que guarde solo cuando se llene contactos
- Tienen que poder ver a todos los pacientes disponibles para que elijan a quien van a atender (sin solicitud).
- Deberían poder hacer el diagnóstico pero no necesariamente asignárselo para darle seguimiento.
- Cuando se da el primer seguimiento, dar un plazo de 30 días, si no le da seguimiento que lo libere para otra persona.
- Funcionalidad para poder liberar del lado de administración o recepción.
- Filtro por Tratamiento y columna nueva.
- Estudiantes que también puedan crear pacientes (dejar registrado quien lo crea y quien hizo el primer diagnóstico).
- Docentes no habría que agregar.
- Agregar una sección de seguridad de manejo de datos personales.
- Que en la barra de búsqueda salgan las etiquetas del diente y el TRATAMIENTO ESPECÍFICO (18 Endodoncia monorradicular).
- Agregar el filtro y que se pueda buscar por esa etiqueta también.
- Mejorar la interfaz gráfica.
- Validar una forma para que puedan mostrar la demo en vivo.
- Validación de no admitir texto en el campo de DUI.



Indicaciones:

**Propósito:** Traducir los resultados del pipeline de datos de la Fase I en una interfaz analítica intuitiva y accionable, reduciendo la carga cognitiva y potenciando la narrativa de datos (_Data Storytelling_).

### 1. Directrices Técnicas de la Interfaz / Cuadro de Mando

- **Gramática de Gráficos y Codificación Visual:**
    - Elección rigurosa de marcas y canales visuales (posición y longitud para comparaciones precisas; color y saturación únicamente con propósito funcional o de alerta).
    - Prohibición de elementos que distorsionen la percepción (gráficos 3D, gráficos circulares densos o ejes truncados sin justificación analítica).

- **Reducción de Carga Cognitiva (Principios Gestalt & Tufte):**
    - Optimización del _Data-to-Ink ratio_ (maximizar señal útil, eliminar _chartjunk_, bordes innecesarios y saturación tipográfica).
    - Aplicación de leyes de proximidad, similitud y cerramiento para agrupar métricas operacionales por contexto lógico.

- **Flujos de Interactividad (Heurística de Shneiderman):**
    - _Overview first:_ Tarjetas de indicadores clave (KPIs) y panorámica general del sistema.
    - _Zoom and filter:_ Filtros cruzados globales sincronizados (por tiempo, severidad, arquitectura o estado).
    - _Details-on-demand:_ _Tooltips_ enriquecidos, tablas de inspección y vistas desplegables (_drill-down_).
    
- **Narrativa de Datos (_Storytelling_):**
    - La interfaz debe articular un flujo guiado: 
    - **Diagnóstico inicial  --> Detección de patrones y anomalías --> Análisis de causa raíz --> Recomendación de acción/decisión.**

### 2. Entregables Requeridos

1. **Adenda al Artículo IEEE (2 a 3 páginas adicionales):**
    - _Sección V-B: Arquitectura Visual y Diseño de la Interfaz:_ Justificación teórica de canales, taxonomía de tareas visuales y decisiones para minimizar carga cognitiva.
    - _Capturas anotadas:_ Figuras en alta resolución demostrando casos de uso reales.
    
2. **Aplicación / Dashboard Funcional:** Enlace interactivo en la nube o repositorio reproducible (Streamlit, Dash, Looker Studio, Power BI, etc.) conectado a la data limpia.

### **3. Conjunto de datos**

**Los equipos cuentan con dos modalidades de trabajo:**

- ****Modalidad A (Continuidad):** Mantener el dataset de la Fase I (limpio y transformado), siempre que **no haya sido observado** en la retroalimentación anterior y su volumen/riqueza permita responder preguntas complejas sin que los gráficos queden despoblados al filtrar.**
- ****Modalidad B (Elección Libre de Mayor Escala):** Seleccionar voluntariamente un nuevo conjunto de datos público —o por requerimiento docente si el anterior fue observado— para contar con mayor escala y granularidad.**
    - ****Volumen recomendado:** Mínimo sugerido de **5,000 a 10,000+ registros** para garantizar densidad en segmentaciones multidimensionales.**
    - ****Dimensionalidad:** Al menos **8 a 12 variables** con mezcla de atributos temporales, categóricos (nominales y ordinales) y métricas continuas.**
    - ****Requisito metodológico:** Si eligen un dataset nuevo, deben aplicar y documentar previamente el pipeline de limpieza, imputación y estandarización de la Fase I antes de conectarlo a la interfaz.**

# Rúbrica de Evaluación: Visualización Interactiva, UX & Data Storytelling

Traducción de Datos Limpios a Interfaces Intuitivas de Baja Carga Cognitiva

|Criterio & Ponderación|Excelente (100% - 90%)|Bueno (89% - 75%)|Regular (74% - 60%)|Insuficiente (< 60%)|
|---|---|---|---|---|
|1. Calidad, Escala y Tratamiento del Dataset  <br>Peso: 15%|Dataset de escala robusta (≥5,000 filas, ≥8-10 vars) completamente limpio y consistente. Soporta segmentaciones profundas sin perder representatividad. Si es nuevo, presenta pipeline de limpieza previo impecable.|Dataset adecuado para visualización interactiva. Datos procesados correctamente, con ligeras limitaciones en granularidad o número de dimensiones analizables.|Volumen ajustado que limita la interactividad (filtros que dejan vistas vacías) o pipeline de limpieza del nuevo dataset documentado superficialmente.|Dataset excesivamente pequeño que impide el análisis interactivo, utiliza datos crudos/sucios o no justifica el preprocesamiento previo.|
|2. Gramática Visual y Eficacia Perceptiva  <br>Peso: 25%|Elección óptima de gráficos acorde al tipo de variable. Uso prioritario de canales visuales precisos (posición, longitud). Escalas legibles y sin distorsión perceptiva.|Gráficos adecuados y funcionales. Selección correcta en la mayoría de variables con detalles menores de escala o densidad de marcas.|Uso de gráficos subóptimos para las variables analizadas. Presencia de saturación visual o gráficos difíciles de interpretar rápidamente.|Errores graves en encodificación visual (gráficos inapropiados, ejes truncados sin justificar, uso de gráficos 3D o circulares densos).|
|3. Carga Cognitiva y Diseño UI (Gestalt & Tufte)  <br>Peso: 20%|Máximo ratio señal/ruido. Paleta de color semántica, funcional y accesible. Distribución limpia aplicando Gestalt. Cero chartjunk o elementos distractores.|Interfaz limpia y ordenada. Buena paleta de color y alineación visual, aunque con leves redundancias en leyendas o etiquetas.|Carga cognitiva moderada: exceso de colores sin propósito funcional, densidad excesiva de elementos o falta de jerarquía tipográfica.|Sobrecarga visual extrema: colores estridentes o aleatorios, desorden en la composición y saturación de elementos irrelevantes.|
|4. Interactividad y Usabilidad (Shneiderman)  <br>Peso: 20%|Flujo interactivo fluido: vista general intuitiva, filtros cruzados globales reactivos y tooltips contextuales ricos en información a demanda.|Interactividad funcional con filtros operativos. Permite explorar y filtrar datos eficazmente, con transiciones o respuestas aceptables.|Interactividad limitada o desarticulada (filtros que no afectan todas las vistas relevantes o tooltips con información redundante).|Interfaz estática o interactividad rota (filtros que generan errores, navegación confusa sin orden lógico).|
|5. Data Storytelling, Adenda IEEE y Demostración  <br>Peso: 20%|Narrativa contundente desde alerta a acción. Adenda IEEE rigurosa con capturas anotadas.|Buena estructura orientada al negocio. Documentación clara y estructurada con explicaciones comprensibles.|Gráficos aislados con narrativa débil. Documentación incompleta o con explicaciones superficiales.|Carencia total de narrativa. Adenda IEEE ausente, dashboard inoperativo.|