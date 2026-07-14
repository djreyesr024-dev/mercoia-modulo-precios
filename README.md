# Módulo de consulta de precios agrícolas – MercoIA

Repositorio técnico del módulo de consulta de precios agrícolas desarrollado para el proyecto **MercoIA**, orientado a permitir que usuarios consulten precios mayoristas de productos agropecuarios mediante WhatsApp.

El módulo recibe mensajes en lenguaje natural, identifica el producto y la fecha solicitada, consulta información de precios publicada en los anexos diarios de **SIPSA del DANE**, procesa los datos relevantes y genera una respuesta estructurada para el usuario final.

---

## 1. Descripción general

El módulo de precios de MercoIA funciona como un asistente conversacional conectado a WhatsApp. Su propósito es facilitar la consulta de precios mayoristas de productos agrícolas en mercados de referencia, principalmente **Centroabastos S.A.** y **Corabastos S.A.**, usando como fuente de información los archivos diarios publicados por SIPSA.

El sistema fue diseñado para recibir consultas como:

```text
precio del banano
precio de la mora de ayer
cuánto está la papa criolla
precio del plátano hartón verde de anteayer
```

A partir del mensaje del usuario, el flujo identifica la intención, extrae el producto consultado, interpreta la fecha cuando está presente, descarga la base SIPSA correspondiente, normaliza el nombre del producto y construye una respuesta con precios por mercado.

---

## 2. Objetivo del módulo

Automatizar la consulta de precios mayoristas de productos agropecuarios mediante una interfaz conversacional por WhatsApp, entregando información clara y útil sobre:

- producto consultado;
- fecha del reporte;
- plaza mayorista;
- región;
- precio en pesos colombianos por kilogramo;
- variación aproximada frente al día de mercado anterior;
- disponibilidad o no disponibilidad de información.

---

## 3. Estado actual del desarrollo

El módulo cuenta con un flujo funcional implementado en **n8n**, integrado inicialmente con **Evolution API** para la recepción y envío de mensajes por WhatsApp.

La versión documentada en este repositorio corresponde al flujo desarrollado y probado con:

- n8n como orquestador principal;
- Evolution API como mecanismo inicial de integración con WhatsApp;
- OpenAI como componente de apoyo para la normalización de productos;
- SIPSA Diario del DANE como fuente principal de datos;
- archivos XLSX como formato de entrada;
- nodos Code de n8n con lógica en JavaScript;
- servidor Hostinger como ambiente utilizado durante el desarrollo y despliegue inicial.

Actualmente, el proyecto se encuentra en proceso de transición hacia **WhatsApp Business Platform – Cloud API de Meta**, con el objetivo de reemplazar la dependencia de Evolution API por una integración oficial y más estable.

---

## 4. Arquitectura general

El flujo general del módulo es el siguiente:

```text
Usuario en WhatsApp
        ↓
API de mensajería
        ↓
Webhook en n8n
        ↓
Extracción de variables del mensaje
        ↓
Clasificación del mensaje
        ↓
Detección de producto y fecha
        ↓
Validación de productos ambiguos
        ↓
Generación de URL SIPSA
        ↓
Descarga del archivo XLSX
        ↓
Lectura y limpieza de datos
        ↓
Normalización del producto consultado
        ↓
Extracción de precios reales
        ↓
Construcción de respuesta
        ↓
Envío de respuesta al usuario por WhatsApp
```

---

## 5. Tecnologías utilizadas

| Tecnología | Función dentro del módulo |
|---|---|
| WhatsApp | Canal de comunicación con el usuario final |
| Evolution API | Integración inicial para recibir y enviar mensajes |
| WhatsApp Business Platform – Cloud API | Integración oficial prevista para la siguiente fase |
| n8n | Orquestación del flujo automatizado |
| OpenAI | Normalización semántica del producto solicitado |
| SIPSA Diario – DANE | Fuente principal de información de precios |
| XLSX | Formato de los archivos diarios consultados |
| JavaScript | Lógica utilizada en nodos Code de n8n |
| Hostinger | Ambiente de despliegue utilizado durante el desarrollo |

---

## 6. Fuente de datos

La fuente principal de datos es **SIPSA Diario del DANE**, particularmente los anexos diarios publicados en formato XLSX.

El sistema genera dinámicamente la URL del archivo según la fecha requerida. Si el usuario solicita una fecha específica, el flujo intenta consultar el archivo correspondiente a esa fecha. Si el usuario no indica fecha, el sistema busca la base más reciente disponible.

Ejemplo general de ruta utilizada:

```text
https://www.dane.gov.co/files/operaciones/SIPSA/anex-SIPSADiario-[FECHA].xlsx
```

Donde `[FECHA]` corresponde a la fecha construida dinámicamente por el flujo de n8n.

---

## 7. Mercados configurados

La versión actual del módulo trabaja principalmente con precios de los siguientes mercados mayoristas:

| Región | Plaza mayorista |
|---|---|
| Santander | Centroabastos S.A. |
| Bogotá D.C. | Corabastos S.A. |

Los precios se reportan en pesos colombianos por kilogramo, según la información disponible en la base SIPSA consultada.

---

## 8. Flujo automatizado en n8n

El archivo principal del workflow se encuentra en:

```text
workflows/mercoia_precios_n8n_sin_credenciales.json
```

Este archivo corresponde a una versión exportada del flujo de n8n sin credenciales sensibles.

El workflow incluye nodos para:

- recibir mensajes mediante webhook;
- extraer variables del evento de WhatsApp;
- detectar saludos, despedidas y consultas;
- evitar el reprocesamiento de mensajes enviados por el bot;
- detectar productos ambiguos;
- conservar contexto temporal de una consulta;
- interpretar fechas en lenguaje natural;
- generar URLs candidatas de SIPSA;
- descargar archivos XLSX;
- leer y limpiar la información;
- generar una lista oficial de productos disponibles;
- normalizar el producto consultado mediante un agente de IA;
- extraer precios reales desde la base;
- calcular variaciones aproximadas;
- construir la respuesta final;
- enviar la respuesta al usuario por WhatsApp.

---

## 9. Manejo de mensajes

El sistema clasifica los mensajes entrantes en tres grupos principales:

| Tipo de mensaje | Acción del sistema |
|---|---|
| Saludo sin producto | Envía mensaje de bienvenida y ejemplos de consulta |
| Despedida | Envía mensaje de cierre |
| Consulta de precio | Ejecuta el flujo de búsqueda y respuesta |

Ejemplos de saludos detectados:

```text
hola
buenos días
buenas
hols
hla
ola
```

Ejemplos de despedidas detectadas:

```text
gracias
muchas gracias
hasta luego
chao
eso es todo
```

---

## 10. Manejo de productos ambiguos

Algunos productos pueden tener varias opciones en SIPSA. Por ejemplo:

- papa;
- plátano.

Si el usuario escribe:

```text
precio de la papa de ayer
```

el sistema puede solicitar aclaración:

```text
¿Cuál precio deseas consultar?
1. Papa negra
2. Papa criolla
```

Después de que el usuario aclara la opción, el flujo conserva el contexto de la consulta inicial, incluyendo la fecha solicitada.

---

## 11. Normalización de productos

El módulo utiliza una lista oficial de productos obtenida desde la base SIPSA consultada. Luego, un agente de IA compara el mensaje del usuario con esa lista y selecciona el producto más probable.

Ejemplos de normalización:

| Entrada del usuario | Producto normalizado |
|---|---|
| aullama | Ahuyama |
| auyama | Ahuyama |
| sanahoria | Zanahoria |
| sebolla | Cebolla |
| papa negra | Papa negra |
| papa criolla | Papa criolla |
| platano guineo | Plátano guineo |
| platano harton verde | Plátano hartón verde |

El sistema evita reemplazar silenciosamente una variedad específica no disponible por un producto genérico. Por ejemplo, si el usuario solicita `tomate cherry` y en la base solo aparece `Tomate`, el flujo debe indicar que no se encontró el producto exacto.

---

## 12. Procesamiento de datos

Durante la ejecución del flujo, el sistema extrae y organiza los siguientes campos principales:

| Campo interno | Descripción |
|---|---|
| `nombre` | Nombre del producto reportado por SIPSA |
| `precio_centroabastos` | Precio del producto en Centroabastos S.A. |
| `var_centroabastos` | Variación reportada para Centroabastos S.A. |
| `precio_corabastos` | Precio del producto en Corabastos S.A. |
| `var_corabastos` | Variación reportada para Corabastos S.A. |

El sistema no estima precios cuando la información no está disponible. En esos casos, informa al usuario que no se encontró dato para el producto, la plaza o la fecha consultada.

---

## 13. Cálculo de variación

SIPSA reporta variaciones porcentuales. El módulo transforma esa información en una variación aproximada en pesos colombianos para que el usuario reciba una respuesta más comprensible.

La lógica general es:

```text
precio_anterior = precio_actual / (1 + variacion_porcentual / 100)

variacion_pesos = precio_actual - precio_anterior
```

En la respuesta final, el usuario no recibe necesariamente el porcentaje original, sino una frase en lenguaje natural, por ejemplo:

```text
subió 200 COP por kilogramo
```

o:

```text
disminuyó 300 COP por kilogramo
```

---

## 14. Formato de respuesta

La respuesta final enviada al usuario contiene:

- disponibilidad de la información;
- fecha del reporte;
- producto consultado;
- mercado o plaza mayorista;
- región;
- precio en COP por kilogramo;
- variación aproximada;
- pregunta de cierre.

Ejemplo de respuesta:

```text
✅ Información disponible para hoy: Miércoles 18 de junio de 2026.

Informe de precios del banano:

En Santander, según Centroabastos S.A., el precio del banano es de *2.500 COP por kilogramo*.
Variación: subió *125 COP por kilogramo* con respecto al día de mercado anterior.

En Bogotá D.C., según Corabastos S.A., el precio del banano es de *2.388 COP por kilogramo*.
Variación: disminuyó *68 COP por kilogramo* con respecto al día de mercado anterior.

¿Deseas consultar otro precio?
```

---

## 15. Ambiente de ejecución

El módulo fue desarrollado para ejecutarse principalmente en un ambiente basado en n8n.

Componentes requeridos:

- instancia de n8n;
- servidor o ambiente de despliegue;
- acceso a internet para consultar archivos SIPSA;
- configuración de webhook público;
- credenciales de Evolution API o API de mensajería equivalente;
- credenciales de OpenAI;
- workflow importado en n8n;
- permisos de ejecución para nodos HTTP Request, Code y lectura de archivos XLSX.

Durante el desarrollo se utilizó un ambiente desplegado en Hostinger para alojar la instancia de trabajo y ejecutar el flujo automatizado.

---

## 16. Variables y credenciales requeridas

Este repositorio no incluye credenciales reales, tokens, API keys ni información sensible.

Para ejecutar el flujo en un ambiente propio se deben configurar, según corresponda, variables o credenciales asociadas a:

```text
N8N_BASE_URL
N8N_WEBHOOK_URL
OPENAI_API_KEY
EVOLUTION_API_URL
EVOLUTION_API_TOKEN
EVOLUTION_INSTANCE_NAME
META_ACCESS_TOKEN
META_PHONE_NUMBER_ID
META_VERIFY_TOKEN
```

Las variables relacionadas con Meta Cloud API se incluyen como referencia para la migración prevista hacia la API oficial de WhatsApp Business Platform.

---

## 17. Estructura del repositorio

```text
mercoia-modulo-precios/
│
├── README.md
│
├── docs/
│   ├── base_datos.md
│   ├── pipeline_modulo_precios.md
│   └── pruebas_y_validacion.md
│
├── src/
│   ├── feature_extraction/
│   └── pipeline/
│
├── workflows/
│   └── mercoia_precios_n8n_sin_credenciales.json
│
└── .gitignore
```

Descripción general:

| Carpeta o archivo | Contenido |
|---|---|
| `README.md` | Documento principal del repositorio |
| `docs/` | Documentación técnica del módulo |
| `src/` | Fragmentos de referencia sobre la lógica del sistema |
| `workflows/` | Workflow exportado desde n8n sin credenciales |
| `.gitignore` | Archivo de exclusión para evitar subir archivos no deseados |

---

## 18. Importación del workflow en n8n

Para revisar o reutilizar el flujo:

1. Ingresar a la instancia de n8n.
2. Crear un nuevo workflow o usar la opción de importación.
3. Importar el archivo:

```text
workflows/mercoia_precios_n8n_sin_credenciales.json
```

4. Revisar los nodos importados.
5. Configurar credenciales de APIs externas.
6. Actualizar URLs, tokens o nombres de instancia según el ambiente.
7. Activar el webhook correspondiente.
8. Realizar pruebas con mensajes controlados antes de exponer el flujo a usuarios finales.

---

## 19. Pruebas realizadas

Durante el desarrollo se contemplaron pruebas funcionales sobre:

- saludos;
- despedidas;
- consultas simples;
- errores ortográficos;
- productos ambiguos;
- conservación de contexto;
- fechas relativas;
- fechas específicas;
- formato de respuesta;
- variación en pesos;
- disponibilidad de base SIPSA;
- casos de producto no encontrado;
- comportamiento con varios usuarios.

La documentación asociada se encuentra en:

```text
docs/pruebas_y_validacion.md
```

---

## 20. Manejo de errores

El módulo contempla diferentes escenarios de error:

| Escenario | Respuesta esperada |
|---|---|
| No se encuentra la base SIPSA | Se informa que no hay base disponible |
| El producto no existe en la base | Se solicita escribir nuevamente el producto |
| El producto es ambiguo | Se solicita aclaración |
| No hay precio para una plaza | Se informa que no hay información disponible |
| Falla el envío por WhatsApp | Se recomienda revisar estado de la API de mensajería |
| La instancia de Evolution API está desconectada | Se requiere reconexión o cambio de proveedor |

---

## 21. Limitaciones actuales

La versión actual presenta algunas limitaciones identificadas durante el desarrollo:

- dependencia inicial de Evolution API;
- posible desconexión de la instancia de WhatsApp cuando se usa Evolution API;
- necesidad de revisar estabilidad en escenarios con múltiples usuarios;
- dependencia de la disponibilidad diaria de archivos SIPSA;
- ausencia de una base de datos histórica propia;
- necesidad de fortalecer la migración hacia Meta Cloud API;
- necesidad de documentar con mayor detalle el ambiente final de producción.

---

## 22. Próximos pasos

Los principales pasos pendientes son:

PENDIENTE...

---

## 23. Seguridad y manejo de credenciales

Por seguridad, este repositorio no debe contener:

- tokens de Evolution API;
- tokens de Meta Cloud API;
- claves de OpenAI;
- credenciales de n8n;
- URLs privadas de webhooks activos;
- números telefónicos reales de usuarios;
- información sensible del servidor.

Las credenciales deben configurarse directamente en el ambiente de ejecución o mediante variables de entorno.

---

## 24. Contacto

Para solicitar información adicional, coordinar la réplica del código o revisar componentes necesarios para la ejecución del módulo, comunicarse al correo institucional del proyecto:

```text
proyectomercoia@gmail.com
```
<img width="1284" height="816" alt="image" src="https://github.com/user-attachments/assets/5a02a2bc-e2c9-49c4-ac49-851f2ba8ba60" />

