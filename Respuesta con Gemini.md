¡Hola! Como desarrollador de software, me encanta este desafío. Vamos a estructurar un proyecto **CRUD (Create, Read, Update, Delete)** de Tours utilizando **Flutter** y **Firebase**, pero con un giro moderno: utilizaremos la metodología de **Antigravity** para orquestar el desarrollo mediante agentes y roles.

---

## 1. Metodología Antigravity: Agentes y Flujo de Trabajo

Para que tus estudiantes comprendan cómo se construye software de nivel profesional, dividiremos el trabajo en **Agentes Inteligentes**. Cada agente tiene un rol, una skill (habilidad) y un entregable.

### Estructura de Agentes
| Agente | Rol | Skill | Tarea Principal |
| :--- | :--- | :--- | :--- |
| **Architect** | Definir Estructura | Diseño de Folder Layering | Crear `pubspec.yaml` y carpetas. |
| **Cloud Eng.** | Firebase Specialist | NoSQL Design | Configurar Firestore y reglas de seguridad. |
| **Dev Flutter** | Programmer | Dart/Flutter Logic | Implementar UI y lógica CRUD. |
| **QA Agent** | Tester | Debugging | Validar que el flujo funcional sea correcto. |

### Flujo de Trabajo (Workflow)
1.  **Orquestación:** El Architect define la estructura de carpetas.
2.  **Infraestructura:** El Cloud Eng. conecta Firebase.
3.  **Implementación:** El Dev construye las vistas y servicios.
4.  **Despliegue:** Se verifica la persistencia en la consola de Firebase.

---

## 2. Configuración de Infraestructura (Firebase & Pubspec)

### Paso 1: Consola de Firebase
1.  Ve a [Firebase Console](https://console.firebase.google.com/).
2.  Crea un proyecto llamado `CRUD Tours`.
3.  Habilita **Cloud Firestore** en modo de prueba (test mode).
4.  Crea una colección llamada `tours`.

### Paso 2: Librerías (pubspec.yaml)
El **Architect Agent** integra las dependencias necesarias:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0
  cloud_firestore: ^5.0.0
```

---

## 3. Estructura de Carpetas del Proyecto
Siguiendo la metodología, organizamos el código por **responsabilidades**:

```text
crud_tours/
├── lib/
│   ├── main.dart             # Punto de entrada
│   ├── models/
│   │   └── tour_model.dart    # Estructura del dato
│   ├── services/
│   │   └── firebase_service.dart # Lógica CRUD
│   └── screens/
│       ├── home_screen.dart   # Lista de Tours (Read/Delete)
│       └── add_tour_screen.dart # Formulario (Create/Update)
```

---

## 4. Implementación del Código Funcional

### A. El Modelo (tour_model.dart)
Define los tres campos solicitados: destino, precio y estrellas.

```dart
class Tour {
  String id;
  String destino;
  double precio;
  int estrellas;

  Tour({required this.id, required this.destino, required this.precio, required this.estrellas});

  // Convertir de Firestore a Objeto
  factory Tour.fromMap(Map<String, dynamic> data, String id) {
    return Tour(
      id: id,
      destino: data['destino'] ?? '',
      precio: (data['precio'] ?? 0).toDouble(),
      estrellas: data['estrellas'] ?? 0,
    );
  }

  // Convertir de Objeto a Map para Firestore
  Map<String, dynamic> toMap() {
    return {
      'destino': destino,
      'precio': precio,
      'estrellas': estrellas,
    };
  }
}
```

### B. El Servicio CRUD (firebase_service.dart)
Aquí es donde ocurre la magia de Firebase.

```dart
import 'cloud_firestore/cloud_firestore.dart';
import '../models/tour_model.dart';

class FirebaseService {
  final CollectionReference tourCollection = FirebaseFirestore.instance.collection('tours');

  // CREATE
  Future<void> addTour(Tour tour) async {
    await tourCollection.add(tour.toMap());
  }

  // READ (Stream para cambios en tiempo real)
  Stream<List<Tour>> getTours() {
    return tourCollection.snapshots().map((snapshot) {
      return snapshot.docs.map((doc) => Tour.fromMap(doc.data() as Map<String, dynamic>, doc.id)).toList();
    });
  }

  // UPDATE
  Future<void> updateTour(Tour tour) async {
    await tourCollection.doc(tour.id).update(tour.toMap());
  }

  // DELETE
  Future<void> deleteTour(String id) async {
    await tourCollection.doc(id).delete();
  }
}
```

### C. La Interfaz de Usuario (home_screen.dart)


```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';
import '../models/tour_model.dart';

class HomeScreen extends StatelessWidget {
  final FirebaseService _service = FirebaseService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("CRUD Tours Antigravity")),
      body: StreamBuilder<List<Tour>>(
        stream: _service.getTours(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          final tours = snapshot.data!;
          return ListView.builder(
            itemCount: tours.length,
            itemBuilder: (context, index) {
              return ListTile(
                title: Text(tours[index].destino),
                subtitle: Text("\$${tours[index].precio} - ${tours[index].estrellas} ⭐"),
                trailing: IconButton(
                  icon: Icon(Icons.delete, color: Colors.red),
                  onPressed: () => _service.deleteTour(tours[index].id),
                ),
                onTap: () => _showForm(context, tours[index]), // Para actualizar
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () => _showForm(context, null), // Para crear nuevo
      ),
    );
  }

  void _showForm(BuildContext context, Tour? tour) {
    // Aquí se abriría un diálogo o pantalla con los controladores para:
    // Destino (TextField), Precio (NumberField), Estrellas (Slider o Input)
    // Al guardar, se llama a _service.addTour o _service.updateTour
  }
}
```

---

## 5. Guía para Estudiantes (Metodología Práctica)

Para que tus alumnos dominen **Antigravity**, pídeles que sigan este orden mental:

1.  **Estado "Zero":** Crear la carpeta `CRUD Tours` y ejecutar `flutter create .`.
2.  **Fase Agente Cloud:** Entrar a la consola de Firebase, crear la DB y descargar el `google-services.json`. **Sin esto, la app no tiene "cerebro" de datos.**
3.  **Fase Agente Architect:** Configurar el `main.dart` para inicializar Firebase:
    ```dart
    void main() async {
      WidgetsFlutterBinding.ensureInitialized();
      await Firebase.initializeApp();
      runApp(MyApp());
    }
    ```
4.  **Ciclo de Skills:** Implementar primero el modelo (¿qué datos tengo?), luego el servicio (¿cómo los muevo?) y finalmente la UI (¿cómo los veo?).

> **Nota de Software Creator:** Asegúrate de que los estudiantes entiendan que el `id` en Firestore es automático. No necesitan crearlo ellos, pero es vital para poder borrar o editar el documento específico después.

¿Te gustaría que desarrolle el código completo del formulario de entrada (`add_tour_screen.dart`) para que los alumnos solo tengan que copiar y pegar?
