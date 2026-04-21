# proyecto-crudpastillero-21-abril-2026
proyecto en flutter 
¡Hola! Entiendo perfectamente el desafío. Como arquitecto de software, he diseñado esta guía estructurada para llevarte de la mano en la creación de **bdcrudruleta0481**.

Lo interesante aquí es la integración de **Antigravity**. Para fines educativos, trataremos a Antigravity como el **orquestador de lógica y estado**, permitiendo que los estudiantes vean el desarrollo no solo como "escribir código", sino como un sistema de agentes con responsabilidades claras.

---

## 1. Metodología de Trabajo (Agentes y Roles)

Antes de tocar el código, definimos cómo se organiza la inteligencia del proyecto:

* **Agente Estructurador (Arquitecto):** Define la estructura de carpetas y configuración de Firebase.
* **Agente de Datos (DBA):** Se encarga de la comunicación con Firestore (CRUD).
* **Agente de Interfaz (UI Designer):** Gestiona los widgets y la experiencia de usuario.
* **Agente de Flujo (Antigravity Core):** Conecta la UI con los datos mediante "Skills" (habilidades).

---

## 2. Configuración Inicial y Firebase

### Paso 1: Creación de la carpeta y proyecto
Abre tu terminal y ejecuta:
```bash
mkdir crudruleta
cd crudruleta
flutter create bdcrudruleta0481
```

### Paso 2: Configuración en Console Firebase
1.  Ve a [Firebase Console](https://console.firebase.google.com/).
2.  Crea un proyecto llamado `bdcrudruleta0481`.
3.  Registra tu app (Android/iOS/Web).
4.  **Firestore Database:** Haz clic en "Crear base de datos", selecciona "Modo de prueba" (para desarrollo rápido) y elige una ubicación.
5.  Crea una colección llamada `usuarios`.

### Paso 3: Librerías (pubspec.yaml)
Agrega estas dependencias en tu archivo `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.24.2
  cloud_firestore: ^4.14.0
  # Antigravity se integra como el gestor de lógica
  antigravity: ^1.0.0 
```
*Ejecuta `flutter pub get` en la terminal.*

---

## 3. Estructura de Carpetas (Arquitectura Limpia)

Para que el proyecto sea mantenible, utilizaremos la siguiente estructura:

```text
lib/
├── agents/          # Definición de roles (Antigravity)
├── skills/          # Lógica de negocio (CRUD)
├── models/          # Modelo de datos (Usuario)
├── views/           # Pantallas (UI)
└── main.dart        # Punto de entrada
```

---

## 4. Implementación del Código Funcional

### Modelo de Datos (`models/usuario.dart`)
```dart
class Usuario {
  String id;
  String nombre;
  String fechaNacimiento;
  String numeroTelefono;

  Usuario({required this.id, required this.nombre, required this.fechaNacimiento, required this.numeroTelefono});

  Map<String, dynamic> toMap() => {
    "nombre": nombre,
    "fecha_nacimiento": fechaNacimiento,
    "numero_telefono": numeroTelefono,
  };

  factory Usuario.fromSnapshot(String id, Map<String, dynamic> data) => Usuario(
    id: id,
    nombre: data['nombre'] ?? '',
    fechaNacimiento: data['fecha_nacimiento'] ?? '',
    numeroTelefono: data['numero_telefono'] ?? '',
  );
}
```

### Skill de Datos (CRUD) (`skills/user_skills.dart`)
Aquí es donde el Agente de Datos aplica sus habilidades.

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/usuario.dart';

class UserSkills {
  final CollectionReference _db = FirebaseFirestore.instance.collection('usuarios');

  // CREATE
  Future<void> addUsuario(Usuario user) => _db.add(user.toMap());

  // READ
  Stream<List<Usuario>> getUsuarios() {
    return _db.snapshots().map((snapshot) =>
        snapshot.docs.map((doc) => Usuario.fromSnapshot(doc.id, doc.data() as Map<String, dynamic>)).toList());
  }

  // UPDATE
  Future<void> updateUsuario(Usuario user) => _db.doc(user.id).update(user.toMap());

  // DELETE
  Future<void> deleteUsuario(String id) => _db.doc(id).delete();
}
```

### Interfaz de Usuario (`views/home_page.dart`)
Un CRUD funcional con un formulario simple.

```dart
import 'package:flutter/material.dart';
import '../skills/user_skills.dart';
import '../models/usuario.dart';

class HomePage extends StatelessWidget {
  final UserSkills _skills = UserSkills();
  final _nombreCtrl = TextEditingController();
  final _fechaCtrl = TextEditingController();
  final _telCtrl = TextEditingController();

  void _showForm(BuildContext context, [Usuario? user]) {
    if (user != null) {
      _nombreCtrl.text = user.nombre;
      _fechaCtrl.text = user.fechaNacimiento;
      _telCtrl.text = user.numeroTelefono;
    }

    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      builder: (_) => Padding(
        padding: EdgeInsets.only(bottom: MediaQuery.of(context).viewInsets.bottom, left: 15, right: 15, top: 15),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(controller: _nombreCtrl, decoration: InputDecoration(labelText: 'Nombre')),
            TextField(controller: _fechaCtrl, decoration: InputDecoration(labelText: 'Fecha Nacimiento (AAAA-MM-DD)')),
            TextField(controller: _telCtrl, decoration: InputDecoration(labelText: 'Teléfono')),
            ElevatedButton(
              onPressed: () {
                final newUser = Usuario(id: user?.id ?? '', nombre: _nombreCtrl.text, fechaNacimiento: _fechaCtrl.text, numeroTelefono: _telCtrl.text);
                user == null ? _skills.addUsuario(newUser) : _skills.updateUsuario(newUser);
                Navigator.pop(context);
              },
              child: Text(user == null ? 'Crear' : 'Actualizar'),
            )
          ],
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("CRUD Ruleta 0481")),
      body: StreamBuilder<List<Usuario>>(
        stream: _skills.getUsuarios(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return LinearProgressIndicator();
          return ListView.builder(
            itemCount: snapshot.data!.length,
            itemBuilder: (context, i) {
              final u = snapshot.data![i];
              return ListTile(
                title: Text(u.nombre),
                subtitle: Text("${u.numeroTelefono} | ${u.fechaNacimiento}"),
                trailing: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    IconButton(icon: Icon(Icons.edit), onPressed: () => _showForm(context, u)),
                    IconButton(icon: Icon(Icons.delete), onPressed: () => _skills.deleteUsuario(u.id)),
                  ],
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(onPressed: () => _showForm(context), child: Icon(Icons.add)),
    );
  }
}
```

---

## 5. Archivo Principal (`main.dart`)

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'views/home_page.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Inicialización crítica de Firebase
  runApp(MaterialApp(
    debugShowCheckedModeBanner: false,
    home: HomePage(),
    theme: ThemeData(primarySwatch: Colors.indigo),
  ));
}
```

---

### Flujo de Trabajo para Estudiantes:
1.  **Definición de Agentes:** Los estudiantes deben decidir qué "Skill" necesita cada agente (Ej: El Agente de Validación no deja guardar si el teléfono no tiene 10 dígitos).
2.  **Flujo Antigravity:** La UI envía una intención, la Skill de Antigravity procesa los datos en Firestore y la UI reacciona al Stream de datos en tiempo real.
3.  **Práctica:** Intentar agregar un cuarto campo llamado "Correo" siguiendo la misma estructura de Agente -> Skill -> UI.
