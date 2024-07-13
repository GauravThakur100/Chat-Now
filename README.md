# Chat Now

Chat Now is an Android application that facilitates real-time group chatting using Firebase Authentication and Firebase Realtime Database. This project demonstrates the use of MVVM architecture, data binding, and LiveData.

## Features

- Anonymous Authentication using Firebase Auth.
- Create and join multiple chat groups.
- Real-time message sending and receiving within chat groups.
- User-friendly UI with smooth transitions and RecyclerViews.

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/yourusername/ChatNow.git
   ```
2. **Open the project in Android Studio.**

3. **Set up Firebase:**
   - Create a Firebase project at [Firebase Console](https://console.firebase.google.com/).
   - Add your Android app to the Firebase project.
   - Download the `google-services.json` file and place it in the `app` directory of your project.
   - Enable Firebase Authentication (Anonymous) and Firebase Realtime Database.

4. **Sync the project with Gradle files:**
   ```gradle
   implementation 'com.google.firebase:firebase-auth:21.0.6'
   implementation 'com.google.firebase:firebase-database:20.0.5'
   implementation 'com.firebaseui:firebase-ui-auth:8.0.2'
   implementation 'androidx.lifecycle:lifecycle-extensions:2.2.0'
   implementation 'androidx.databinding:databinding-runtime:4.1.3'
   ```

## Project Structure

```plaintext
com.mastercoding.chatapp
├── model
│   ├── ChatGroup.java
│   ├── ChatMessage.java
├── repository
│   ├── Repository.java
├── viewmodel
│   ├── MyViewModel.java
├── views
│   ├── ChatActivity.java
│   ├── GroupsActivity.java
│   ├── LoginActivity.java
│   ├── adapters
│   │   ├── ChatAdapter.java
│   │   ├── GroupAdapter.java
├── res
│   ├── layout
│   │   ├── activity_chat.xml
│   │   ├── activity_groups.xml
│   │   ├── activity_login.xml
│   │   ├── item_card.xml
│   │   ├── row_chat.xml
│   │   ├── dialog_layout.xml
│   ├── values
│   │   ├── strings.xml
│   │   ├── styles.xml
```

## Code Overview

### Model

#### ChatGroup.java

```java
package com.mastercoding.chatapp.model;

public class ChatGroup {
    String groupName;

    public ChatGroup(String groupName) {
        this.groupName = groupName;
    }

    public String getGroupName() {
        return groupName;
    }

    public void setGroupName(String groupName) {
        this.groupName = groupName;
    }
}
```

#### ChatMessage.java

```java
package com.mastercoding.chatapp.model;

import com.google.firebase.auth.FirebaseAuth;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.TimeZone;

public class ChatMessage {
    String senderId;
    String text;
    long time;
    public boolean isMine;

    public ChatMessage(String senderId, String text, long time) {
        this.senderId = senderId;
        this.text = text;
        this.time = time;
    }

    public ChatMessage() {}

    public String getSenderId() {
        return senderId;
    }

    public void setSenderId(String senderId) {
        this.senderId = senderId;
    }

    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }

    public long getTime() {
        return time;
    }

    public void setTime(long time) {
        this.time = time;
    }

    public boolean isMine() {
        if (senderId.equals(FirebaseAuth.getInstance().getCurrentUser().getUid())) {
            return true;
        }
        return false;
    }

    public void setMine(boolean mine) {
        isMine = mine;
    }

    public String convertTime() {
        SimpleDateFormat sdf = new SimpleDateFormat("HH:mm");
        Date date = new Date(getTime());
        sdf.setTimeZone(TimeZone.getDefault());
        return sdf.format(date);
    }
}
```

### Repository

#### Repository.java

```java
package com.mastercoding.chatapp.repository;

import android.content.Context;
import android.content.Intent;
import androidx.annotation.NonNull;
import androidx.lifecycle.MutableLiveData;

import com.google.android.gms.tasks.OnCompleteListener;
import com.google.android.gms.tasks.Task;
import com.google.firebase.auth.AuthResult;
import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.database.DataSnapshot;
import com.google.firebase.database.DatabaseError;
import com.google.firebase.database.DatabaseReference;
import com.google.firebase.database.FirebaseDatabase;
import com.google.firebase.database.ValueEventListener;
import com.mastercoding.chatapp.model.ChatGroup;
import com.mastercoding.chatapp.model.ChatMessage;
import com.mastercoding.chatapp.views.GroupsActivity;

import java.util.ArrayList;
import java.util.List;

public class Repository {
    MutableLiveData<List<ChatGroup>> chatGroupMutableLiveData;
    FirebaseDatabase database;
    DatabaseReference reference;
    DatabaseReference groupReference;
    MutableLiveData<List<ChatMessage>> messagesLiveData;

    public Repository() {
        this.chatGroupMutableLiveData = new MutableLiveData<>();
        database = FirebaseDatabase.getInstance();
        reference = database.getReference();
        messagesLiveData = new MutableLiveData<>();
    }

    public void firebaseAnonymousAuth(Context context) {
        FirebaseAuth.getInstance().signInAnonymously()
                .addOnCompleteListener(new OnCompleteListener<AuthResult>() {
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        if (task.isSuccessful()) {
                            Intent i = new Intent(context, GroupsActivity.class);
                            i.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                            context.startActivity(i);
                        }
                    }
                });
    }

    public String getCurrentUserId() {
        return FirebaseAuth.getInstance().getUid();
    }

    public void signOUT() {
        FirebaseAuth.getInstance().signOut();
    }

    public MutableLiveData<List<ChatGroup>> getChatGroupMutableLiveData() {
        List<ChatGroup> groupsList = new ArrayList<>();
        reference.addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot snapshot) {
                groupsList.clear();
                for (DataSnapshot dataSnapshot : snapshot.getChildren()) {
                    ChatGroup group = new ChatGroup(dataSnapshot.getKey());
                    groupsList.add(group);
                }
                chatGroupMutableLiveData.postValue(groupsList);
            }

            @Override
            public void onCancelled(@NonNull DatabaseError error) {}
        });
        return chatGroupMutableLiveData;
    }

    public void createNewChatGroup(String groupName) {
        reference.child(groupName).setValue(groupName);
    }

    public MutableLiveData<List<ChatMessage>> getMessagesLiveData(String groupName) {
        groupReference = database.getReference().child(groupName);
        List<ChatMessage> messagesList = new ArrayList<>();
        groupReference.addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot snapshot) {
                messagesList.clear();
                for (DataSnapshot dataSnapshot : snapshot.getChildren()) {
                    ChatMessage message = dataSnapshot.getValue(ChatMessage.class);
                    messagesList.add(message);
                }
                messagesLiveData.postValue(messagesList);
            }

            @Override
            public void onCancelled(@NonNull DatabaseError error) {}
        });
        return messagesLiveData;
    }

    public void sendMessage(String messageText, String chatGroup) {
        DatabaseReference ref = database.getReference(chatGroup);
        if (!messageText.trim().equals("")) {
            ChatMessage msg = new ChatMessage(
                    FirebaseAuth.getInstance().getCurrentUser().getUid(),
                    messageText,
                    System.currentTimeMillis()
            );
            String randomKey = ref.push().getKey();
            ref.child(randomKey).setValue(msg);
        }
    }
}
```

### ViewModel

#### MyViewModel.java

```java
package com.mastercoding.chatapp.viewmodel;

import android.app.Application;
import android.content.Context;
import androidx.annotation.NonNull;
import androidx.lifecycle.AndroidViewModel;
import androidx.lifecycle.MutableLiveData;
import com.mastercoding.chatapp.repository.Repository;
import com.mastercoding.chatapp.model.ChatGroup;
import com.mastercoding.chatapp.model.ChatMessage;
import java.util.List;

public class MyViewModel extends AndroidViewModel {
    Repository repository;

    public MyViewModel(@NonNull Application application) {
        super(application);
        repository = new Repository();
    }

    public void signUpAnonymousUser() {
        Context c = this.getApplication();
        repository.firebaseAnonymousAuth(c);
    }

    public String getCurrentUserId() {
        return repository.getCurrentUserId();
    }

    public void signOut() {
        repository.signOUT();
    }

    public MutableLiveData<List<ChatGroup>> getGroupList() {
        return repository.getChatGroupMutableLiveData();
    }

    public void createNewGroup(String groupName) {
        repository.createNewChatGroup(groupName);
    }

    public MutableLiveData<List<ChatMessage>> getMessagesLiveData(String groupName) {
        return repository.getMessagesLiveData(groupName);
    }

    public void sendMessage(String msg, String chatGroup) {
        repository.sendMessage(msg, chatGroup);
    }
}
```

### Views and Adapters

#### ChatActivity.java

```java
package com.mastercoding

.chatapp.views;

import android.os.Bundle;
import android.widget.EditText;
import android.widget.ImageButton;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;
import androidx.lifecycle.ViewModelProvider;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;
import com.mastercoding.chatapp.R;
import com.mastercoding.chatapp.viewmodel.MyViewModel;
import com.mastercoding.chatapp.views.adapters.ChatAdapter;

public class ChatActivity extends AppCompatActivity {
    RecyclerView recyclerView;
    ChatAdapter chatAdapter;
    MyViewModel myViewModel;
    EditText editText;
    ImageButton sendButton;
    String groupName;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_chat);

        recyclerView = findViewById(R.id.recyclerViewChat);
        editText = findViewById(R.id.editTextChat);
        sendButton = findViewById(R.id.sendButton);
        TextView textViewGroupName = findViewById(R.id.textViewGroupName);

        groupName = getIntent().getStringExtra("groupName");
        textViewGroupName.setText(groupName);

        myViewModel = new ViewModelProvider(this).get(MyViewModel.class);

        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        chatAdapter = new ChatAdapter();
        recyclerView.setAdapter(chatAdapter);

        myViewModel.getMessagesLiveData(groupName).observe(this, chatMessages -> {
            chatAdapter.setChatMessages(chatMessages);
            recyclerView.smoothScrollToPosition(chatAdapter.getItemCount() - 1);
        });

        sendButton.setOnClickListener(view -> {
            String messageText = editText.getText().toString();
            myViewModel.sendMessage(messageText, groupName);
            editText.setText("");
        });
    }
}
```

#### GroupsActivity.java

```java
package com.mastercoding.chatapp.views;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.lifecycle.ViewModelProvider;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;
import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.mastercoding.chatapp.R;
import com.mastercoding.chatapp.viewmodel.MyViewModel;
import com.mastercoding.chatapp.views.adapters.GroupAdapter;

public class GroupsActivity extends AppCompatActivity {
    RecyclerView recyclerView;
    GroupAdapter groupAdapter;
    MyViewModel myViewModel;
    FloatingActionButton floatingActionButton;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_groups);

        recyclerView = findViewById(R.id.recyclerViewGroups);
        floatingActionButton = findViewById(R.id.fabCreateGroup);

        myViewModel = new ViewModelProvider(this).get(MyViewModel.class);

        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        groupAdapter = new GroupAdapter();
        recyclerView.setAdapter(groupAdapter);

        myViewModel.getGroupList().observe(this, chatGroups -> groupAdapter.setChatGroups(chatGroups));

        floatingActionButton.setOnClickListener(view -> showCreateGroupDialog());
    }

    private void showCreateGroupDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        View view = getLayoutInflater().inflate(R.layout.dialog_layout, null);
        builder.setView(view);

        EditText editTextGroupName = view.findViewById(R.id.editTextGroupName);

        builder.setPositiveButton("Create", (dialogInterface, i) -> {
            String groupName = editTextGroupName.getText().toString();
            myViewModel.createNewGroup(groupName);
        });

        builder.setNegativeButton("Cancel", (dialogInterface, i) -> dialogInterface.dismiss());

        AlertDialog dialog = builder.create();
        dialog.show();
    }
}
```

#### LoginActivity.java

```java
package com.mastercoding.chatapp.views;

import android.content.Intent;
import android.os.Bundle;
import androidx.appcompat.app.AppCompatActivity;
import androidx.lifecycle.ViewModelProvider;
import com.mastercoding.chatapp.R;
import com.mastercoding.chatapp.viewmodel.MyViewModel;

public class LoginActivity extends AppCompatActivity {
    MyViewModel myViewModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);

        myViewModel = new ViewModelProvider(this).get(MyViewModel.class);

        if (myViewModel.getCurrentUserId() != null) {
            Intent intent = new Intent(LoginActivity.this, GroupsActivity.class);
            startActivity(intent);
            finish();
        } else {
            myViewModel.signUpAnonymousUser();
        }
    }
}
```

### Adapters

#### ChatAdapter.java

```java
package com.mastercoding.chatapp.views.adapters;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;
import com.mastercoding.chatapp.R;
import com.mastercoding.chatapp.model.ChatMessage;
import java.util.ArrayList;
import java.util.List;

public class ChatAdapter extends RecyclerView.Adapter<ChatAdapter.ChatViewHolder> {
    List<ChatMessage> chatMessages = new ArrayList<>();

    public void setChatMessages(List<ChatMessage> chatMessages) {
        this.chatMessages = chatMessages;
        notifyDataSetChanged();
    }

    @NonNull
    @Override
    public ChatViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.row_chat, parent, false);
        return new ChatViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull ChatViewHolder holder, int position) {
        ChatMessage chatMessage = chatMessages.get(position);

        if (chatMessage.isMine()) {
            holder.textViewMessage.setText(chatMessage.getText());
            holder.textViewTime.setText(chatMessage.convertTime());
            holder.textViewMessage.setBackgroundResource(R.drawable.my_message_bg);
            holder.textViewTime.setVisibility(View.VISIBLE);
        } else {
            holder.textViewMessage.setText(chatMessage.getText());
            holder.textViewTime.setText(chatMessage.convertTime());
            holder.textViewMessage.setBackgroundResource(R.drawable.other_message_bg);
            holder.textViewTime.setVisibility(View.GONE);
        }
    }

    @Override
    public int getItemCount() {
        return chatMessages.size();
    }

    public static class ChatViewHolder extends RecyclerView.ViewHolder {
        TextView textViewMessage, textViewTime;

        public ChatViewHolder(@NonNull View itemView) {
            super(itemView);
            textViewMessage = itemView.findViewById(R.id.textViewMessage);
            textViewTime = itemView.findViewById(R.id.textViewTime);
        }
    }
}
```

#### GroupAdapter.java

```java
package com.mastercoding.chatapp.views.adapters;

import android.content.Intent;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;
import com.mastercoding.chatapp.R;
import com.mastercoding.chatapp.model.ChatGroup;
import com.mastercoding.chatapp.views.ChatActivity;
import java.util.ArrayList;
import java.util.List;

public class GroupAdapter extends RecyclerView.Adapter<GroupAdapter.GroupViewHolder> {
    List<ChatGroup> chatGroups = new ArrayList<>();

    public void setChatGroups(List<ChatGroup> chatGroups) {
        this.chatGroups = chatGroups;
        notifyDataSetChanged();
    }

    @NonNull
    @Override
    public GroupViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_card, parent, false);
        return new GroupViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull GroupViewHolder holder, int position) {
        ChatGroup chatGroup = chatGroups.get(position);
        holder.textViewGroupName.setText(chatGroup.getGroupName());

        holder.itemView.setOnClickListener(view -> {
            Intent intent = new Intent(view.getContext(), ChatActivity.class);
            intent.putExtra("groupName", chatGroup.getGroupName());
            view.getContext().startActivity(intent);
        });
    }

    @Override
    public int getItemCount() {
        return chatGroups.size();
    }

    public static class GroupViewHolder extends RecyclerView.ViewHolder {
        TextView textViewGroupName;

        public GroupViewHolder(@NonNull View itemView) {
            super(itemView);
            textViewGroupName = itemView.findViewById(R.id.textViewGroupName);
        }
    }
}
```

### Layouts

#### activity_chat.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".views.ChatActivity">

    <TextView
        android:id="@+id/textViewGroupName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:text="Group Name"
        android:textSize="18sp"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraint

Start_toStartOf="parent" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerViewChat"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_marginTop="8dp"
        android:layout_marginBottom="8dp"
        app:layout_constraintTop_toBottomOf="@id/textViewGroupName"
        app:layout_constraintBottom_toTopOf="@+id/sendButton"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        tools:listitem="@layout/row_chat" />

    <EditText
        android:id="@+id/editTextChat"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_margin="8dp"
        android:hint="Type a message"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toStartOf="@id/sendButton" />

    <ImageButton
        android:id="@+id/sendButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="8dp"
        android:background="?attr/selectableItemBackgroundBorderless"
        android:src="@drawable/ic_send"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/editTextChat" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

#### activity_groups.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".views.GroupsActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerViewGroups"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_margin="16dp"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        tools:listitem="@layout/item_card" />

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/fabCreateGroup"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:src="@drawable/ic_add"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

#### activity_login.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".views.LoginActivity">

    <TextView
        android:id="@+id/textViewLogin"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:text="Logging in..."
        android:textSize="24sp"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

#### item_card.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

    <TextView
        android:id="@+id/textViewGroupName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:text="Group Name"
        android:textSize="18sp" />

</androidx.cardview.widget.CardView>
```

#### row_chat.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="8dp">

    <TextView
        android:id="@+id/textViewMessage"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="@drawable/message_bg"
        android:padding="8dp"
        android:text="Message"
        android:textSize="16sp" />

    <TextView
        android:id="@+id/textViewTime"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Time"
        android:textSize="12sp"
        android:layout_marginTop="4dp" />

</LinearLayout>
```

#### dialog_layout.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp">

    <EditText
        android:id="@+id/editTextGroupName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Group Name"
        android:inputType="text"
        android:padding="8dp" />

</LinearLayout>
```

#### strings.xml

```xml
<resources>
    <string name="app_name">Chat Now</string>
    <string name="group_name">Group Name</string>
    <string name="type_message">Type a message</string>
    <string name="create_group">Create Group</string>
</resources>
```

#### styles.xml

```xml
<resources>

    <!-- Base application theme -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

</resources>
```
