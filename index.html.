import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:cloud_firestore/cloud_firestore.dart';

// ⚠️ Usá tu firebase_options.dart real
import 'firebase_options.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(const CanchaApp());
}

class CanchaApp extends StatelessWidget {
  const CanchaApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData.dark(),
      home: const AuthGate(),
    );
  }
}

/// =======================
/// AUTH GATE (AUTO LOGIN)
/// =======================
class AuthGate extends StatelessWidget {
  const AuthGate({super.key});

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<User?>(
      stream: FirebaseAuth.instance.authStateChanges(),
      builder: (context, snap) {
        if (snap.connectionState == ConnectionState.waiting) {
          return const Scaffold(
            body: Center(child: CircularProgressIndicator()),
          );
        }
        if (snap.hasData) return const Home();
        return const Login();
      },
    );
  }
}

/// =======================
/// LOGIN / REGISTER
/// =======================
class Login extends StatefulWidget {
  const Login({super.key});
  @override
  State<Login> createState() => _LoginState();
}

class _LoginState extends State<Login> {
  final email = TextEditingController();
  final pass = TextEditingController();

  Future<void> login() async {
    await FirebaseAuth.instance.signInWithEmailAndPassword(
      email: email.text,
      password: pass.text,
    );
  }

  Future<void> register() async {
    final cred = await FirebaseAuth.instance
        .createUserWithEmailAndPassword(
      email: email.text,
      password: pass.text,
    );

    await FirebaseFirestore.instance
        .collection('users')
        .doc(cred.user!.uid)
        .set({
      'uid': cred.user!.uid,
      'email': email.text,
      'username': null,
      'photoUrl': null,
      'followers': [],
      'following': [],
      'isPrivate': false,
      'verified': false,
      'createdAt': Timestamp.now(),
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: SizedBox(
          width: 320,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              TextField(controller: email),
              TextField(controller: pass, obscureText: true),
              const SizedBox(height: 12),
              ElevatedButton(onPressed: login, child: const Text('Entrar')),
              TextButton(
                  onPressed: register,
                  child: const Text('Crear cuenta')),
            ],
          ),
        ),
      ),
    );
  }
}

/// =======================
/// HOME
/// =======================
class Home extends StatefulWidget {
  const Home({super.key});
  @override
  State<Home> createState() => _HomeState();
}

class _HomeState extends State<Home> {
  int index = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: index == 0 ? const Feed() : const Profile(),
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: index,
        onTap: (i) => setState(() => index = i),
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: ''),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: ''),
        ],
      ),
    );
  }
}

/// =======================
/// FEED REAL (VACÍO SI NO HAY POSTS)
/// =======================
class Feed extends StatelessWidget {
  const Feed({super.key});

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<QuerySnapshot>(
      stream: FirebaseFirestore.instance
          .collection('posts')
          .orderBy('createdAt', descending: true)
          .snapshots(),
      builder: (context, snap) {
        if (!snap.hasData) {
          return const Center(child: CircularProgressIndicator());
        }
        if (snap.data!.docs.isEmpty) {
          return const Center(
            child: Text(
              'No hay publicaciones',
              style: TextStyle(color: Colors.grey),
            ),
          );
        }
        return ListView(
          children: snap.data!.docs.map((d) {
            return Card(
              color: Colors.black,
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  ListTile(
                    title: Text(
                      d['username'] ?? '',
                      style: const TextStyle(color: Colors.white),
                    ),
                  ),
                  if (d['imageUrl'] != null)
                    Image.network(d['imageUrl']),
                  if (d['caption'] != null)
                    Padding(
                      padding: const EdgeInsets.all(8),
                      child: Text(
                        d['caption'],
                        style: const TextStyle(color: Colors.white),
                      ),
                    ),
                ],
              ),
            );
          }).toList(),
        );
      },
    );
  }
}

/// =======================
/// PROFILE REAL
/// =======================
class Profile extends StatelessWidget {
  const Profile({super.key});

  @override
  Widget build(BuildContext context) {
    final uid = FirebaseAuth.instance.currentUser!.uid;

    return StreamBuilder<DocumentSnapshot>(
      stream: FirebaseFirestore.instance
          .collection('users')
          .doc(uid)
          .snapshots(),
      builder: (context, snap) {
        if (!snap.hasData) {
          return const Center(child: CircularProgressIndicator());
        }

        final u = snap.data!;
        final followers = (u['followers'] as List).length;

        return Padding(
          padding: const EdgeInsets.all(20),
          child: Column(
            children: [
              if (u['photoUrl'] != null)
                CircleAvatar(
                  radius: 40,
                  backgroundImage: NetworkImage(u['photoUrl']),
                ),
              const SizedBox(height: 12),
              Text(
                u['username'] ?? '',
                style: const TextStyle(fontSize: 18),
              ),
              if (u['verified'] == true)
                const Icon(Icons.verified, color: Colors.blue),
              const SizedBox(height: 8),
              Text(
                '$followers seguidores',
                style: const TextStyle(color: Colors.grey),
              ),
              const Spacer(),
              ElevatedButton(
                onPressed: () => FirebaseAuth.instance.signOut(),
                child: const Text('Cerrar sesión'),
              )
            ],
          ),
        );
      },
    );
  }
}
