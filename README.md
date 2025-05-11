![Maven Central Version](https://img.shields.io/maven-central/v/io.github.dwarshb/firebase-cmp)
![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/dwarshb/firebase-cmp/publish.yml)
![GitHub top language](https://img.shields.io/github/languages/top/dwarshb/firebase-cmp)

# Firebase-CMP 🔥
A Compose Multiplatform library that leverages Firebase Rest API to integrate features like Firebase Authentication, Firebase RealtimeDatabase and Gemini in multiplatform application targeting Android, iOS, Desktop & Web.

### Why Firebase REST API for Authentication ?
We have seen that there are multiple Firebase SDK available for Android, iOS & Web Platform, but there is no stable SDK for Compose Multiplatform. And I have seen various examples of Compose Multiplatform that uses REST API to showcase their use cases. So to use a single code base and target multiple platform, I preferred to use Firebase REST API 

## 📦 Get Started
### 1. Add dependency in build.gradle
```gradle
commonMain {
  implementation("io.github.dwarshb:firebase-cmp:1.0.1")
  //Ktor Dependencies
}
```
> #### :red_circle: Important - Make sure to add Ktor dependencies as well in your project. Check https://ktor.io/docs/client-create-multiplatform-application.html#code

Once the dependency is added, Sync the project

---
### 2. Setup Firebase project
Before moving on to development, you will need to setup Firebase project and enable the required features:

✅ To use email/password authentication you need to Enable Firebase Authentication feature and turn on Email/Password toggle.

✅ To use Firebase Realtime Database feature, you need to enable the feature and **copy the Database URL**.
  For example : `https://YOUR-DATABASE.firebaseio.com/`
  
✅ To use Gemini feature, you need to **copy the API Key** available in  Gemini Dashboard. Or you can find it in https://aistudio.google.com/

#### 📽️ You can check below video to learn more about Firebase Setup:

[![Firebase Configuration](https://img.youtube.com/vi/bdmw_jD6T7M/0.jpg)](https://youtu.be/bdmw_jD6T7M)

Once the Firebase Project is configured, you can visit the project settings to get API Key that will be used by the library.

![image](https://github.com/user-attachments/assets/6d1723ec-e868-463a-a6dd-9982517aa96b)


---
### 3. Initialize the Library

<blockquote>Note : Replace the below credentials with yours</blockquote>

```kotlin
val firebaseAPIKey : MutableState<String> = remember {  mutableStateOf("FIREBASE_API_KEY")}
val firebaseDatabaseUrl = remember {  mutableStateOf("https://myproject.firebaseio.com/") }
val STORAGE_URL = "https://firebasestorage.googleapis.com/v0/b/YOUR_PROJECT_ID.appspot.com/o"
val geminiKey = remember { mutableStateOf("YOUR_GEMINI_API_KEY") }
```
Initialize the Firebase and pass the required credentials
```kotlin
val firebase = Firebase()
firebase.initialize(
    apiKey = firebaseAPIKey.value,
    databaseUrl = firebaseDatabaseUrl.value,
    storageUrl = STORAGE_URL)
```

Gemini.kt file is part of this library so if you want to use Gemini, you need to pass model and API Key as given in below example
```kotlin
var geminiModel : MutableState<String> = remember { mutableStateOf("gemini-2.0-flash") }
firebase.setGemini(geminiModel.value,geminiKey.value)
```
---

### 4. Functionality
#### 4.1 Firebase Authentication
```kotlin
//Initialize FirebaseAuth
var firebaseAuth = FirebaseAuth()

//Create user with email and password
suspend fun signUp(
        email: String,
        password: String,
        confirmPassword: String,
        completion: onCompletion<String>
    ) {
        if (password == confirmPassword) {
                firebaseAuth.signUpWithEmailAndPassword(email,password, object : onCompletion<AuthResponse> {
                    override fun onSuccess(T: AuthResponse) {
                        //User created successfully
                        // AuthResponse contains T.localId,T.email,T.refreshToken
                        completion.onSuccess(T.idToken)
                    }

                    override fun onError(e: Exception) {
                        completion.onError(e)
                    }
                })
        } else {
            completion.onError(Exception("Password doesn't match"))
        }
    }

//Login existing user with email and password
suspend fun login(
        email: String,
        password: String,
        completion: onCompletion<String>
    ) {
            firebaseAuth.login(email,password,object : onCompletion<AuthResponse> {
                override fun onSuccess(T: AuthResponse) {
                    //User logged in successfully
                    // AuthResponse contains T.localId,T.email,T.refreshToken
                    completion.onSuccess(T.idToken)
                }
                override fun onError(e: Exception) {
                    completion.onError(e)
                }
            })
    }

```
<hr/>

#### 4.2 Firebase Realtime Database
```kotlin
//Initialize Firebase Database
var firebaseDatabase = FirebaseDatabase()

// To read data from Firebase Realtime Database user readFirebaseDatabase() function
// For example
suspend fun readFromDatabase(userId: String,onCompletion: onCompletion<String>) {
    
    val childPath = listOf("chats",userId,"messages") //For path="chats/userId/messages"
    
    val query = ""
    
    firebaseDatabase.readFirebaseDatabase(childPath,query, 
        object : onCompletion<String> {
            override fun onSuccess(t: String) {
                //t is a response received in Json formatted String
                onCompletion.onSuccess(t)
            }

            override fun onError(e: Exception) {
                e.printStackTrace()
                onCompletion.onError(e)
            }
      })
}


//To write a data in Firebase Realtime Databse you can use the available methods like postFirebaseDatabase(), putFirebaseDatabase() or patchFirebaseDatabase() based on the request
suspend fun writeMessageToFirebase(message String, userId:String, onCompletion: onCompletion<String>) {
    
    val params = HashMap<String,Any>()
    params.put("message",message)
    params.put("userId",userId)
    params.put("id",Clock.System.now().toEpochMilliseconds()),

    val childPath = listOf("chats",userId,"messages") //For path="chats/userId/messages"
        
    firebaseDatabase.patchFirebaseDatabase(childPath, params,
        object : onCompletion<String> {
              override fun onSuccess(t: String) {
                  onCompletion.onSuccess(t)
              }

              override fun onError(e: Exception) {
                  e.printStackTrace()
                  onCompletion.onError(e)
              }
        })
}
```
#### 4.3 Gemini (WIP)
Currently Gemini class provides two method generatePrompt and Conversational AI
```kotlin
//Initialize Gemini
val gemini = Gemini()


gemini.generatePrompt("Hey, whats the whether today",
    object : onCompletion<String> {
        override fun onSuccess(response: String) {
            //Handle response
        }

        override fun onError(e: Exception) {
           e.printStackTrace()
            //Handle error
        }
    }
)

val systemInstructions = "You are Cat"

gemini.conversationalAI(systemInstructions,
    createContentFromConversation(messagesList),
    object : onCompletion<String> {
        override fun onSuccess(response: String) {
            //Handle Response from model
        }

        override fun onError(e: Exception) {
            e.printStackTrace()
        }
    }
)
```
For conversationalAI(), checkout full implementation [here in sample app.](https://github.com/dwarshb/chaitalk/blob/master/composeApp/src/commonMain/kotlin/com/dwarshb/chaitalk/chat/Chat.kt#L152)

---

### Reference:
https://ai.google.dev/gemini-api/docs/text-generation#supported-models

https://kmp.jetbrains.com/#newProject

https://firebase.google.com/docs/reference/rest/auth

https://firebase.google.com/docs/reference/rest/database

---


This is a Kotlin Multiplatform project targeting Android, iOS, Web, Desktop.

* `/composeApp` is for code that will be shared across your Compose Multiplatform applications.
  It contains several subfolders:
  - `commonMain` is for code that’s common for all targets.
  - Other folders are for Kotlin code that will be compiled for only the platform indicated in the folder name.
    For example, if you want to use Apple’s CoreCrypto for the iOS part of your Kotlin app,
    `iosMain` would be the right folder for such calls.

* `/iosApp` contains iOS applications. Even if you’re sharing your UI with Compose Multiplatform, 
  you need this entry point for your iOS app. This is also where you should add SwiftUI code for your project.


Learn more about [Kotlin Multiplatform](https://www.jetbrains.com/help/kotlin-multiplatform-dev/get-started.html),
[Compose Multiplatform](https://github.com/JetBrains/compose-multiplatform/#compose-multiplatform),
[Kotlin/Wasm](https://kotl.in/wasm/)…

We would appreciate your feedback on Compose/Web and Kotlin/Wasm in the public Slack channel [#compose-web](https://slack-chats.kotlinlang.org/c/compose-web).
If you face any issues, please report them on [GitHub](https://github.com/JetBrains/compose-multiplatform/issues).

You can open the web application by running the `:composeApp:wasmJsBrowserDevelopmentRun` Gradle task.

---
<div style="width:100%">
	<div style="width:50%; display:inline-block">
		<h2 align="center">
      :handshake: Open for Contribution
		</h2>	
	</div>	
</div>
