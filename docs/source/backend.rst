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
