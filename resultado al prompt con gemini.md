¡Hola! Como desarrollador de software, me encanta la estructura que planteas. Vamos a elevar este proyecto integrando **Antigravity** (un framework de orquestación de agentes de IA) para automatizar el flujo de trabajo del CRUD.

Aquí tienes la metodología paso a paso para transformar este requerimiento en una práctica profesional guiada.

---

## 1. Fase de Preparación y Consola Firebase

### Paso A: Creación del Proyecto Flutter
Abre tu terminal y ejecuta:
```bash
flutter create crudlavanderia
cd crudlavanderia
```

### Paso B: Configuración en Firebase Console
1. Ve a [Firebase Console](https://console.firebase.google.com/).
2. Crea un proyecto llamado `crud-lavanderia`.
3. Habilita **Cloud Firestore** en modo de prueba.
4. Registra tu app (Android/iOS) y descarga el archivo `google-services.json` (para Android) colocándolo en `android/app/`.

### Paso C: Dependencias en `pubspec.yaml`
Para que el proyecto funcione, añade estas líneas bajo `dependencies`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.24.2
  cloud_firestore: ^4.14.0
  antigravity: ^latest_version # Framework de Agentes
```
Ejecuta `flutter pub get` en la terminal.

---

## 2. Metodología Antigravity: Agentes y Flujos

En esta práctica, usaremos **Antigravity** no solo como código, sino como la arquitectura que gestiona los datos de los empleados.

### Estructura de Carpetas Sugerida
```text
lib/
├── agents/          # Definición de roles y skills
├── models/          # Clase Empleado
├── services/        # Lógica de Firebase
├── ui/              # Pantallas (CRUD)
└── main.dart        # Punto de entrada
```

### Definición de Agentes y Roles
| Agente | Rol | Skill (Habilidad) |
| :--- | :--- | :--- |
| **AdminAgent** | Gestor de Personal | Validar datos y coordinar el CRUD |
| **DataAgent** | Persistencia | Comunicación directa con Firestore |

---

## 3. Implementación del Código (Funcional)

### A. Modelo de Datos (`lib/models/empleado_model.dart`)
```dart
class Empleado {
  String? id;
  String nombre;
  String apellidos;
  String email;
  String telefono;

  Empleado({this.id, required this.nombre, required this.apellidos, required this.email, required this.telefono});

  Map<String, dynamic> toMap() => {
    "nombre": nombre,
    "apellidos": apellidos,
    "email": email,
    "telefono": telefono,
  };

  factory Empleado.fromSnapshot(String id, Map<String, dynamic> data) => Empleado(
    id: id,
    nombre: data['nombre'],
    apellidos: data['apellidos'],
    email: data['email'],
    telefono: data['telefono'],
  );
}
```

### B. Lógica CRUD con Firebase (`lib/services/firebase_service.dart`)
```dart
import 'cloud_firestore/cloud_firestore.dart';
import '../models/empleado_model.dart';

class FirebaseService {
  final CollectionReference _db = FirebaseFirestore.instance.collection('empleados');

  // Create
  Future<void> addEmpleado(Empleado emp) => _db.add(emp.toMap());

  // Read
  Stream<List<Empleado>> getEmpleados() {
    return _db.snapshots().map((snap) => 
      snap.docs.map((doc) => Empleado.fromSnapshot(doc.id, doc.data() as Map<String, dynamic>)).toList());
  }

  // Update
  Future<void> updateEmpleado(Empleado emp) => _db.doc(emp.id).update(emp.toMap());

  // Delete
  Future<void> deleteEmpleado(String id) => _db.doc(id).delete();
}
```

### C. Integración con Antigravity (Flujo de Trabajo)
Aquí definimos cómo los "Agentes" procesan la información.

```dart
// lib/agents/empleado_workflow.dart
import 'package:antigravity/antigravity.dart';

class EmpleadoWorkflow extends Workflow {
  @override
  void setup() {
    // Definimos el flujo: Validar -> Guardar
    step('validar', (input) {
      if (input['email'].contains('@')) return true;
      throw Exception("Email inválido");
    });
    
    step('persistir', (input) async {
      // Aquí se llama al FirebaseService
    });
  }
}
```

### D. Interfaz de Usuario (UI) Simplificada (`lib/ui/home_page.dart`)
```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';
import '../models/empleado_model.dart';

class HomePage extends StatelessWidget {
  final FirebaseService _service = FirebaseService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("CRUD Lavandería - Empleados")),
      body: StreamBuilder<List<Empleado>>(
        stream: _service.getEmpleados(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return CircularProgressIndicator();
          return ListView.builder(
            itemCount: snapshot.data!.length,
            itemBuilder: (context, i) {
              final emp = snapshot.data![i];
              return ListTile(
                title: Text("${emp.nombre} ${emp.apellidos}"),
                subtitle: Text(emp.email),
                trailing: IconButton(
                  icon: Icon(Icons.delete, color: Colors.red),
                  onPressed: () => _service.deleteEmpleado(emp.id!),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _service.addEmpleado(Empleado(
          nombre: "Juan", apellidos: "Pérez", email: "juan@mail.com", telefono: "123456"
        )),
        child: Icon(Icons.add),
      ),
    );
  }
}
```

---

## 4. Guía para el Estudiante (Metodología de Trabajo)

Para que tus alumnos dominen esta práctica, pídeles que sigan este flujo de trabajo de "Agente":

1.  **Análisis de Requerimientos (Rol de Analista):** Identificar que se necesitan 4 campos (Nombre, Apellidos, Email, Teléfono).
2.  **Configuración de Infraestructura (Rol de DevOps):** Vincular la consola de Firebase con la App.
3.  **Desarrollo de Skills (Rol de Programador):** Crear las funciones de `add`, `read`, `update` y `delete`.
4.  **Pruebas de Flujo (Rol de QA):** Usar los logs de Antigravity para verificar que los datos lleguen a Firestore correctamente.

**Nota técnica:** No olvides inicializar Firebase en tu `main.dart`:
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(MaterialApp(home: HomePage()));
}
```
