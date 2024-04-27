Frontend
========

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
