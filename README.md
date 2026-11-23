# Internship-Pro5
Create a 10-question multiple-choice quiz with scoring, feedback and a results screen
import 'package:flutter/material.dart';

// ---------- 1. Question model ----------
class Question {
  final String text;
  final List<String> options;
  final int answerIndex;
  const Question(this.text, this.options, this.answerIndex);
}

// ---------- 2. Questions kept in a separate list ----------
const questions = [
  Question('Which language does Flutter use?', [
    'Java',
    'Dart',
    'Swift',
    'Go',
  ], 1),
  Question('Everything in Flutter is a...', [
    'Widget',
    'Activity',
    'Page',
    'Script',
  ], 0),
  Question('Which kind of widget can change its look using setState?', [
    'StatelessWidget',
    'InheritedWidget',
    'StatefulWidget',
    'Container',
  ], 2),
  Question('Which widget arranges its children vertically?', [
    'Row',
    'Column',
    'Stack',
    'Wrap',
  ], 1),
  Question('Which widget places children on top of each other?', [
    'Column',
    'Row',
    'Stack',
    'Expanded',
  ], 2),
  Question('Which company created Flutter?', [
    'Google',
    'Apple',
    'Microsoft',
    'Meta',
  ], 0),
  Question('Which command creates a new Flutter project?', [
    'flutter new',
    'flutter create',
    'flutter init',
    'flutter start',
  ], 1),
  Question('Which Dart keyword declares a compile-time constant?', [
    'var',
    'final',
    'static',
    'const',
  ], 3),
  Question('Which feature updates code in a running app and keeps its state?', [
    'Hot restart',
    'Hot reload',
    'Cold boot',
    'Full rebuild',
  ], 1),
  Question('Which widget builds list items lazily as they scroll into view?', [
    'Column',
    'Row',
    'Stack',
    'ListView.builder',
  ], 3),
];

// ---------- 3. Sanity check for the question data ----------
void checkQuestions() {
  var ok = questions.length == 10;
  for (final q in questions) {
    if (q.options.length != 4 ||
        q.answerIndex < 0 ||
        q.answerIndex >= q.options.length) {
      ok = false;
      print('FAIL: bad question -> ${q.text}');
    }
  }
  print(
    '${ok ? 'PASS' : 'FAIL'}: 10 questions, 4 options each, valid answer index',
  );
}

void main() {
  checkQuestions();
  runApp(
    MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(colorSchemeSeed: Colors.teal, useMaterial3: true),
      home: const QuizPage(),
    ),
  );
}

// ---------- 4. Quiz screen and state ----------
class QuizPage extends StatefulWidget {
  const QuizPage({super.key});
  @override
  State<QuizPage> createState() => _QuizPageState();
}

class _QuizPageState extends State<QuizPage> {
  int index = 0;
  int score = 0;
  int? selected; // null means no answer chosen yet

  void choose(int i) {
    if (selected != null)
      return; // ignore taps once answered (no double scoring)
    setState(() {
      selected = i;
      if (i == questions[index].answerIndex) score++;
    });
  }

  void next() => setState(() {
    index++;
    selected = null;
  });

  void restart() => setState(() {
    index = 0;
    score = 0;
    selected = null;
  });

  String resultMessage() {
    final percent = score / questions.length;
    if (percent == 1) return 'Perfect score! Flutter expert!';
    if (percent >= 0.7) return 'Great job! You know your Flutter.';
    if (percent >= 0.4) return 'Good effort. Keep practising!';
    return 'Time to revisit the basics. Try again!';
  }

  Color? tileColor(int i, Question q) {
    if (selected == null) return null;
    if (i == q.answerIndex) return Colors.green.shade100;
    if (i == selected) return Colors.red.shade100;
    return null;
  }

  Widget? trailingIcon(int i, Question q) {
    if (selected == null) return null;
    if (i == q.answerIndex) {
      return const Icon(Icons.check_circle, color: Colors.green);
    }
    if (i == selected) return const Icon(Icons.cancel, color: Colors.red);
    return null;
  }

  @override
  Widget build(BuildContext context) {
    // ----- Results screen -----
    if (index >= questions.length) {
      return Scaffold(
        appBar: AppBar(
          backgroundColor: Colors.teal,
          foregroundColor: Colors.white,
          title: const Text('Results'),
        ),
        body: Center(
          child: Padding(
            padding: const EdgeInsets.all(24),
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                const Icon(Icons.emoji_events, size: 80, color: Colors.amber),
                const SizedBox(height: 16),
                Text(
                  'Score: $score / ${questions.length}',
                  style: const TextStyle(
                    fontSize: 32,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 8),
                Text(
                  resultMessage(),
                  textAlign: TextAlign.center,
                  style: const TextStyle(fontSize: 18),
                ),
                const SizedBox(height: 24),
                ElevatedButton.icon(
                  onPressed: restart,
                  icon: const Icon(Icons.refresh),
                  label: const Text('Try again'),
                ),
              ],
            ),
          ),
        ),
      );
    }

    // ----- Question screen -----
    final q = questions[index];
    final isLast = index == questions.length - 1;

    return Scaffold(
      appBar: AppBar(
        backgroundColor: Colors.teal,
        foregroundColor: Colors.white,
        title: Text('Question ${index + 1} of ${questions.length}'),
        actions: [
          Padding(
            padding: const EdgeInsets.only(right: 16),
            child: Center(child: Text('Score: $score')),
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            LinearProgressIndicator(value: index / questions.length),
            const SizedBox(height: 24),
            Text(q.text, style: const TextStyle(fontSize: 20)),
            const SizedBox(height: 16),
            for (var i = 0; i < q.options.length; i++)
              Card(
                color: tileColor(i, q),
                child: ListTile(
                  title: Text(q.options[i]),
                  trailing: trailingIcon(i, q),
                  onTap: () => choose(i),
                ),
              ),
            const SizedBox(height: 16),
            if (selected != null)
              ElevatedButton(
                onPressed: next,
                child: Text(isLast ? 'See results' : 'Next'),
              ),
          ],
        ),
      ),
    );
  }
}
