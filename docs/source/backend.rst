Backend
=====

.. _Login Page:

LoginPage.dart
------------

The first interactable page after the splash page. This page is managed by Firebase and its corresponding database.

The login page has to interact with the database and actively monitor the input fields in a stateful widget (mutable).
.. code-block:: console

   class _LoginPageState extends State<LoginPage> {
        final FirebaseAuth _auth = FirebaseAuth.instance;
        final TextEditingController _emailController = TextEditingController();
        final TextEditingController _passwordController = TextEditingController();
        final _formKey = GlobalKey<FormState>();

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

