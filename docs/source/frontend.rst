Frontend
========

.. _Authorisation Page Form:

AuthPageForm.dart
-----------------

.. code-block:: dart
   class _AuthPageFormState extends State<AuthPageForm> {
   
     bool _error = false;

While this page is primarily built by the frontend, there is the backend variable ``bool_error`` that tracks if an error has occured on the page for the backend to handle. There are multiple instances of this to make sure the system catches any errors.
