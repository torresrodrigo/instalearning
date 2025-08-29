## 🎯 Objetivo  
Crear una aplicación en **SwiftUI (Swift 6)** que simule la experiencia de Instagram pero enfocada en **aprendizaje**.  
El flujo debe permitir estudiar temas a través de historias y luego reforzar el aprendizaje con **juegos de preguntas (quizzes)** generados a partir de esas historias.  

---

## 🛠 Tecnologías  
- Lenguaje: **Swift 6**  
- Framework: **SwiftUI**  
- Arquitectura: **MVP + Clean Architecture** (extensible a MVVM si se requiere)  
- Async/Await y Concurrency moderna  
- Preparado para integración con APIs externas (ej: Notion API)  
- Mock Data al inicio  

---

## 🚀 Features principales  

### 1. Pantalla **Home**  
- Barra superior con una **colección horizontal** de *temas* (similar a historias de Instagram).  
- Cada **tema** contiene varias *historias* con información relevante a estudiar.  
- En el centro, tarjetas con **métricas del usuario** sobre el estudio (ej: progreso, tiempo invertido, temas completados).  
  - ⚠️ Placeholder para métricas, escalable a futuro.  

### 2. Pantalla de **Temas**  
- Lista con todos los temas disponibles.  
- Cada tema muestra su **nombre, descripción y número de historias**.  
- Opción para crear/cargar un nuevo tema.  

### 3. Pantalla de **Historias de un Tema**  
- Experiencia tipo Instagram Stories:  
  - Cada historia se muestra a pantalla completa.  
  - Puede contener texto, imágenes o código de ejemplo.  
  - Navegación con swipe izquierda/derecha para avanzar/retroceder entre historias.  

### 4. **Juego de Preguntas (Quiz)**  
- Cada tema tendrá un conjunto de **preguntas** generadas en base a sus historias.  
- Tipos de preguntas:  
  - **Opción múltiple** (una correcta entre varias).  
  - **Verdadero/Falso**.  
  - **Completar código sencillo** (ej: “¿Qué falta en este snippet?”).  
- Pantalla con UI estilo **Airbnb / Duolingo**:  
  - Animaciones fluidas al seleccionar respuesta.  
  - Feedback inmediato: ✔️ Correcto o ❌ Incorrecto.  
- Al finalizar el quiz:  
  - Mostrar puntaje.  
  - Opción de repetir el quiz o volver al Home.  

### 5. Carga de Temas  
- Fase inicial: **datos mockeados en el proyecto** (incluyendo historias y preguntas).  
- Fase futura: **integración con Notion API** para cargar dinámicamente temas, historias y quizzes.  
- Preparar capa de repositorio para migrar de mock a API.  

---

## 🎨 Estilo visual y animaciones  

- Inspirado en **Airbnb y Duolingo**:  
  - **Interfaz amigable, minimalista, colorida y con microinteracciones**.  
  - Paleta clara y moderna.  
  - Cards y botones con **bordes redondeados y sombras suaves**.  
- Animaciones estilo Airbnb:  
  - Transiciones con **fade in/out + scale**.  
  - Efectos con **matchedGeometryEffect** al navegar de lista de temas a detalle.  
  - Swipe suave para pasar historias.  
- Animaciones estilo Duolingo para el quiz:  
  - Botones que hacen “bounce” al elegir respuesta.  
  - Checkmark ✅ animado si aciertas, shake ❌ si fallas.  

