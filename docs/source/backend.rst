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

To retrieve a list of random ingredients,
you can use the ``lumache.get_random_ingredients()`` function:

.. autofunction:: lumache.get_random_ingredients

The ``kind`` parameter should be either ``"meat"``, ``"fish"``,
or ``"veggies"``. Otherwise, :py:func:`lumache.get_random_ingredients`
will raise an exception.

.. autoexception:: lumache.InvalidKindError

For example:

>>> import lumache
>>> lumache.get_random_ingredients()
['shells', 'gorgonzola', 'parsley']

Yeah
----------------
i was here - up2112135

