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

An instance of ``FirebaseAuth`` is created that handles the authentication of the user inputs via the Firebase API. 

The ``_emailController`` and ``_passwordController`` handle the text input as well as retrieve them for the authentication instance.

Finally, a ``GlobalKey`` is made to uniquely identify the form and ensure that instance is authenticated once.


Creating recipes
----------------

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

