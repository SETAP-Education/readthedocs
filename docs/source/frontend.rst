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
