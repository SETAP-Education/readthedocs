Backend
=======

.. _Authorisation Page Form:

AuthPageForm.dart
-----------------

.. code-block:: dart
   class _AuthPageFormState extends State<AuthPageForm> {
   
     bool _error = false;

While this page is primarily built by the frontend, there is the backend variable ``bool_error`` that tracks if an error has occured on the page for the backend to handle. There are multiple instances of this to make sure the system catches any errors.


.. _Display Name Page:

DisplayNamePage.dart
--------------------

.. code-block:: dart

   class _DisplayUserState extends State<DisplayUser> {
     final TextEditingController _nameController = TextEditingController();
     User? _user;
     List<String> _selectedInterests = [];
     List<String> _interestsList = [];
     String _displayName = '';
     QuizManager quizManager = QuizManager();

Here the user state has a ``_nameController`` that controls the input for user name and credentials. Some of the variables are defined here, including ``User?`` which allows for user details to be passed without requiring a username, and required variables like ``_selectedInterests`` and ``_displayName`` that are required to the user to be identified and to produce the landing page contents. Errors are thrown if these variables are not supplied. ``quizManager`` creates an instance of the ``QuizManager()`` class that manages each instance of a generated quiz.

.. code-block:: dart

   void initState() {
       super.initState();
       _checkAuthState();
       _fetchInterests();
       _fetchUserData();
     }

Here the ``initState()`` checks the stateful widgets of the current user authentication, selected interests and user data such as display name and associated interests.

.. code-block:: dart

   void _checkAuthState() {
       FirebaseAuth.instance.authStateChanges().listen((User? user) {
         if (mounted) {
           setState(() {
             _user = user;
             _fetchUserData(); // Fetch user data when the user changes
           });

Here this function checks if the user is authenticated on initialisation. This may occur when the session refreshes and the program checks if the user is still logged in and authorised to be logged in.

.. code-block:: dart

   void _fetchInterests() async {
       try {
         DocumentSnapshot interestsDoc = await FirebaseFirestore.instance
             .collection('interests')
             .doc('interests')
             .get();
   
         if (interestsDoc.exists) {
           setState(() {
             _interestsList = List<String>.from(interestsDoc['interests']);
           });

The ``_fetchInterests()`` function check the firestore for a ``DocumentSnapshot`` that includes the list of interests users can select from. If the instance exists and is retrieved, the ``_interestsList`` variable updates. If it's not found, an error is displayed.

.. code-block:: dart

   void _fetchUserData() async {
       if (_user != null) {
         try {
           DocumentSnapshot userDoc =
               await FirebaseFirestore.instance.collection('users').doc(_user!.uid).get();
   
           if (userDoc.exists) {
             setState(() {
               _displayName = userDoc.get('displayName') ?? '';
               _selectedInterests = List<String>.from(userDoc.get('interests') ?? []);
               _nameController.text = _displayName;
             });

The same principle applies as the ``_fetchInterests`` function but instead the firestore checks if the userdata for the authenticated user exists. It retrieves the display name and user selected interests. If no user is found, an error is thrown.

.. code-block:: dart

   void _setDisplayName(String userId, String displayName) {
       final users = FirebaseFirestore.instance.collection('users');
       users.doc(userId).set({ //sets display name
         'displayName': displayName,
       }, SetOptions(merge: true)).then((_) {
         print('Display Name set successfully!');
       }).catchError((error) {
         print('Failed to set Display Name: $error');

This function takes the user details from ``FirebaseFirestore.instance.collection('users')`` and reassigns the display name from user input using ``users.doc(userId).set(
'displayName': displayName,}`` This then updates on userId which is then converted back to the firestore.

.. code-block:: dart

   void _saveInterests(String userId, List<String> interests) {
       final users = FirebaseFirestore.instance.collection('users');
   
       // Saving the interests for the user
       users.doc(userId).set({
         'interests': interests,
       }, void _saveInterests(String userId, List<String> interests) {
    final users = FirebaseFirestore.instance.collection('users');

    // Saving the interests for the user
    users.doc(userId).set({
      'interests': interests,
    }, SetOptions(merge: true)).then((_) {
      print('Interests saved successfully!');.then((_) {
         print('Interests saved successfully!');

The same as above, but instead the variables manupulated are the ``List,String. interests`` which are selected using buttons. Once this is retrieved, it's stored using ``SetOptions(merge: true))`` which ensures only the selected fields are updated and the other fields are left untouched.

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

.. code-block:: dart
   
   Future<bool> _fetchThemePreference(String userId) async {
       try {
         // Retrieve theme preference from Firestore
         DocumentSnapshot documentSnapshot = await FirebaseFirestore.instance
             .collection('users')
             .doc(userId)
             .get();
   
         if (documentSnapshot.exists) {
           return documentSnapshot['darkMode'] ?? false;
         }
   
         // Default to light mode if not specified
         return false;
       } catch (e) {
         print('Error fetching theme preference: $e');
         // Default to light mode in case of error
         return false;
       }
     }
   
     void _setTheme(bool isDarkMode, BuildContext context) {
       context.read<ThemeNotifier>().setTheme(isDarkMode);

When progressing from the login page to the landing page, the Firestore is checked if ``documentSnapshot.exists``, meaning the user selected the dark theme prior to logging in, meaning the landing page will also display as dark mode. If the theming data wasn't modified or can't be retrieved, the landing page will default to light mode. ``setTheme(isDarkMode)`` is the setter of this if the conditions are fulfilled.

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

.. code-block:: dart
   
   Future<void> _register() async {
       try {
   
         bool satisfysMinCharacters = _passwordController.text.length >= minCharacters;
         bool hasOneNumber = _passwordController.text.contains(RegExp(r'[0-9]'));
   
         if (!satisfysMinCharacters || !hasOneNumber) {
           // Password does not satisfy constraints 
   
           globalErrorManager.pushError("Bad password");

The ``_register()`` function contains various constraints the user has to adhere to to register. In the example above, the user must ``satisfyMinCharacters`` defined in the ``_passwordController.text.length`` and meet the boolean ``hasOneNumber`` ranging from 0 to 9. Should the user not meet these requirements, an error is thrown and the user is prompted to try a different password. In the frontend, a cross and checkmark icon tracks the user's progress in fulfilling the requirements.

.. code-block:: dart

   UserCredential userCredential = await _auth.createUserWithEmailAndPassword(
           email: _emailController.text,
           password: _passwordController.text,
         );
   
         User? user = userCredential.user;
   
         if (user != null) {
           await _createDatabase();
           Navigator.pushReplacement(
             context,
             MaterialPageRoute(builder: (context) => DisplayUser()),

If all goes well, Firebase is called to create a user in the database with ``userCredential`` and the ``_createDatabase()`` function. Once that process is complete, the page navigates to the ``DisplayUser()`` function located in the ``DisplayNamePage.dart`` file.

.. _Quiz Summary Page:

QuizSummaryPage.dart
--------------------

.. code-block:: dart

   class QuizSummaryPage extends StatelessWidget {
     final List<QuizQuestion> loadedQuestions;
     final Map<String, dynamic> quizAttemptData;
     final int earnedXp; 
   
     QuizSummaryPage({
       required this.loadedQuestions,
       required this.quizAttemptData,
       required this.earnedXp, 
     });

The ``QuizSummaryPage`` takes 3 parameters that populate the data in the summary page: ``loadedQuestions`` for the questions generated that question, ``quizAttemptData`` for user responses and ``earnedXp`` to display xp earned as a result of completing the quiz.

.. code-block:: dart

   Future<dynamic> getUserResponseForQuestion(QuizQuestion question, Map<String, dynamic> quizAttemptData, int questionIndex) async {
       dynamic userResponse; // Initialize the userResponse variable
   
       quizAttemptData['userSummary'].forEach((questionId, summary) {
         // Check if 'userResponse' exists for the current question
         if (summary.containsKey('userResponse')) {
           // Retrieve 'userResponse' based on question type
           var summaryUserResponse = summary['userResponse'];
   
           // Check question type (assuming multiple choice or fill in the blank)
           if (question.type == QuestionType.multipleChoice && summaryUserResponse is List<int>) {
             // Handle 'userResponse' as List<int> (multiple choice)
             print('Question ID: $questionId, User Response (Multiple Choice): $summaryUserResponse');
             userResponse = summaryUserResponse;
           } else if (question.type == QuestionType.fillInTheBlank && summaryUserResponse is String) {
             // Handle 'userResponse' as String (fill in the blank)
             print('Question ID: $questionId, User Response (Fill in the Blank): $summaryUserResponse');
             userResponse = summaryUserResponse;
      ...
   return userResponse;
   }

Here the retrieved variables are compared between the question data and the attempt data. ``summaryUserResponse`` retrieves response data if it's found to match to the given question. This then gets compared to the conditionals below depending on whether the question type is ``multipleChoice`` or ``fillInTheBlank``. Once this has been done for all questions, the frontend can display the results and "mark" accordingly.

.. _Quiz Page:

QuizPage.dart
-------------

.. code-block:: dart
   
   class _QuizPageState extends State<QuizPage> {
     late TextEditingController fillInTheBlankController;
     late QuizManager quizManager;
     late Quiz quiz;
     late List<QuizQuestion> loadedQuestions = [];
     int currentQuestionIndex = 0;
     bool quizCompleted = false;
     Map<String, dynamic> userSummary = {};
     bool quizSubmitted = false;
     int earnedXp = 0; 
     // Replace the quizId being passed in, it is static for testing purposes.
     Map<String, dynamic> quizAttemptData = {};

Initialises the variables required to build the quiz layout and contents.

.. code-block:: dart
   
   Future<void> loadQuiz(String quizId) async {
       print("Loading quiz with ID: $quizId");
   
       Quiz? loadedQuiz = await quizManager.getQuizWithId(quizId);
   
       if (loadedQuiz != null) {
         setState(() {
           quiz = loadedQuiz;
         });
   
         // Print quiz details
         print("Loaded quiz: ${quiz.name}");
         print("Question IDs: ${quiz.questionIds}");
   
         List<QuizQuestion> questions = [];
         for (String questionId in quiz.questionIds) {
           print(
               "1 Fetching Question: $questionId, list length: ${questions.length}");
   
           // Fetch the question document directly from Firestore using QuizManager instead
           QuizQuestion? question =
               await QuizManager().getQuizQuestionById(questionId);
   
           if (question != null) {
             questions.add(question);
   
             // Print question type
             print("Question Text: ${question.questionText}");
             print("Question Type: ${question.type}");
   
             print("Added question, list length: ${questions.length}");

Here a quiz is loaded using the ``quizManager`` functions to build the structure of the quiz. If loaded successfully, details are collected including ``quiz.name`` and ``questionIds``. An empty list ``questions`` is initialised and a loop iterates through the loaded questions. If they are not empty, they are added to the list. Once it's iterated enough times to fill the question total, the state is updated.

.. code-block:: dart

   void moveToNextOrSubmit() async {
       if (currentQuestionIndex >= loadedQuestions.length) {
         // Prevents accessing an index that is out of bounds
         return;
       }
   
       QuizQuestion currentQuestion = loadedQuestions[currentQuestionIndex];
       String questionId = quiz.questionIds[currentQuestionIndex]; // Get the correct questionId
   
       Map<String, dynamic> questionSummary;
   
       if (currentQuestion.type == QuestionType.multipleChoice) {
         if (currentQuestion.answer is QuestionMultipleChoice) {
           questionSummary = checkUserAnswers(
             currentQuestion,
             questionId,
             currentQuestion.type,
             userSummary,
           );
         } else {
           print("Error: Incorrect question type for multiple-choice question.");
           return;
         }
       } else if (currentQuestion.type == QuestionType.fillInTheBlank) {
         if (currentQuestion.answer is QuestionFillInTheBlank) {
           questionSummary = checkUserAnswers(
             currentQuestion,
             questionId,
             currentQuestion.type,
             userSummary,

This function handles the navigaton inside the quiz, moving between question numbers. ``currentQuestionIndex >= loadedQuestions.length`` makes sure the navigation cannot progress past the total number of questions. ``questionSummary`` accounts for the responses to that particular question whereas ``userSummary`` evaluates answers across the whole quiz. There are two conditionals for the two ``QuestionTypes`` that handle input differently.

.. code-block:: dart
   
   if (currentQuestionIndex < loadedQuestions.length - 1) {
         setState(() {
           currentQuestionIndex++;
           quizCompleted = false;
         });
         await displayQuestion(currentQuestionIndex, quiz.questionIds);
       } else {
         setState(() {
           quizCompleted = true;
         });
   
         await storeUserAnswersInFirebase(userSummary);
         Map<String, dynamic> quizAttemptData = createQuizAttemptData(userSummary);

The conditional checks if the user is on a question that is not the last. If that's the case ``quizCompleted = false`` when pressing buttons to progress. If it's anything else i.e. the final question, it will complete the quiz upon progression.

.. code-block:: dart

   Map<String, dynamic> createQuizAttemptData(Map<String, dynamic> userSummary) {
       int quizTotal = loadedQuestions.length;
   
       return {
         'timestamp': FieldValue.serverTimestamp(),
         'userResults': {
           'quizTotal': quizTotal,  // Update this with the actual maximum points
           'userTotal': calculateUserTotal(userSummary),
         },
         'userSummary': userSummary,
       };

After the quiz is labelled as completed, it will compile the metadata of the quiz and send it to ``QuizSummaryPage.dart``.

.. code-block:: dart
   
   Map<String, dynamic> quizAttemptData = {
           'timestamp': FieldValue.serverTimestamp(),
           'userResults': {
             'quizTotal': quizTotal, // Update this with the actual number of questions
             'userTotal': calculateUserTotal(userSummary),
           },
           'userSummary': userSummary,
         };
   
         // Store data in Firebase
         await quizAttemptDocument.set(quizAttemptData);
   
         // Calculate the User XP to add
   
   
         int xpGain = calculateXpGain(userSummary, widget.multiplier);
   
         earnedXp = xpGain; 

This code "snapshots" the user data after compiling the quiz and sends it back to the Firestore using ``quizAttemptDocument.set(quizAttemptData)``. This will then be retrieved for the summary page and any time the user wants to review a quiz attempt.

.. code-block:: dart

   var userDoc = await FirebaseFirestore.instance.collection("users").doc(userId).get();
         int currentXp = 0; 
   
         if (userDoc.data() != null) {
           if (userDoc.data()!.containsKey("xpLvl")) {
             currentXp = userDoc.data()!["xpLvl"];
           }
         }
   
         currentXp += xpGain;
   
         FirebaseFirestore.instance.collection("users").doc(userId).update({ "xpLvl": currentXp }, );

This code snippet retrieves the ``currentXp`` from user data and takes the ``xpGain`` generated from the quiz attempt to apply to the user's account. Once the user returns to the landing page, their Xp should update accordingly.

.. code-block:: dart

   userSummary.forEach((questionId, details) {
         if (details['correctIncorrect'] == 'Correct') {
           userTotal += 1;
         }
       });
   
       return userTotal;
     }
   
     int calculateXpGain(Map<String, dynamic> userSummary, double multiplier) {
       int xp = 0; 
   
       userSummary.forEach((questionId, details) {
         if (details['correctIncorrect'] == 'Correct') {
           var q = loadedQuestions.where((element)  { return element.questionId == questionId; });
           xp +=  q.first.difficulty;
         }
       });
   
       xp = xp ~/ loadedQuestions.length;
   
       xp = (xp.toDouble() * multiplier).toInt();
   
       return xp;

The precise calculation is outlined here. ``userTotal`` is defined by the amount of answers the user got correct. This is then returned and used in ``calculateXpGain`` in ``userSummary``. 

.. _Landing Page:

LandingPage.dart
----------------

.. code-block:: dart

   class _LandingPageState extends State<LandingPage> {
     User? _user;
     List<String> userInterests = [];
     int xpLevel = 0; // Assuming XP level is an integer
     late String _displayName = "Placeholder";
     late List<String> otherTopics = [];
   
     late List<QuizQuestion> loadedQuestions = [];
     Map<String, dynamic> quizAttemptData = {};
     Map<String, dynamic> userSummary = {};
     late QuizManager quizManager;
     String quizName = "";
     late Quiz quiz;
   
     @override
     void initState() {
       super.initState();
       _checkAuthState();
       quizManager = QuizManager();
     }
   
     void _checkAuthState() {
       FirebaseAuth.instance.authStateChanges().listen((User? user) {
         if (mounted) {
           setState(() {
             _user = user;
           });
           if (user != null) {
             _fetchOtherTopics();
             _getUserInterests(user.uid);
             _getUserXPLevel(user.uid);
             _getUserDisplayName(user.uid); // Call to get user display name
           }

Inside the Landing Page widget, there are listeners for ``authStateChanges()`` that alter the UI depending on the authentication states of the ``User`` variable. This class also creates an instance of the ``QuizManager`` which shows various quizzes and quiz data thata is retrieved from the datastore associated with the user. Many variables are retrieved due to the amount of information the landing page displays, including ``quizAttemptData`` like in the recent quizzes section and ``user`` data that relates to experience and user interests.

.. code-block:: dart

   void _getUserInterests(String uid) async {
       try {
         DocumentSnapshot userSnapshot =
             await FirebaseFirestore.instance.collection('users').doc(uid).get();
   
         if (userSnapshot.exists) {
           setState(() {
             userInterests = List<String>.from(userSnapshot.get('interests'));

The UI for the quizzes are split between interests and topics that weren't in the user's interests. Here the function checks if ``userSnapshot.exists`` meaning that the user has selected at least one interest and places the interest and its associated widget in the user interests section of the UI.

.. code-block:: dart

   void _fetchOtherTopics() async {
       try {
         if (_user != null) {
           // Get user's interests from Firestore
           DocumentSnapshot userSnapshot = await FirebaseFirestore.instance.collection('users').doc(_user!.uid).get();
           if (userSnapshot.exists) {
             List<String> userInterests = List<String>.from(userSnapshot.get('interests'));
   
             // Query Firestore to get all interests
             DocumentSnapshot interestsSnapshot = await FirebaseFirestore.instance.collection('interests').doc('interests').get();
   
             if (interestsSnapshot.exists) {
               List<String> allInterests = List<String>.from(interestsSnapshot.get('interests'));
   
               // Extract other topics that are not in the user's interests
               List<String> remainingInterests = allInterests.where((interest) => !userInterests.contains(interest)).toList();
   
               // Set the remaining interests as topics
               setState(() {
                 otherTopics = remainingInterests.map((interest) => '$interest').toList();

Here the comments are self explanatory. If the user isn't null, the Firestore is checked to get the user's interests. Whatever is ``!userInterests.contains(interest)`` (not in user's interests) will be added as "other topics".

.. code-block:: dart
   
   Map<String, dynamic> createQuizAttemptData(Map<String, dynamic> userSummary) {
       int quizTotal = loadedQuestions.length;
   
       return {
         'timestamp': FieldValue.serverTimestamp(),
         'userResults': {
           'quizTotal': quizTotal,
           'userTotal': -1,
         },
         'userSummary': userSummary,
       };

Quiz metadata is produced to be retrieved when a user clicks on a quiz in quiz history. Much of this is handled in the backend of ``QuizSummaryPage.dart``.

.. _   Settings Page:

SettingsPage.dart
-----------------

.. code-block:: dart
   
   class _SettingsDisplayUserState extends State<SettingsDisplayUser> {
     final TextEditingController _nameController = TextEditingController();
     User? _user;
     List<String> _selectedInterests = [];
     List<String> _interestsList = [];
     String _displayName = '';
     QuizManager quizManager = QuizManager();

The settings page is functionally identical to ``DisplayNamePage.dart``. More of the difference are explained on the front end. As the code snippet above shows, the data being modified is the same as if a new user was registering.

.. _Splash Page:

SplashPage.dart
---------------

.. code-block:: dart
   
   class _OpeningPageState extends State<OpeningPage> {
   
     void _checkAuthState() async {
   
       User? firebaseUser = await FirebaseAuth.instance.authStateChanges().first;
       
       if (firebaseUser != null) {
         print("User signed in");
         Navigator.pushReplacement(context, MaterialPageRoute(builder:(context) {
           return LandingPage();
         },));
       }
       else {
         print("User not signed in");
       }

This code section checks with Firebase is there is an authorised user logged in. If user is not null, they are signed in and sent to the ``LandingPage()``, else they stay on the splash page. This is especially useful for keeping sessions logged in such as when Internet access disconnects or the page refreshes.

.. _Quiz Page:

quiz.dart
---------

This is one of the most important files in the project as it defines how the quiz operates using factories and many variables.

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
   
   Map<String, dynamic> toFirestore() {
       return {
         "correctAnswer": correctAnswer,
         "userResponse": userResponse,
       };
     }
   
     factory QuestionFillInTheBlank.fromMap(Map<String, dynamic> map) {
       return QuestionFillInTheBlank(
         correctAnswer: map["correctAnswer"] ?? "",
         userResponse: map["userResponse"] ?? "",
       );

Similar to ``QuestionMultipleChoice``, a map of ``correctAnswer`` and ``userResponse`` is made to send back to the Firestore. Functionally they are not different but their user input mediums are different (selection vs textbox). 

.. code-block:: dart

   Map<String, dynamic> checkUserAnswers(
     QuizQuestion question,
     String questionId,
     QuestionType currentType,
     Map<String, dynamic> userSummary,
   ) {
     if (currentType == QuestionType.multipleChoice) {
       if (question.answer is QuestionMultipleChoice) {
         return checkMultipleChoiceAnswer(
           question.answer as QuestionMultipleChoice,
           questionId,
           userSummary,
         );
       } else {
         print("Error: Incorrect question type for multiple-choice question.");
         return userSummary;
       }
     } else if (currentType == QuestionType.fillInTheBlank) {
       if (question.answer is QuestionFillInTheBlank) {
         return checkFillInTheBlankAnswer(
           question.answer as QuestionFillInTheBlank,
           questionId,
           userSummary,
         );
       } else {
         print("Error: Incorrect question type for fill-in-the-blank question.");
         return userSummary;

The function names are self-explanatory. The function first checks what kind of question is being checked. If multiple choice, ``checkMultipleChoiceAnswer`` is returned with ``userSummary`` and ``questionId``. The same applies for fill in the blank questions, instead ``checkFillInTheBlankAnswer`` is called that performs the same operattion with questions of that type.

.. code-block:: dart
   
   Map<String, dynamic> checkMultipleChoiceAnswer(
     QuestionMultipleChoice question,
     String questionId,
     Map<String, dynamic> userSummary,
   ) {
     List<int> correctAnswers = question.correctAnswers;
     List<int> selectedOptions = question.selectedOptions;
     correctAnswers.sort();
     selectedOptions.sort();
   
     print("$selectedOptions");
   
     if (areListsEqual(correctAnswers, selectedOptions)) {
       userSummary[questionId] = {
         'correctIncorrect': 'Correct',
         'userResponse': question.selectedOptions,
         'correctAnswers': correctAnswers,
       };
     } else {
       // The user's answer is incorrect
       print("Incorrect! User selected the wrong options.");
       userSummary[questionId] = {
         'correctIncorrect': 'Incorrect',
         'userResponse': question.selectedOptions,
         'correctAnswers': correctAnswers,

This is what is actually called in ``checkUserAnswers`` for multiple choice questions. if the ``selectedOptions`` and ``correctAnswers`` are equal, it's returned as true and the user gets that question right, else it's incorrect and they get it wrong. They are compared using lists.

.. code-block:: dart

   Map<String, dynamic> checkFillInTheBlankAnswer(
     QuestionFillInTheBlank question,
     String questionId,
     Map<String, dynamic> userSummary,
   ) {
     // Get the correct answer for the question
     String correctAnswer = question.correctAnswer.toLowerCase();
   
     // Get the user's response
     String userResponse = question.userResponse.toLowerCase();
   
     // Check if the user's response matches the correct answer
     bool isCorrect = correctAnswer == userResponse;
   
     // Update the user summary
     userSummary[questionId] = {
       'correctIncorrect': isCorrect ? 'Correct' : 'Incorrect',
       'userResponse': userResponse,
       'correctAnswer': correctAnswer,
     };

Similar to above, though the strings are compared directly by converting both the answer and user response to lowercase using ``.toLowerCase()`` that ensures that capitalisation isn't an issue in the marking process.

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

.. code-block:: dart

   Future<List<QuizQuestion>> getQuizQuestions() async {
       var db = FirebaseFirestore.instance;
   
       var questionRef = await db
           .collection("questions")
           .withConverter(
               fromFirestore: QuizQuestion.fromFirestore,
               toFirestore: (QuizQuestion q, _) => q.toFirestore())
           .get();
   
       return List.generate(
           questionRef.docs.length, (index) => questionRef.docs[index].data());
     }

The ``QuizQuestion`` here is taken from the Firestore to be used elsewhere like in quiz.dart. The opposite conversion is done with this function.

.. code-block:: dart

   static void addQuizQuestionToDatabase(QuizQuestion question) {
       var db = FirebaseFirestore.instance;
   
       db.collection("questions").doc().set(question.toFirestore());
     }

Similar function exist in the same file for conversion of entire quizzes.

.. code-block:: dart

   Future<String> generateQuiz(List<String> tags, int userLevel, int range, int questionCount, { String name = "" }) async {
   
       print("Quiz Generating...");
   
       String outputQuizId = "";
   
       var db = FirebaseFirestore.instance;
   
       // This returns all questions that fit 
       var questionRef = await db
           .collection("questions")
           .where("tags", arrayContainsAny: tags)
           .where("difficulty", isGreaterThan: (userLevel - range))
           .where("difficulty", isLessThan: (userLevel + range))
           .withConverter(
               fromFirestore: QuizQuestion.fromFirestore,
               toFirestore: (QuizQuestion q, _) => q.toFirestore())
           .get();
   
       var questions = List.generate(
           questionRef.docs.length, (index) => questionRef.docs[index].data());

This function is important for generating quizzes based on the user's data that gets fed back into the Firestore using ``q.toFirestore())`` that gets used by other functions. The quiz manager will attempt to find as many questions that match the above criteria, but if none or too little are found, the manager will pick randomly:

.. code-block:: dart

   print("Randomly picking Questions");
         
         var rng = Random();
         for (int i = 0; i < questionCount; i++) {
           var j = rng.nextInt(questions.length);
   
           while (questionNumbers.contains(j)) {
             j = rng.nextInt(questions.length);

As shown with the ``rng`` variable.

.. code-block:: dart

   Future<List<RecentQuiz>> getRecentQuizzesForUser(String userId) async {
       List<RecentQuiz> recent = [];
   
       var db = FirebaseFirestore.instance; 
   
       var userRef = db.collection("users").doc(userId);
   
       var quizzes = await userRef.collection("quizHistory").get();
   
       for (var i in quizzes.docs) {
         Map<String, dynamic> data = i.data(); 
   
         String id = i.id;
   
         Quiz? q = await getQuizWithId(id);
         
         if (q != null) {
   
           // We have a quiz
   
           RecentQuiz recentQuiz = RecentQuiz(); 
           recentQuiz.id = id; 
           recentQuiz.name = q.name;
           recentQuiz.timestamp = data["timestamp"];
           recentQuiz.xpEarned = data.containsKey("xpGain") ? data["xpGain"] : 0;
           
           recent.add(recentQuiz);

The Firebase data store is queried for ``users`` and ``quizHistory``. A ``for loop`` then iterates through each quiz taken and retrieves its id, name, date taken and xp gained. The quizzes are then sorted by recency as shown in the code snippet below:

.. code-block:: dart

   recent.sort(((a, b) {
         return b.timestamp.compareTo(a.timestamp);
       }));

.. _   XP Logic:

xpLogic.dart
---------------

This file refers to the logic inside the ranking system each user has. It's initialised with

.. code-block:: dart

   int xp = 10;
   int level = 0;
   String rank = '';
   // max level is 10
   bool reachedMaxLevel = false;

Users have a number, a written rank name associated with the xp number and a ``maxLevel`` cap that turns on a boolean.

.. code-block:: dart

   class XpInterface {
   
     static List<String> rankList = ['Bronze', 'Silver', 'Gold', 'Platinum', 'Emerald'];
     static List<int> rankThesholds = [ 20, 40, 60, 80, 100 ];
   
     static int getLevel(int xp) {
        for (int i = 0; i < rankThesholds.length; i++) {
         if (xp < rankThesholds[i]) {
           return i;
         }
       }
   
       return 0;
     }
   
     static String getRank(int xp) {
       
       for (int i = 0; i < rankList.length; i++) {
         if (xp < rankThesholds[i]) {
           return rankList[i];

The various ranks, their associated levels and thresholds are defined, which can visually be fed back into ``LandingPage.dart``.

.. code-block:: dart
   
   void setXp(BuildContext context, int quizXp){
     xp += quizXp;
   
     // check if level has changed
     if(checkIfLeveledUp() == true){
       // only need to set level and rank if user has leveled up, otherwise just wasting time
       level = getLevel();
       if(reachedMaxLevel == true){
         ScaffoldMessenger.of(context).showSnackBar(
           SnackBar(
             content: Text('Congratulations you have reached the maximum level! You are now at level ${level} and rank ${rank}'),
           ),
         );
       }
       else{
         ScaffoldMessenger.of(context).showSnackBar(
           SnackBar(
             content: Text('Congratulations you leveled up! You are now at level ${level} and rank ${rank}'),
           ),

This function is called after the user completes a quiz and updates their xp level according to their performance which is decided by the quiz/questions. There is also a boolean check if the xp increases rank or reaches max level.

.. code-block:: dart
   
   bool checkIfLeveledUp(){
     // compare level before and after xp from completed quiz is added
     int prevLevel = level;
     int newLevel = getLevel();
     if(prevLevel != newLevel){
       return true;
     }
     else{
       return false;

``checkIfLeveledUp()`` from the above function is defined here. It takes the previous level based on xp and compares it to the current. This only runs after a quiz is completed, for simplicity.

.. code-block:: dart
   
   String getRank(int xp){
     List<String> rankList = ['Bronze', 'Silver', 'Gold', 'Platinum', 'Emerald'];
     String rank = '';
     for(int i=0;i<=9;i++){
       if(xp == i){
         rank = rankList[i];
       }
     }
     return rank;
   }

While the ranks are defined further up in this file, the actual rank is applied by converting ``xp`` into ``rank`` by comparing them with ``i``. Since the xp boundaries are directly linked to the ranks in the list, the indexes link to each other directly i.e. Bronze = 20, Silver = 40 etc.

.. _Theme Notifier:

ThemeNotifier.dart
------------------

.. code-block:: dart
   
   class ThemeNotifier extends ChangeNotifier {
     bool _isDarkMode = true;
     ThemeData _currentTheme = AppTheme.lightTheme;
   
     ThemeData get currentTheme => _currentTheme;
   
     bool get isDarkMode => _isDarkMode;

The theme data is initialised in ``ThemeNotifier.dart`` the default is ``lightTheme``, which is toggled by setting ``isDarkMode`` to ``true``.

.. code-block:: dart

   void setTheme(bool isDarkMode) {
       _isDarkMode = isDarkMode;
       _currentTheme = isDarkMode ? AppTheme.darkTheme : AppTheme.lightTheme;
       notifyListeners();
   
       // Update user preference in Firestore if user is logged in
       _updateUserThemePreference(isDarkMode);
     }
   
     void toggleTheme() {
       _isDarkMode = !_isDarkMode;
       _currentTheme = _isDarkMode ? AppTheme.darkTheme : AppTheme.lightTheme;
       notifyListeners();
   
       // Update user preference in Firestore if user is logged in
       _updateUserThemePreference(_isDarkMode);

``setTheme`` is concerned with the update of light mode to dark mode. ``toggleTheme`` switches between the two by manipulating the ``_isDarkMode`` variable by setting it to true/false based on user preference.

.. code-block:: dart

   Future<void> _updateUserThemePreference(bool isDarkMode) async {
       final user = FirebaseAuth.instance.currentUser;
   
       if (user != null) {
         try {
           await FirebaseFirestore.instance
               .collection('users')
               .doc(user.uid)
               .update({'darkMode': isDarkMode});
   
   The app theme preferences are uploaded to the Firestore for the associated ``user``. This is intended for when users log in or switch between pages, maintaining their theme preference when transitioning between pages.

.. _Recent Quizzes:

RecentQuizzes.dart
------------------

.. code-block:: dart
   
   class RecentQuizzesState extends State<RecentQuizzes> {
   
     QuizManager quizManager = QuizManager();
     User? _user;
     late List<QuizQuestion> loadedQuestions = [];
     Map<String, dynamic> quizAttemptData = {};
     Map<String, dynamic> userSummary = {};
     int earnedXp = 0; 
     late Quiz quiz;

Variables that get passed to the front end to displayed are initialised here. Not all of the variables stored in something like ``userSummary`` will be used, but the important ones like ``earnedXp``.


.. code-block:: dart
   
   Future<void> _getloadedQuestions(String quizId) async {
       // int currentQuestionIndex = 0;
   
       if (mounted) {
         // print("Loading quiz with ID: $quizId");
   
         Quiz? loadedQuiz = await quizManager.getQuizWithId(quizId);
   
         if (loadedQuiz != null) {
           setState(() {
             quiz = loadedQuiz;
           });
   
           List<QuizQuestion> questions = [];
           for (String questionId in quiz.questionIds) {
             QuizQuestion? question = await QuizManager().getQuizQuestionById(questionId);
   
             if (question != null) {
               questions.add(question);

Here the question "snapshot" from the quiz attempt is retrieved. It will take the question ids used in the attempt and reload them into the quiz builder along with the quiz attempt data below:

.. code-block:: dart

   Future<void> _loadQuizAttemptData(String quizId) async {
       if (_user != null && mounted) {
         try {
           final CollectionReference userCollection = FirebaseFirestore.instance.collection('users');
           final DocumentReference userDoc = userCollection.doc(_user!.uid);
   
           final CollectionReference quizHistoryCollection = userDoc.collection('quizHistory').doc(quizId).collection('attempts');
   
           final QuerySnapshot attemptsSnapshot = await quizHistoryCollection.orderBy('timestamp', descending: true).limit(1).get();
   
           if (attemptsSnapshot.docs.isNotEmpty && mounted) {
             final attemptData = attemptsSnapshot.docs.first.data();
             setState(() {
               if (attemptData != null) {
                 Map<String, dynamic> attemptDataMap = attemptData as Map<String, dynamic>;
                 Map<String, dynamic> userResults = attemptDataMap['userResults'];
                 Map<String, dynamic> userSummary = attemptDataMap['userSummary'];
                 Timestamp? timestamp = attemptDataMap['timestamp'];
   
                 if (userResults.isNotEmpty && userSummary.isNotEmpty && timestamp != null) {
                   quizAttemptData = {
                     'timestamp': FieldValue.serverTimestamp(),
                     'userResults': {
                       'quizTotal': userResults['quizTotal'],
                       'userTotal': userResults['userTotal'],
                     },
                     'userSummary': userSummary,


.. code-block:: dart
   
   final List<String> months = [
         'January',
         'February',
         'March',
         'April',
         'May',
         'June',
         'July',
         'August',
         'September',
         'October',
         'November',
         'December'
       ];
   
     String _nicifyDateTime(DateTime dateTime) {
   
       return "${dateTime.day} ${months[dateTime.month - 1]}";

As shown on the front end, ``_nicifyDateTime`` specifies the months and returns the quiz's timestamp in the day and month completed as opposed to Dart's default method of timestamps which uses nanoseconds, which isn't user friendy to display.

.. _User Info:

UserInfo.dart
-------------

.. code-block:: dart

   var db = FirebaseFirestore.instance;
   
       var collection = await db.collection("users").doc(widget.userId).get();
   
       var data = collection.data();
   
       if (data == null ) {
         return; 
       }
   
       if (data.containsKey("xpLvl")) {
         setState(() {
          
           currentXpOverall = data["xpLvl"];
           print(currentXpOverall);
           currentLevel = XpInterface.getLevel(currentXpOverall);
   
           if (XpInterface.rankList[currentLevel] == "Emerald") {
             currentLevelProgress = currentXpOverall - prevLevelMax;
           }
           else {
             currentLevelMax = XpInterface.rankThesholds[currentLevel];
             if (currentLevel > 0) {
               prevLevelMax = XpInterface.rankThesholds[currentLevel - 1];
             }
             else {
               prevLevelMax = 0;
             }
             
             currentLevelProgress = currentXpOverall - prevLevelMax;

The variables are retrieved from the Firestore in a format that the application can use in other functions. ``xpLevel`` and ``users`` are used frequently from here in other applications, typically with ``users.[variable]``.

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
     create: (context) => ThemeNotifier(),
     runApp(const MyApp());
   }
