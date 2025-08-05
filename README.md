import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(MellowaApp());
}

class MellowaApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Mellowa 💜',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(fontFamily: 'Poppins'),
      home: SplashScreen(),
    );
  }
}

class SplashScreen extends StatefulWidget {
  @override
  State<SplashScreen> createState() => _SplashScreenState();
}

class _SplashScreenState extends State<SplashScreen> with TickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller =
        AnimationController(vsync: this, duration: Duration(seconds: 2));
    _animation = CurvedAnimation(parent: _controller, curve: Curves.easeInOut);
    _controller.forward();
    _navigateNext();
  }

  _navigateNext() async {
    await Future.delayed(Duration(seconds: 3));
    final prefs = await SharedPreferences.getInstance();
    final userType = prefs.getString('userType');
    if (userType == null) {
      Navigator.pushReplacement(context,
          MaterialPageRoute(builder: (_) => UserTypeSelector()));
    } else {
      Navigator.pushReplacement(
          context, MaterialPageRoute(builder: (_) => MainHome(userType)));
    }
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.deepPurple.shade100,
      body: Center(
        child: ScaleTransition(
          scale: _animation,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.flutter_dash, size: 100, color: Colors.deepPurple),
              SizedBox(height: 20),
              Text("Mellowa 💜",
                  style: TextStyle(
                      fontSize: 28,
                      fontWeight: FontWeight.bold,
                      color: Colors.deepPurple)),
              Text("A peaceful world of moments",
                  style: TextStyle(color: Colors.purple.shade700)),
            ],
          ),
        ),
      ),
    );
  }
}

class UserTypeSelector extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.deepPurple.shade50,
      body: Center(
        child: Padding(
          padding: const EdgeInsets.symmetric(horizontal: 32.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text("Who are you?",
                  style: TextStyle(
                      fontSize: 26,
                      fontWeight: FontWeight.w600,
                      color: Colors.deepPurple)),
              SizedBox(height: 40),
              ElevatedButton.icon(
                style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.deepPurple,
                    minimumSize: Size(double.infinity, 50)),
                icon: Icon(Icons.star),
                label: Text("Creator"),
                onPressed: () async {
                  final prefs = await SharedPreferences.getInstance();
                  await prefs.setString('userType', 'creator');
                  Navigator.pushReplacement(context,
                      MaterialPageRoute(builder: (_) => MainHome('creator')));
                },
              ),
              SizedBox(height: 20),
              ElevatedButton.icon(
                style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.purpleAccent,
                    minimumSize: Size(double.infinity, 50)),
                icon: Icon(Icons.person),
                label: Text("User"),
                onPressed: () async {
                  final prefs = await SharedPreferences.getInstance();
                  await prefs.setString('userType', 'user');
                  Navigator.pushReplacement(context,
                      MaterialPageRoute(builder: (_) => MainHome('user')));
                },
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class MainHome extends StatelessWidget {
  final String userType;
  MainHome(this.userType);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Text(
          'Welcome $userType to Mellowa 💜\nNext: Login Page →',
          style: TextStyle(fontSize: 22, color: Colors.deepPurple),
          textAlign: TextAlign.center,
        ),
      ),
    );
  }
}
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:google_sign_in/google_sign_in.dart';

class LoginPage extends StatefulWidget {
  final void Function(String userType) onLoginComplete;
  const LoginPage({required this.onLoginComplete});

  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  bool isLogin = true;
  String email = '';
  String password = '';
  String userType = 'user'; // 'user' or 'creator'

  Future<void> signInWithGoogle() async {
    final GoogleSignInAccount? googleUser = await GoogleSignIn().signIn();
    if (googleUser == null) return;
    final googleAuth = await googleUser.authentication;
    final credential = GoogleAuthProvider.credential(
      accessToken: googleAuth.accessToken, idToken: googleAuth.idToken,
    );
    await FirebaseAuth.instance.signInWithCredential(credential);
    widget.onLoginComplete(userType);
  }

  Future<void> signInWithEmail() async {
    try {
      if (isLogin) {
        await FirebaseAuth.instance.signInWithEmailAndPassword(email: email, password: password);
      } else {
        await FirebaseAuth.instance.createUserWithEmailAndPassword(email: email, password: password);
      }
      widget.onLoginComplete(userType);
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(e.toString())));
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.deepPurple[50],
      body: Center(
        child: SingleChildScrollView(
          child: Padding(
            padding: const EdgeInsets.all(20),
            child: Card(
              shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(20)),
              elevation: 8,
              child: Padding(
                padding: const EdgeInsets.all(24),
                child: Column(
                  children: [
                    Text("Mellowa 💜", style: TextStyle(fontSize: 32, fontWeight: FontWeight.bold, color: Colors.deepPurple)),
                    SizedBox(height: 16),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        ChoiceChip(
                          label: Text('User'),
                          selected: userType == 'user',
                          onSelected: (_) => setState(() => userType = 'user'),
                        ),
                        SizedBox(width: 10),
                        ChoiceChip(
                          label: Text('Creator'),
                          selected: userType == 'creator',
                          onSelected: (_) => setState(() => userType = 'creator'),
                        ),
                      ],
                    ),
                    SizedBox(height: 20),
                    TextField(
                      decoration: InputDecoration(labelText: 'Email'),
                      onChanged: (val) => email = val,
                    ),
                    SizedBox(height: 10),
                    TextField(
                      decoration: InputDecoration(labelText: 'Password'),
                      obscureText: true,
                      onChanged: (val) => password = val,
                    ),
                    SizedBox(height: 20),
                    ElevatedButton(
                      onPressed: signInWithEmail,
                      child: Text(isLogin ? 'Login' : 'Signup'),
                      style: ElevatedButton.styleFrom(
                        backgroundColor: Colors.deepPurple,
                        shape: StadiumBorder(),
                        padding: EdgeInsets.symmetric(horizontal: 40, vertical: 14),
                      ),
                    ),
                    TextButton(
                      onPressed: () => setState(() => isLogin = !isLogin),
                      child: Text(isLogin ? "Don't have an account? Signup" : "Already have an account? Login"),
                    ),
                    Divider(height: 30),
                    Text("or", style: TextStyle(color: Colors.grey)),
                    SizedBox(height: 10),
                    ElevatedButton.icon(
                      onPressed: signInWithGoogle,
                      icon: Icon(Icons.account_circle),
                      label: Text("Continue with Google"),
                      style: ElevatedButton.styleFrom(backgroundColor: Colors.purple[200]),
                    ),
                  ],
                ),
              ),
            ),
          ),
        ),
      ),
    );
  }
}

import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:firebase_auth/firebase_auth.dart';

class ChatsPage extends StatelessWidget {
  final String currentUser = FirebaseAuth.instance.currentUser!.uid;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.deepPurple.shade50,
      appBar: AppBar(
        backgroundColor: Colors.deepPurple,
        title: Text("Chats 💬"),
        actions: [
          IconButton(icon: Icon(Icons.phone), onPressed: () {
            Navigator.push(context, MaterialPageRoute(builder: (_) => CallsPage()));
          }),
        ],
      ),
      body: StreamBuilder(
        stream: FirebaseFirestore.instance
            .collection('chats')
            .where('users', arrayContains: currentUser)
            .snapshots(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return CircularProgressIndicator();
          final chats = snapshot.data!.docs;
          return ListView.builder(
            itemCount: chats.length,
            itemBuilder: (context, index) {
              final chat = chats[index];
              return ListTile(
                title: Text(chat['name'] ?? 'User'),
                subtitle: Text(chat['lastMessage'] ?? ''),
                trailing: Icon(Icons.lock_outline, color: Colors.grey),
                onTap: () {
                  Navigator.push(context, MaterialPageRoute(
                      builder: (_) => ChatDetailPage(chatId: chat.id)));
                },
              );
            },
          );
        },
      ),
    );
  }
}

class ChatDetailPage extends StatefulWidget {
  final String chatId;
  ChatDetailPage({required this.chatId});

  @override
  State<ChatDetailPage> createState() => _ChatDetailPageState();
}

class _ChatDetailPageState extends State<ChatDetailPage> {
  TextEditingController msgController = TextEditingController();

  void sendMessage() {
    if (msgController.text.trim().isEmpty) return;
    FirebaseFirestore.instance
        .collection('chats')
        .doc(widget.chatId)
        .collection('messages')
        .add({
      'text': msgController.text.trim(),
      'sender': FirebaseAuth.instance.currentUser!.uid,
      'timestamp': Timestamp.now(),
    });
    msgController.clear();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.white,
      appBar: AppBar(title: Text("Chat 💜"), backgroundColor: Colors.deepPurple),
      body: Column(
        children: [
          Expanded(
            child: StreamBuilder(
              stream: FirebaseFirestore.instance
                  .collection('chats')
                  .doc(widget.chatId)
                  .collection('messages')
                  .orderBy('timestamp', descending: true)
                  .snapshots(),
              builder: (context, snapshot) {
                if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
                final messages = snapshot.data!.docs;
                return ListView.builder(
                  reverse: true,
                  itemCount: messages.length,
                  itemBuilder: (context, index) {
                    final msg = messages[index];
                    return Align(
                      alignment: msg['sender'] == FirebaseAuth.instance.currentUser!.uid
                          ? Alignment.centerRight
                          : Alignment.centerLeft,
                      child: Container(
                        margin: EdgeInsets.all(6),
                        padding: EdgeInsets.all(10),
                        decoration: BoxDecoration(
                          color: msg['sender'] == FirebaseAuth.instance.currentUser!.uid
                              ? Colors.deepPurple.shade100
                              : Colors.grey.shade300,
                          borderRadius: BorderRadius.circular(12),
                        ),
                        child: Text(msg['text']),
                      ),
                    );
                  },
                );
              },
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Row(
              children: [
                Expanded(child: TextField(controller: msgController)),
                IconButton(
                  icon: Icon(Icons.send, color: Colors.deepPurple),
                  onPressed: sendMessage,
                )
              ],
            ),
          )
        ],
      ),
    );
  }
}

class CallsPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Placeholder for calls UI (can use agora / Jitsi later)
    return Scaffold(
      appBar: AppBar(title: Text("Calls 📞"), backgroundColor: Colors.deepPurple),
      body: Center(child: Text("Call feature coming soon!")),
    );
  }
}video_player: ^2.8.1
cloud_firestore: ^4.13.1import 'package:flutter/material.dart';
import 'package:video_player/video_player.dart';
import 'package:cloud_firestore/cloud_firestore.dart';

class ReelsPage extends StatefulWidget {
  @override
  _ReelsPageState createState() => _ReelsPageState();
}

class _ReelsPageState extends State<ReelsPage> {
  PageController _pageController = PageController();
  List<String> videoUrls = [];

  @override
  void initState() {
    super.initState();
    fetchReels();
  }

  Future<void> fetchReels() async {
    final snapshot = await FirebaseFirestore.instance.collection('reels').get();
    if (snapshot.docs.isNotEmpty) {
      videoUrls = snapshot.docs.map((doc) => doc['videoUrl'] as String).toList();
    } else {
      // Dummy reels
      videoUrls = [
        'https://flutter.github.io/assets-for-api-docs/assets/videos/bee.mp4',
        'https://flutter.github.io/assets-for-api-docs/assets/videos/butterfly.mp4',
      ];
    }
    setState(() {});
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      body: PageView.builder(
        scrollDirection: Axis.vertical,
        controller: _pageController,
        itemCount: videoUrls.length,
        itemBuilder: (context, index) {
          return VideoPlayerWidget(url: videoUrls[index]);
        },
      ),
    );
  }
}

class VideoPlayerWidget extends StatefulWidget {
  final String url;
  const VideoPlayerWidget({required this.url});

  @override
  _VideoPlayerWidgetState createState() => _VideoPlayerWidgetState();
}

class _VideoPlayerWidgetState extends State<VideoPlayerWidget> {
  late VideoPlayerController _controller;

  @override
  void initState() {
    super.initState();
    _controller = VideoPlayerController.network(widget.url)
      ..initialize().then((_) {
        _controller.setLooping(true);
        _controller.play();
        setState(() {});
      });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return _controller.value.isInitialized
        ? Stack(
            fit: StackFit.expand,
            children: [
              FittedBox(
                fit: BoxFit.cover,
                child: SizedBox(
                  width: _controller.value.size.width,
                  height: _controller.value.size.height,
                  child: VideoPlayer(_controller),
                ),
              ),
              Positioned(
                right: 20,
                bottom: 60,
                child: Column(
                  children: [
                    Icon(Icons.favorite, color: Colors.white),
                    SizedBox(height: 10),
                    Icon(Icons.comment, color: Colors.white),
                    SizedBox(height: 10),
                    Icon(Icons.share, color: Colors.white),
                  ],
                ),
              ),
            ],
          )
        : Center(child: CircularProgressIndicator());
  }
}
