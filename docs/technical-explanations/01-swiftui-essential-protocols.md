# SwiftUI Essential Protocols: Identifiable, Hashable & Codable

## 🎯 Overview

En el ecosistema de SwiftUI y Clean Architecture, hay tres protocolos que forman la foundation de entities bien diseñadas: `Identifiable`, `Hashable`, y `Codable`. Estos no son simplemente "requirements" de SwiftUI - son decisiones arquitectónicas fundamentales que determinan el performance, maintainability, y scalability de tu aplicación.

**Analogía del Documento de Identidad**: Imagina que tus data entities son ciudadanos en un país digital. Cada ciudadano necesita:
- **Identifiable**: Una cédula de identidad única (para que el gobierno los reconozca)
- **Hashable**: Un código de barras que permite búsqueda instantánea en bases de datos
- **Codable**: La habilidad de traducirse a diferentes idiomas (JSON, XML, etc.)

---

## 🧠 Deep Dive: Identifiable Protocol

### ¿Qué Problema Resuelve?

SwiftUI usa un algoritmo de "diffing" para determinar qué views necesitan actualizarse cuando los datos cambian. Sin identifiers estables, SwiftUI no puede distinguir entre "este item fue editado" vs "este item fue eliminado y uno nuevo fue agregado".

### Analogía: La Lista de Estudiantes

Imagina que eres un profesor con una lista de estudiantes:

```swift
// ❌ Problema: Sin IDs estables
class Classroom {
    var students = ["Ana", "Bob", "Carlos"]
    
    func moveStudent() {
        students.swapAt(0, 1) // Ana y Bob intercambian posiciones
        // ¿Cómo sabe SwiftUI que Ana se movió y no que fue reemplazada?
    }
}
```

SwiftUI ve esto como:
- Posición 0: "Ana" → "Bob" (CAMBIÓ - recrear view)
- Posición 1: "Bob" → "Ana" (CAMBIÓ - recrear view)

```swift
// ✅ Solución: Con IDs estables
struct Student: Identifiable {
    let id = UUID()
    let name: String
}

class Classroom {
    var students = [
        Student(name: "Ana"),
        Student(name: "Bob"), 
        Student(name: "Carlos")
    ]
    
    func moveStudent() {
        students.swapAt(0, 1)
        // SwiftUI entiende: "Ana ID-123 se movió a posición 1"
        // Resultado: Smooth animation, no recreation
    }
}
```

### Cuándo Usar Automatic vs Manual IDs

#### Automatic UUID (Datos Locales)
```swift
struct LocalNote: Identifiable {
    let id = UUID() // SwiftUI maneja esto automáticamente
    var content: String
}
```
**Ventajas**: Simple, no conflictos
**Desventajas**: No persiste across app launches
**Usar cuando**: Data que nunca dejará el device

#### Manual ID (Datos Sincronizados)
```swift
struct ServerNote: Identifiable {
    let id: String // Server-provided ID
    var content: String
}
```
**Ventajas**: Persiste, sync-friendly, no duplicates
**Desventajas**: Requiere más setup, possible conflicts
**Usar cuando**: Data que se syncroniza con server/cloud

---

## 🧠 Deep Dive: Hashable Protocol

### ¿Qué Problema Resuelve?

Swift usa hash tables para operaciones O(1) en collections. Sin Hashable proper, las operaciones se vuelven O(n) - incredibly slow en large datasets.

### Analogía: La Biblioteca vs El Diccionario

**Sin Hashable (Biblioteca desorganizada)**:
```swift
// Para encontrar un libro, tienes que revisar cada estante
func findBook(title: String, in books: [Book]) -> Book? {
    for book in books { // O(n) - slow!
        if book.title == title {
            return book
        }
    }
    return nil
}
```

**Con Hashable (Diccionario con índice)**:
```swift
struct Book: Hashable {
    let isbn: String // Unique identifier
    let title: String
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(isbn) // Solo hash immutable properties
    }
    
    static func == (lhs: Book, rhs: Book) -> Bool {
        lhs.isbn == rhs.isbn
    }
}

// Swift puede crear un hash table
var library: Set<Book> = [] // O(1) lookups!
```

### Implementation Guidelines

#### ✅ Correcto: Hash Solo Properties Immutable
```swift
struct User: Hashable {
    let id: UUID              // Immutable - safe to hash
    var name: String          // Mutable - DON'T hash
    var isLoggedIn: Bool      // Mutable - DON'T hash
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(id) // Solo el immutable ID
    }
    
    static func == (lhs: User, rhs: User) -> Bool {
        lhs.id == rhs.id // Comparison también usa solo ID
    }
}
```

#### ❌ Incorrecto: Hash Mutable Properties
```swift
struct BadUser: Hashable {
    let id: UUID
    var name: String
    var isLoggedIn: Bool
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
        hasher.combine(name)        // ❌ Mutable property
        hasher.combine(isLoggedIn)  // ❌ Mutable property
    }
}

// Problema: Si name cambia después de añadir a Set,
// el object se "pierde" en el hash table
var users: Set<BadUser> = []
var user = BadUser(id: UUID(), name: "Ana", isLoggedIn: false)
users.insert(user)

user.name = "Ana María" // Hash cambió!
print(users.contains(user)) // false - perdido en hash table!
```

### Performance Impact

```swift
// Sin Hashable proper: O(n) para cada operación
let slowUsers: [User] = loadUsers()
func updateUser(id: UUID) {
    // Linear search - gets slower with more users
    for i in 0..<slowUsers.count {
        if slowUsers[i].id == id {
            // Found after searching all items
        }
    }
}

// Con Hashable proper: O(1) para operaciones
let fastUsers: Set<User> = Set(loadUsers())
func updateUser(id: UUID) {
    // Hash table lookup - constant time regardless of size
    if let user = fastUsers.first(where: { $0.id == id }) {
        // Found instantly
    }
}
```

---

## 🧠 Deep Dive: Codable Protocol

### ¿Qué Problema Resuelve?

En Clean Architecture, las entities necesitan cruzar boundaries entre layers. Codable permite que los objects se serializen/deserializen automatically, enabling clean data flow.

### Analogía: El Traductor Universal

Imagina que tus data entities son diplomáticos que necesitan comunicarse en diferentes contexts:
- **Network layer**: Hablan JSON
- **Database layer**: Hablan SQL/Core Data
- **UI layer**: Hablan SwiftUI

```swift
struct Diplomat: Codable {
    let id: String
    let name: String
    let country: String
}

// El "traductor" (Codable) permite communication:
let jsonData = try JSONEncoder().encode(diplomat) // Swift → JSON
let diplomat2 = try JSONDecoder().decode(Diplomat.self, from: jsonData) // JSON → Swift
```

### Clean Architecture Benefits

#### Repository Pattern Flexibility
```swift
// Domain layer - Protocol abstraction
protocol TopicRepository {
    func fetchTopics() async throws -> [Topic]
    func saveTopic(_ topic: Topic) async throws
}

// Data layer - Multiple implementations, same interface
class NetworkRepository: TopicRepository {
    func fetchTopics() async throws -> [Topic] {
        let data = try await api.get("/topics")
        return try JSONDecoder().decode([Topic].self, from: data) // Codable!
    }
}

class LocalRepository: TopicRepository {
    func fetchTopics() async throws -> [Topic] {
        let data = try Data(contentsOf: fileURL)
        return try JSONDecoder().decode([Topic].self, from: data) // Same code!
    }
}

class MockRepository: TopicRepository {
    func fetchTopics() async throws -> [Topic] {
        return mockTopics // No serialization needed
    }
}
```

### Advanced Codable Techniques

#### Custom Property Names
```swift
struct Topic: Codable {
    let id: String
    let displayName: String
    let isActive: Bool
    
    enum CodingKeys: String, CodingKey {
        case id
        case displayName = "display_name"  // Maps to snake_case API
        case isActive = "is_active"
    }
}
```

#### Computed Properties Exclusion
```swift
struct Topic: Codable {
    let id: String
    let stories: [Story]
    
    // Computed property - automatically excluded from Codable
    var progress: Double {
        guard !stories.isEmpty else { return 0 }
        let completed = stories.filter(\.isCompleted).count
        return Double(completed) / Double(stories.count)
    }
    
    // Only id and stories are encoded/decoded
}
```

#### Custom Encoding/Decoding
```swift
struct Topic: Codable {
    let id: String
    let createdAt: Date
    
    enum CodingKeys: String, CodingKey {
        case id
        case createdAt = "created_at"
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        id = try container.decode(String.self, forKey: .id)
        
        // Custom date parsing
        let dateString = try container.decode(String.self, forKey: .createdAt)
        let formatter = ISO8601DateFormatter()
        guard let date = formatter.date(from: dateString) else {
            throw DecodingError.dataCorrupted(
                DecodingError.Context(codingPath: decoder.codingPath, 
                                    debugDescription: "Invalid date format")
            )
        }
        createdAt = date
    }
}
```

---

## ⚖️ Trade-offs y Decisiones

### Cuándo Usar Cada Protocol

#### Identifiable
**✅ Usar cuando**:
- Displaying data en SwiftUI Lists/ForEach
- Necesitas smooth animations
- Working con large datasets que cambian frequently

**❌ No usar cuando**:
- Simple value types que no se muestran en UI
- Data que nunca cambia o se actualiza
- Performance no es crítico y simplicity es preferida

#### Hashable  
**✅ Usar cuando**:
- Storing en Sets o usando como Dictionary keys
- Performance de lookups es crítico
- Working con large collections

**❌ No usar cuando**:
- Entity tiene muchas mutable properties
- Equality logic es compleja o cambia frequently
- Simple arrays son sufficient para tu use case

#### Codable
**✅ Usar cuando**:
- Data needs to be serialized (JSON, files, network)
- Building repository pattern con multiple data sources
- API integration es planned o current

**❌ No usar cuando**:
- Data nunca leaves memory (UI-only models)
- Complex object graphs con cycles
- Custom serialization requirements muy específicos

### Performance Considerations

#### Memory Impact
```swift
// Efficient: Minimal memory overhead
struct LightweightTopic: Identifiable, Hashable, Codable {
    let id: String
    let name: String
}

// Heavy: Unnecessary complexity
struct HeavyTopic: Identifiable, Hashable, Codable {
    let id: String
    let name: String
    let metadata: [String: Any] // Not Codable!
    let computedValue: Double   // Expensive computation
    
    // Overcomplicated conformances...
}
```

#### Network Impact
```swift
// Efficient JSON structure
{
  "id": "topic-1",
  "name": "SwiftUI Basics",
  "story_count": 5
}

// Inefficient: Over-detailed serialization
{
  "id": "topic-1", 
  "name": "SwiftUI Basics",
  "created_at": "2025-08-29T10:00:00Z",
  "updated_at": "2025-08-29T11:30:00Z",
  "version": 1,
  "metadata": {
    "created_by": "user-123",
    "tags": ["beginner", "ui", "swift"],
    "difficulty": "easy"
  }
  // ... 20+ more fields
}
```

---

## 🚀 Real-world Applications en InstaLearning

### Entities Design
```swift
// Perfect balance of protocols for learning app
struct Topic: Identifiable, Hashable, Codable {
    let id: String              // Server-provided, stable
    var name: String            // User-editable
    var description: String     // Rich content
    private var _stories: [Story] // Backing storage
    
    // Computed property for UI
    var storyCount: Int {
        _stories.count
    }
    
    var progress: Double {
        guard !_stories.isEmpty else { return 0 }
        let completed = _stories.filter(\.isCompleted).count
        return Double(completed) / Double(_stories.count)
    }
    
    // MARK: - Hashable (Performance)
    func hash(into hasher: inout Hasher) {
        hasher.combine(id) // Only immutable ID
    }
    
    static func == (lhs: Topic, rhs: Topic) -> Bool {
        lhs.id == rhs.id
    }
    
    // MARK: - Codable (Architecture)
    enum CodingKeys: String, CodingKey {
        case id, name, description
        case _stories = "stories"
        // progress y storyCount excluded (computed)
    }
}

struct Story: Identifiable, Hashable, Codable {
    let id: String
    let topicId: String
    var title: String
    var content: String
    var order: Int
    var isCompleted: Bool
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
    }
    
    static func == (lhs: Story, rhs: Story) -> Bool {
        lhs.id == rhs.id
    }
}
```

### SwiftUI Integration
```swift
struct TopicsListView: View {
    @State private var topics: [Topic] = []
    
    var body: some View {
        NavigationView {
            List(topics) { topic in // Identifiable enables this
                TopicRow(topic: topic)
                    .onTapGesture {
                        updateTopicProgress(topic.id) // Hashable enables O(1) lookup
                    }
            }
            .task {
                await loadTopics() // Codable enables clean serialization
            }
        }
    }
}
```

---

## 📚 Related Concepts

- **[Clean Architecture en iOS](./02-clean-architecture-ios.md)**: Cómo estos protocols fit en layered architecture
- **[SwiftUI Performance](./03-swiftui-performance.md)**: Deep dive en optimization techniques
- **[Repository Pattern](./04-repository-pattern.md)**: Using Codable para data layer abstraction
- **[Advanced Swift Types](./05-advanced-swift-types.md)**: Beyond basic protocol conformance

---

## 💡 Key Takeaways

1. **Identifiable** = SwiftUI performance y smooth animations
2. **Hashable** = O(1) collection operations (hash only immutable properties!)
3. **Codable** = Clean architecture boundary crossing
4. **Together** = Scalable, maintainable, performant iOS apps

Estos protocols no son afterthoughts - son architectural foundations que enable professional-grade iOS development. Master them, y tu code will scale de prototype a production seamlessly.

---

**📝 Document Created**: August 29, 2025  
**📝 Last Updated**: August 29, 2025  
**📝 Author**: Senior iOS Mentor + Rodrigo Torres  
**📝 Level**: Mid → Senior Transition