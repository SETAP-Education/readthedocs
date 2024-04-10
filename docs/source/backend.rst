Backend
=====

.. _Login Page:

LoginPage.dart
----------------

The first interactable page after the splash page. This page is managed by Firebase and its corresponding database.

The login page has to interact with the database and actively monitor the input fields in a stateful widget (mutable).

.. code-block:: dart

   class _LoginPageState extends State<LoginPage> {
      final FirebaseAuth _auth = FirebaseAuth.instance;
      final TextEditingController _emailController = TextEditingController();
      final TextEditingController _passwordController = TextEditingController();
      final _formKey = GlobalKey<FormState>();

An instance of ``FirebaseAuth`` is created that handles the authentication of the user inputs via the Firebase API. The ``_emailController`` and ``_passwordController`` handle the text input as well as retrieve them for the authentication instance. Finally, a ``GlobalKey`` is made to uniquely identify the form and ensure that instance is authenticated once.

There is also input sanitisation for security purposes, as well as providing active user feedback.

.. code-block:: dart

   if (email.isNotEmpty) {
      try {
         var user = await FirebaseAuth.instance.fetchSignInMethodsForEmail(email);
         if (user.isNotEmpty) {
            await FirebaseAuth.instance.sendPasswordResetEmail(email: email);

A gesture detector checks the email field through ``onTap`` and cleans up any empty spaces with ``_emailController.text.trim();``. A check is performed to see if the field is empty and if not, action is taken, i.e. the password reset email. There are also additional checks to identify if an email is in fact in the app's database.

.. code-block:: dart

   if (value.isEmpty) {
           setState(() {
             _error = true;
             print("You have an email valid error.");
           });
         } else if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
           setState(() {
             _error = true;
           });
         } else {
           setState(() {
             _error = false; // Reset error state

This is the code that checks that the email format is in acceptable paramaters and the field is not empty. The first two cases will trigger errors if the field is entered as empty or if the email does not fit accepted parameters. The last case will proceed is the field is not empty and matches the regular expression.

.. code-block:: dart

   if (_formKey.currentState!.validate()) {
         try {
           UserCredential userCredential = await _auth.signInWithEmailAndPassword(
             email: _emailController.text,
             password: _passwordController.text,
           );
           if (userCredential.user != null) {
             Navigator.pushReplacement(
               context,
               MaterialPageRoute(builder: (context) => LandingPage()),
             );
           }
         } catch (e) {
           setState(() {
             _error = true;
             _errorMessages.insert(0, "Please ensure all of your login details are correct.");

This code snippet actually validates the email and password fields to move them onto the next page using ``Navigator.pushReplacement``. If the credentials aren't validated against the database, an error is thrown.

.. _Registration Page:

RegistrationPage.dart
---------------------

.. code-block:: dart

   Future<void> _register() async {
       try {
         UserCredential userCredential = await _auth.createUserWithEmailAndPassword(
           email: _emailController.text,
           password: _passwordController.text,
         );
   
         User? user = userCredential.user;
   
         if (user != null) {
           await _createDatabase();
           Navigator.pushReplacement(
             context,
             MaterialPageRoute(builder: (context) => LandingPage()),
           );
         }

Here, the ``_register()`` function attempts to take the input credentials from the fields and place them into the Firebase database. given ``user != null``, it will create an entry in the database and automatically log them into the application.

.. code-block:: dart

   } catch (e) {
         print("Registration failed: $e");
         ScaffoldMessenger.of(context).showSnackBar(
           SnackBar(
             content: Text("Registration failed. Please try again."),
             duration: Duration(seconds: 3),

Should the entry be null, it will display an error for 3 seconds.

.. _Landing Page:

LandingPage.dart
----------------

.. code-block:: dart

   class _LandingPageState extends State<LandingPage> {
     User? _user;
   
     final QuizManager quizManager = QuizManager();
   
     @override
     void initState() {
       super.initState();
       _checkAuthState();
     }
   
     void _checkAuthState() {
       FirebaseAuth.instance.authStateChanges().listen((User? user) {
         if (mounted) {
           setState(() {
             _user = user; // Set the current user
           });

Inside the Landing Page widget, there are listeners for ``authStateChanges()`` that alter the UI depending on the authentication states of the ``User`` variable. This class also creates an instance of the ``QuizManager`` which shows various quizzes and quiz data thata is retrieved from the datastore associated with the user.

.. _Quiz Page:

quiz.dart
---------

The main logic of the quizzes users will interact with.

.. code-block:: dart

   class QuestionAnswer {
   
     void debugPrint() {}
   
     Map<String, dynamic> toFirestore() { return {}; }
   }

The ``QuestionAnswer`` class is responsible for taking user input and converting it to a format that makes it suitable for use in the Firestore database. ``Map<String, dynamic> toFirestore()`` makes this conversion.

.. code-block:: dart

   class QuestionMultipleChoice extends QuestionAnswer {
   
     QuestionMultipleChoice({ required this.options, required this.correctAnswers });
   
     // Multiple choice have multiple options
     // and also a list of correct answers in case 1 or more is correct
     // Probably should detect if there is 1 or more and display UI accordingly
     List<String> options;
     List<int> correctAnswers; 

``QuestionMultipleChoice`` is a subclass of ``QuestionAnswer`` that creates the logic of a multiple choice question. The options the user chooses are represented as strings whereas the number of options (from 1 to n) are represented as integers.

.. code-block:: dart

   Map<String, dynamic> toFirestore() {
       return {
         "options": options, 
         "correctAnswers": correctAnswers
       };
     }
   
     factory QuestionMultipleChoice.fromMap(Map<String, dynamic> map) {
       return QuestionMultipleChoice(
           options: map["options"] is Iterable ? List.from(map["options"]) : List.empty(), 
           correctAnswers:  map["correctAnswers"] is Iterable ? List.from(map["correctAnswers"]) : List.empty()
         );

Here a map is formed to make a keypair of the options and correct answers to be placed in the Firestore. The factory option also creates a QuestionMultipleChoice object from the Firestore map, allowing for flexible conversion of data from the app and the database.

.. code-block:: dart

  String questionText = ""; 

  // This is the type of question. This determines how the question will be displayed/answered
  QuestionType type = QuestionType.none; 

  // Stores the difficulty of the question 
  int difficulty = 0; 

  // List of tags/topics for sorting
  List<String> tags = List.empty(growable: true);

  QuestionAnswer answer = QuestionAnswer();

Here the basic parameters that make up a ``Question`` are initialised. These values are then converted and mapped to a Firestore format much like the previous examples.

.. code-block:: dart

    question.questionText = data["questionText"];
    question.type = QuestionType.values[data["type"]];
    question.difficulty = data.containsKey("difficulty") ? data["difficulty"] : 0;
    question.tags = data["tags"] is Iterable ? List.from(data["tags"]) : List.empty();

    if (question.type == QuestionType.multipleChoice) {
      question.answer = QuestionMultipleChoice.fromMap(data["answer"]);
    }

    return question;

Here data is retrieved from the Firestore using a ``factory`` constructor to assign various properties to `QuizQuestion`, adding extra variables should the question be of the `multipleChoice` type. This then creates a ``QuizQuestion`` object that is used throughout the application.

.. code-block:: dart

   class Quiz { 
   
     Quiz();
   
     // Name/Id of the Quiz
     String name = ""; 
   
     // Quiz creator if any
     // Can be null
     String? creator;
   
     // The share code 
     // Can be null
     String? shareCode;
   
     // Quizzes can be tagged for specific topics
     List<String> tags = List.empty(growable: true);
   
     // Store a List of Ids to questions
     // These questions will be stored outside of the quiz
     List<String> questionIds = List.empty(growable: true); 
   
   
     // This is not stored in the database and is loaded later when the quiz starts 
     List<QuizQuestion> loadedQuestions = List.empty(growable: true);

In a similar fashion to the question structure, here we have the function that generates the quiz itself, including variables like ``creator`` and ``shareCode`` which identifies the quiz through "metadata".

.. code-block:: dart

  // Get the number of questions in the quiz
  int length() { 
    return questionIds.length;
  }

  // Returns a string for the name of the creator
  // or if its auto generated it returns "Auto-Generated"
  String getCreator() {
    return isQuizGenerated() ? "Auto-Generated" : creator!;
  }

  // This returns the sharecode if it exists 
  // If it does not exist it returns an empty string 
  String getShareCode() {
    return shareCode != null ? shareCode! : "";
  }

  // Get if the quiz is generated or is user created
  bool isQuizGenerated() {
    return creator == null ? true : false; 
  }

The quiz properties differ to the question properties as some of its "metadata" would be viewed outside of the context it's contained in, such as the ``creator`` being displayed in the description of the quiz. Hence the use of getters. The method of retrieving these variables from the firestore are identical to the ``Quiz`` example above.

.. _Quiz Manager:

quizManager.dart
----------------

The quiz manager is focused on collating the quizzes together and retrieving them based on their properties through "tags" such as ``difficulty`` and ``shareCode`` which all take part in identifying an individual quiz.

.. code-block:: dart

   class QuizManager {
     // Searches for quizzes with the specified tags
     // Returns an empty list if none exist
     Future<List<Quiz>> getQuizzesWithTags(List<String> tags) {
       return Future(() => List.empty());
     }

The ``QuizManager`` class picks through the tagging system and converts Firestore data into outputs the manager can compare with user input to.

.. code-block:: dart

   // Grab the object with a converter
       var quizRef = await db
           .collection("quizzes")
           .doc(id)
           .withConverter(
               fromFirestore: Quiz.fromFirestore,
               toFirestore: (Quiz quiz, _) => quiz.toFirestore())
           .get();
   
       // Test if the quiz exists
       // If it doesn't return null
       if (!quizRef.exists) {
         return Future(() => null);
       }

``withConverter`` can intercahnge between the object and the database fields. We take the converted parameters and test against various conditionals such as if the reference exists (if no, return null). This is done with all the search parameters for quizzes.

.. _Level Logic:

levellogic.dart
---------------

This file refers to the logic inside the ranking system each user has. It's initialised with

.. code-block:: dart

   int xp = 10;
   int level = 0;
   String rank = '';
   // max level is 10
   bool reachedMaxLevel = false;

Users have a number, a written rank name associated with the xp number and a maxLevel cap that turns on a boolean.

.. code-block:: dart
   
   void setXp(int quizXp){
     xp += quizXp;
   
     // check if level has changed
     if(checkIfLeveledUp() == true){
       if(reachedMaxLevel == true){
         // print 'congrats! you have leveled up as far as possible' message
       }
       else{
         // print 'congrats! you leveled up' message
       }
     }
   
     level = setLevel();
     rank = setRank();

This function is called after the user completes a quiz and updates their xp level according to their performance which is decided by the quiz/questions. There is also a boolean check if the xp increases rank or reaches max level.

.. code-block:: dart
   
   int setLevel(){
     List<int> levels = [100,300,500,1000,1500,2250,3000,4000,5000,7000];
     int currentLevel = 0;
     for(int i=0;i<=9;i++){
       if(xp < levels[i]){
         currentLevel = i;
         break;
       }
       else{
         currentLevel = 10;
         reachedMaxLevel = true;
       }
     }
     return currentLevel;
   }

``setLevel`` is called whenever xp changes, which is used often in the other functions such as ``setXp``. This also sets the rank boundaries that divide the string rank names below:

.. code-block:: dart
   
   String setRank(){
     List<String> rankList = ['Copper', 'Silver', 'Gold', 'Pearl', 'Jade', 'Ruby', 'Sapphire', 'Emerald', 'Opal', 'Diamond'];
     String rank = '';
     for(int i=0;i<=9;i++){
       if(level == i){
         rank = rankList[i];
       }
     }
     return rank;


.. _Main:

main.dart
---------
There is not much in terms of backend for the ``main.dart`` file. It's responsible for initialising the frontend widgets as well as the Firebase services the application uses often.

.. code-block:: dart
   
   void main() async {
     WidgetsFlutterBinding.ensureInitialized();
     
     await Firebase.initializeApp(
       options: DefaultFirebaseOptions.currentPlatform,
     );
     
     runApp(const MyApp());
   }
