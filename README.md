# Fine-tuning con LoRA y QLoRA (`Qwen2.5-1.5B-Instruct`)

Práctica de Fine-tuning **QLoRA (4 bits)** ejecutada en GPU Tesla T4 gratuita (Google Colab) para adaptar el estilo y formato de respuesta de un modelo de lenguaje al ámbito de **gestión de alquileres**.

## Contenido del repositorio
- `08_finetuning.ipynb`: Notebook completo y ejecutado con todas las salidas visibles, gráfica de evolución del *loss*, explicaciones paso a paso y comparativa antes vs. después.

## Resumen de resultados
- **Modelo base:** `Qwen/Qwen2.5-1.5B-Instruct`.
- **Configuración LoRA:** `target_modules="all-linear"`, `r=8`, `lora_alpha=16`, `10 épocas`, `learning_rate=3e-4`.
- **Dataset:** 18 ejemplos propios de incidencias de alquiler con formato estricto de 5 líneas (titular + 3 viñetas con `- ` + firma fija `— Equipo de Gestión de Alquileres`).
- **Evaluación sobre 4 preguntas inéditas:**
  - **Modelo Base:** `0/4` cumplen el formato.
  - **Modelo Ajustado (LoRA):** `3/4` cumplen el formato exacto (75% de acierto).

## Conclusiones y comparativa: Modelo Base (sin LoRA) vs. Modelo Ajustado (con QLoRA)

1. **Cumplimiento de la estructura y formato:**
   - **Modelo Base (sin LoRA):** No mantiene en ningún caso la estructura solicitada (`0/4`). Responde con párrafos extensos, listas numeradas (`1., 2., 3.`), cambios al inglés sin motivo (en la consulta sobre las visitas del propietario) y fallos de comprensión ante vocabulario local de vivienda en España (como *"persiana"*), donde se evade inventando la palabra *"malentendu"* (mezcla con el francés *malentendu*). Además, malinterpreta conceptos cotidianos, confundiendo *"se ha fundido la vitrocerámica"* con un proceso químico de fundición de materiales.
   - **Modelo Ajustado (con QLoRA):** Tras entrenar únicamente el adaptador sobre las capas lineales (`all-linear`, `r=8`, `alpha=16`, 10 épocas) con 18 ejemplos propios de gestión de alquileres, el modelo adopta el formato estricto aprendido: un titular sin guion, tres viñetas iniciadas por `"- "` y la firma literal `— Equipo de Gestión de Alquileres`.

2. **Análisis de alucinaciones y límites observados (Fine-tuning vs. RAG):**
   - **La forma frente a los datos:** El descenso pronunciado del *training loss* demuestra que LoRA fija eficazmente el estilo y la estructura sintáctica con pocos ejemplos.
   - **Sobreajuste, ruido de tokenización y alucinaciones factuales:** Al ajustar un modelo ligero (`Qwen2.5-1.5B-Instruct`) durante 10 épocas con un dataset reducido (18 ejemplos), el modelo fuerza el tono de inmobiliaria e inventa información inexistente cuando no tiene contexto real (como asegurar que existe un seguro obligatorio, inventar una fianza de *30.500 euros netos* en la prueba de contrato o generar pequeños artefactos de decodificación como caracteres no reconocidos ``).
   - **Conclusión técnica:** El *fine-tuning* con QLoRA resuelve **cómo debe responder** el asistente (formato, tono y cierre), pero para evitar alucinaciones y aportar datos reales de contratos o normativas debe combinarse en producción con un sistema **RAG**.
