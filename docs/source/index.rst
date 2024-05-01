Welcome to Quizzical - A tag-based quizzing application
=======================================================


Building
--------
Make sure you have an up to date Flutter SDK. 

First begin by cloning the repo 
`git clone https://github.com/SETAP-Education/EducationApp.git`

Once this has been done open the terminal in the project sub directory of `education_app`.

In this directory if its the first time setup run. 
`flutter pub get` to retrieve all packages. If packages are out of date, use 

.. code-block:: dart

   flutter pub upgrade

Then you can run `flutter run -d chrome` to run the project in chrome. 

Dependencies
------------
These are the associated packages that the application interacts with. These will automatically be installed and applied upon running ``pubspec.yaml``. If you are building from the files, you need to make sure your Flutter SDK is up to date and linked to your IDE.

flutter:

sdk: flutter

cupertino_icons: 1.0.8

firebase_core: 2.30.0

firebase_auth: 4.19.2

cloud_firestore: 4.17.0

google_fonts: 6.2.1

provider: ^6.1.2

fake_cloud_firestore: ^2.5.1

firebase_auth_mocks: ^0.13.0

Contents
--------

.. toctree::

   backend
   frontend
