import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';
import 'package:shared_preferences/shared_preferences.dart';
import 'package:uuid/uuid.dart';

void main() {
  runApp(const FlashcardsApp());
}

class FlashcardsApp extends StatelessWidget {
  const FlashcardsApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flashcard Quiz',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.teal,
          brightness: Brightness.light,
        ),
        textTheme: GoogleFonts.poppinsTextTheme(),
        useMaterial3: true,
      ),
      darkTheme: ThemeData(
        brightness: Brightness.dark,
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal, brightness: Brightness.dark),
        textTheme: GoogleFonts.poppinsTextTheme(ThemeData.dark().textTheme),
      ),
      themeMode: ThemeMode.system,
      home: const FlashcardHomePage(),
    );
  }
}

class Flashcard {
  String id;
  String question;
  String answer;

  Flashcard({required this.id, required this.question, required this.answer});

  factory Flashcard.fromJson(Map<String, dynamic> json) =>
      Flashcard(id: json['id'], question: json['question'], answer: json['answer']);

  Map<String, dynamic> toJson() => {
        'id': id,
        'question': question,
        'answer': answer,
      };
}

class FlashcardHomePage extends StatefulWidget {
  const FlashcardHomePage({super.key});

  @override
  State<FlashcardHomePage> createState() => _FlashcardHomePageState();
}

class _FlashcardHomePageState extends State<FlashcardHomePage> {
  final String _storageKey = 'flashcards_v2';
  final Uuid _uuid = const Uuid();

  List<Flashcard> _cards = [];
  int _currentIndex = 0;
  bool _showAnswer = false;
  bool _loading = true;

  @override
  void initState() {
    super.initState();
    _loadFlashcards();
  }

  Future<void> _loadFlashcards() async {
    final prefs = await SharedPreferences.getInstance();
    final data = prefs.getString(_storageKey);
    if (data != null) {
      final decoded = jsonDecode(data) as List;
      _cards = decoded.map((e) => Flashcard.fromJson(e)).toList();
    } else {
      _cards = [
        Flashcard(
            id: _uuid.v4(),
            question: 'What is Flutter?',
            answer: 'A UI toolkit by Google for building natively compiled applications.')
      ];
    }
    setState(() => _loading = false);
  }

  Future<void> _saveCards() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(
        _storageKey, jsonEncode(_cards.map((c) => c.toJson()).toList()));
  }

  void _nextCard() {
    if (_cards.isEmpty) return;
    setState(() {
      _currentIndex = (_currentIndex + 1) % _cards.length;
      _showAnswer = false;
    });
  }

  void _previousCard() {
    if (_cards.isEmpty) return;
    setState(() {
      _currentIndex = (_currentIndex - 1 + _cards.length) % _cards.length;
      _showAnswer = false;
    });
  }

  Future<void> _addOrEditCard({Flashcard? editCard}) async {
    final qController = TextEditingController(text: editCard?.question ?? '');
    final aController = TextEditingController(text: editCard?.answer ?? '');

    await showDialog(
      context: context,
      builder: (_) => AlertDialog(
        backgroundColor: Theme.of(context).colorScheme.surfaceContainer,
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
        title: Text(editCard == null ? 'Add Flashcard' : 'Edit Flashcard'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(
              controller: qController,
              decoration: const InputDecoration(labelText: 'Question'),
            ),
            const SizedBox(height: 12),
            TextField(
              controller: aController,
              decoration: const InputDecoration(labelText: 'Answer'),
            ),
          ],
        ),
        actions: [
          TextButton(onPressed: () => Navigator.pop(context), child: const Text('Cancel')),
          ElevatedButton(
              onPressed: () {
                if (qController.text.isEmpty || aController.text.isEmpty) return;
                if (editCard != null) {
                  final idx = _cards.indexWhere((c) => c.id == editCard.id);
                  _cards[idx] = Flashcard(
                    id: editCard.id,
                    question: qController.text,
                    answer: aController.text,
                  );
                } else {
                  _cards.add(Flashcard(
                      id: _uuid.v4(),
                      question: qController.text,
                      answer: aController.text));
                  _currentIndex = _cards.length - 1;
                }
                _saveCards();
                setState(() {});
                Navigator.pop(context);
              },
              child: const Text('Save')),
        ],
      ),
    );
  }

  void _deleteCard() async {
    if (_cards.isEmpty) return;
    final confirm = await showDialog<bool>(
      context: context,
      builder: (_) => AlertDialog(
        title: const Text('Delete card?'),
        content: const Text('This action cannot be undone.'),
        actions: [
          TextButton(onPressed: () => Navigator.pop(context, false), child: const Text('Cancel')),
          ElevatedButton(
              onPressed: () => Navigator.pop(context, true),
              child: const Text('Delete')),
        ],
      ),
    );
    if (confirm == true) {
      _cards.removeAt(_currentIndex);
      _currentIndex = _cards.isEmpty ? 0 : _currentIndex % _cards.length;
      await _saveCards();
      setState(() {});
    }
  }

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);

    return Scaffold(
      body: Container(
        decoration: const BoxDecoration(
          gradient: LinearGradient(
            colors: [Color(0xFF00C9A7), Color(0xFF008C72)],
            begin: Alignment.topLeft,
            end: Alignment.bottomRight,
          ),
        ),
        child: SafeArea(
          child: _loading
              ? const Center(child: CircularProgressIndicator(color: Colors.white))
              : Column(
                  children: [
                    const SizedBox(height: 10),
                    Text('Flashcard Quiz',
                        style: GoogleFonts.poppins(
                          fontSize: 24,
                          fontWeight: FontWeight.w700,
                          color: Colors.white,
                        )),
                    const SizedBox(height: 10),
                    Expanded(
                      child: Center(
                        child: _cards.isEmpty
                            ? Column(
                                mainAxisSize: MainAxisSize.min,
                                children: [
                                  const Text('No flashcards yet!',
                                      style: TextStyle(
                                          color: Colors.white, fontSize: 18)),
                                  const SizedBox(height: 8),
                                  ElevatedButton.icon(
                                    style: ElevatedButton.styleFrom(
                                        backgroundColor: Colors.white),
                                    onPressed: () => _addOrEditCard(),
                                    icon: const Icon(Icons.add),
                                    label: const Text('Add Card'),
                                  )
                                ],
                              )
                            : GestureDetector(
                                onTap: () =>
                                    setState(() => _showAnswer = !_showAnswer),
                                child: AnimatedContainer(
                                  duration: const Duration(milliseconds: 300),
                                  padding: const EdgeInsets.all(24),
                                  margin:
                                      const EdgeInsets.symmetric(horizontal: 20),
                                  decoration: BoxDecoration(
                                    borderRadius: BorderRadius.circular(24),
                                    color: Colors.white.withOpacity(0.95),
                                    boxShadow: [
                                      BoxShadow(
                                          color: Colors.black26,
                                          blurRadius: 15,
                                          offset: const Offset(0, 8))
                                    ],
                                  ),
                                  child: Column(
                                    mainAxisAlignment: MainAxisAlignment.center,
                                    children: [
                                      Text(
                                        _showAnswer
                                            ? 'Answer'
                                            : 'Question',
                                        style: TextStyle(
                                            color: Colors.grey[600],
                                            fontWeight: FontWeight.w500),
                                      ),
                                      const SizedBox(height: 12),
                                      AnimatedSwitcher(
                                        duration:
                                            const Duration(milliseconds: 300),
                                        child: Text(
                                          _showAnswer
                                              ? _cards[_currentIndex].answer
                                              : _cards[_currentIndex].question,
                                          key: ValueKey(_showAnswer),
                                          textAlign: TextAlign.center,
                                          style: const TextStyle(
                                              fontSize: 22,
                                              height: 1.3,
                                              color: Colors.black87),
                                        ),
                                      ),
                                    ],
                                  ),
                                ),
                              ),
                      ),
                    ),
                    const SizedBox(height: 8),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        IconButton(
                            onPressed: _previousCard,
                            icon: const Icon(Icons.arrow_back_ios,
                                color: Colors.white)),
                        Text(
                          '${_cards.isEmpty ? 0 : _currentIndex + 1}/${_cards.length}',
                          style: const TextStyle(color: Colors.white),
                        ),
                        IconButton(
                            onPressed: _nextCard,
                            icon: const Icon(Icons.arrow_forward_ios,
                                color: Colors.white)),
                      ],
                    ),
                    Padding(
                      padding: const EdgeInsets.symmetric(
                          horizontal: 16, vertical: 10),
                      child: Row(
                        mainAxisAlignment: MainAxisAlignment.spaceAround,
                        children: [
                          _roundButton(Icons.add, 'Add',
                              () => _addOrEditCard()),
                          _roundButton(Icons.edit, 'Edit',
                              _cards.isEmpty ? null : () => _addOrEditCard(editCard: _cards[_currentIndex])),
                          _roundButton(Icons.delete, 'Delete',
                              _cards.isEmpty ? null : _deleteCard),
                        ],
                      ),
                    )
                  ],
                ),
        ),
      ),
    );
  }

  Widget _roundButton(IconData icon, String label, VoidCallback? onPressed) {
    return Column(
      children: [
        ElevatedButton(
          style: ElevatedButton.styleFrom(
            backgroundColor:
                onPressed == null ? Colors.grey : Colors.white.withOpacity(0.9),
            shape: const CircleBorder(),
            padding: const EdgeInsets.all(14),
          ),
          onPressed: onPressed,
          child: Icon(icon, color: Colors.teal),
        ),
        const SizedBox(height: 4),
        Text(label,
            style: const TextStyle(
                color: Colors.white, fontWeight: FontWeight.w500)),
      ],
    );
  }
}
