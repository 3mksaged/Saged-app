import 'dart:convert';
import 'dart:math';
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const SajidFamilyApp());
}

class SajidFamilyApp extends StatelessWidget {
  const SajidFamilyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'منظومة الإدارة الأسرية',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        useMaterial3: true,
        fontFamily: 'Roboto',
        colorScheme: ColorScheme.fromSeed(
          seedColor: const Color(0xFF0F766E),
          primary: const Color(0xFF0F766E),
          secondary: const Color(0xFFF59E0B),
          surface: const Color(0xFFF8FAFC),
        ),
      ),
      home: const SplashScreen(),
    );
  }
}

// ---------------- شاشة الفحص والتأكد من الحساب النشط ----------------
class SplashScreen extends StatefulWidget {
  const SplashScreen({super.key});

  @override
  State<SplashScreen> createState() => _SplashScreenState();
}

class _SplashScreenState extends State<SplashScreen> {
  @override
  void initState() {
    super.initState();
    _checkActiveUser();
  }

  void _checkActiveUser() async {
    final prefs = await SharedPreferences.getInstance();
    final activeEmail = prefs.getString('active_user_email');

    if (activeEmail != null) {
      final userDataRaw = prefs.getString('user_$activeEmail');
      if (userDataRaw != null) {
        final Map<String, dynamic> userData = jsonDecode(userDataRaw);
        if (!mounted) return;
        if (userData['role'] == 'student') {
          Navigator.pushReplacement(
            context,
            MaterialPageRoute(
              builder: (_) => StudentDashboard(email: activeEmail, userData: userData),
            ),
          );
          return;
        } else if (userData['role'] == 'supervisor') {
          Navigator.pushReplacement(
            context,
            MaterialPageRoute(
              builder: (_) => MotherDashboard(email: activeEmail, userData: userData),
            ),
          );
          return;
        }
      }
    }

    if (!mounted) return;
    Navigator.pushReplacement(
      context,
      MaterialPageRoute(builder: (_) => const WelcomeScreen()),
    );
  }

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Center(
        child: CircularProgressIndicator(color: Color(0xFF0F766E)),
      ),
    );
  }
}
// ---------------- الشاشة الترحيبية ----------------
class WelcomeScreen extends StatelessWidget {
  const WelcomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        decoration: const BoxDecoration(
          gradient: LinearGradient(
            colors: [Color(0xFF0F766E), Color(0xFF1E293B)],
            begin: Alignment.topCenter,
            end: Alignment.bottomCenter,
          ),
        ),
        child: SafeArea(
          child: Column(
            children: [
              Expanded(
                child: Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Container(
                        padding: const EdgeInsets.all(22),
                        decoration: BoxDecoration(
                          color: Colors.white.withOpacity(0.12),
                          shape: BoxShape.circle,
                          border: Border.all(color: Colors.white30, width: 2),
                        ),
                        child: const Icon(Icons.auto_stories_rounded,
                            size: 75, color: Color(0xFFF59E0B)),
                      ),
                      const SizedBox(height: 25),
                      const Text(
                        'منظومة الإدارة الأسرية',
                        style: TextStyle(
                            fontSize: 28,
                            fontWeight: FontWeight.bold,
                            color: Colors.white),
                      ),
                      const SizedBox(height: 8),
                      const Text(
                        'متابعة المذاكرة والدروس والمصاريف بذكاء',
                        style: TextStyle(fontSize: 14, color: Colors.white70),
                      ),
                    ],
                  ),
                ),
              ),
              Padding(
                padding: const EdgeInsets.symmetric(horizontal: 30, vertical: 20),
                child: Column(
                  children: [
                    ElevatedButton(
                      style: ElevatedButton.styleFrom(
                        backgroundColor: const Color(0xFFF59E0B),
                        foregroundColor: Colors.black,
                        minimumSize: const Size(double.infinity, 55),
                        shape: RoundedRectangleBorder(
                            borderRadius: BorderRadius.circular(16)),
                      ),
                      onPressed: () {
                        Navigator.push(
                          context,
                          MaterialPageRoute(builder: (_) => const AuthOptionScreen()),
                        );
                      },
                      child: const Text('ابدأ الآن',
                          style: TextStyle(
                              fontSize: 18, fontWeight: FontWeight.bold)),
                    ),
                    const SizedBox(height: 25),
                    const Text(
                      'المبرمج ساجد',
                      style: TextStyle(
                          fontSize: 12,
                          color: Colors.white38,
                          letterSpacing: 2,
                          fontWeight: FontWeight.w600),
                    ),
                  ],
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// ---------------- خيار تسجيل دخول أم حساب جديد ----------------
class AuthOptionScreen extends StatelessWidget {
  const AuthOptionScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('دخول المنظومة'), centerTitle: true),
      body: Padding(
        padding: const EdgeInsets.all(24.0),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton.icon(
              icon: const Icon(Icons.login_rounded),
              label: const Text('تسجيل دخول بحساب حالي',
                  style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
              style: ElevatedButton.styleFrom(
                minimumSize: const Size(double.infinity, 60),
                backgroundColor: const Color(0xFF0F766E),
                foregroundColor: Colors.white,
                shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(16)),
              ),
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (_) => const LoginScreen()),
                );
              },
            ),
            const SizedBox(height: 20),
            OutlinedButton.icon(
              icon: const Icon(Icons.person_add_alt_1_rounded),
              label: const Text('إنشاء حساب جديد',
                  style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
              style: OutlinedButton.styleFrom(
                minimumSize: const Size(double.infinity, 60),
                foregroundColor: const Color(0xFF0F766E),
                side: const BorderSide(color: Color(0xFF0F766E), width: 2),
                shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(16)),
              ),
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (_) => const RoleSelectionScreen()),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
// ---------------- شاشة تسجيل الدخول ----------------
class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});

  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  final _formKey = GlobalKey<FormState>();

  void _login() async {
    if (_formKey.currentState!.validate()) {
      final email = _emailController.text.trim();
      final password = _passwordController.text.trim();

      final prefs = await SharedPreferences.getInstance();
      final userDataRaw = prefs.getString('user_$email');

      if (userDataRaw == null) {
        _showError('هذا البريد غير مسجل مسبقاً');
        return;
      }

      final Map<String, dynamic> userData = jsonDecode(userDataRaw);
      if (userData['password'] != password) {
        _showError('كلمة المرور غير صحيحة');
        return;
      }

      await prefs.setString('active_user_email', email);

      if (!mounted) return;
      if (userData['role'] == 'student') {
        Navigator.pushAndRemoveUntil(
          context,
          MaterialPageRoute(
              builder: (_) => StudentDashboard(email: email, userData: userData)),
          (route) => false,
        );
      } else {
        Navigator.pushAndRemoveUntil(
          context,
          MaterialPageRoute(
              builder: (_) => MotherDashboard(email: email, userData: userData)),
          (route) => false,
        );
      }
    }
  }

  void _showError(String msg) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(msg), backgroundColor: Colors.red),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('تسجيل الدخول')),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                controller: _emailController,
                decoration: const InputDecoration(
                  labelText: 'البريد الإلكتروني',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.email),
                ),
                validator: (v) => v == null || v.isEmpty ? 'مطلوب' : null,
              ),
              const SizedBox(height: 15),
              TextFormField(
                controller: _passwordController,
                obscureText: true,
                decoration: const InputDecoration(
                  labelText: 'كلمة المرور',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.lock),
                ),
                validator: (v) => v == null || v.isEmpty ? 'مطلوب' : null,
              ),
              const SizedBox(height: 25),
              ElevatedButton(
                style: ElevatedButton.styleFrom(
                  minimumSize: const Size(double.infinity, 50),
                  backgroundColor: const Color(0xFF0F766E),
                  foregroundColor: Colors.white,
                ),
                onPressed: _login,
                child: const Text('دخول', style: TextStyle(fontSize: 16)),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// ---------------- اختيار نوع الحساب ----------------
class RoleSelectionScreen extends StatelessWidget {
  const RoleSelectionScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('نوع الحساب الجديد')),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ListTile(
              shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(15)),
              tileColor: Colors.teal.shade50,
              leading: const Icon(Icons.person, color: Color(0xFF0F766E), size: 36),
              title: const Text('حساب طالب', style: TextStyle(fontWeight: FontWeight.bold)),
              subtitle: const Text('إعداد جدول المذاكرة ومواعيد الدروس'),
              onTap: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (_) => const StudentRegistrationScreen()),
                );
              },
            ),
            const SizedBox(height: 20),
            ListTile(
              shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(15)),
              tileColor: Colors.amber.shade50,
              leading: const Icon(Icons.supervisor_account, color: Color(0xFFD97706), size: 36),
              title: const Text('حساب مشرفة (الأم)', style: TextStyle(fontWeight: FontWeight.bold)),
              subtitle: const Text('متابعة جداول ومصاريف الأبناء'),
              onTap: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (_) => const MotherRegistrationScreen()),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
// ---------------- إنشاء حساب طالب مع إيميل وباسورد ----------------
class StudentRegistrationScreen extends StatefulWidget {
  const StudentRegistrationScreen({super.key});

  @override
  State<StudentRegistrationScreen> createState() => _StudentRegistrationScreenState();
}

class _StudentRegistrationScreenState extends State<StudentRegistrationScreen> {
  final _formKey = GlobalKey<FormState>();
  String? name;
  String? stage;
  String? grade;
  String? email;
  String? password;

  final Map<String, List<String>> gradesMap = {
    'ابتدائي': [
      'الأول الإبتدائي', 'الثاني الإبتدائي', 'الثالث الإبتدائي',
      'الرابع الإبتدائي', 'الخامس الإبتدائي', 'السادس الإبتدائي'
    ],
    'إعدادي': ['الأول الإعدادي', 'الثاني الإعدادي', 'الثالث الإعدادي'],
    'ثانوي': ['الأول الثانوي', 'الثاني الثانوي', 'الثالث الثانوي'],
  };

  void _registerStudent() async {
    if (_formKey.currentState!.validate()) {
      _formKey.currentState!.save();
      final prefs = await SharedPreferences.getInstance();

      if (prefs.containsKey('user_$email')) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('البريد الإلكتروني مستخدم بالفعل!'), backgroundColor: Colors.red),
        );
        return;
      }

      String studentCode = "S-${Random().nextInt(899999) + 100000}";

      Map<String, dynamic> userData = {
        'role': 'student',
        'name': name,
        'stage': stage,
        'grade': grade,
        'email': email,
        'password': password,
        'code': studentCode,
        'studySchedule': {},
      };

      await prefs.setString('user_$email', jsonEncode(userData));
      await prefs.setString('active_user_email', email!);

      if (!mounted) return;
      Navigator.pushAndRemoveUntil(
        context,
        MaterialPageRoute(
          builder: (_) => StudentDashboard(email: email!, userData: userData),
        ),
        (route) => false,
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('تسجيل حساب طالب')),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Form(
          key: _formKey,
          child: ListView(
            children: [
              TextFormField(
                decoration: const InputDecoration(labelText: 'الاسم الكامل', border: OutlineInputBorder()),
                validator: (v) => v == null || v.isEmpty ? 'مطلوب' : null,
                onSaved: (v) => name = v,
              ),
              const SizedBox(height: 12),
              DropdownButtonFormField<String>(
                decoration: const InputDecoration(labelText: 'المرحلة الدراسية', border: OutlineInputBorder()),
                value: stage,
                items: gradesMap.keys.map((s) => DropdownMenuItem(value: s, child: Text(s))).toList(),
                onChanged: (val) => setState(() { stage = val; grade = null; }),
                validator: (v) => v == null ? 'مطلوب' : null,
                onSaved: (v) => stage = v,
              ),
              const SizedBox(height: 12),
              if (stage != null) ...[
                DropdownButtonFormField<String>(
                  decoration: const InputDecoration(labelText: 'الصف الدراسي', border: OutlineInputBorder()),
                  value: grade,
                  items: gradesMap[stage]!.map((g) => DropdownMenuItem(value: g, child: Text(g))).toList(),
                  onChanged: (val) => setState(() => grade = val),
                  validator: (v) => v == null ? 'مطلوب' : null,
                  onSaved: (v) => grade = v,
                ),
                const SizedBox(height: 12),
              ],
              TextFormField(
                decoration: const InputDecoration(labelText: 'البريد الإلكتروني', border: OutlineInputBorder()),
                keyboardType: TextInputType.emailAddress,
                validator: (v) => v == null || !v.contains('@') ? 'إيميل غير صحيح' : null,
                onSaved: (v) => email = v!.trim(),
              ),
              const SizedBox(height: 12),
              TextFormField(
                obscureText: true,
                decoration: const InputDecoration(labelText: 'كلمة المرور', border: OutlineInputBorder()),
                validator: (v) => v == null || v.length < 6 ? 'كلمة المرور يجب أن لا تقل عن 6 أحرف' : null,
                onSaved: (v) => password = v!.trim(),
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                style: ElevatedButton.styleFrom(
                  backgroundColor: const Color(0xFF0F766E),
                  foregroundColor: Colors.white,
                  minimumSize: const Size(double.infinity, 50),
                ),
                onPressed: _registerStudent,
                child: const Text('إنشاء الحساب والاستمرار'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
// ---------------- لوحة تحكم الطالب الكاملة ----------------
class StudentDashboard extends StatefulWidget {
  final String email;
  final Map<String, dynamic> userData;

  const StudentDashboard({super.key, required this.email, required this.userData});

  @override
  State<StudentDashboard> createState() => _StudentDashboardState();
}

class _StudentDashboardState extends State<StudentDashboard> {
  int _selectedIndex = 0;
  late Map<String, dynamic> currentUserData;

  @override
  void initState() {
    super.initState();
    currentUserData = widget.userData;
  }

  void _logout() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.remove('active_user_email');
    if (!mounted) return;
    Navigator.pushAndRemoveUntil(
      context,
      MaterialPageRoute(builder: (_) => const WelcomeScreen()),
      (route) => false,
    );
  }

  void _editData(String field, String title) {
    TextEditingController controller =
        TextEditingController(text: currentUserData[field]?.toString() ?? '');
    showDialog(
      context: context,
      builder: (_) => AlertDialog(
        title: Text('تعديل $title'),
        content: TextField(controller: controller),
        actions: [
          TextButton(onPressed: () => Navigator.pop(context), child: const Text('إلغاء')),
          ElevatedButton(
            onPressed: () async {
              setState(() {
                currentUserData[field] = controller.text;
              });
              final prefs = await SharedPreferences.getInstance();
              await prefs.setString('user_${widget.email}', jsonEncode(currentUserData));
              if (!mounted) return;
              Navigator.pop(context);
            },
            child: const Text('حفظ'),
          )
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: const Color(0xFF0F766E),
        foregroundColor: Colors.white,
        title: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(currentUserData['name'] ?? '',
                style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            Text(currentUserData['grade'] ?? '',
                style: const TextStyle(fontSize: 12, color: Colors.white70)),
          ],
        ),
        actions: [
          Container(
            padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
            decoration: BoxDecoration(
              color: Colors.white24,
              borderRadius: BorderRadius.circular(8),
            ),
            child: Text(currentUserData['code'] ?? '',
                style: const TextStyle(fontWeight: FontWeight.bold, fontSize: 13)),
          ),
          PopupMenuButton<String>(
            icon: const Icon(Icons.more_vert),
            onSelected: (val) {
              if (val == 'logout') _logout();
              if (val == 'edit_stage') _editData('stage', 'المرحلة الدراسية');
              if (val == 'edit_grade') _editData('grade', 'الصف الدراسي');
            },
            itemBuilder: (_) => [
              const PopupMenuItem(value: 'edit_stage', child: Text('تعديل المرحلة')),
              const PopupMenuItem(value: 'edit_grade', child: Text('تعديل الصف الدراسي')),
              const PopupMenuDivider(),
              const PopupMenuItem(
                  value: 'logout',
                  child: Text('تسجيل الخروج', style: TextStyle(color: Colors.red))),
            ],
          )
        ],
      ),
      body: _selectedIndex == 0
          ? StudyScheduleView(email: widget.email, userData: currentUserData)
          : const Center(child: Text('جدول مواعيد الدروس والتنبيهات')),
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _selectedIndex,
        selectedItemColor: const Color(0xFF0F766E),
        onTap: (i) => setState(() => _selectedIndex = i),
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.calendar_month), label: 'جدول المذاكرة'),
          BottomNavigationBarItem(icon: Icon(Icons.alarm), label: 'مواعيد الدروس'),
        ],
      ),
    );
  }
}

// ---------------- تفاصيل جدول المذاكرة مع الحفظ الذاتي ----------------
class StudyScheduleView extends StatefulWidget {
  final String email;
  final Map<String, dynamic> userData;

  const StudyScheduleView({super.key, required this.email, required this.userData});

  @override
  State<StudyScheduleView> createState() => _StudyScheduleViewState();
}

class _StudyScheduleViewState extends State<StudyScheduleView> {
  final List<String> days = ['السبت', 'الأحد', 'الاثنين', 'الثلاثاء', 'الأربعاء', 'الخميس', 'الجمعة'];
  late Map<String, dynamic> scheduleData;

  @override
  void initState() {
    super.initState();
    scheduleData = widget.userData['studySchedule'] ?? {};
  }

  void _toggleStatus(String day, int index) async {
    List<dynamic> currentDayStatuses = List.from(scheduleData[day] ?? [0, 0, 0, 0]);
    currentDayStatuses[index] = (currentDayStatuses[index] + 1) % 3;

    setState(() {
      scheduleData[day] = currentDayStatuses;
      widget.userData['studySchedule'] = scheduleData;
    });

    final prefs = await SharedPreferences.getInstance();
    await prefs.setString('user_${widget.email}', jsonEncode(widget.userData));
  }

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      padding: const EdgeInsets.all(12),
      itemCount: days.length,
      itemBuilder: (_, index) {
        String day = days[index];
        List<dynamic> statuses = scheduleData[day] ?? [0, 0, 0, 0];

        return Card(
          margin: const EdgeInsets.only(bottom: 12),
          child: Padding(
            padding: const EdgeInsets.all(12.0),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(day, style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold)),
                const Divider(),
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceAround,
                  children: List.generate(4, (i) {
                    int st = statuses[i];
                    return InkWell(
                      onTap: () => _toggleStatus(day, i),
                      child: Container(
                        padding: const EdgeInsets.all(10),
                        decoration: BoxDecoration(
                          color: st == 1
                              ? Colors.green.shade100
                              : st == 2
                                  ? Colors.red.shade100
                                  : Colors.grey.shade100,
                          borderRadius: BorderRadius.circular(8),
                          border: Border.all(
                            color: st == 1 ? Colors.green : st == 2 ? Colors.red : Colors.grey.shade300,
                          ),
                        ),
                        child: Icon(
                          st == 1 ? Icons.check_circle : st == 2 ? Icons.cancel : Icons.radio_button_unchecked,
                          color: st == 1 ? Colors.green : st == 2 ? Colors.red : Colors.grey,
                        ),
                      ),
                    );
                  }),
                ),
              ],
            ),
          ),
        );
      },
    );
  }
}
// ---------------- تسجيل الأم ----------------
class MotherRegistrationScreen extends StatefulWidget {
  const MotherRegistrationScreen({super.key});

  @override
  State<MotherRegistrationScreen> createState() => _MotherRegistrationScreenState();
}

class _MotherRegistrationScreenState extends State<MotherRegistrationScreen> {
  final _formKey = GlobalKey<FormState>();
  String? name;
  String? email;
  String? password;
  String? studentCodeInput;

  void _registerMother() async {
    if (_formKey.currentState!.validate()) {
      _formKey.currentState!.save();
      final prefs = await SharedPreferences.getInstance();

      if (prefs.containsKey('user_$email')) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('الإيميل مستخدم مسبقاً'), backgroundColor: Colors.red),
        );
        return;
      }

      Map<String, dynamic> userData = {
        'role': 'supervisor',
        'name': name,
        'email': email,
        'password': password,
        'studentCodes': [studentCodeInput],
      };

      await prefs.setString('user_$email', jsonEncode(userData));
      await prefs.setString('active_user_email', email!);

      if (!mounted) return;
      Navigator.pushAndRemoveUntil(
        context,
        MaterialPageRoute(
          builder: (_) => MotherDashboard(email: email!, userData: userData),
        ),
        (route) => false,
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('تسجيل الأم / المشرفة')),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Form(
          key: _formKey,
          child: ListView(
            children: [
              TextFormField(
                decoration: const InputDecoration(labelText: 'الاسم الكامل', border: OutlineInputBorder()),
                validator: (v) => v == null || v.isEmpty ? 'مطلوب' : null,
                onSaved: (v) => name = v,
              ),
              const SizedBox(height: 12),
              TextFormField(
                decoration: const InputDecoration(labelText: 'معرف الطالب (الكود)', border: OutlineInputBorder()),
                validator: (v) => v == null || v.isEmpty ? 'يرجى إدخال كود الطالب' : null,
                onSaved: (v) => studentCodeInput = v!.trim(),
              ),
              const SizedBox(height: 12),
              TextFormField(
                decoration: const InputDecoration(labelText: 'البريد الإلكتروني', border: OutlineInputBorder()),
                keyboardType: TextInputType.emailAddress,
                validator: (v) => v == null || !v.contains('@') ? 'إيميل غير صحيح' : null,
                onSaved: (v) => email = v!.trim(),
              ),
              const SizedBox(height: 12),
              TextFormField(
                obscureText: true,
                decoration: const InputDecoration(labelText: 'كلمة المرور', border: OutlineInputBorder()),
                validator: (v) => v == null || v.length < 6 ? '6 أحرف على الأقل' : null,
                onSaved: (v) => password = v!.trim(),
              ),
              const SizedBox(height: 20),
              ElevatedButton(
                style: ElevatedButton.styleFrom(
                  backgroundColor: const Color(0xFFD97706),
                  foregroundColor: Colors.white,
                  minimumSize: const Size(double.infinity, 50),
                ),
                onPressed: _registerMother,
                child: const Text('إنشاء حساب ودخول'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// ---------------- لوحة تحكم الأم المتقدمة ----------------
class MotherDashboard extends StatelessWidget {
  final String email;
  final Map<String, dynamic> userData;

  const MotherDashboard({super.key, required this.email, required this.userData});

  void _logout(BuildContext context) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.remove('active_user_email');
    Navigator.pushAndRemoveUntil(
      context,
      MaterialPageRoute(builder: (_) => const WelcomeScreen()),
      (route) => false,
    );
  }

  @override
  Widget build(BuildContext context) {
    List<dynamic> codes = userData['studentCodes'] ?? [];

    return Scaffold(
      appBar: AppBar(
        title: Text('لوحة المتابعة: ${userData['name'] ?? ''}'),
        backgroundColor: const Color(0xFFD97706),
        foregroundColor: Colors.white,
        actions: [
          IconButton(
            icon: const Icon(Icons.logout),
            onPressed: () => _logout(context),
          )
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text('الطلاب المرتبطين بحسابك:',
                style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            const SizedBox(height: 10),
            Expanded(
              child: ListView.builder(
                itemCount: codes.length,
                itemBuilder: (_, i) {
                  return Card(
                    child: ListTile(
                      leading: const Icon(Icons.person, color: Color(0xFF0F766E)),
                      title: Text('كود الطالب: ${codes[i]}'),
                      subtitle: const Text('انقر لمتابعة جدول المذاكرة ومواعيد الدفع'),
                      trailing: const Icon(Icons.arrow_forward_ios, size: 16),
                      onTap: () {
                        ScaffoldMessenger.of(context).showSnackBar(
                          SnackBar(content: Text('جاري فتح بيانات الطالب ${codes[i]}')),
                        );
                      },
                    ),
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}
