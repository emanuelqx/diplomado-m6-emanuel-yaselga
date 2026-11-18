# Sistema de decisión para PagoYa — Grupo omega
**Integrantes:** Emanuel Yaselga · Edwin Cevallos · Evelyn Cueva · Pablo Segarra   ·   **Problema:** segmentación de clientes y detección de usuarios anómalos
*Diplomado en Big Data e IA · CEC-EPN · Módulo 6 · Datos simulados*

## 1. Problema y datos
PagoYa trata a todos sus usuarios igual: mismos límites, mismos controles. Buscamos **grupos de clientes con comportamiento parecido** para darles productos y controles distintos, e identificar a los **usuarios anómalos** que conviene revisar.
Datos: **781 usuarios**, perfiles agregados de 2 996 transacciones (ingreso, edad, antigüedad, número y monto de operaciones, % de madrugada, % desde dispositivo nuevo, distancia, intentos de PIN). Sin faltantes ni duplicados. Las variables de dinero son muy asimétricas, así que usamos logaritmo y escalado. **El fraude no se usó para formar los grupos**: sirve solo para validarlos.

**Orden del proceso:** primero detectamos las anomalías y las retiramos; después segmentamos a los 754 usuarios restantes. Los extremos arrastran los centroides de K-Means: al incluirlos, la estabilidad baja (ARI 0,75) y aparece un segmento pequeño cuyo único rasgo son los intentos de PIN; al retirarlos, la estabilidad sube a ARI 0,86 y ese segmento desaparece.

## 2. Modelos comparados
| Modelo | Uso | Resultado |
|---|---|---|
| **Isolation Forest + LOF** (5 %) | Anomalías | Cada uno marca 39 usuarios; coinciden en 27 (3,5 %). Tasa de fraude 16,7 % frente a 8,6 % del resto |
| **K-Means, k = 4** (elegido) | Segmentar 754 usuarios | Silueta 0,108 · Davies-Bouldin 2,29 · estabilidad ARI 0,86 con 10 semillas |
| Jerárquico Ward, k = 4 | Segmentar | Silueta 0,076 · Davies-Bouldin 2,55 |
| DBSCAN | Segmentar | No encuentra grupos de densidad (98 % ruido con eps = 1,5; un solo grupo con eps = 2,0 y 2,5) |
| K-Means **con** anomalías (comparación) | Segmentar 781 usuarios | Silueta 0,108 · estabilidad ARI 0,75 · segmento más pequeño de 64 usuarios |

**Validación de los detectores con anomalías sembradas** (15 perfiles anómalos conocidos, inyectados en una copia de los datos solo para evaluar): Isolation Forest los encuentra todos (recall 1,0); LOF no encuentra ninguno (0,0), porque los 15 forman un grupo compacto y con 20 vecinos LOF los ve como vecindario normal. La regla «los marcan ambos» es conservadora y sirve para anomalías dispersas, pero no detectaría un grupo organizado.

La silueta es baja con cualquier k (0,11–0,14): los clientes forman un continuo, no tipos puros. Elegimos k = 4 porque los cuatro perfiles son interpretables y accionables, con un grupo más pequeño de 173 usuarios.

## 3. Qué explica el resultado
| Segmento | Usuarios | Qué lo define | Tasa de fraude | % de los fraudes |
|---|---|---|---|---|
| 0 · Básicos de bajo monto | 197 (25 %) | Ingreso USD 576, pagos de USD 32 | 4,6 % | 14 % |
| 1 · Diurnos estables | 185 (24 %) | Ingreso USD 1 092; nunca de madrugada ni desde dispositivos nuevos | 4,2 % | 7 % |
| 2 · Nocturnos | 173 (22 %) | 30 % de sus pagos de madrugada; 4,5 operaciones | 12,5 % | 34 % |
| 3 · Alto valor multidispositivo | 199 (25 %) | Ingreso USD 1 274, pagos de USD 127; 26 % desde dispositivo nuevo | 13,1 % | 42 % |
| Anómalos | 27 (3 %) | Poca historia (1,6 operaciones), 44 % de madrugada, intentos de PIN altos | 16,7 % | 3 % |

El fraude no se usó para formar los grupos y aun así separa dos niveles de riesgo: 4–5 % en los segmentos 0 y 1 frente a 12–13 % en los 2 y 3.
Ejemplo de anómalo: U0284 paga siempre desde un dispositivo nuevo, su pago mayor es el 52 % de su ingreso y promedia 2,3 intentos de PIN.

## 4. Recomendación con cifras
- **Alto valor multidispositivo (25 % de usuarios, 42 % de los fraudes):** verificación de identidad (biometría) al cambiar de dispositivo; límites altos con confirmación si el pago supera el 40 % del ingreso.
- **Nocturnos (22 %, 34 % de los fraudes):** alertas en tiempo real de operaciones de madrugada, límite a montos nocturnos y educación antifraude.
- Estos dos segmentos suman el **47 % de los usuarios y el 76 % de los fraudes**: ahí rinde más el monitoreo.
- **Básicos de bajo monto (25 %, tasa 4,6 %) y Diurnos estables (24 %, tasa 4,2 %):** menos fricción; productos de entrada y de ahorro. Son el 49 % de la base y el 21 % de los fraudes.
- **Anómalos (3 %):** cola de revisión manual semanal y límites temporales hasta verificar identidad. **Sin bloqueo automático:** «anómalo» no es «fraudulento».

## 5. Limitaciones y próximos pasos
- Datos **simulados**; los segmentos deben recalcularse con datos reales.
- Grupos poco separados (silueta 0,11): un cliente cerca del límite puede cambiar de segmento. Conviene recalcular cada trimestre y vigilar la estabilidad.
- El 5 % de anomalías es una decisión de capacidad de revisión, no un dato. Los anómalos son pocos y con pocas operaciones, así que su mayor tasa de fraude es una señal orientativa.
- La regla «Isolation Forest y LOF» no detectó anomalías sembradas en grupo (LOF, recall 0); si el riesgo es un grupo organizado, usar Isolation Forest solo o «alguno de los dos».
- Próximos pasos: piloto A/B de cada acción por segmento y variables adicionales (tipo de comercio, canal, historial).

# Proyecto final integrador (40 pts · grupal)
Se define el **lunes 16 de noviembre** (Clase 4). Esta carpeta se usa solo en el repositorio del grupo
(un integrante; los demás se agregan como colaboradores).

Archivos obligatorios (el verificador los revisa):

| Archivo | Contenido |
|---|---|
| `proyecto/proyecto.ipynb` | Cuaderno ejecutado de principio a fin con datos de la app simulada | cumplido
| `proyecto/README.md` | Informe ejecutivo (1–2 páginas): problema, datos, modelos, resultados, recomendación, limitaciones | cumplido
| `proyecto/ficha_modelo.json` | Ficha del modelo: métricas, fecha, semilla y versiones | cumplido
| `proyecto/presentacion.pdf` | Diapositivas de la defensa | cumplido

**Plazo: jueves 19 de noviembre de 2026, 18:00.** Se califica el último commit anterior a esa hora.
