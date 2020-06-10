# Android’s Room in Kotlin ft. MVVM Architecture and Coroutines

If you’re looking for an explanation on **Room** implementation on Android using Kotlin and one of it’s Coroutine feature with MVVM architecture, then this one is for you.

Let’s see what I have in store for you.

  1.  What is Room, Kotlin, MVVM, Coroutines?
  2.  Advantages of Room over SQLite?
  3.  Important Annotation in Room.
  4.  Step-by-Step Simple Insert and Read Example
  5.  Conclusion

So let’s get started.

## 1. What is Room, Kotlin, MVVM, Coroutines?

**Answer:** Let's see what are the important concepts in ROOM and MVVM.

**Room**
**Room Database:** Database layer on top of SQLite database that takes care of mundane tasks that you used to handle with an SQLiteOpenHelper. Database holder that serves as an access point to the underlying SQLite database. The Room database uses the DAO to issue queries to the SQLite database.

**Entity:** When working with Architecture Components, this is an annotated class that describes a database table.

**SQLite database:** On the device, data is stored in an SQLite database. For simplicity, additional storage options, such as a web server, are omitted. The Room persistence library creates and maintains this database for you.

**DAO:** Data access object. A mapping of SQL queries to functions. You used to have to define these painstakingly in your SQLiteOpenHelper class. When you use a DAO, you call the methods, and Room takes care of the rest.

**Kotlin:** Kotlin is an open-source, statically-typed programming language that supports both object-oriented and functional programming. Kotlin provides similar syntax and concepts from other languages, including C#, Java, and Scala, among many others. Kotlin does not aim to be unique — instead, it draws inspiration from decades of language development. It exists in variants that target the JVM (Kotlin/JVM), JavaScript (Kotlin/JS), and native code (Kotlin/Native).

**MVVM**
**ViewModel:** Provides data to the UI. Acts as a communication center between the Repository and the UI. Hides where the data originates from the UI. ViewModel instances survive configuration changes.

**LiveData:** A data holder class that can be observed. Always holds/caches latest version of data. Notifies its observers when the data has changed. LiveData is lifecycle aware. UI components just observe relevant data and don’t stop or resume observation. LiveData automatically manages all of this since it’s aware of the relevant lifecycle status changes while observing.

**Repository:** A class that you create, for example using the WordRepository class. You use the Repository for managing multiple data sources.


<img src="https://i.imgur.com/UsNsFfN.png" />


**Coroutines**: Coroutines are a great new feature of Kotlin which allow you to write asynchronous code in a sequential fashion. … However, like RxJava, coroutines have a number of little subtleties that you end up learning for yourself during development time, or tricks that you pick up from others.


## 2. Advantages of Room over SQLite?

- In case of SQLite, There is no compile time verification of raw SQLite queries. But in Room there is SQL validation at compile time.

- As your schema changes, you need to update the affected SQL queries manually. Room solves this problem.

- You need to use lots of boilerplate code to convert between SQL queries and Java data objects. But, Room maps our database objects to Java Object without boilerplate code.

- Room is built to work with LiveData and RxJava for data observation, while SQLite does not.


## 3. Important Annotations in Room.


<img src="https://i.imgur.com/KEXp8s2.png" />


## 4. Implementation Step-by-Step?
As said before, this example uses MVVM with Room using Kotlin and Coroutines. Let's dive into the steps of doing it.

### **Step1:** Add dependencies to your project:

```xml
dependencies {
...
...
    // - - Room Persistence Library
    def room_version = "2.2.5"
    implementation "androidx.room:room-runtime:$room_version"
    kapt "androidx.room:room-compiler:$room_version"

    // optional - Kotlin Extensions and Coroutines support for Room
    implementation "androidx.room:room-ktx:$room_version"

    // - - ViewModel and LiveData
    def lifecycle_version = "2.2.0"
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:$lifecycle_version"

    // - - Kotlin Coroutines
    def coroutines_version = "1.3.7"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:$coroutines_version"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:$coroutines_version"
...
...
}
```


### **Step2:** Create different folders that relate to MVVM:


<img src="https://i.imgur.com/ug5rEp2.png" />


### **Step3:** Design your MainActivity which should look like this:
 
 ```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#FFFFFF"
    tools:context=".view.MainActivity">

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideline"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintGuide_percent="0.5" />

    <View
        android:layout_width="0dp"
        android:layout_height="1dp"
        android:background="#8D8D8D"
        app:layout_constraintBottom_toTopOf="@+id/guideline"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

    <androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/constraintLayout"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toTopOf="@+id/guideline"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent">

        <androidx.appcompat.widget.AppCompatTextView
            android:id="@+id/lblInsertHeading"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="20dp"
            android:text="Enter the Username and Password"
            android:textColor="#000000"
            android:textSize="18sp"
            android:textStyle="bold"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <androidx.appcompat.widget.AppCompatEditText
            android:id="@+id/txtUsername"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:textColor="#000000"
            android:hint="Username"
            android:textColorHint="#8D8D8D"
            android:layout_marginTop="20dp"
            android:layout_marginStart="40dp"
            android:layout_marginEnd="40dp"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/lblInsertHeading" />

        <androidx.appcompat.widget.AppCompatEditText
            android:id="@+id/txtPassword"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:textColor="#000000"
            android:hint="Password"
            android:textColorHint="#8D8D8D"
            android:layout_marginTop="20dp"
            android:layout_marginStart="40dp"
            android:layout_marginEnd="40dp"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/txtUsername" />

        <androidx.appcompat.widget.AppCompatButton
            android:id="@+id/btnAddLogin"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="20dp"
            android:layout_marginStart="40dp"
            android:layout_marginEnd="40dp"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/txtPassword"
            android:text="Insert Credentials"/>

        <androidx.appcompat.widget.AppCompatTextView
            android:id="@+id/lblInsertResponse"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:text="- - -"
            android:textSize="18sp"
            android:textStyle="bold"
            android:layout_margin="20dp"
            android:gravity="center"
            android:lineSpacingExtra="5dp"
            android:textColor="@android:color/holo_orange_light"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/btnAddLogin" />

    </androidx.constraintlayout.widget.ConstraintLayout>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="@+id/guideline">

        <androidx.appcompat.widget.AppCompatTextView
            android:id="@+id/lblReadHeading"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Reading and Displaying Data From Room"
            android:textColor="#000000"
            android:textSize="18sp"
            android:textStyle="bold"
            android:layout_marginTop="20dp"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/btnReadLogin" />

        <androidx.appcompat.widget.AppCompatTextView
            android:id="@+id/lblUseraname"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:text="- - -"
            android:textSize="18sp"
            android:textStyle="bold"
            android:layout_margin="20dp"
            android:gravity="center"
            android:lineSpacingExtra="5dp"
            android:padding="10dp"
            android:textColor="@android:color/holo_orange_light"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/lblReadHeading" />

        <androidx.appcompat.widget.AppCompatTextView
            android:id="@+id/lblPassword"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:text="- - -"
            android:textSize="18sp"
            android:textStyle="bold"
            android:layout_margin="20dp"
            android:gravity="center"
            android:lineSpacingExtra="5dp"
            android:padding="10dp"
            android:textColor="@android:color/holo_orange_light"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/lblUseraname" />

        <androidx.appcompat.widget.AppCompatButton
            android:id="@+id/btnReadLogin"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:textSize="12sp"
            android:layout_marginTop="10dp"
            android:layout_marginStart="40dp"
            android:layout_marginEnd="40dp"
            android:text="Click To Read Credentials From Room"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <androidx.appcompat.widget.AppCompatTextView
            android:id="@+id/lblReadResponse"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_margin="20dp"
            android:gravity="center"
            android:lineSpacingExtra="5dp"
            android:text="- - -"
            android:textColor="@android:color/holo_orange_light"
            android:textSize="18sp"
            android:textStyle="bold"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/lblPassword" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
```

### **Step4:** Now let's create the Room Database INSTANCE:

**LoginDatabase.kt**

```kotlin
package com.example.room.mvvm.room

import android.content.Context
import androidx.room.*
import com.example.room.mvvm.model.LoginTableModel

@Database(entities = arrayOf(LoginTableModel::class), version = 1, exportSchema = false)
abstract class LoginDatabase : RoomDatabase() {

    abstract fun loginDao() : DAOAccess

    companion object {

        @Volatile
        private var INSTANCE: LoginDatabase? = null

        fun getDataseClient(context: Context) : LoginDatabase {

            if (INSTANCE != null) return INSTANCE!!

            synchronized(this) {

                INSTANCE = Room
                    .databaseBuilder(context, LoginDatabase::class.java, "LOGIN_DATABASE")
                    .fallbackToDestructiveMigration()
                    .build()

                return INSTANCE!!

            }
        }

    }

}
```

### **Step5:** Next, let's create a class for creating a Table for Room DB:

```kotlin
package com.example.room.mvvm.model

import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "Login")
data class LoginTableModel (

    @ColumnInfo(name = "username")
    var Username: String,

    @ColumnInfo(name = "password")
    var Password: String

) {

    @PrimaryKey(autoGenerate = true)
    @ColumnInfo(name = "id")
    var Id: Int? = null

}
```

### **Step6:** Next is to create the queries using DAO Interface:

```kotlin
package com.example.room.mvvm.room

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.room.Dao
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import com.example.room.mvvm.model.LoginTableModel

@Dao
interface DAOAccess {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun InsertData(loginTableModel: LoginTableModel)

    @Query("SELECT * FROM Login WHERE Username =:username")
    fun getLoginDetails(username: String?) : LiveData<LoginTableModel>

}
```

### **Step7:** Next we create the Repository Class:

```kotlin
package com.example.room.mvvm.repository

import android.content.Context
import androidx.lifecycle.LiveData
import com.example.room.mvvm.model.LoginTableModel
import com.example.room.mvvm.room.LoginDatabase
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers.IO
import kotlinx.coroutines.launch

class LoginRepository {

    companion object {

        var loginDatabase: LoginDatabase? = null

        var loginTableModel: LiveData<LoginTableModel>? = null

        fun initializeDB(context: Context) : LoginDatabase {
            return LoginDatabase.getDataseClient(context)
        }

        fun insertData(context: Context, username: String, password: String) {

            loginDatabase = initializeDB(context)

            CoroutineScope(IO).launch {
                val loginDetails = LoginTableModel(username, password)
                loginDatabase!!.loginDao().InsertData(loginDetails)
            }

        }

        fun getLoginDetails(context: Context, username: String) : LiveData<LoginTableModel>? {

            loginDatabase = initializeDB(context)

            loginTableModel = loginDatabase!!.loginDao().getLoginDetails(username)

            return loginTableModel
        }

    }
}
```

### **Step8:** Next and very important step is to have a ViewModel in the project:

```kotlin
package com.example.room.mvvm.viewmodel

import android.content.Context
import androidx.lifecycle.LiveData
import androidx.lifecycle.ViewModel
import com.example.room.mvvm.model.LoginTableModel
import com.example.room.mvvm.repository.LoginRepository

class LoginViewModel : ViewModel() {

    var liveDataLogin: LiveData<LoginTableModel>? = null

    fun insertData(context: Context, username: String, password: String) {
       LoginRepository.insertData(context, username, password)
    }

    fun getLoginDetails(context: Context, username: String) : LiveData<LoginTableModel>? {
        liveDataLogin = LoginRepository.getLoginDetails(context, username)
        return liveDataLogin
    }

}
```

### **Step9:** Finally, we code the MainActivity kotlin file:

```kotlin
package com.example.room.mvvm.view

import android.content.Context
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.Toast
import androidx.lifecycle.Observer
import androidx.lifecycle.ViewModelProvider
import com.example.room.mvvm.R
import com.example.room.mvvm.viewmodel.LoginViewModel
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {

    lateinit var loginViewModel: LoginViewModel

    lateinit var context: Context

    lateinit var strUsername: String
    lateinit var strPassword: String

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        context = this@MainActivity

        loginViewModel = ViewModelProvider(this).get(LoginViewModel::class.java)

        btnAddLogin.setOnClickListener {

            strUsername = txtUsername.text.toString().trim()
            strPassword = txtPassword.text.toString().trim()

            if (strPassword.isEmpty()) {
                txtUsername.error = "Please enter the username"
            }
            else if (strPassword.isEmpty()) {
                txtPassword.error = "Please enter the username"
            }
            else {
                loginViewModel.insertData(context, strUsername, strPassword)
                lblInsertResponse.text = "Inserted Successfully"
            }
        }

        btnReadLogin.setOnClickListener {

            strUsername = txtUsername.text.toString().trim()

            loginViewModel.getLoginDetails(context, strUsername)!!.observe(this, Observer {

                if (it == null) {
                    lblReadResponse.text = "Data Not Found"
                    lblUseraname.text = "- - -"
                    lblPassword.text = "- - -"
                }
                else {
                    lblUseraname.text = it.Username
                    lblPassword.text = it.Password

                    lblReadResponse.text = "Data Found Successfully"
                }
            })
        }
    }
}
```

For any clarifications please refer to the repository.

## **Conclusion**

Hopefully this guide introduced you to a lesser known yet useful form of Android application data storage called ROOM with Kotlin and MVVM.

I hope it will help you too.
