# dynamic-Firebase-configuration
Here's a step-by-step guide to set up a React Native project with dynamic Firebase configuration for multiple Firebase projects


Step 1: Install Required Libraries Install the core Firebase library for React Native:

npm install @react-native-firebase/app Install additional Firebase modules for the features you need: Firestore:

npm install @react-native-firebase/firestore Authentication:

npm install @react-native-firebase/auth Messaging (FCM):

npm install @react-native-firebase/messaging Step 2: Prepare Firebase Configuration Files You need configuration files for both Android and iOS for each project.

For Android: Download google-services.json for each Firebase project:

Go to your Firebase console > Project settings > Download google-services.json. Save them in a custom directory inside your project:

android/app/firebase-configs/ Example:

google-services-IN.json google-services-EE.json For iOS: Download GoogleService-Info.plist for each Firebase project:

Go to your Firebase console > Project settings > Download GoogleService-Info.plist. Save them in a custom directory inside your project:

ios/firebase-configs/ Example:

GoogleService-Info-IN.plist GoogleService-Info-EE.plist Step 3: Disable Automatic Firebase Initialization You must disable automatic Firebase initialization to allow dynamic configuration.

For Android: Open android/app/build.gradle. Add the following block to disable automatic initialization: gradle

firebase { enableAutomaticResourceManagement = false } For iOS: Open AppDelegate.m or AppDelegate.swift. Remove or comment out the line: objc

[FIRApp configure]; Step 4: Create a Dynamic Firebase Initialization Function

Create a helper file (e.g., firebaseSetup.js) to handle dynamic Firebase initialization.

import { initializeApp, getApps, deleteApp } from '@react-native-firebase/app'; import messaging from '@react-native-firebase/messaging'; import { Platform } from 'react-native';

// Firebase configurations for each project const firebaseConfigIN = Platform.select({ ios: require('./ios/firebase-configs/GoogleService-Info-IN.plist'), android: require('./android/app/firebase-configs/google-services-IN.json'), });

const firebaseConfigEE = Platform.select({ ios: require('./ios/firebase-configs/GoogleService-Info-EE.plist'), android: require('./android/app/firebase-configs/google-services-EE.json'), });

// Function to initialize Firebase export const configureFirebase = async (project) => { const apps = getApps();

// Delete any existing Firebase app if (apps.length > 0) { await deleteApp(apps[0]); }

// Choose the appropriate configuration const config = project === 'IN' ? firebaseConfigIN : firebaseConfigEE;

// Initialize Firebase with the selected configuration const firebaseApp = initializeApp(config);

// Configure messaging messaging().setBackgroundMessageHandler(async (remoteMessage) => { console.log('Message handled in the background:', remoteMessage); });

return firebaseApp; };

Step 5: Add Buttons for Switching Firebase Projects In your main React Native component, add buttons to let users switch between Firebase projects.

import React, { useState } from 'react'; import { View, Button, Text } from 'react-native'; import { configureFirebase } from './firebaseSetup';

const App = () => { const [selectedProject, setSelectedProject] = useState(null);

const handleProjectSelection = async (project) => { setSelectedProject(project); await configureFirebase(project); console.log(${project} Firebase project configured); };

return ( <View style={{ padding: 20 }}> <Button title="Select IN Project" onPress={() => handleProjectSelection('IN')} /> <Button title="Select EE Project" onPress={() => handleProjectSelection('EE')} /> <Text style={{ marginTop: 20 }}>Selected Project: {selectedProject}</Text> </View> ); };

export default App;

Step 6: Use Firebase Services Once Firebase is dynamically configured, you can use any Firebase service.

Authentication Example:

import auth from '@react-native-firebase/auth';

const signIn = async (email, password) => { try { const userCredential = await auth().signInWithEmailAndPassword(email, password); console.log('User signed in:', userCredential.user); } catch (error) { console.error('Error signing in:', error); } };

Firestore Example:

import firestore from '@react-native-firebase/firestore';

const fetchData = async () => { try { const snapshot = await firestore().collection('your-collection').get(); const data = snapshot.docs.map(doc => doc.data()); console.log('Fetched data:', data); } catch (error) { console.error('Error fetching Firestore data:', error); } };

Messaging Example:

import messaging from '@react-native-firebase/messaging';

const setupNotifications = async () => { try { const token = await messaging().getToken(); console.log('FCM Token:', token);

`messaging().onMessage(async (remoteMessage) => {`
  `console.log('Foreground message:', remoteMessage);`
`});`
} catch (error) { console.error('Error setting up messaging:', error); } };

Step 7: Test Your Setup Run your app and click on the buttons to switch between Firebase projects. Verify that: Authentication works with the selected project. Firestore reads/writes data from/to the selected project. Notifications are routed through the selected project's configuration.

Tips for Scaling: Add More Projects: Add additional google-services.json and GoogleService-Info.plist files for new projects and update your firebaseSetup.js file. Environment Variables: Use .env files or a config file to manage the list of Firebase projects and their identifiers dynamically. With this setup, you can easily switch between multiple Firebase projects in your React Native app while fully utilizing all Firebase services.

