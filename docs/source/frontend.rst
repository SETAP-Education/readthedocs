Frontend
========

Overview
========
The front end of this application has been standardised across pages with a clear design focus. This doc will cover some of the front end choices and inner workings, but in terms of what you can expect to see throughout the app, you can find:

Fonts
-----
Google fonts: Nunito ver. 6.2.1

Colours
-------
Split into two themes (light and dark), we aimed to make the theme colours cohesive in both modes of use.

**Light**

Primary Colour: 0xFF19c37d   

Secondary Colour: 0xFF333333

Primary Container Colour: ARGB(255, 255, 255, 255)

Secondary Container Colour: ARGB(255, 231, 231, 231)

Error Colour: FF9800FF

Background Colour: ARGB(255, 230, 231, 236)

Text Colour: 0xFF333333

**Dark**

Primary Colour: 0xFF19c37d   

Secondary Colour: 0xFFE7E7E7

Primary Container Colour: 0xFF202226

Secondary Container Colour: ARGB(255, 65, 68, 74)

Error Colour: FF9800FF

Background Colour: 0xFF131517

Text Colour: 0xFFE7E7E7

Iconography
-----------


Pages
=====
.. _Authorisation Page Form:

AuthPageForm.dart
-----------------

.. code-block:: dart

   LayoutBuilder(
                      builder: (BuildContext context, BoxConstraints constraints) {
                        return Container(
                          padding: const EdgeInsets.all(20.0),
                          decoration: BoxDecoration(
                           
                            borderRadius: BorderRadius.circular(15.0),
                          ),
                          child: _buildFormBase(),

This section of the code is responsible for setting out the constraints of the authorisation form that users will fill out. Its size changes depending on the values the parent passes into it. ``_buildFormBase()`` returns the form elements that is displayed in the container.

.. code-block:: dart

   child: SizedBox(
                   width: 400,
                   height: 400,
                   child: Center(
                     child: Image.asset(
                       'images/quiz_app_logo_2.png',
                       width: 400,
                       height: 400,
                     ),
                   ),
                 ),
               ),
               Padding(
                 padding: EdgeInsets.only(left: 50, top: 50),
                 child: widget.child

Here the layout for the image is placed into a ``SizedBox`` to maintain aspect ratio at 400 x 400 px. ``widget.child`` represents the layout for the form fields, adding insets that place it away from the image child. This function prepares the cohesive layout of the splash, registration and login forms that allows them to be filled with their relevant fields.

.. _Display Name Page:

DisplayNamePage.dart
--------------------

This page is mainly responsible for handling the adjustment of the user's display name and selected interests.

.. code-block:: dart

   Text("What should we call you?",
                                 style: GoogleFonts.nunito(
                                     fontSize: 26, fontWeight: FontWeight.w600))
                           ],
                         ),
                     ),
                   SizedBox(height: 20),
                   Center(
                     child: Container(
                       width: 600,
                       child: TextFormField(
                         controller: _nameController,
                         decoration: InputDecoration(
                           labelText: 'Display Name',

Text input is decorated and uniform, using the Google font ``nunito`` across the application. Cohesive font sizes and weights are also used across the application.

.. code-block:: dart
   
   SizedBox(height: 20),
                       Center(
                         child: Button(
                           width: 400,
                           important: true,
                           onClick: () {
                             // Get the entered display name
                             String displayName = _nameController.text.trim();
   
                             // Check if display name or interests are empty
                             if (displayName.isEmpty) {
                                 // Add an error message to the error manager
                                 print("No display name");
                                 globalErrorManager.pushError('Display name cannot be empty');
                             } else if (_selectedInterests.isEmpty) {
                                 // Add an error message to the error manager
                                 print("No interests");
                                 globalErrorManager.pushError('You must select at least one interest');
                             } else {
                                 // If there are no errors, proceed with setting the display name and interests
                                 _setDisplayName(_user!.uid, displayName);
                                 _saveInterests(_user!.uid, _selectedInterests);

Here is the majority of the error handling relating to user input is managed. The text field is checked with ``displayname.isEmpty`` and the button field for interests is checked with ``_selectedInterests.isEmpty``. Direct feedback is given back to the user and once the requirements are satisfied, the user is moved onto a diagnostic test (as a new user).

.. _Error Displayer Page:

ErrorDisplayer.dart
-------------------

.. code-block:: dart

   return Positioned(
           top: screenHeight * 0.01,
           left: screenWidth * 0.15,
           right: screenWidth * 0.15,
           child: Container(
             decoration: BoxDecoration(
               color: Colors.transparent,
             ),
             child: ListView.separated(
               physics: NeverScrollableScrollPhysics(), 
               shrinkWrap: true,
               itemCount: globalErrorManager.errors.length,
               separatorBuilder: (BuildContext context, int index) {
                 return SizedBox(height: 8);

The container that holds error messages is defined here with the sizes in ``top, left and right`` placing the error messages at the top of the screen. The background is ``Colors.transparent`` and when multiple errors occur, there is a gap of ``height: 8`` between them. Errors are added to an index and passed into the container.

.. code-block:: dart

   _timer = Timer(Duration(seconds: 4), () {
                   setState(() {
                     if (globalErrorManager.errors.length > index) {
                       globalErrorManager.errors.removeAt(index);

An internal timer exists that makes the container and held error message remain on screen for ``4 seconds``. There is also the code snippet below that dismisses the error message through user gesture ``onTap()``.

.. code-block:: dart

   MouseRegion(
                         cursor: SystemMouseCursors.click,
                         child: GestureDetector(
                           onTap: () {
                             setState(() {
                               globalErrorManager.errors.removeAt(index);


.. _Login Page:

LoginPage.dart
--------------------

.. code-block:: dart
   
   child: TextFormField(
                   controller: _passwordController,
                   obscureText: !_showPassword, // Correct placement of obscureText
                   decoration: InputDecoration(
                     labelText: 'Password',
                     contentPadding: EdgeInsets.symmetric(horizontal: 20, vertical: 15),
                     focusedBorder: OutlineInputBorder(
                       borderSide: BorderSide(color: textColour),
                       borderRadius: BorderRadius.circular(30.0),
                     ),
                     enabledBorder: OutlineInputBorder(
                       borderSide: BorderSide(color: textColour),
                       borderRadius: BorderRadius.circular(30.0),
                     ),
                     suffixIcon: Padding(
                       padding: EdgeInsets.only(right: 8.0), // Adjust the padding as needed
                       child: IconButton(
                         icon: Icon(
                           _showPassword ? Icons.visibility_off : Icons.visibility,
                         ),
                         color: textColour,
                         onPressed: () {
                           setState(() {
                             _showPassword = !_showPassword;

While much of the login page is handled by Firebase and the cloud authentication system, elements of the UI have had modifications applied for security and privacy. Here ``obscureText`` is set by default, hiding the input for the password from the user when it's typed in. The ``sufficIcon`` widget then defines a visibility icon on the input line that ``onPressed`` will ``_showPassword``, setting ``obscureText`` to false.

.. code-block:: dart

   child: GestureDetector(
                     onTap: () async {
                       final email = _emailController.text.trim();
                       if (email.isNotEmpty) {
                         try {
                           var user = await FirebaseAuth.instance.fetchSignInMethodsForEmail(email);
                           if (user.isNotEmpty) {
                             await FirebaseAuth.instance.sendPasswordResetEmail(email: email);
                             
                             globalErrorManager.pushError("Password reset email sent to $email real");
                               
                             
                           } else {
   
                             globalErrorManager.pushError("Password reset email sent to $email not real");
                             
                           }
                         } catch (e) {
                           print('Error: $e');
                         }
                       } else {
                           globalErrorManager.pushError("Please enter an email");

More email interaction is defined here. When the email field ``isNotEmpty``, the backend will trim any spaces and send it to the backend for authentication that the email exists. If it matches, the email will be sent and a message shown to feedback that. If the email doesn't exist in the Firestore, an error is thrown that the email doesn't exists and nothing happens. The same occurs when nothing is input in the text field. The error manager can also give feedback to the user that an email has been sent through the error labelling system, even if nothing has produced an error.

.. _Registration Page:

RegistrationPage.dart
---------------------

The registration page is functionally the same in terms of widgets and visual design, with one big exception.

.. code-block:: dart
   
   Widget _buildPasswordRequirement(String text, bool satisfied) {
       return Row(
         children: [
           satisfied ? Icon(Icons.done, color: Colors.green,) : Icon(Icons.close, color: Colors.red),
           const SizedBox(width: 8.0),
           Text(text, style: GoogleFonts.nunito(color: satisfied ? Colors.grey : Theme.of(context).textTheme.bodyMedium!.color!, fontSize: 18, decoration: satisfied ?  TextDecoration.lineThrough : TextDecoration.none),)
         ]

While the user is inputting text in the password field, ``_buildPasswordRequirement`` listens and updates according to conditions set out in the backend. The widget initialises two conditions and icons with a red cross. When a condition is satisfied, it updates to a green check mark. Once both are satisfied and there are no errors in formatting and matching passwords, the account can be registered.

.. code-block:: dart

   child: Column(
           crossAxisAlignment: CrossAxisAlignment.start,
           children: [
             _buildPasswordRequirement("Minimum of 8 characters", satisfysMinCharacters),
             _buildPasswordRequirement("Contains a number", hasOneNumber)



.. _Quiz Summary Page:

QuizSummaryPage.dart
---------------------

.. code-block:: dart

   SizedBox(height: 16.0),
                 buildQuizResults(quizAttemptData, context),
                 for (int i = 0; i < loadedQuestions.length; i++)
                   FractionallySizedBox(
                     widthFactor: 2 / 3,
                     child: Container(
                       margin: EdgeInsets.only(bottom: 16.0),
                      
                       decoration: BoxDecoration(
                          color: Theme.of(context).colorScheme.primaryContainer,
                          borderRadius: BorderRadius.circular(24),
                       ),
                       child: Padding(
                         padding: const EdgeInsets.all(16.0),
                         child: loadedQuestions.isNotEmpty
                             ? buildQuizSummaryItem(loadedQuestions[i], i, quizAttemptData)

The variables retrieved from the backend are passed to the frontend to build the widgets in accordance to the ``loadedQuestions``, adding a widget for each question and including the user attempt inside.

.. code-block:: dart

   RichText(text: TextSpan(
                   text: "$userTotal",
                   style: GoogleFonts.nunito(fontSize: 22.0, color: Theme.of(context).textTheme.bodyMedium!.color),
                   children: [
                     TextSpan(
                       text: " / $quizTotal",
                       style: GoogleFonts.nunito(fontSize: 16.0, color: Theme.of(context).textTheme.bodyMedium!.color!.withOpacity(0.5))
                     ),
                     TextSpan(
                       text: " answered correctly",
                       style: GoogleFonts.nunito(fontSize: 20.0, color: Theme.of(context).textTheme.bodyMedium!.color)
                     )
                   ]
                 )),
   
                 Text("${(userTotal / quizTotal) * 100}%", style: GoogleFonts.nunito(fontSize: 32.0, fontStyle: FontStyle.italic, fontWeight: FontWeight.bold)),
   
                 Row(
                   mainAxisAlignment: MainAxisAlignment.center,
                   crossAxisAlignment: CrossAxisAlignment.center,
                   children: [
                     Text("You earned:  ", style: GoogleFonts.nunito(fontSize: 22)),
                     Text("${earnedXp}xp" , style: GoogleFonts.nunito(fontSize: 28, color: Theme.of(context).colorScheme.primary, fontStyle: FontStyle.italic, fontWeight:       FontWeight.bold))

The metadata about the quiz results are displayed here including ``userTotal`` (amount of questions user got right), ``quizTotal`` (amount of questions) and ``earnedXP`` (xp earned for each right question). This is displayed at the top of the page.

.. code-block:: dart

   return Column(
         crossAxisAlignment: CrossAxisAlignment.center,
         children: [
           SizedBox(height: 10),
           Text(
             question.questionText,
             style: GoogleFonts.nunito(fontSize: 20, fontWeight: FontWeight.bold),
           ),
           SizedBox(height: 20),
           if (question.type == QuestionType.multipleChoice)
             buildMultipleChoiceQuestion(question.answer as QuestionMultipleChoice, userResponse),
           if (question.type == QuestionType.fillInTheBlank)
             buildFillInTheBlankQuestion(question.answer as QuestionFillInTheBlank, userResponse),

This is the actual containers that hold the question and answer responses. It will be built differently depending on whether the question type is ``multipleChoice`` or ``fillInTheBlank``.

.. code-block:: dart
   
   return ListView.builder(
         shrinkWrap: true,
         physics: NeverScrollableScrollPhysics(),
         itemCount: question.options.length,
         itemBuilder: (context, index) {
           String option = question.options[index];
           bool isSelected = userResponse.contains(index);
           bool isCorrect = question.correctAnswers.contains(index);
   
           Color backgroundColour = isSelected
               ? (isSelected && isCorrect ? Colors.green : Colors.red)
               : Colors.transparent;
   
           Color borderColour = isSelected
               ? (isSelected && isCorrect ? Colors.green : Colors.red)
               : (isCorrect ? Colors.green : Colors.grey)

Here the code defines the marking criteria for multiple choice answers. When a correct answer is selected, its outline and background colour fills to green. When a wrong answer is selected, it's filled and outlined to red while the correct answer is outlined green. When there was no option selected, the correct option is outlined green while the rest are grey.


.. code-block:: dart

   Widget buildFillInTheBlankQuestion(QuestionFillInTheBlank question, String userResponse) {
       print("The user response: ${userResponse}, The correct response: ${question.correctAnswer}");
       print("FITB USER RESPONSE: $userResponse");
   
       Color backgroundColour = userResponse.isEmpty
           ? Colors.transparent
           : (userResponse.toLowerCase() == question.correctAnswer.toLowerCase())
               ? Colors.green
               : Colors.red;
   
       Color borderColour = userResponse.isEmpty
           ? Colors.blue
           : (userResponse.toLowerCase() == question.correctAnswer.toLowerCase())
               ? Colors.green
               : Colors.red;

The same principle is applied here, but for the ``fillInTheBlank`` question type.

.. code-block:: dart
   
   child: Center(
           child: Text(
             userResponse.isEmpty
                 ? 'Not answered - The correct Answer is: "${question.correctAnswer}"'
                 : userResponse.toLowerCase() == question.correctAnswer.toLowerCase()
                     ? 'Correct! Your answer: ${userResponse} ✓'
                     : 'Incorrect. Your answer: ${userResponse} ✘ | The correct Answer is: "${question.correctAnswer}"',

Below the fill in the blank question, this widget takes the question and attempt data to give feedback on responses.



.. _Quiz Page:

QuizPage.dart
-------------

.. code-block:: dart

   if (currentQuestionIndex > 0)
                         Padding(
                           padding: const EdgeInsets.only(left: 550),
                           child: IconButton(
                             // color: tertiary,
                             // hoverColor: secondary,
                             icon: Icon(Icons.arrow_left, color: Theme.of(context).colorScheme.primary,),
                             tooltip: 'Previous question',
                             onPressed: () {
                               if (currentQuestionIndex > 0) {
                                 // If there is a previous question, move to it
                                 currentQuestionIndex--;
                                 displayQuestion(currentQuestionIndex, quiz.questionIds);
                                 setState(() {
                                   quizCompleted = false;
                                 });

Here is the button code for navigating through the quiz, in this case, a previous question. ``if (currentQuestionIndex > 0)`` i.e. any question number aside from the first, it will decrement the question index and change the ui contents to the question before. It will also set the ``quizCompleted`` state to false (more important for the last question in set).

.. code-block:: dart

   else
                         SizedBox(width: 48), // Add a placeholder SizedBox when the condition is false
                       Padding(
                         padding: const EdgeInsets.only(right: 550),
                         child: IconButton(
                           // color: tertiary,
                           // hoverColor: secondary,
                           icon: Icon(Icons.arrow_right, color: Theme.of(context).colorScheme.primary),
                           tooltip: currentQuestionIndex < loadedQuestions.length - 1
                               ? 'Next Question'
                               : 'Submit Quiz',
                           onPressed: () async {
                             if (currentQuestionIndex == loadedQuestions.length) {
                               // If there are more questions, store user answers in Firebase
                               print("Just before storing the userSummary: $userSummary");
                               // await storeUserAnswersInFirebase2(userSummary);
                             }
                             moveToNextOrSubmit();

These buttons control loading the next question as well as the ``Submit Quiz`` button. This is handled by the ``moveToNextOrSubmit`` function in the backend of this page.

.. code-block:: dart

   Widget buildQuizPage(QuizQuestion question) {
       return Column(
         crossAxisAlignment: CrossAxisAlignment.center,
         children: [
           SizedBox(height: 20),
           Text(
             question.questionText,
             style: TextStyle(fontSize: 30, fontWeight: FontWeight.bold),
             textAlign: TextAlign.center,
           ),
           SizedBox(height: 45),
           //This is where the question will be asked / written to the page. The question format for posing the question is universal for all question types thus doesn't need to be type specific.
   
           if (question.type == QuestionType.multipleChoice)
             buildMultipleChoiceQuestion(question.answer as QuestionMultipleChoice),
           if (question.type == QuestionType.fillInTheBlank)
             buildFillInTheBlankQuestion(question.answer as QuestionFillInTheBlank, question.key),

The basic quiz page structure is set out where the question and user input will be laid out. This differs depending on the question type.

.. code-block:: dart

   return InkWell(
             onTap: () {
               setState(() {
                 if (isSelected) {
                   question.selectedOptions.remove(index);
                 } else {
                   question.selectedOptions.add(index);
                 }
               });
             },
             child: Container(
               padding: EdgeInsets.all(10),
               margin: EdgeInsets.symmetric(vertical: 8, horizontal: 100),
               decoration: BoxDecoration(
                 color: isSelected ? Colors.blue : Colors.white,
                 borderRadius: BorderRadius.circular(20),
                 border: Border.all(
                   color: Colors.blue,
                   width: 1,

The ``multipleChoice`` question type makes use of selectable ``InkWell`` and ``Container`` widgets that hold the question answers that users can select.

.. code-block:: dart

   child: TextField(
           controller: question.controller,
           key: key,
           onChanged: (text) {
             setState(() {
               question.userResponse = text;
             });
           },
           decoration: InputDecoration(
             hintText: "Enter your answer here",
             border: OutlineInputBorder(
               borderRadius: BorderRadius.circular(25),
             ),
             filled: true,
             fillColor: primaryColour,

The ``fillInTheBlank`` question type uses a ``TextField`` widget that can then be retrieved and tested against the answer stored in the database.

.. _Landing Page:

LandingPage.dart
----------------

.. code-block:: dart

   Container(
                                               decoration: BoxDecoration(
                                                 borderRadius: BorderRadius.circular(12),
                                                 color: Theme.of(context).colorScheme.primary
                                               ),
                                               padding: EdgeInsets.symmetric(horizontal: 20.0, vertical: 6.0),
                                               child: Text("Review", style: GoogleFonts.nunito(fontSize: 20, fontWeight: FontWeight.w800))
                                             ),
                                             SizedBox(height: 12.0),
                                             Text("Take a review of all topics and difficulties to see how much you've improved!", style: GoogleFonts.nunito(fontSize: 18, fontWeight: FontWeight.bold)),
                                             SizedBox(height: 6.0),
                                             Text("8 Questions • ${userInterests.toString().substring(1, userInterests.toString().length - 1)}", style: GoogleFonts.nunito(fontSize: 14, fontWeight: FontWeight.bold, color: Theme.of(context).colorScheme.primary))

At the top of the landing page, a banner is generated that allows the user to take a special quiz of ``8 questions`` in length comprised of their ``userInterests``. It's generated the same as any other quiz in the backend and labelled ``Review``.

.. code-block:: dart
   
   Text(
                                       'Pick a topic to begin a quiz!',
                                       style: GoogleFonts.nunito(fontSize: 18),
                                     ),
   
                                     const SizedBox(height: 20),
                                     FutureBuilder<List<String>>(
                                       future: Future.value(userInterests),
                                       builder: (context, snapshot) {
                                         if (snapshot.connectionState == ConnectionState.waiting) {
                                           return Center(child: CircularProgressIndicator());
                                         } else if (snapshot.hasError) {
                                           return Center(child: Text('Error loading interests'));
                                         } else {
                                           List<String> interests = snapshot.data ?? [];
                                           int numInterests = interests.length;
                                           int numInterestsPerRow = 4; // Adjust the number of interests per row as needed
                                           int numRows = (numInterests / numInterestsPerRow).ceil();
                                           List<Widget> rows = List.generate(numRows, (rowIndex) {
                                             List<Widget> rowChildren = [];
                                             for (int i = 0; i < numInterestsPerRow; i++) {
                                               int index = rowIndex * numInterestsPerRow + i;
                                               const SizedBox(height: 10);
                                               if (index < numInterests) {
                                                 rowChildren.add(
                                                   Flexible(
                                                     child: Padding(
                                                       padding: const EdgeInsets.symmetric(horizontal: 10.0, vertical: 10),
                                                       child: InkWell(
                                                         onTap: () async {
                                                             print('Interest ${index + 1}: ${interests[index]} pressed');
   
                                                             // Generate a new quiz
                                                             String id = await quizManager.generateQuiz([ interests[index] ], xpLevel, 20, 5);
                                                             
                                                             Navigator.push(context, MaterialPageRoute(builder:(context) {
                                                               return QuizPage(quizId: id);

The ``interests`` container is created here, with the user's selected interests being retrieved with ``snapshot.data``. The interests are sorted in a grid and placed in ``InkWell`` containers. Selecting one of these interests will ``generateQuiz`` of said interests at the index selected and move the user to the ``QuizPage``. This structure is exactly the same for "other interests".


.. _Splash Page:

SplashPage.dart
-------------

.. code-block:: dart


   Column (
                   mainAxisAlignment: MainAxisAlignment.center,
                   crossAxisAlignment: CrossAxisAlignment.start,
                   children: [
                     Text("Welcome to", style: GoogleFonts.nunito(
                           fontSize: 30.0,
                           fontWeight: FontWeight.bold,
                           fontStyle: FontStyle.italic,
                           color: secondaryColour
                         )),
   
                     Text(
                       'Quizzical 🎓!',
                       style: GoogleFonts.nunito(
                         fontSize: 60.0,
                         fontWeight: FontWeight.bold,
                         color: secondaryColour
                       ),
                     ),
                     SizedBox(height: 10),
   
                      Text(
                       'Learning doesn\'t have to be boring!',
                       style: GoogleFonts.nunito(
                         fontSize: 24.0,
                         fontWeight: FontWeight.w600,
                         fontStyle: FontStyle.italic
                       ),

Makes up the large splash text that the user is shown upon opening the application for the first time. This page is mainly populated by other files such as ``RegistrationPage.dart`` and ``LoginPage.dart``.

.. _App Theme:

AppTheme.dart
-------------

.. code-block:: dart

   class AppTheme {
   
     static ThemeData lightTheme = ThemeData(
       colorScheme: const ColorScheme.light(
         background: Colors.transparent,
         primary: Color(0xFF19c37d),
         secondary: Color(0xFF333333),
         primaryContainer: Color.fromARGB(255, 255, 255, 255),
         secondaryContainer: Color.fromARGB(255, 231, 231, 231),
         error: Colors.orange,
       ),
        scaffoldBackgroundColor: Color.fromARGB(255, 230, 231, 236),
       textTheme: const TextTheme(
         bodyMedium: TextStyle(color: Color(0xFF333333)),
       ),
       textSelectionTheme: TextSelectionThemeData(
         cursorColor: Colors.blue,

The app theme is defined here using ``hexadecimal`` and ``ARGB``. These can easily be adjusted and changed to change UI colour schemes throughout the app. Here it's defined for light mode, another definition is made for dark mode.

.. code-block:: dart

   static TextStyle defaultBodyText(BuildContext context) {
       return GoogleFonts.nunito( 
         fontSize: 18,
         fontWeight: FontWeight.w300,
         letterSpacing: -0.5,
         color: Theme.of(context).colorScheme.secondary,
       );

Much like colour theming, text styles can be defined and used throughout the application.

.. code-block:: dart
   
   static AppBar buildAppBar(BuildContext context, String title, bool includeTitleAndIcons, bool autoImply, String dialogTitle, Text contentText) {//, bool automaticallyImplyLeading) {
       // Get the current theme
       ThemeNotifier themeNotifier = Provider.of<ThemeNotifier>(context, listen: false);
   
       // Define icons for light and dark mode
       Icon lightModeIcon = Icon(Icons.light_mode_outlined, color: Theme.of(context).colorScheme.secondary);
       Icon darkModeIcon = Icon(Icons.dark_mode_outlined, color: Theme.of(context).colorScheme.secondary);
   
       // Determine the current icon based on the theme
       Icon currentIcon = themeNotifier.isDarkMode ? lightModeIcon : darkModeIcon;

Here an appbar holds the toggle button for light/dark mode. The current theme is held by the theme ``themeNotifier``, the backend the notifies the app of the active theme and updates when toggled.

.. code-block:: dart

   static ElevatedButton buildElevatedButton({
       required VoidCallback onPressed,
       required String buttonText,
       BuildContext? context,
     }) {
       return ElevatedButton(
         onPressed: onPressed,
         style: ElevatedButton.styleFrom(
           backgroundColor: Color(0xFF19c37d),
           shape: RoundedRectangleBorder(
             borderRadius: BorderRadius.circular(15.0), // 15 for rounded edges, 5 for curved corners
           ),
           // Add other button style configurations as needed
         ),
         child: Text(
           buttonText,
           style: context != null ? AppTheme.defaultBodyText(context) : null,

The button shape and style is defined with ``RoundedRectangleBorder`` and ``backgroundColor`` dictating the shape and colour of various buttons across the app.

.. _Button:

Button.dart
-----------

``Button.dart`` is a dedicated file that is used to define the design of a button to more specific constraints. This then gets passed to functions that use UI elements to prevent duplication of code.

.. code-block:: dart

   class Button extends StatelessWidget {
   
     const Button({ super.key, this.onClick, this.child, this.important = false, this.width = double.infinity });
   
     final bool important;
     final Function? onClick; 
     final Widget? child; 
     final double width;

The variables the button widget is concerned with is defined here, mostly related to generic functions a button would require such as ``Function?`` (action to perform on click) and ``Widget?`` which gets passed by the function the Button is being used by.
