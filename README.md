# Building a Simple Video Conferencing App with VideoSDK in Android
- In this blog post, we’ll go through integrating the VideoSDK into an Android project to create a simple video conferencing app. The VideoSDK provides powerful APIs for real-time video and audio communication, making it an excellent choice for developers looking to implement video conferencing features.

# Prerequisites
- Make sure you have the following before we begin.

1. Basics of Android development.
2. Android Studio installed on your computer.
    -  Java Development Kit.
    -  Android Studio `3.0` or `later`.
    -  Android SDK API Level `21` or `higher`.
    -  A mobile device that runs `Android 5.0` or `later`.  
3. VideoSDK account to get your API key.

##  **Table of Contents**

| **Content Index**                          | **Content Title**                 |
|------------------------------------------------|-----------------------------------------|
|`1.`                            | [ **Basic Implementation**](#creating-a-new-android-project)                                     |
|`2.`                               | [ **JoiningScreen Implementation**](#step-2-creating-joining-screen)                             |
|`3.`                              | [ **MeetingScreen Implementation**](#step-3-finally-we-are-creating-a-meeting-screen)            |
|`4.`                             | [ **Conclusion**](#conclusion)                                                                   |
|`5.`                              | [ **FAQS Section**](#faqs-section)                                                               |







# Architecture of my Project Implementation.
![Diagram.JPG](https://github.com/RomilMovaliya/VideoSDK-Task/blob/main/DocumentationStuff/Concept%20map%20(Community).jpg)

# Creating A New Android Project

- Create a new Android project and select Empty activity.
- In `File > New > New Project > Empty Views Activity` then create a project.
  
<img src="https://github.com/RomilMovaliya/VideoSDK-Task/blob/main/DocumentationStuff/Empty%20Activity.png" alt="EmptyActivity.PNG"><br>

# Integrating Video SDK with our Android Project
- Adding the following Maven url in the Maven Central,
- In `settings.gradle` file,
  
 ```groovy

dependencyResolutionManagement {
    repositories {
        // ...
        google()
        mavenCentral()
        maven { url "https://maven.aliyun.com/repository/jcenter" }
    }
}
```

- Adding following dependency in `app/build.gradle`.
 ```groovy
dependencies {
  implementation 'live.videosdk:rtc-android-sdk:0.1.34'

  // This library is used for performing a Network call to generate a meeting id
  implementation 'com.amitshekhar.android:android-networking:1.0.2'

  // other app dependencies as it is
  }
```

# Adding some permissions into our project
- In `/app/Manifests/AndroidManifest.xml`, adding the following permissions after `</application>`.
- In `AndroidManifest.xml` file,
  
```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CAMERA" />
  ```

> `Note` : If your project has set `android.useAndroidX=true`, then you need to set `android.enableJetifier=true` in the `gradle.properties` file to migrate your project to AndroidX and avoid duplicate class conflict.


# Step 1: First We Initializing VideoSDK
Create MainApplication class which will extend the `android.app.Application`.
In `MainApplication.java` file,

``` Java
import android.app.Application;
import live.videosdk.rtc.android.VideoSDK;

public class MainApplication extends Application {
  @Override
  public void onCreate() {
    super.onCreate();
    VideoSDK.initialize(getApplicationContext());
  }
}
```

- After initializing the videosdk then we are adding this class in the `AndroidManifest.xml` file.

```xml
<application
    android:name=".MainApplication" >
   <!-- ... -->
</application>

```

# Step 2: Creating Joining Screen

- Creating a new Activity called `JoinActivity`.
- This activity has main 3 UI elements,
 1. `Create Button` : This button creates the new meeting.
 2. `TextField for meetingId` : It contains the meetingId that you provide.
 3. `Join Button` : This button will join the meeting with meetingId that you provide.
 
- Now We are adding the below content inside the `/app/res/layout/activity_join.xml`.
- In `activity_join.xml` file,

``` xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical"
    tools:context=".JoinActivity">

    <Button
        android:id="@+id/btnCreateMeeting"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="16dp"
        android:text="Create Meeting" />

    <TextView
        style="@style/TextAppearance.AppCompat.Headline"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="OR" />

    <com.google.android.material.textfield.TextInputLayout
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginVertical="16dp"
        android:hint="Enter Meeting ID">

        <EditText
            android:id="@+id/etMeetingId"
            android:layout_width="250dp"
      android:layout_height="wrap_content" />
    </com.google.android.material.textfield.TextInputLayout>

    <Button
        android:id="@+id/btnJoinMeeting"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Join Meeting" />
</LinearLayout>
```

# Create Meeting API Integration Stage
- Here we are creating a private variable `sampleToken` in the `JoinActivity.java` class that basically holds the token that you generated from [VideoSDK Dashboard](https://www.videosdk.live/). It is used for `VideoSDK config` as well as `meetingId generating` purpose.
  
![Integration.JPG](https://github.com/RomilMovaliya/VideoSDK-Task/blob/main/DocumentationStuff/Slide2.JPG)

- In `JoinActivity.java` file,

```java

public class JoinActivity extends AppCompatActivity {

  //Replace with the token you generated from the VideoSDK Dashboard
  private String sampleToken ="";

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    //...
  }
}



```

- Now we Create `JoinButton` for navigate with meetingId and token to the `MeetingActivity.java` through `onclick event`.
- In `JoinActivity.java` file,

```java

public class JoinActivity extends AppCompatActivity {

  //Replace with the token you generated from the VideoSDK Dashboard
  private String sampleToken ="";

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_join);

    final Button btnCreate = findViewById(R.id.btnCreateMeeting);
    final Button btnJoin = findViewById(R.id.btnJoinMeeting);
    final EditText etMeetingId = findViewById(R.id.etMeetingId);

    btnCreate.setOnClickListener(v -> {
      createMeeting(sampleToken);
    });

    btnJoin.setOnClickListener(v -> {
      Intent intent = new Intent(JoinActivity.this, MeetingActivity.class);
      intent.putExtra("token", sampleToken);
      intent.putExtra("meetingId", etMeetingId.getText().toString());
      startActivity(intent);
    });
  }

  private void createMeeting(String token) {
    // we will explore this method in the next step
  }
}


```
- We are implementing `createMeeting()` method that will make an API call to VideoSDK Server to get a roomId and direct navigate to the `MeetingActivity` with token and generated meetingId.
- In `JoinActivity.java` file,

```java
public class JoinActivity extends AppCompatActivity {
  //...onCreate

  private void createMeeting(String token) {
      // we will make an API call to VideoSDK Server to get a roomId
      AndroidNetworking.post("https://api.videosdk.live/v2/rooms")
        .addHeaders("Authorization", token) //we will pass the token in the Headers
        .build()
        .getAsJSONObject(new JSONObjectRequestListener() {
            @Override
            public void onResponse(JSONObject response) {
                try {
                    // response will contain `roomId`
                    final String meetingId = response.getString("roomId");

                    // starting the MeetingActivity with received roomId and our sampleToken
                    Intent intent = new Intent(JoinActivity.this, MeetingActivity.class);
                    intent.putExtra("token", sampleToken);
                    intent.putExtra("meetingId", meetingId);
                    startActivity(intent);
                } catch (JSONException e) {
                    e.printStackTrace();
                }
            }

            @Override
            public void onError(ANError anError) {
                anError.printStackTrace();
                Toast.makeText(JoinActivity.this, anError.getMessage(), Toast.LENGTH_SHORT).show();
            }
        });
  }
}
```

- Our Application is based on audio and video communication so we need `RECORD_AUDIO`, `CAMERA` permission during runtime that reason we are implementing their logic inside the `JoinActivity` class.
- In `JoinActivity.java` file,

```java

public class JoinActivity extends AppCompatActivity {
  private static final int PERMISSION_REQ_ID = 22;

  private static final String[] REQUESTED_PERMISSIONS = {
    Manifest.permission.RECORD_AUDIO,
    Manifest.permission.CAMERA
  };

  private boolean checkSelfPermission(String permission, int requestCode) {
    if (ContextCompat.checkSelfPermission(this, permission) != PackageManager.PERMISSION_GRANTED) {
      ActivityCompat.requestPermissions(this, REQUESTED_PERMISSIONS, requestCode);
      return false;
    }
    return true;
  }

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    //... button listeneres
    checkSelfPermission(REQUESTED_PERMISSIONS[0], PERMISSION_REQ_ID);
    checkSelfPermission(REQUESTED_PERMISSIONS[1], PERMISSION_REQ_ID);
  }
}


```

> Note :  Make sure to include the import `import android.Manifest;`  for Manifest in your `JoinActivity` class. If it is not there then it will show you error `Cannot resolve symbol 'RECORD_AUDIO'" usually occurs because the Manifest class hasn't been imported properly`.



# Step 3: Finally We are Creating a Meeting Screen
- Create Activity Called, `MeetingActivity`. 
- Add the following content into `/app/res/layout/activity_meeting.xml` file.
- In `activity_meeting.xml` file,
  
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical"
    tools:context=".MeetingActivity">

    <TextView
        android:id="@+id/tvMeetingId"
        style="@style/TextAppearance.AppCompat.Display1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Meeting Id" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/rvParticipants"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content">

        <Button
            android:id="@+id/btnMic"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginVertical="8dp"
            android:text="Mic"/>

        <Button
            android:id="@+id/btnLeave"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginVertical="8dp"
            android:layout_marginHorizontal="8dp"
            android:text="Leave"/>

        <Button
            android:id="@+id/btnWebcam"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginVertical="8dp"
            android:text="Webcam" />

    </LinearLayout>

</LinearLayout>

```

# Meeting Initializing Stage
- Now come back to our `JoinActivity.java` file. After getting token and meetigId from JoinActivity set-ups the following things,

1. Configuring VideoSDK with token
2. Initialize meeting with some required parameters such as `meetingId`, `participantName`, `micEnabled`, `webcamEnabled` and many more.
3. Use `MeetingEventListener` to listening event such as `Meeting Join/Left` and `Participant Join/Left`.
4. Joining the room using `meeting.join()` method.
 
- In `MeetingActivity.java` file,

```java

public class MeetingActivity extends AppCompatActivity {
  // declare the variables we will be using to handle the meeting
  private Meeting meeting;
  private boolean micEnabled = true;
  private boolean webcamEnabled = true;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_meeting);

    final String token = getIntent().getStringExtra("token");
    final String meetingId = getIntent().getStringExtra("meetingId");
    final String participantName = "John Doe";

    // 1. Configuration VideoSDK with Token
    VideoSDK.config(token);

    // 2. Initialize VideoSDK Meeting
    meeting = VideoSDK.initMeeting(
            MeetingActivity.this, meetingId, participantName,
            micEnabled, webcamEnabled, null, null, false, null, null);

    // 3. Add event listener for listening upcoming events
    meeting.addEventListener(meetingEventListener);

    //4. Join VideoSDK Meeting
    meeting.join();

    ((TextView)findViewById(R.id.tvMeetingId)).setText(meetingId);
  }

  // creating the MeetingEventListener
  private final MeetingEventListener meetingEventListener = new MeetingEventListener() {
    @Override
    public void onMeetingJoined() {
      Log.d("#meeting", "onMeetingJoined()");
    }

    @Override
    public void onMeetingLeft() {
      Log.d("#meeting", "onMeetingLeft()");
      meeting = null;
      if (!isDestroyed()) finish();
    }

    @Override
    public void onParticipantJoined(Participant participant) {
      Toast.makeText(MeetingActivity.this, participant.getDisplayName() + " joined", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onParticipantLeft(Participant participant) {
      Toast.makeText(MeetingActivity.this, participant.getDisplayName() + " left", Toast.LENGTH_SHORT).show();
    }
  };
}

```

# Step 4: Handle Local Participant Media
- After entering into the meeting. It's time to `enable/disble` loaclparticipate for `webcam` and `mic`.
- for that we will use below methods in meeting class.
  
 1. `enableWebcam / disableWebcam` : for Webcam.
 2. `muteMic / unmuteMic` : for mic.

- In `MeetingActivity.java` file,

```java

public class MeetingActivity extends AppCompatActivity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_meeting);
    //...Meeting Setup is Here

    // actions
    setActionListeners();
  }

  private void setActionListeners() {
    // toggle mic
    findViewById(R.id.btnMic).setOnClickListener(view -> {
      if (micEnabled) {
        // this will mute the local participant's mic
        meeting.muteMic();
        Toast.makeText(MeetingActivity.this, "Mic Disabled", Toast.LENGTH_SHORT).show();
      } else {
        // this will unmute the local participant's mic
        meeting.unmuteMic();
        Toast.makeText(MeetingActivity.this, "Mic Enabled", Toast.LENGTH_SHORT).show();
      }
      micEnabled=!micEnabled;
    });

    // toggle webcam
    findViewById(R.id.btnWebcam).setOnClickListener(view -> {
      if (webcamEnabled) {
        // this will disable the local participant webcam
        meeting.disableWebcam();
        Toast.makeText(MeetingActivity.this, "Webcam Disabled", Toast.LENGTH_SHORT).show();
      } else {
        // this will enable the local participant webcam
        meeting.enableWebcam();
        Toast.makeText(MeetingActivity.this, "Webcam Enabled", Toast.LENGTH_SHORT).show();
      }
      webcamEnabled=!webcamEnabled;
    });

    // leave meeting
    findViewById(R.id.btnLeave).setOnClickListener(view -> {
      // this will make the local participant leave the meeting
      meeting.leave();
    });
  }
}

```


# Step 5: Handling the Participants View
1. We show all the participates in the recyler view so that reason we create one `item_remote_peer.xml` file in the `res/layout` folder.
- In `item_remote_peer.xml` file,

```xml

<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="200dp"
    android:background="@color/cardview_dark_background"
    tools:layout_height="200dp">

    <live.videosdk.rtc.android.VideoView
        android:id="@+id/participantView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:visibility="gone" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom"
        android:background="#99000000"
        android:orientation="horizontal">

        <TextView
            android:id="@+id/tvName"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:padding="4dp"
            android:textColor="@color/white" />

    </LinearLayout>

</FrameLayout>

```

2. Creating a recycler view adapter called `ParticipantAdapter` that shows the list and create `PeerViewHolder` in the adapter that extends from `RecyclerView.ViewHolder`.
- In `ParticipantAdapter.java` file,

```java

public class ParticipantAdapter extends RecyclerView.Adapter<ParticipantAdapter.PeerViewHolder> {

  @NonNull
  @Override
  public PeerViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
      return new PeerViewHolder(LayoutInflater.from(parent.getContext()).inflate(R.layout.item_remote_peer, parent, false));
  }

  @Override
  public void onBindViewHolder(@NonNull PeerViewHolder holder, int position) {
  }

  @Override
  public int getItemCount() {
      return 0;
  }

  static class PeerViewHolder extends RecyclerView.ViewHolder {
    // 'VideoView' to show Video Stream
    public VideoView participantView;
    public TextView tvName;
    public View itemView;

    PeerViewHolder(@NonNull View view) {
        super(view);
        itemView = view;
        tvName = view.findViewById(R.id.tvName);
        participantView = view.findViewById(R.id.participantView);
    }
  }
}

```

3. Now We are initializing list in the constructor of the `ParticipantAdapter`.
- In `ParticipantAdapter.java` file,
  
```java

public class ParticipantAdapter extends RecyclerView.Adapter<ParticipantAdapter.PeerViewHolder> {

  // creating a empty list which will store all participants
  private final List<Participant> participants = new ArrayList<>();

  public ParticipantAdapter(Meeting meeting) {
    // adding the local participant(You) to the list
    participants.add(meeting.getLocalParticipant());

    // adding Meeting Event listener to get the participant join/leave event in the meeting.
    meeting.addEventListener(new MeetingEventListener() {
      @Override
      public void onParticipantJoined(Participant participant) {
        // add participant to the list
        participants.add(participant);
        notifyItemInserted(participants.size() - 1);
      }

      @Override
      public void onParticipantLeft(Participant participant) {
        int pos = -1;
        for (int i = 0; i < participants.size(); i++) {
          if (participants.get(i).getId().equals(participant.getId())) {
            pos = i;
            break;
          }
        }
        // remove participant from the list
        participants.remove(participant);

        if (pos >= 0) {
          notifyItemRemoved(pos);
        }
      }
    });
  }

  // replace getItemCount() method with following.
  // this method returns the size of total number of participants
  @Override
  public int getItemCount() {
    return participants.size();
  }
  //...
}

```

4. Right now We have all listed our participants. So Let's set up the `view holder` to display a participant video.

- In `ParticipantAdapter.java` file,
```java

public class ParticipantAdapter extends RecyclerView.Adapter<ParticipantAdapter.PeerViewHolder> {

  // replace onBindViewHolder() method with following.
  @Override
  public void onBindViewHolder(@NonNull PeerViewHolder holder, int position) {
    Participant participant = participants.get(position);

    holder.tvName.setText(participant.getDisplayName());

    // adding the initial video stream for the participant into the 'VideoView'
    for (Map.Entry<String, Stream> entry : participant.getStreams().entrySet()) {
      Stream stream = entry.getValue();
      if (stream.getKind().equalsIgnoreCase("video")) {
        holder.participantView.setVisibility(View.VISIBLE);
        VideoTrack videoTrack = (VideoTrack) stream.getTrack();
        holder.participantView.addTrack(videoTrack)
        break;
      }
    }
    // add Listener to the participant which will update start or stop the video stream of that participant
    participant.addEventListener(new ParticipantEventListener() {
      @Override
      public void onStreamEnabled(Stream stream) {
        if (stream.getKind().equalsIgnoreCase("video")) {
          holder.participantView.setVisibility(View.VISIBLE);
          VideoTrack videoTrack = (VideoTrack) stream.getTrack();
          holder.participantView.addTrack(videoTrack)
        }
      }

      @Override
      public void onStreamDisabled(Stream stream) {
        if (stream.getKind().equalsIgnoreCase("video")) {
          holder.participantView.removeTrack();
          holder.participantView.setVisibility(View.GONE);
        }
      }
    });
  }
}

```

5. Finally Add the adapter in the `MeetingActivity`.
- In `MeetingActivity.java` file,
```java

@Override
protected void onCreate(Bundle savedInstanceState) {
  //Meeting Setup...
  //...
  final RecyclerView rvParticipants = findViewById(R.id.rvParticipants);
  rvParticipants.setLayoutManager(new GridLayoutManager(this, 2));


 // 2. Initialize VideoSDK Meeting
        meeting = VideoSDK.initMeeting(
                MeetingActivity.this, meetingId, participantName,
                micEnabled, webcamEnabled, null, null, false, null, null);



  // **Important**:  Initialize the ParticipantAdapter here after meeting Initializing.
  rvParticipants.setAdapter(new ParticipantAdapter(meeting));

}

```

# Final Result of Our Dedicatiion

![OutputScreenshot.JPG](https://github.com/RomilMovaliya/VideoSDK-Task/blob/main/DocumentationStuff/Slide1.JPG)


# conclusion
- Now you have a basic video conferencing app bundled with VideoSDK! This example is basic, and can be extended by adding features such as chat, user interfaces, and advanced video controls.

- See the [official VideoSDK documentation](https://docs.videosdk.live/android/guide/video-and-audio-calling-api-sdk/quick-start) for more detailed information and features.

## FAQS Section

> **1. Why we are creating adapter in the Android?** <br>
- We use adapter that serves bridge between data source and UI component that always display the data.
- For example, Adapter optimize the performance by resuing the views. when a view goes off-screen, its resources can be reused for new data, which helps to reduce memory usage and improve scrolling performance.
- `data source` : such as `arrays`, `lists`, or `databases`.
- `UI component` : such as `RecyclerView`, `ListView`, or `GridView`.

<br>

> **2. What is VideoSDK?** <br>
- VideoSDK is a `platform that provides APIs and set of tools` for developers to add `real-time video` and `audio interactions` to a variety of applications, such as video conferencing, live streaming, and virtual events.

<br>

> **3. How do I get my VideoSDK API key?** <br>
- First of all you have to register on [VideoSDK Website](https://www.videosdk.live/) and you will see the dashboard after completing sign in/sign up process. Then you will get `API from API section`.

<br>

> **4. Can I use the VideoSDK for free?** <br>
- The VideoSDK offers a variety of pricing plans, But as a beginner we start with `PAY-AS-YOU-USE`, which is best plan, including a free tier with limited usage. See their pricing page for details on features and restrictions.



